---
title: "[프로그래머스] 디스크 컨트롤러 - 자바"
date: 2023-11-30T19:26:15+09:00
draft: false
pin: false
summary: 프로그래머스 Level 3 디스크 컨트롤러 자바 풀이
tags:
  - programmers
  - stack/queue
---

> # [프로그래머스] 디스크 컨트롤러 - 자바

## Code
---

```java
class Solution {  
  
    static class Task {  
        private int requestTime;  
        private int processingTime;  
  
        public Task(int requestTime, int processingTime) {  
            this.requestTime = requestTime;  
            this.processingTime = processingTime;  
        }  
  
    }  
  
    public int solution(int[][] jobs) {  
  
        PriorityQueue<Task> ready = new PriorityQueue<>((o1, o2) -> o1.processingTime - o2.processingTime);  
        PriorityQueue<Task> wait = new PriorityQueue<>((o1, o2) -> o1.requestTime - o2.requestTime);  
        int totalCostedTime = 0;  
        int totalWaitedTime = 0;  
        int idleTime = 0;  
        int count = 0;  
  
        for (int[] job : jobs) {  
            wait.offer(new Task(job[0], job[1]));  
        }  
  
        while (count < jobs.length) {  
            while (!wait.isEmpty() && wait.peek().requestTime <= totalCostedTime) {  
                ready.offer(wait.poll());  
            }  
  
            if (ready.isEmpty()) {  
                idleTime += wait.peek().requestTime - totalCostedTime;  
                totalCostedTime = wait.peek().requestTime;  
            } else {  
                Task job = ready.poll();  
                totalWaitedTime += totalCostedTime - job.requestTime;  
                totalCostedTime += job.processingTime;  
                count++;  
            }  
        }  
        return (totalCostedTime + totalWaitedTime - idleTime) / jobs.length;  
    }  
}
```


### 풀이방법
---

1. 작업중에 요청된 작업들은 소요시간 작은순으로 처리
2.  진행중인 작업이 없다면 요청 시간 빠른순 처리

위 두개의 요구사항을 만족하기 위해 우선순위 큐 2개 선언
- 작업 대기 큐
- 작업 진행 큐

1. 작업 진행 큐는 요청된 작업이 모두 완료될때까지 실행
2. 진행큐에는 대기큐에서 현재 총 작업소요시간보다  빠른 요청시간을 가진 작업들만 들어올수있음
3. 더 이상 빠른 요청시간이 없고 대기큐에 작업이 있는 경우 idleTime과 총 소요시간을 업데이트
4. 현재까지 총소요시간 - 요청시간을 빼면 작업의 대기시간을 알수있음
5. _(총소요시간 + 총대기시간  - IDLE시간) / 총 작업의 개수_ 는 총 작업시간의 평균을 구할수있다.
