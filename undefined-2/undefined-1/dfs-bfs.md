# DFS/BFS

## 타겟 넘버

```java
public class Solution {
    int answer = 0;

    public int solution(int[] numbers, int target) {

        dfs(numbers, 0, 0, target);

        return answer;
    }

    private void dfs(int[] numbers, int sum, int depth, int target) {
        if (depth == numbers.length) {
            if (sum == target) {
                answer++;
            }
        } else {
            dfs(numbers, sum + numbers[depth], depth + 1, target);
            dfs(numbers, sum - numbers[depth], depth + 1, target);
        }
    }
}
```
