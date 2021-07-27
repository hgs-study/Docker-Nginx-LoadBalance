## nginx 로드밸런싱
```
  worker 인스턴스 3개와 nginx 인스턴스 1개를 이용하여 부하분산 테스트
```

### 순서
```
  ㅇ 도커 설치
  ㅇ 도커 허브에서 이미지 pull
  ㅇ 도커 실행
```

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



