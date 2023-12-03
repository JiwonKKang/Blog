---
title: "[프로그래머스] 개인정보 수집 유효기간 - 자바"
date: 2023-12-02T01:33:13+09:00
draft: false
pin: false
summary: "[프로그래머스] 개인정보 수집 유효기간 - 2023 KAKAO BLIND RECRUITMENT"
tags:
  - programmers
  - kakao
---

> # [프로그래머스] 개인정보 수집 유효기간 - 자바

### 나의 풀이
---

```java
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.*;

class Solution {

    static class Privacy {
        private Date  startDate;
        private String termType;
        private SimpleDateFormat format = new SimpleDateFormat("yyyy.MM.dd");
        private Calendar cal = Calendar.getInstance();

        public Privacy(String date, String termType) throws ParseException {

            this.startDate = format.parse(date);
            this.termType = termType;
        }

        public boolean isValid(String today, Map<String, Integer> termMap) throws ParseException {

            cal.setTime(startDate);
            Integer expireDate = termMap.get(termType);
            cal.add(Calendar.MONTH, expireDate);
            Date todayDate = format.parse(today);
            return cal.getTime().after(todayDate);

        }
    }

    public int[] solution(String today, String[] terms, String[] privacies) throws ParseException {
        List<Integer> answer = new ArrayList<>();
        Map<String, Integer> termMap = new HashMap<>();

        List<Privacy> privacyList = new ArrayList<>();

        for (String term : terms) {
            String[] splitTerm = term.split(" ");
            termMap.put(splitTerm[0], Integer.parseInt(splitTerm[1]));
        }

        for (String pri : privacies) {
            String[] dateAndTerm = pri.split(" ");
            String date = dateAndTerm[0];
            String term = dateAndTerm[1];

            Privacy privacy = new Privacy(date, term);

            privacyList.add(privacy);
        }

        for (Privacy privacy : privacyList) {
            if (!privacy.isValid(today, termMap)) {
                answer.add(privacyList.indexOf(privacy) + 1);
            }
        }

        return answer.stream().mapToInt(Integer::intValue).toArray();
    }
}
```


### 더 좋은 풀이
---
```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

class Solution {
    public int[] solution(String today, String[] terms, String[] privacies) {
        List<Integer> answer = new ArrayList<>();
        Map<String, Integer> termMap = new HashMap<>();
        int date = getDate(today);

        for (String s : terms) {
            String[] term = s.split(" ");

            termMap.put(term[0], Integer.parseInt(term[1]));
        }
        for (int i = 0; i < privacies.length; i++) {
            String[] privacy = privacies[i].split(" ");

            if (getDate(privacy[0]) + (termMap.get(privacy[1]) * 28) <= date) {
                answer.add(i + 1);
            }
        }
        return answer.stream().mapToInt(integer -> integer).toArray();
    }

    private int getDate(String today) {
        String[] date = today.split("\\.");
        int year = Integer.parseInt(date[0]);
        int month = Integer.parseInt(date[1]);
        int day = Integer.parseInt(date[2]);
        return (year * 12 * 28) + (month * 28) + day;
    }
}
```

### 통찰

- 코딩테스트에서는 Date같은 유틸리티 클래스를 사용해서 풀도록 만들거 같진않다. 되도록이면 쓰지않는 방향으로 풀도록하자