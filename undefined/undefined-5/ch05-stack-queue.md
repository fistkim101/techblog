# CH05 Stack, Queue(자료구조)

<figure><img src="../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

```java
package stack.main1;

import java.util.Scanner;
import java.util.Stack;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        String str = scanner.next();
        Stack<Character> stack = new Stack<>();
        for (char c : str.toCharArray()) {
            if (c == '(') {
                stack.push(c);
            } else {
                if (stack.isEmpty()) {
                    System.out.println("NO");
                    return;
                }
                stack.pop();
            }
        }

        if (stack.isEmpty()) {
            System.out.println("YES");
        } else {
            System.out.println("NO");
        }
    }
}

```

<figure><img src="../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

```java
package stack.main2;

import java.util.Scanner;
import java.util.Stack;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        String str = scanner.next();
        Stack<Character> stack = new Stack<>();
        for (char c : str.toCharArray()) {
            if (c == ')') {
                while (stack.pop() != '(') {
                }
            } else {
                stack.push(c);
            }
        }

        stack.forEach(System.out::print);
    }
}

```

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

```java
package stack.main3;

import java.util.Scanner;
import java.util.Stack;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();
        int[][] matric = new int[n][n];
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                matric[i][j] = scanner.nextInt();
            }
        }

        int m = scanner.nextInt();
        int[] moves = new int[m];
        for (int i = 0; i < m; i++) {
            moves[i] = scanner.nextInt();
        }

        int count = 0;
        Stack<Integer> stack = new Stack<>();
        for (int number : moves) {
            int column = number - 1;
            for (int rowNum = 0; rowNum < n; rowNum++) {
                if (matric[rowNum][column] != 0) {
                    int dollType = matric[rowNum][column];
                    matric[rowNum][column] = 0;

                    if (!stack.isEmpty() && stack.peek() == dollType) {
                        count += 2;
                        stack.pop();
                    } else {
                        stack.push(dollType);
                    }
                    break;
                }
            }
        }

        System.out.println(count);
    }
}

```

<figure><img src="../../.gitbook/assets/image (128).png" alt=""><figcaption></figcaption></figure>

```java
package stack.main4;

import java.util.Scanner;
import java.util.Stack;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        String str = scanner.next();
        Stack<Integer> stack = new Stack<>();
        for (char c : str.toCharArray()) {
            if (Character.isDigit(c)) {
                stack.push(Integer.parseInt(String.valueOf(c)));
            } else {
                int right = Integer.parseInt(String.valueOf(stack.pop()));
                int left = Integer.parseInt(String.valueOf(stack.pop()));
                int result = calculate(left, right, c);
                stack.push(result);
            }
        }

        System.out.println(stack.get(0));
    }

    private static int calculate(int left, int right, char cal) {
        if (cal == '*') {
            return left * right;
        } else if (cal == '/') {
            return left / right;
        } else if (cal == '+') {
            return left + right;
        } else {
            return left - right;
        }
    }
}
```

<figure><img src="../../.gitbook/assets/image (129).png" alt=""><figcaption></figcaption></figure>

```java
package stack.main5;

import java.util.Scanner;
import java.util.Stack;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        String str = scanner.next();
        Stack<Character> stack = new Stack<>();
        int count = 0;
        for (int i = 0; i < str.length(); i++) {
            char c = str.toCharArray()[i];
            if (c == '(') {
                stack.push(c);
            } else {
                stack.pop();
                if (str.toCharArray()[i - 1] == '(') {
                    count += stack.size();
                } else {
                    count++;
                }
            }
        }

        System.out.println(count);
    }
}

```

<figure><img src="../../.gitbook/assets/image (130).png" alt=""><figcaption></figcaption></figure>

```java
package stack.main6;

import java.util.LinkedList;
import java.util.Queue;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();
        int m = scanner.nextInt();

        Queue<Integer> queue = new LinkedList<>();
        for (int i = 1; i <= n; i++) {
            queue.offer(i);
        }

        while (true) {
            for (int i = 1; i < m; i++) {
                queue.offer(queue.poll());
            }
            queue.poll();
            if (queue.size() == 1) {
                System.out.println(queue.poll());
                return;
            }
        }

    }
}
```











