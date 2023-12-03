---
title: "컨테이너 배포를 위한 AWS EC2 환경 세팅"
date: 2023-08-14T09:58:26+09:00
draft: false
pin: false 
summary: "프로젝트 배포를위해 EC2 콘솔에 배포환경을 세팅해보자"
tags: [docker, aws]
---

> # 컨테이너 배포를 위한 AWS EC2 환경 세팅

EC2 인스턴스를 만드는건 쉽기때문에 생략했다. 프리티어로 과금 안되는선에서 만들면 된다.

다음으로는 보안그룹을 설정해야한다. 

`보안 그룹은 AWS에서 제공하는 방화벽 모음이다.`

서비스를 제공하는 애플리케이션이라면 상관 없지만 RDS처럼 외부에서 함부로 접근하면 안되는 인스턴스는 허용된 IP에서만 접근하도록 설정이 필요하다.

- 인바운드(Inbound): 외부 -> 인스턴스 내부 허용
- 아웃바운드(Outbound): 인스턴스 내부 -> 외부 허용

여기서 나의 어플리케이션을 다른 사용자가 접속하여 사용할수 있게 하기 위해서는 인바운드 규칙을 추가 해야 한다.

![](https://user-images.githubusercontent.com/26623547/172119445-3bab7f12-f006-4149-a935-eaa8019f118a.png)

> ## Docker Compose 설치 및 배포

EC2에 접속하여 도커 및 Git을 설치해 보자.

#### Git 설치

```
#Perform a quick update on your instance:
$ sudo yum update -y

#Install git in your EC2 instance
$ sudo yum install git -y

#Check git version
$ git version
```

#### 도커, 도커 컴포즈 설치 및 시작

docker client와 server(docker daemon)간 통신 방식은 기본적으로 unix domain socket(IPC socket)을 사용하며, 내부적으로 /var/run/docker.sock파일을 사용하여 통신한다.  
따라서, 아래와 같이 설치 후 권한을 부여한다.

```
//도커 설치  
$ sudo yum install docker
$ docker -v

// 도커 컴포즈 설치 
$ sudo curl -L https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose   

// 도커 시작하기     
$ sudo systemctl start docker

// 실행 권한 적용   
$ sudo chmod +x /usr/local/bin/docker-compose    
$ sudo chmod 666 /var/run/docker.sock
$ docker-compose -v
```

#### Docker Compose 파일이 존재하는 소스 내려받기

```
$ git clone https://github.com/WonYong-Jang/Pharmacy-Recommendation.git
```

#### Docker 환경변수

로컬에서 개발할 때, DB 계정 정보나 외부에 노출되면 안되는 값들을 따로 제외하여 관리하였고 이를 도커 컨테이너를 실행할 때 전달해주어야 하는데 이때 .ENV 파일을 사용할 수 있다.

`docker-compose를 사용할때 .env라는 파일에 환경변수를 사용하면 자동으로 참조하여 사용할 수 있다.`

이를 동일하게 EC2에서도 docker-compose파일이 있는 위치에 아래와 같이 환경변수 값들을 추가해 주어서 docker-compose 실행시 해당 값들을 참조하여 컨테이너를 실행시킬 수 있게 해준다.

```
$ vi .env   

SPRING_DATASOURCE_USERNAME=root
SPRING_DATASOURCE_PASSWORD=1234
```

.env 파일을 생성후 아래 명령어를 통해 값을 확인 가능하다.

```
$ docker-compose config    
```

#### JDK 설치 및 jar 파일 생성

아래와 같이 Amazon에서 제공하는 OpenJDK인 Amazon Coretto를 다운받아 간편하게 설치할 수 있다.

```
# aws coreetto 다운로드
$ sudo curl -L https://corretto.aws/downloads/latest/amazon-corretto-11-x64-linux-jdk.rpm -o jdk11.rpm

# jdk11 설치
$ sudo yum localinstall jdk11.rpm
```

```
// 테스트 케이스 제외하고, jar 파일 빌드만 진행
$ ./gradlew clean build -x test
```

#### Docker 이미지 받고 Docker Compose 실행

```
$ docker-compose up --build
```

docker-compose를 이용해서 애플리케이션을 실행시킨다.

![](https://user-images.githubusercontent.com/26623547/172131535-a3d87de5-dac4-43cb-a5fb-bb16245826ed.png)

위에서 적용한 Elastic IP를 통해 접속하게 되면, 정상적으로 애플리케이션을 접속하는 것을 확인 할 수 있다.

---
**Reference**

[https://bcp0109.tistory.com/356](https://bcp0109.tistory.com/356)  
https://zzang9ha.tistory.com/329?category=954133](https://zzang9ha.tistory.com/329?category=954133)