---
title: "[프로그래머스] 성격 유형 검사 - 자바"
date: 2023-12-02T21:45:05+09:00
draft: false
pin: false
summary: "[프로그래머스] 성격 유형 검사 - 2022 KAKAO TECH INTERNSHIP"
tags:
  - programmers
---


> # [프로그래머스] 성격 유형 검사 - 자바

### 나의 풀이
---
```java
import java.util.HashMap;

class Solution {
    public String solution(String[] survey, int[] choices) {
        String answer = "";
        HashMap<Character, Integer> map = new HashMap<>();
        char[] array = {'R', 'T', 'C', 'F', 'J', 'M', 'A', 'N'};

        for (char c : array) {
            map.put(c, 0);
        }

        for (int i = 0; i < survey.length; i++) {
            char[] questionArray = survey[i].toCharArray();
            if (choices[i] < 4) {
                map.put(questionArray[0], map.get(questionArray[0]) + Math.abs(4 - choices[i]));
            } else {
                map.put(questionArray[1], map.get(questionArray[1]) + Math.abs(4 - choices[i]));
            }
        }

        String[] array1 = {"RT", "CF", "JM", "AN"};

        for (String str : array1) {
            char[] charArray = str.toCharArray();
            if (map.get(charArray[0]) < map.get(charArray[1])) {
                answer += charArray[1];
            } else {
                answer += charArray[0] ;
            }
        }

        return answer;
    }
}
```

### 접근 
---
1. Map을 사용하여 각 유형 별 점수 저장
2. 미리 array1에 유형을 사전순으로 정렬하여 점수비교만 하면되게함

### 배운 점  
---
- String에 + 연산으로 문자를 뒤에 이을 수 있는것을 알았다.