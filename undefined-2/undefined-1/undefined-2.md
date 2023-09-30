# 정렬

## K번째 수

```java
package programers.array;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

class Solution {
    public int[] solution(int[] array, int[][] commands) {
        int[] answer = new int[commands.length];
        for (int i = 0; i < commands.length; i++) {
            int startIndex = commands[i][0] - 1;
            int endIndex = commands[i][1] - 1;
            int targetIndex = commands[i][2] - 1;
            int[] scopedArray = getScopedArray(array, startIndex, endIndex);
            Arrays.sort(scopedArray);
            answer[i] = scopedArray[targetIndex];
        }

        return answer;
    }

    private int[] getScopedArray(int[] array, int startIndex, int endIndex) {
        List<Integer> temp = new ArrayList<>();
        for (int i = startIndex; i <= endIndex; i++) {
            temp.add(array[i]);
        }

        int[] result = new int[temp.size()];
        for (int i = 0; i < result.length; i++) {
            result[i] = temp.get(i);
        }

        return result;
    }
}

class Main {
    public static void main(String[] args) {
        Solution T = new Solution();

        int[] array = {1, 5, 2, 6, 3, 7, 4};
        int[][] commands = {{2, 5, 3}, {4, 4, 1}, {1, 7, 3}};
        int[] result = T.solution(array, commands);

        for (int num : result) {
            System.out.print(num + " ");
        }
    }
}
```

## 가장 큰 수

```java
package programers.array.solution2;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

class Number implements Comparable<Number> {
    int value;

    public Number(int value) {
        this.value = value;
    }

    @Override
    public int compareTo(Number o) {
        String first = String.valueOf(this.value);
        String second = String.valueOf(o.value);

        int firstValue = Integer.parseInt(first + second);
        int secondValue = Integer.parseInt(second + first);

        return secondValue - firstValue;
    }
}

class Solution {
    public String solution(int[] numbers) {
        List<Number> targets = new ArrayList<>();
        for (int num : numbers) {
            targets.add(new Number(num));
        }
        Collections.sort(targets);

        if (targets.get(0).value == 0) {
            return "0";
        }

        StringBuilder stringBuilder = new StringBuilder();
        for (Number number : targets) {
            stringBuilder.append(number.value);
        }

        return stringBuilder.toString();
    }
}

class Main {
    public static void main(String[] args) {
        Solution T = new Solution();
//        String answer = T.solution(new int[]{6, 10, 2});
        String answer = T.solution(new int[]{3, 30, 34, 5, 9}); // 9534330
        System.out.println(answer);
    }
}
```

## H-Index

```java
package programers.array.solution3;

import java.util.Arrays;

class Solution {
    public int solution(int[] citations) {
        Arrays.sort(citations);
        int max = 0;
        for (int i = 0; i < citations.length; i++) {
            int biggerSize = citations.length - i;
            int current = citations[i];
            if (current >= biggerSize) {
                max = Math.max(max, biggerSize);
            }
        }

        return max;
    }
}

class Main {
    public static void main(String[] args) {
        Solution T = new Solution();
        int answer = T.solution(new int[]{3, 0, 6, 1, 5}); // 3
        System.out.println(answer);
    }
}
```
