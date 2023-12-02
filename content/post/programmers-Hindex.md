이
---
title: "[프로그래머스] H-Index - 자바"
date: 2023-12-03T01:26:14+09:00
draft: true
pin: false
summary: "[프로그래머스] H-Index Level 2 - 자바 풀이"
tags:
  - programmers
---

> # [프로그래머스] H-Index - 자바

### 나의 풀이
---

```java
import java.util.*;

class Solution {
    public int solution(int[] citations) {
        List<Integer> answerList = new ArrayList<>();

        Arrays.sort(citations);

        for (int i = 0; i < citations.length; i++) {
            if (citations[i] >= citations.length - i) {
                answerList.add(citations.length - i);
            }
        }
        return answerList.isEmpty() ? 0 : answerList.get(0);
    }
}
```

### 접근
---
1. citations를 정렬하게되면 배열의 값이 인용횟수, citations.length-i가 인용횟수 이상의 논문 갯수
2. **논문의 갯수가 H-Index**

### 삽질
---
- 논문의 갯수를 리턴해야하는데 계속 논문인용횟수를 리턴해서 테스트 케이스를 통과하지못했다;;
- 문제를 잘 이해하도록 하자...

