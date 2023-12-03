---
title: "[프로그래머스] 소수찾기 - 자바"
date: 2023-12-03T14:17:59+09:00
draft: false
pin: false
summary: "[프로그래머스] 소수찾기 Level 2 - 자바풀이"
tags:
  - programmers
  - dfs
  - backtracking
---


> # [프로그래머스] 소수찾기 - 자바

### 나의 풀이 (substring 사용)
---
```java
import java.util.*;

class Solution {
    HashSet<Integer> set = new HashSet<>();
    
    public int solution(String numbers) {
        int answer = 0;
        
        recursive("", numbers);
        
        return set.size();
    }
    
    private void recursive(String current, String others) {
        if (!current.isEmpty()) {
            Integer num = Integer.valueOf(current);
            if (isPrime(num)) set.add(num);
        }
        
        for(int i=0; i < others.length(); i++) {
            recursive(current + others.charAt(i), others.substring(0, i) + others.substring(i+1));
        }
    }
    
    private boolean isPrime(int num) {
        if (num < 2) return false;
        
        for (int i=2; i <= (int) Math.sqrt(num); i++) {
            if (num % i == 0) return false;
        }
        return true;
    }
}
```

### 접근 
---
1. DFS로 순열 조합을 만든다.
2. substring을 사용하여 i번째 수를 제외하고 다음 재귀를 호출한다.
3. 소수를 판단하여 Set에 넣는다.
4. Set의 사이즈를 반환한다.

### 다른 풀이 (Backtracking 사용)
---

```java
import java.util.HashSet;  
  
class Solution {  
    HashSet<Integer> set = new HashSet<>();  
    boolean[] visited = new boolean[7];  
  
    public int solution(String numbers) {  
        int answer = 0;  
  
        recursive("", numbers);  
  
        return set.size();  
    }  
  
    private void recursive(String current, String others) {  
        if (!current.isEmpty()) {  
            Integer num = Integer.valueOf(current);  
            if (isPrime(num)) set.add(num);  
        }  
  
        for(int i=0; i < others.length(); i++) {  
            if (!visited[i]) {  
                visited[i] = true;  
                recursive(current + others.charAt(i), others);  
                visited[i] = false;  
            }  
        }  
    }  
  
    private boolean isPrime(int num) {  
        if (num < 2) return false;  
  
        for (int i=2; i <= (int) Math.sqrt(num); i++) {  
            if (num % i == 0) return false;  
        }  
        return true;  
    }  
}
```

### 접근
---
1. boolean 배열을 사용해 이미 사용한 수인지 확인한다.
2. 재귀 호출을 끝까지하고 돌아오면 수를 다시 사용할수있게 false로 되돌린다.

### 배운 점
---
- 공통적으로 DFS를 사용하여 가능한 순열을 찾고있다.
- 순열을 만들 때 i번째의 수를 제외하는 방법이 substring이냐 boolean배열이냐의 차이점이다.