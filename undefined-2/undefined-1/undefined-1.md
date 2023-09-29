# 스택/큐

## 같은 숫자는 싫어

```java
package programers.stack;

import java.util.Stack;

public class Solution {
    public int[] solution(int[] arr) {
        Stack<Integer> stack = new Stack<>();
        for (int number : arr) {
            if (stack.isEmpty() || stack.peek() != number) {
                stack.add(number);
            }
        }

        int[] anwser = new int[stack.size()];
        for (int i = anwser.length - 1; i >= 0; i--) {
            anwser[i] = stack.pop();
        }

        return anwser;
    }
}

class Main {
    public static void main(String[] args) {
        Solution T = new Solution();
        int[] answer = T.solution(new int[]{1, 1, 3, 3, 0, 1, 1});
        for (int a : answer) {
            System.out.println(a);
        }
    }
}
```

## 올바른 괄호

```java
package programers.stack.solution2;

import java.util.Stack;

class Solution {
    boolean solution(String s) {
        Stack<Character> stack = new Stack<>();
        for (Character c : s.toCharArray()) {
            if (c == ')') {
                if (stack.isEmpty()) {
                    return false;
                }
                while ('(' != stack.pop()) {
                }
            } else {
                stack.add(c);
            }
        }

        if (stack.isEmpty()) {
            return true;
        } else {
            return false;
        }
    }
}
```

## 기능 개발

```java
package programers.stack.solution3;

import java.util.LinkedList;
import java.util.Queue;

class Task {
    int progress;
    int speed;

    public Task(int progress, int speed) {
        this.progress = progress;
        this.speed = speed;
    }

    public void doTask() {
        progress += speed;
    }
}

class Solution {
    public int[] solution(int[] progresses, int[] speeds) {
        Queue<Task> taskQueue = new LinkedList<>();
        for (int i = 0; i < progresses.length; i++) {
            Task task = new Task(progresses[i], speeds[i]);
            taskQueue.offer(task);
        }

        Queue<Integer> answerQueue = new LinkedList<>();
        while (!taskQueue.isEmpty()) {
            int tempAnswer = 0;
            while (!taskQueue.isEmpty() && taskQueue.peek().progress >= 100) {
                tempAnswer++;
                taskQueue.poll();
            }
            if (tempAnswer != 0) {
                answerQueue.offer(tempAnswer);
            }

            for (Task task : taskQueue) {
                task.doTask();
            }
        }

        int[] answer = new int[answerQueue.size()];
        for (int i = 0; i < answer.length; i++) {
            if (!answerQueue.isEmpty()) {
                answer[i] = answerQueue.poll();
            }
        }

        return answer;
    }
}

class Main {
    public static void main(String[] args) {
        Solution T = new Solution();
        int[] answer = T.solution(new int[]{93, 30, 55}, new int[]{1, 30, 5});
        for (int n : answer) {
            System.out.print(n + " ");
        }
    }
}
```

## 프로세스

```java
package programers.stack.solution4;

import java.util.LinkedList;
import java.util.Queue;

class Process {
    int index;
    int priority;

    public Process(int index, int priority) {
        this.index = index;
        this.priority = priority;
    }
}

class Solution {
    public int solution(int[] priorities, int location) {
        Queue<Process> processQueue = new LinkedList<>();
        for (int i = 0; i < priorities.length; i++) {
            Process process = new Process(i, priorities[i]);
            processQueue.offer(process);
        }

        Queue<Integer> finishedProcessIndexes = new LinkedList<>();
        while (!processQueue.isEmpty()) {
            Process picked = processQueue.poll();

            for (Process process : processQueue) {
                if (process.priority > picked.priority) {
                    processQueue.offer(picked);
                    picked = null;
                    break;
                }
            }

            if (picked != null) {
                finishedProcessIndexes.offer(picked.index);
            }
        }

        int answer = 1;
        while (!finishedProcessIndexes.isEmpty()) {
            if (location == finishedProcessIndexes.poll()) {
                return answer;
            }
            answer++;
        }

        return -1;
    }
}

class Main {
    public static void main(String[] args) {
        Solution T = new Solution();
//        int answer = T.solution(new int[]{2, 1, 3, 2}, 2); // 1
        int answer = T.solution(new int[]{1, 1, 9, 1, 1, 1}, 0); // 5
        System.out.println(answer);
    }
}
```

## 다리를 지나는 트럭

```java
import java.util.LinkedList;
import java.util.Queue;

class Truck {
    int weight;
    int index;

    public Truck(int weight) {
        this.weight = weight;
        this.index = 0;
    }

    public void forward() {
        this.index += 1;
    }
}

class Solution {
    public int solution(int bridge_length, int weight, int[] truck_weights) {
        Queue<Truck> truckQueue = new LinkedList<>();
        for (int w : truck_weights) {
            truckQueue.offer(new Truck(w));
        }

        int currentWeight = 0;
        int time = 0;
        Queue<Truck> outTruckQueue = new LinkedList<>();
        while (true) {
            outTruckQueue.forEach(Truck::forward);
            if (!outTruckQueue.isEmpty() && outTruckQueue.peek().index > bridge_length) {
                Truck t = outTruckQueue.poll();
                currentWeight -= t.weight;
            }

            if (!truckQueue.isEmpty() && (currentWeight + truckQueue.peek().weight) <= weight && outTruckQueue.size() < bridge_length) {
                Truck t = truckQueue.poll();
                t.forward();
                currentWeight += t.weight;
                outTruckQueue.offer(t);
            }

            time += 1;

            if (truckQueue.isEmpty() && outTruckQueue.isEmpty()) {
                break;
            }
        }

        return time;
    }
}

```

## 주식가격

```java
package programers.stack.solution6;

class Solution {
    public int[] solution(int[] prices) {
        int[] answers = new int[prices.length];
        for (int i = 0; i < prices.length; i++) {
            int current = prices[i];

            int seconds = 0;
            for (int j = i + 1; j < prices.length; j++) {
                seconds++;
                if (prices[j] < current) {
                    break;
                }
            }

            answers[i] = seconds;
        }

        return answers;
    }
}
```
