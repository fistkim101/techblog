# 완전탐색

## 최소직사각형

```java
class Solution {
    public int solution(int[][] sizes) {
        int maxLong = 0;
        int maxShort = 0;

        for (int[] size : sizes) {
            int tempMax = Math.max(size[0], size[1]);
            int tempMin = Math.min(size[0], size[1]);

            maxLong = Math.max(maxLong, tempMax);
            maxShort = Math.max(maxShort, tempMin);
        }

        return maxLong * maxShort;
    }
}
```

## 모의고사

```java
package programers.search.solution2;

import java.util.ArrayList;
import java.util.List;

class Solution {
    public int[] solution(int[] answers) {
        int[][] patterns = {
                {1, 2, 3, 4, 5},
                {2, 1, 2, 3, 2, 4, 2, 5},
                {3, 3, 1, 1, 2, 2, 4, 4, 5, 5}
        };

        int[] scores = new int[3];
        for (int i = 0; i < answers.length; i++) {
            for (int j = 0; j < 3; j++) {
                if (answers[i] == patterns[j][i % patterns[j].length]) {
                    scores[j]++;
                }
            }
        }

        int maxScore = Math.max(scores[0], Math.max(scores[1], scores[2]));
        List<Integer> result = new ArrayList<>();
        for (int i = 0; i < 3; i++) {
            if (scores[i] == maxScore) {
                result.add(i + 1);
            }
        }

        return result.stream().mapToInt(i -> i).toArray();
    }
}

class Main {
    public static void main(String[] args) {
        Solution T = new Solution();
        int[] answer = T.solution(new int[]{1, 2, 3, 4, 5}); // 1
//        int[] answer = T.solution(new int[]{1,3,2,4,2}); // 1, 2, ,3
        for (int num : answer) {
            System.out.print(num + " ");
        }
    }
}
```

## 카펫

```java
package programers.search.solution4;

class Solution {
    public int[] solution(int brown, int yellow) {
        int total = brown + yellow;
        for (int i = 1; i <= Math.sqrt(total); i++) {
            if (total % i == 0) {
                int height = i;
                int weight = total / height;

                if ((weight - 2) * (height - 2) == yellow) {
                    return new int[]{weight, height};
                }
            }
        }

        int[] answer = {};
        return answer;
    }

}
```

## 피로도

```java
class Solution {
    public int solution(int k, int[][] dungeons) {
        int[] visited = new int[dungeons.length];
        explore(dungeons, visited, k, 0);

        return max;
    }

    private int max = 0;

    private void explore(int[][] dungeons, int[] visited, int k, int count) {
        for (int i = 0; i < dungeons.length; i++) {
            if (visited[i] == 0 && k >= dungeons[i][0]) {
                visited[i] = 1;
                explore(dungeons, visited, k - dungeons[i][1], count + 1);
                visited[i] = 0;
            }
        }

        if (max < count) {
            max = count;
        }
    }
}
```
