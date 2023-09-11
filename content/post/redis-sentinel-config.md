---
title: Redis-Sentitnel 구성
date: 2023-09-11T09:48:08+09:00
draft: false
pin: false
summary: Redis-sentinel 구성하기
tags:
  - redis
---

> # Redis sentinel이란?

운영환경에서 레디스는 일반적으로 마스터와 복제로 구성됩니다.   운영중 예기치 않게 마스터가 다운되었다면, 관리자가 이를 감지해서 복제를 마스터로 올리고 클라이언트들이 새로운 마스터에 접속할 수 있도록 해 주어야 합니다.   

센티널은 마스터와 복제를 감시하고 있다가 마스터가 다운되면 이를 감지해서 관리자의 개입없이 자동으로 복제를 마스터로 올려줍니다.

센티널은 다음과 같은 기능을 합니다.  

- **모니터링 Monitoring** : 센티널은 레디스 마스터, 복제들을 제대로 동작하는지 지속적으로 감시합니다.

- **자동 장애조치 Automatic Failover** : 센티널은 레디스 마스터가 예기치 않게 다운되었을 때 복제를 새로운 마스터로 승격시켜 줍니다.   그리고 복제가 여러 대 있을 경우 이 복제들이 새로운 마스터로 부터 데이터를 받을 수 있도록 재 구성하고, 다운된 마스터가 재 시작했을 때 복제로 전환되어 새로운 마스터를 바라볼 수 있도록 합니다.

- **알림 Notification** : 센티널은 감시하고 있는 레디스 인스턴스들이 failover 되었을 때 Pub/Sub으로 Application(client)에게 알리거나 shell script로 관리자에게 이메일이나 SMS로 알릴 수 있습니다.


우선 Redis와 Sentinel은 아래와 같이 동작합니다.


> 정상 운영시

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb879Y4%2Fbtq16tHGvXM%2FQzvi5VCOIcLkPNvnYp7NsK%2Fimg.png)

>한대 장애시


![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb6XTsx%2Fbtq2clOKSLF%2FAB0KRcR2VvhMiOrUQ1XFp1%2Fimg.png)


> 장애 서버 복구시


![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fdbx7PG%2Fbtq18EhvJHC%2Fj9ZlHSKGnTFOKdM7EdKGP1%2Fimg.png)

> # 준비

일단 클라우드 플랫폼을 통해 VM을 3개 만들어줍니다.

VM 2대에

```bash
sudo apt-get install redis-server
sudo apt-get install redis-sentinel
```

을 통해 Redis와 Sentinel을 설치해줍니다. 

redis와 sentinel은 port만 다르다면 정삭적으로 동작합니다.

하지만 서로 다른 머신에 설치하고 운영하는게 좀더 바람직하다고 생각합니다.

그리고 남은 VM 1대에는 sentinel만 설치해줍니다.(어플리케이션 서버)

> # Redis 설정

```bash
vi /etc/redis/redis.conf
```

를 통해 redis 설정파일로 들어가줍니다.


일단 VM들끼리 서로 통신할수있게 네트워크 설정을 해줘야합니다.

#### BIND

Bind로 하나 또는 여러 개의 IP를 지정할 수 있습니다. 이 경우 지정한 IP로만 레디스 서버에 접속할 수 있습니다.   Bind 지시자를 지정하지 않으면 서버에서 사용 가능한 모든 네트워크 인터페이스로 접속을 허용합니다.

**bind 192.168.1.100 127.0.0.1  
bind 127.0.0.1 ::1   -> 로칼 접속만 허용할 경우  
bind * -::*   -> 모든 접속을 허용할 경우 설정(Redis 6.2부터 가능)**

여기서는 

``` bash
bind 0.0.0.0
```

으로 모든 IP에 대해서 허용해주도록 하겠습니다.
#### REPLICAOF

Master-Replica 복제.  
Replicaof 지시자(명령)을 사용해서 레디스 서버의 복제본을 만들 수 있습니다. 
![](http://redisgate.kr/images/server/redos_conf_replica.png)

- 복제는 비동기이지만 지정한 수의 복제가 실행되지 못하면 마스터 서버가 더 이상 데이터를 받아들이지 않게 설정할 수 있습니다.
- 마스터와 복제 서버간 짧은 시간 연결이 끊겼을 경우 부분 재동기를 할 수 있습니다. 복제 백로그 크기(replication backlog size)를 조정해서 부준 재동기 시간을 간접적으로 조정할 수 있습니다.
- 복제는 자동으로 사용자의 개입이 필요하지 않습니다. 복제 서버가 마스터에 재 연결되면 자동으로 복제를 수행합니다.

replicaof [masterip] [masterport] 

를 Replica VM에 설정해줍니다.

> Sentinel 설정

다음으로는 sentinel 설정입니다.

```bash
vi /etc/redis/sentinel.conf
```

로 sentinel 설정 파일에 들어갈수 있습니다.

여기서도 redis 설정과 마찬가지로 

```bash
bind 0.0.0.0 
```

으로 모든 IP에 대해 허용해주도록 하겠습니다.

또한 Sentienl에서는 모니터링할 master redis에 대한 추가적인 설정이 필요합니다.

#### SENTINEL MONITOR

센티널이 레디스 마스터 서버를 모니터하도록 지정합니다.  

센티널은 여러 개의 마스터를 모니터 할 수 있습니다. 

복제 노드를 모니터하지 마세요. 복제 노드 정보는 마스터에서 가져옵니다. master-name은 임의로 지정할 수 있고 중복할 수 없습니다.  
master-name은 알파벳 대소문자, 숫자와 ".-_"를 사용할 수 있습니다.  

quorum(쿼럼)은 레디스 마스터 서버가 다운되었을 때 몇 개의 센티널이 다운되었는지 인지해야 객관적으로 다운되었다고 판정하는 기준입니다.  

sentinel monitor [master-name] [redis-ip] [redis-port] [quorum]  

```bash
sentinel monitor mymaster 127.0.0.1 6379 2
```


> 참고


~~~
추가로 클라우드 플랫폼의 VM을 이용했기때문에 플랫폼에서 각 포트에 대한 방화벽을 열어 주셔야
VM끼리 통신 할 수 있습니다.
~~~


> # Failover 테스트

이렇게 까지 설정을 완료했다면 master redis가 장애가 났을때 Sentinel들이 이를 감지하고
Failover를 통해 master-replica를 서로 바꾸는지 테스트해보도록 합시다.

```bash
tail -f /var/log/redis/redis-server.log
tail -f /var/log/redis/redis-sentinel.log
```

위 2개의 명령어를 통해 각 로그를 확인 할 수 있습니다.


> _출처_

https://co-de.tistory.com/15
http://redisgate.kr/redis/server/redis_conf_han.php
http://redisgate.jp/redis/sentinel/sentinel_conf_han.php



