## Nginx 로드밸런싱
```
  Worker 인스턴스 3개와 nginx 인스턴스 1개를 이용하여 부하분산 테스트
```

### 순서
```
  ㅇ 도커 설치
  ㅇ 도커 허브에서 이미지 pull
  ㅇ 도커 실행
  ㅇ nginx 설치
  ㅇ 로드밸런싱 설정
  ㅇ 로드밸런싱 커스텀 conf 생성
  ㅇ 결과 확인
  ㅇ 흥미로운 주제 : upstream weight
```

### 도커
-----
+ 도커 설치 (AWS linux 2 기준)
```shell
sudo yum update -y

sudo amazon-linux-extras install docker

sudo service docker start
```

+ 도커 허브에서 이미지 pull
```shell
sudo docker pull hgstudy/nginx_load_balance_practice
```

+ 도커 실행 (로그를 바로 보기 위해 백그라운드로 run 하지 않음)
```shell
sudo docker run -p 8080:8080 hgstudy/nginx_load_balance_practice
```

![image](https://user-images.githubusercontent.com/76584547/127164463-342848ed-e902-4e0b-8ffb-06355ac7a376.png)


### Nginx
---
+ nginx 설치
```shell
sudo amazon-linux-extras enable nginx1

sudo yum install nginx

yum list installed nginx

sudo service nginx start
```

+ 로드밸런싱 설정
```shell
  sudo vim /etc/nginx/nginx.conf
```
+ "include /etc/nginx/conf.d/*.conf"를 활용해 로드밸런싱 커스텀 conf를 생성한다. 
![image](https://user-images.githubusercontent.com/76584547/127176236-7aa7b044-dda9-4fb2-a10c-bce3a1ad24c5.png)

+ 로드밸런싱 커스텀 conf 생성
```shell
  sudo vim /etc/nginx/conf.d/load-balance.conf
```
+ 로드밸런싱할 upstream 생성 / server proxy_pass에 upstream 명칭 작성!
```shell
upstream worker-instance {
  server 172.31.11.88:8080 weight=100 max_fails=3 fail_timeout=3s;
  server 172.31.1.117:8080 weight=100 max_fails=3 fail_timeout=3s;
  server 172.31.11.244:8080 weight=100 max_fails=3 fail_timeout=3s;
}

server{
  listen 80;

  location / {
    proxy_pass http://worker-instance;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
  }
}
```
+ 엔진엑스 reload
```shell
  sudo systemctl reload nginx - nginx reload
```
+ 에러 확인
```shell
  sudo tail -f /var/log/nginx/error.log (엔진엑스 에러로그 확인)
```

+ connect() failed (111: Connection refused) while connecting to upstream, client: 에러발생!
+ connect()가 기본적으로 실행을 못하게 되어있다. 

+ 해결 
```shell
  sudo setsebool -P httpd_can_network_connect on
```

+ 요청

<br/>

![image](https://user-images.githubusercontent.com/76584547/127175707-d5794231-d0b1-4fa4-9c2f-5c9999bef45c.png)

+ 1회 요청 시
![image](https://user-images.githubusercontent.com/76584547/127175617-0b2e71f8-a7b8-4460-9574-bab486079115.png)

+ 2회 요청 시 
![image](https://user-images.githubusercontent.com/76584547/127175847-fddda29f-e6eb-42dc-93c4-8c750b3c1673.png)

+ 3회 요청 시 
![image](https://user-images.githubusercontent.com/76584547/127175933-aea839a8-79f0-47a6-9d7b-0f307778bffb.png)

+ 4회 요청 시
![image](https://user-images.githubusercontent.com/76584547/127176002-8bbcf636-b7fe-4132-ac2f-20c71231fd4e.png)

+ 결과
```
  엔진엑스 로드밸런싱 기본이 라운드로빈이기 때문에 인스턴스를 하나씩 요청하는 것을 알 수 있다.
```

### 흥미로운 주제 : upstream weight
-------
+ 현재 가중치(weight)
  + 모두 동등한 100 이었다.
```shell
  upstream worker-instance {
    server 172.31.11.88:8080 weight=100 max_fails=3 fail_timeout=3s;
    server 172.31.1.117:8080 weight=100 max_fails=3 fail_timeout=3s;
    server 172.31.11.244:8080 weight=100 max_fails=3 fail_timeout=3s;
  }
```

+ 기본 설정이 라운드 로빈인데 weight가 제각기 달라도 순서를 보장할까?
  + weight를 6, 4, 2로 설정해봤다 
```shell
upstream worker-instance {
  server 172.31.11.88:8080 weight=6 max_fails=3 fail_timeout=3s;
  server 172.31.1.117:8080 weight=4 max_fails=3 fail_timeout=3s;
  server 172.31.11.244:8080 weight=2 max_fails=3 fail_timeout=3s;
}
```

+ 결과
```
  ㅇ 1-1-2-1-3-2 / 1-1-2-1-3-2 순으로 반복하게 된다.
  ㅇ 라운드로빈은 깨지게 되고 가중치가 높은 서버가 더 많은 요청을 받게 된다.
  ㅇ 6번의 요청 중 3번은 worker01이, 2번은 worker02이, 1번은 worker03이 가중치만큼 받았다.
```

![image](https://user-images.githubusercontent.com/76584547/127182824-7e817f6c-0333-4597-b5af-58320b461b28.png)




