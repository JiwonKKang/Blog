---
title: "[프로그래머스] 이중 우선순위큐 - 자바"
date: 2023-12-02T21:45:05+09:00
draft: false
pin: false
summary: "[프로그래머스] 이중 우선순위 큐  Level 3 - 자바풀이"
tags:
  - programmers
  - heap
---


> # [프로그래머스] 이중 우선순위큐 - 자바

### 나의 풀이
---
```java
import java.util.Arrays;
import java.util.Collections;
import java.util.PriorityQueue;


class Solution {

    class DoublePriorityQueue {
        private PriorityQueue<Integer> minHeap = new PriorityQueue<>();
        private PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());

        public void push(int value) {
            maxHeap.offer(value);
        }

        public void removeMax() {
            while (!minHeap.isEmpty()) {
                maxHeap.offer(minHeap.poll());
            }

            maxHeap.poll();
        }

        public void removeMin() {
            while (!maxHeap.isEmpty()) {
                minHeap.offer(maxHeap.poll());
            }

            minHeap.poll();
        }

        public int getMax() {

            if (minHeap.size() == maxHeap.size()) {
                return 0;
            }

            while (!minHeap.isEmpty()) {
                maxHeap.offer(minHeap.poll());
            }

            return maxHeap.peek().intValue();
        }

        public int getMin() {

            if (minHeap.size() == maxHeap.size()) {
                return 0;
            }

            while (!maxHeap.isEmpty()) {
                minHeap.offer(maxHeap.poll());
            }

            return minHeap.peek().intValue();
        }

    }
    public int[] solution(String[] operations) {
        DoublePriorityQueue doublePriorityQueue = new DoublePriorityQueue();
        for (String operation : operations) {
            String[] commandAndValue = operation.split(" ");
            String command = commandAndValue[0].trim();
            int value = Integer.parseInt(commandAndValue[1].trim());

            if (command.equals("I")) {
                doublePriorityQueue.push(value);
            } else {
                if (value == 1) {
                    doublePriorityQueue.removeMax();
                } else {
                    doublePriorityQueue.removeMin();
                }
            }
        }
        return new int[]{doublePriorityQueue.getMax(), doublePriorityQueue.getMin()};
    }
}

```

### 접근 
---
1. 정렬 기준을 다르게 한 우선순위 큐를 2개 만든다
2. 최대값이나 최솟값을 삭제할때 큐 한개에 모두 넣고 삭제한다
3. 두 큐의 사이즈가 같다는것 곧 큐가 비었다는 의미이기때문에 0을 반환한다.
