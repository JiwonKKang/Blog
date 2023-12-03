---
title: "Kafka란 무엇인가요?"
date: 2023-08-30T00:05:13+09:00 
draft: false
pin: false 
summary: "분산 메세징 시스템 Kafka를 자세히 알아보자"
tags: [kafka]
---

> # kafka란?

카프카(Kafka)는 파이프라인, 스트리밍 분석, 데이터 통합 및 미션 크리티컬 애플리케이션을 위해 설계된 **고성능 분산 이벤트 스트리밍 플랫폼**입니다.

**Pub-Sub 모델의 메시지 큐** 형태로 동작하며 분산환경에 특화되어 있습니다.

> ##  Kafka의 Produce 과정

Kafka Producer가 Broker에게 데이터를 전달하는 과정을 먼저 알아야 합니다.  
단순히 topic 데이터만을 Kafka Broker에게 전송하면 된다고 생각하겠지만, 내부적으로 여러 과정이 있습니다.
기본적으로 아래 그림과 같은 순서대로 진행됩니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F7h66m%2FbtrCflUmNAO%2Fg9Jk6CrWeDzVYfZEG82gK1%2Fimg.png)

Producer Client는 처음부터 Topic data를 보내는 것이 아니라 topic이 저장된 위치를 알기 위해 Metadata를 Kafka Broker 중 하나에게 요청합니다. 요청을 받은 Broker는 Client가 원하는 topic이 위치한 서버 리스트를 보내줍니다. 그리고 **최종적으로 Client는 Broker로부터 받은 서버 리스트를 참고해서, 서버에 직접 접근 및 데이터 전송을 합니다.**

Docker로 구성한 Kafka 클러스터에서는 각 Broker가 hostname을 포함하여 서로의 정보들을 사전에 알고 있습니다. 그래서 클러스터 내부에서  Produce / Consume을 진행해도 간편하게 서비스 이름과 port를 반환해주면됩니다. 

**하지만 문제는 외부에서 Kafka 클러스터에 접근할 때입니다. 각 Broker의 hostname을 반환해주더라도 외부에서는 해당 hostname의 정보를 가지고 있지 않기 때문에  아마 에러가 발생할 겁니다.**

그래서 Kafka는 옵션 값 설정을 통해 내부 / 외부 통신 시 구분해서 서버 주소를 반환해줄 수 있습니다.  

- listeners : Kafka가 서비스를 제공하는 주소 ( 다수 등록 가능 ) 
- advertised.listeners : Client에게 Metadata와 함께 반환할 서버 주소

아래에 예시를 보여드리겠습니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbiwSO6%2FbtrCiaSiWd7%2Fetf4TYA4aQN2jNPojL0By0%2Fimg.png)

Docker 내부에서 통신하는 클라이언트 하나 호스트 머신에서 통신하는 클라이언트가 하나씩있습니다.
각각 39092 port와 39094 port로 포트포워딩 되어있습니다.

- KAFKA_LISTENER_SECURITY_PROTOCOL_MAP : **AA**:PLAINTEXT, **BB**:PLAINTEXT
- KAFKA_LISTENERS : **AA**://kafka-3:39092, **BB**://kafka-3:39094  
- KAFKA_ADVERTISED_LISTENERS : **AA**://kafka-3:39092, **BB**://localhost:39094  
- KAFKA_INTER_BROKER_LISTENER_NAME : **AA**

**KAFKA_LISTENER_SECURITY_PROTOCOL_MAP** 설정을 통해 구분하고 싶은 네트워크 추가와  
보안 프로토콜을 정할 수 있습니다. 위의 예시에서는 "AA"와 "BB"라는 이름의 listener를 추가했습니다.

**KAFKA_LISTENERS** 설정에 위 옵션에서 등록 한 listener마다 서버 주소를 설정해줌으로써 어떤 네트워크를 사용했는지 Kafka가 구분할 수 있습니다. 예시와 같이 설정이 되었을 때, **"kafka-3" hostname에 39092 port로 들어온 서비스는 이제 AA listener로 구분**할 수 있습니다.

**KAFKA_ADVERTISED_LISTENERS** 설정은 위에서 구분한 listener에 맞게 다시 Client에게 반환시킬 서버 주소입니다. Client는 이 서버 주소를 받아 직접 Topic을 produce / consume 합니다. 예시에서는 **AA**를 Docker 내부 통신을 의도했기 때문에 Client에게 hostname을 포함한 주소를 반환해줘도 Client는 해당 hostname을 사전에 알고 있습니다. 하지만 **BB**는 Docker 외부, 즉 Host machine과 통신을 의도했기 때문에, hostname 대신 localhost를 적었습니다. Host machine은 Docker container와 통신하려면 port forwarding 된 localhost를 사용해야 하기 때문입니다.

**KAFKA_INTER_BROKER_LISTENER_NAME"** 설정을 통해서 Kafka가 사전에 내부 통신을 위한 listener가 어떤 이름인지 알 수 있습니다.

하지만 위 그림과 설정에서 이상한 점을 느낄 수 있습니다. Host machine은 Kafka broker의 hostname을 모르기 때문에 localhost로 반환해주는 것인데, 외부 통신을 의도한 **BB** listener는 "kafka-3"이라는 hostname을 직접 명시했을까요? **위의 그림과 설정에서는 Host machine과 Docker container 간의 port forwarding 과정이 빠져있기 때문**입니다. Kafka 클러스터와 **Host machine의 Producer Client만 다시 분리해서 port forwarding 과정을 추가**하면 아래 그림과 같습니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbXwRFz%2FbtrCatklwBy%2FCpYOCsSjrtfTOIMOrNjW8K%2Fimg.png)

Kafka Broker 입장에서는 Host machine과 Docker container 간의 port forwarding은 중요하지 않습니다. 단지 자신에게 들어오는 최종적인 요청 값만 고려하기 때문입니다.


---
Reference
https://taaewoo.tistory.com/59