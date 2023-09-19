---
title: Redis 장애 Case 및 대응
date: 2023-09-18T18:47:36+09:00
draft: false
pin: false
summary: Redis의 주요 장애 Case와 그에 따른 대응 방법을 알아보자
tags:
  - redis
  - error
---

> # Master-Replica 전환 후 Client 의 인식

### _Scenario_

1. Client 는 Master에는 Write. Replica에는 Read request를 날림
2. 어떤 이유로 Master-Replica 전환
3. Client 는 새로 바뀐 정보가 아닌 에전 Master/Replica 정보를 게속 참조하며 Write 가 불가하다는 메세지가 발생됨. 

![](/images/redis-error-case/case1.png)


실제 오류 메세지
```
READONLY You can't write against a read only replica.
```

### 대응

##### 1. 도메인의 Routing 정책

![](/images/redis-error-case/case1-1.png)

기존 Client에서 IP를 통해 Master에 접근을 했지만 실제 운영환경에서는 IP를 통한 접근은 잘하지않고 도메인을 만들어서 해당 도메인이 Routing 정보를 갖고있게하고 해당 Routing 정보를 통해 Master로 가게되는 흐름으로 만듭니다. 

위처럼 하게되면 Client에서는 Redis의 IP를 입력할 필요없이 해당 도메인의 정보만 입력하게되면  
나중에 Redis 서버가 바뀌거나 추가되어도 Client에서 어떠한 변경도 할 필요가 없게됩니다.

만약 Master-Replica가 전환되었을 때 도메인이 새로운 Master로 Routing 하게끔 바뀌어야되는데 
이때 사용할수있는게 AWS Lamda 나 다양한 Health Check 도구등을 이용해서 Redis의 info를 확인해서
'Role'이 Master가 아니라고 감지되었을때는 도메인이 향하는 Routing방향이 변경되도록 할수있습니다.

##### 2. Cluster 환경에서는 Client 의 Meta 정보 Refresh Option 설정

- Lettuce : ClusterTopologyRefreshOptions

> #   full sync 실패로 인한 장애 유발

![](/images/redis-error-case/case2.png)

기본적으로 Master-Replica는 지속적으로 sync하려고합니다. 
이때 어떤 이유로 sync가 끊어지게되면 먼저 손실된 operation을 replica가 수행하여 동기화하려고하는
partial sync(부분동기화)를 시도합니다. 

여기서 부분동기화가 실패하게되면 full sync를 시도합니다. 
full sync는 Master가 영속성을 보장하기위해 주기적으로 디스크에 저장하는 파일 rdb를 통해 이루어집니다.
Master가 rdb를 Replica에게 전달하고 Replica는 rdb를 처리함으로써 full sync가 완료됩니다.

이때 만약 rdb의 크기가 크면
```redis.conf
[ client-output-buffer-limit ] slave 256mb 64mb 60
```
이 설정값에 의해서 256MB가 한번에 메모리에 올라가거나 64MB이상이 60초 이상 메모리에 올라가면
해당 transaction을 abort합니다.

따라서 Master-Replica는 full sync를 계속 시도하며 지속적으로 dump.rdb 재생성을 하고 
Load 및 Latency 가 증가하여 장애를 유발합니다.

### 대응

```redis
[ client-output-buffer-limit ] slave 2048mb 64mb 600
```

위처럼 설정값을 바꾸게 되면 rdb의 크기가 커도 full sync를  할수 있게됩니다.


> # 통신불가로 인한 Buffer 증가, 데이터 삭제

![](/images/redis-error-case/case3.png)

위 사진 처럼 Client에서 크기가 굉장히 큰 데이터 요청이 들어오게되면 데이터를 조회하여 return을 시도합니다.

이때 Client의 어떤 방화벽(Inbound) 문제로 인해서 Client가 데이터를 받을수 없게되면 
반환되지못한 Data가 Memory상에 임시 보관됩니다. 

보통 Client쪽에서는 데이터를 받지 못하게되면 같은요청을 3번까지 retry하기 때문에
큰 용량의 데이터가 Memory에 점점 싸입니다.

임시 보관 Data가 누적될수록 Redis에 할당된 공간을 점점 차지하게 되고 할당된 공간을 모두 사용하면
Redis Memory policy에 따라 write가 불가해지거나, 데이터가 지워지는 현상이 발생합니다.

```
[ maxmemory-policy ] noeviction : 데이터를 지우지않음 -> memory full -> write 불가
[ maxmemory-policy ] volatile-lru, allkey-lru 
			: 정해진 규칙에 의해 삭제 -> memory full -> 기존 데이터 삭제
```


### 대응

```
[ client-output-buffer-limit ] normal 2gb 0 0
```

이 설정값을 변겨해줍니다. 

이 설정값은 일반적으로는 Client에서 Data를 받지 못할 일이 없기에 미설정이 Default입니다.


> # 그 외 장애들 

- Client 무한 증가 : redis에서 timeout 설정해도 특정 library에서는 주기적으로 신호를 보내어 idle connection으로 인식되지 않아 close 되지 않는다. 이에 Client 단에서 close를 꼭 해주거나 tcp 를 임의로 kill하는 작업이 필요함 (관련 parameter : timeout 21600, maxclient 10000) 

<br>

- AOF 쓰기작업 : AOF 는 Client 에서 보내는 명령을 모두 hard disk에 기록하는 파일임. (Append Only File). 이를 통해 RDB로 한꺼번에 쓰지 않아도 되지만, 너무 빈번하게 발생되는 경우 Redis Service 성능에 영향을 준다. (관련 parameter : appendfsync : everysec ), AOF 쓰기가 너무 오래 걸리는 경우 성능에 문제가 되기도 한다. 때문에 대량 쓰기(rdb, aof 생성) 시 fsync를 하지 않도록 설정할 수 있다. (관련 Paramter : no-appendfsync-on-rewrite no) 

<br>

- KEYS, HGETALL 등 과도한 요청으로 인한 장애 : 전체 데이터를 조회하는 식의 요청은 최대한 지양하는 것이 좋다. KEY 명령어는 rename command 를 통해 실행되지 않게 조정할 수 있으며, 전체 데이터를 지우는 flushall, flushdb 등의 명령어도 제한해 놓는 것이 좋다. (관련 parameter : rename-command keys “”)