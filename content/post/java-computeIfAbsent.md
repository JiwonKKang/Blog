---
title: "[Java] Map의 computeIfAbsent"
date: 2023-08-27T22:54:45+09:00
draft: false
pin: false 
summary: "자바 Map의 메서드 computeIfAbsent에 대해 알아보자"
tags: [Java]
---

> # computeIfAbsent

롤 승률 분석 서비스를 만들던 도중 챔피언 이름별로 승률 데이터를 넣어야하는 상황에 
computeIfAbsent라는 Map의 메서드를 알았습니다.

computeIfAbsent 메서드는 첫 번째 파라미터에는 key를  
두 번째 파라미터에는 함수형 인터페이스 Function을 넣어주면  
Map에 key가 존재하지 않을때만 Fucntion을 연산한 결과를 넣어줍니다.

저는 이 메서드를 다음과 같이 활용했습니다.

```java
Map<String, ChampionWinRateDto> championWinRateMap = new HashMap<>();

.
.
.

match.getParticipants().stream()  
        .filter(participant ->
           participant.getTeamPosition().equals(me.getTeamPosition()) && 
           participant.getTeamId() != me.getTeamId())  
        .forEach(participant -> {  
            championWinRateMap  
                    .computeIfAbsent(participant.getChampionName(), 
                    k -> new ChampionWinRateDto())  
                    .incrementTotalMatches()  
                    .incrementWins(participant.isWin() ? 0 : 1);  
        });
```

위 코드를 설명하자면  

매치 정보들에서 자기 자신과 같은 라인에(ex 미드) 에 서서 상대했던 챔피언들을 필터링해서  
그 챔피언의 이름을 Map의 key로 value로는 승률 정보가 들어가게 됩니다.  
key가 처음들어오는 상황에 key를 넣은 뒤 value에 승률 정보를 새로 생성해서 넣어줘야하는 상황이였습니다.

그때 computeIfAbsent라는 메서드를 알게되어 key가 있을 경우에는 그에 대응하는 value(승률 정보)를  
업데이트해주고 없을 경우에는  ```k -> new ChampionWinRateDto()```  함수를 실행시켜  
승률정보DTO를 먼저 생성한뒤에 승률정보를 업데이트해주었습니다.

그리고 추가로
```java
public ChampionWinRateDto incrementTotalMatches() {  
    totalMatches++;  
    return this;  
}
```
다음과 같이 클래스안의 메서드에서 this를 반환해주면  
위의 로직같이 메서드 체이닝이 된다는 사실을 알았고 코드를 좀더 깔끔하게 짤 수 있었습니다.