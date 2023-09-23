# CH08 DFS, BFS 활용

<figure><img src="../../.gitbook/assets/image (137).png" alt=""><figcaption></figcaption></figure>

```java
package dfsbfs.main1;

import java.util.Scanner;

public class Main {
    static int[] arr;
    static int total = 0;

    static String answer = "NO";

    static boolean stopNow = false;

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();
        arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = scanner.nextInt();
            total += arr[i];
        }

        Main main = new Main();
        main.DFS(0, 0);
        System.out.println(answer);
    }

    private void DFS(int index, int sum) {
        if (sum > total / 2 || stopNow) {
            return;
        }

        if (index == arr.length - 1) {
            if (sum == total - sum) {
                answer = "YES";
                stopNow = true;
            }
        } else {
            DFS(index + 1, sum + arr[index]);
            DFS(index + 1, sum);
        }
    }
}
```

<figure><img src="../../.gitbook/assets/image (139).png" alt=""><figcaption></figcaption></figure>

```java
package dfsbfs.main2;

import java.util.Scanner;

public class Main {

    static int max = 0;
    static int limit = 0;
    static int[] weights;

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        limit = scanner.nextInt();
        int count = scanner.nextInt();
        weights = new int[count];
        for (int i = 0; i < count; i++) {
            weights[i] = scanner.nextInt();
        }

        Main main = new Main();
        main.DFS(0, 0);

        System.out.println(max);
    }

    private void DFS(int index, int sum) {
        if (sum > limit) {
            return;
        }

        if (index == weights.length) {
            max = Math.max(sum, max);
        } else {
            DFS(index + 1, sum + weights[index]);
            DFS(index + 1, sum);
        }
    }
}

```

<figure><img src="../../.gitbook/assets/image (140).png" alt=""><figcaption></figcaption></figure>

```java
package dfsbfs.main3;

import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

class Problem {
    int point;
    int time;

    public Problem(int point, int time) {
        this.point = point;
        this.time = time;
    }
}

public class Main {
    static int count;
    static int limit;
    static int pointMax = 0;
    static List<Problem> problems = new ArrayList<>();

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        count = scanner.nextInt();
        limit = scanner.nextInt();

        for (int i = 0; i < count; i++) {
            problems.add(new Problem(scanner.nextInt(), scanner.nextInt()));
        }
        Main main = new Main();
        main.DFS(0, 0, 0);
        System.out.println(pointMax);
    }

    private void DFS(int index, int pointSum, int timeSum) {
        if (timeSum > limit) return;

        if (index == count) {
            pointMax = Math.max(pointSum, pointMax);
        } else {
            DFS(index + 1, pointSum + problems.get(index).point, timeSum + problems.get(index).time);
            DFS(index + 1, pointSum, timeSum);
        }
    }
}

```

<figure><img src="../../.gitbook/assets/image (141).png" alt=""><figcaption></figcaption></figure>

```java
package dfsbfs.main4;

import java.util.Scanner;

public class Main {
    static int[] pm;
    static int n, m;

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        n = scanner.nextInt();
        m = scanner.nextInt();
        pm = new int[m];
        Main main = new Main();
        main.DFS(0);
    }

    private void DFS(int L) {
        if (L == m) {
            for (int x : pm) System.out.print(x + " ");
            System.out.println();
        } else {
            for (int i = 1; i <= n; i++) {
                pm[L] = i;
                DFS(L + 1);
            }
        }
    }
}
```

<figure><img src="../../.gitbook/assets/image (142).png" alt=""><figcaption></figcaption></figure>

```java
package dfsbfs.main5;

import java.util.Scanner;

public class Main {

    static int count;
    static int[] coins;
    static int target;
    static int min = Integer.MAX_VALUE;

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        count = scanner.nextInt();
        coins = new int[count];
        for (int i = count - 1; i >= 0; i--) {
            coins[i] = scanner.nextInt();
        }
        target = scanner.nextInt();

        Main main = new Main();
        main.DFS(0, 0);
        System.out.println(min);
    }

    private void DFS(int L, int sum) {
        if (sum > target || L > min) {
            return;
        }

        if (sum == target) {
            min = Math.min(min, L);
            return;
        } else {
            for (int i = 0; i < count; i++) {
                DFS(L + 1, sum + coins[i]);
            }
        }
    }
}
```

<figure><img src="../../.gitbook/assets/image (143).png" alt=""><figcaption></figcaption></figure>

```java
package dfsbfs.main6;

import java.util.Scanner;

public class Main {
    static int totalCount;
    static int selectCount;
    static int[] numbers;
    static int[] pm;
    static int[] check;

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        totalCount = scanner.nextInt();
        selectCount = scanner.nextInt();
        numbers = new int[totalCount];
        for (int i = 0; i < totalCount; i++) {
            numbers[i] = scanner.nextInt();
        }
        pm = new int[selectCount];
        check = new int[totalCount];
        Main main = new Main();
        main.DFS(0);
    }

    private void DFS(int L) {
        if (L > selectCount) return;

        if (L == selectCount) {
            for (int number : pm) System.out.print(number + " ");
            System.out.println();
        } else {
            for (int i = 0; i < totalCount; i++) {
                if (check[i] == 0) {
                    check[i] = 1;
                    pm[L] = numbers[i];
                    DFS(L + 1);
                    check[i] = 0;
                }

            }
        }

    }
}
```



























