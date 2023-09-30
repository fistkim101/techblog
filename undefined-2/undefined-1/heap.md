# 힙(Heap)

## 더 맵게

```java
package programers.heap.solution1;

import java.util.PriorityQueue;
import java.util.Queue;

class Solution {
    public int solution(int[] scoville, int K) {
        Queue<Integer> priorityQ = new PriorityQueue<>();
        for (int s : scoville) {
            priorityQ.offer(s);
        }

        int answer = 0;
        while (priorityQ.peek() < K) {
            if (priorityQ.size() < 2) {
                return -1;
            }

            answer++;
            int first = priorityQ.poll();
            int second = priorityQ.poll();
            int result = first + (second * 2);
            priorityQ.offer(result);
        }

        return answer;
    }
}

class Main {
    public static void main(String[] args) {
        Solution T = new Solution();
        int answer = T.solution(new int[]{1, 2, 3, 9, 10, 12}, 7);
        System.out.println(answer);
    }
}
```

## 디스크 컨트롤러

```java
package programers.heap.solution2;

import java.util.*;

class Task implements Comparable<Task> {
    int requestTime;
    int takeTime;

    public Task(int requestTime, int takeTime) {
        this.requestTime = requestTime;
        this.takeTime = takeTime;
    }

    @Override
    public int compareTo(Task o) {
        return this.takeTime - o.takeTime;
    }
}

class Solution {
    public int solution(int[][] jobs) {
        List<Task> tasks = new ArrayList<>();
        for (int i = 0; i < jobs.length; i++) {
            Task task = new Task(jobs[i][0], jobs[i][1]);
            tasks.add(task);
        }
        tasks.sort(Comparator.comparingInt(t -> t.requestTime));

        int totalTime = 0;
        int endTime = 0;
        int index = 0;
        Queue<Task> priorityQueue = new PriorityQueue<>();
        while (index < tasks.size() || !priorityQueue.isEmpty()) {
            while (index < tasks.size() && tasks.get(index).requestTime <= endTime) {
                priorityQueue.offer(tasks.get(index++));
            }

            if (priorityQueue.isEmpty()) {
                endTime = tasks.get(index).requestTime;
            } else {
                Task currentTask = priorityQueue.poll();

                int waitingTime = endTime - currentTask.requestTime;
                int actualTime = waitingTime + currentTask.takeTime;
                totalTime += actualTime;
                endTime += currentTask.takeTime;
            }
        }

        return totalTime / jobs.length;
    }
}

class Main {
    public static void main(String[] args) {
        Solution T = new Solution();
        int[][] jobs = {{0, 3}, {1, 9}, {2, 6}};
        int answer = T.solution(jobs);
        System.out.println(answer);
    }
}
```

## 이중우선순위큐

```java
package programers.heap.solution3;import java.util.TreeMap;

class Solution {
    public int[] solution(String[] operations) {
        TreeMap<Integer, Integer> map = new TreeMap<>();
        for (String operation : operations) {
            String[] parts = operation.split(" ");
            String command = parts[0];
            int num = Integer.parseInt(parts[1]);

            if (command.equals("I")) {
                map.put(num, map.getOrDefault(num, 0) + 1);
            } else if (command.equals("D")) {
                if (map.isEmpty()) continue;

                int keyToDelete = (num == 1) ? map.lastKey() : map.firstKey();
                int count = map.get(keyToDelete);
                if (count == 1) {
                    map.remove(keyToDelete);
                } else {
                    map.put(keyToDelete, count - 1);
                }
            }

        }

        if (map.isEmpty()) {
            return new int[]{0, 0};
        }

        return new int[]{map.lastKey(), map.firstKey()};
    }
}
```
