# CH06 Sorting and Searching(정렬, 이분검색과 결정알고리즘)

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

```java
package 정렬_new.main1;

import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();
        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = scanner.nextInt();
        }

        for (int i = 0; i < n - 1; i++) {
            for (int j = i + 1; j < n; j++) {
                if (arr[i] > arr[j]) {
                    int temp = arr[j];
                    arr[j] = arr[i];
                    arr[i] = temp;
                }
            }
        }

        for (int i : arr) {
            System.out.print(i + " ");
        }
    }
}
```

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

```java
package 정렬_new.main2;

import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();
        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = scanner.nextInt();
        }

        for (int i = 0; i < n - 1; i++) {
            for (int j = i + 1; j < n; j++) {
                if (arr[i] > arr[j]) {
                    int temp = arr[j];
                    arr[j] = arr[i];
                    arr[i] = temp;
                }
            }
        }

        for (int i : arr) {
            System.out.print(i + " ");
        }
    }
}
```

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

```java
package 정렬_new.main3;

import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();
        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = scanner.nextInt();
        }

        for (int i = 1; i < n; i++) {
            int temp = arr[i];
            int j;
            for (j = i - 1; j >= 0; j--) {
                if (temp < arr[j]) {
                    arr[j + 1] = arr[j];
                } else {
                    break;
                }
            }

            arr[j + 1] = temp;
        }

        for (int i : arr) {
            System.out.print(i + " ");
        }
    }
}
```

<figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

```java
package 정렬_new.main4;

import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int cacheSize = scanner.nextInt();
        int[] cacheMemory = new int[cacheSize];

        int taskSize = scanner.nextInt();
        int[] tasks = new int[taskSize];
        for (int i = 0; i < taskSize; i++) {
            tasks[i] = scanner.nextInt();
        }

        for (int task : tasks) {
            int hitIndex = -1;
            for (int i = 0; i < cacheSize; i++) {
                if (cacheMemory[i] == task) {
                    hitIndex = i;
                    break;
                }
            }

            if (hitIndex == -1) {
                // cache miss
                for (int i = cacheSize - 1; i >= 1; i--) {
                    cacheMemory[i] = cacheMemory[i - 1];
                }
            } else {
                // cache hit
                for (int i = hitIndex; i >= 1; i--) {
                    cacheMemory[i] = cacheMemory[i - 1];
                }
            }
            cacheMemory[0] = task;
        }

        for (int task : cacheMemory) {
            System.out.print(task + " ");
        }
    }
}
```

<figure><img src="../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

```java
package 정렬.main5;

import java.util.HashSet;
import java.util.Scanner;
import java.util.Set;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int count = scanner.nextInt();
        int[] numbers = new int[count];
        for (int i = 0; i < count; i++) {
            numbers[i] = scanner.nextInt();
        }

        Set<Integer> set = new HashSet<>();
        for (int i = 0; i < count; i++) {
            set.add(numbers[i]);
        }

        if (set.size() == count) {
            System.out.println("U");
        } else {
            System.out.println("D");
        }

    }
}
```

```java
package 정렬_new.main5;

import java.util.Arrays;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();
        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = scanner.nextInt();
        }

        Arrays.sort(arr);
        for (int i = 0; i < n - 1; i++) {
            if (arr[i] == arr[i + 1]) {
                System.out.println("D");
                return;
            }
        }

        System.out.println("U");
    }
}
```

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

```java
package 정렬_new.main6;

import java.util.Arrays;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();
        int[] arr = new int[n];
        int[] correctOrder = new int[n];
        for (int i = 0; i < n; i++) {
            int value = scanner.nextInt();
            arr[i] = value;
            correctOrder[i] = value;
        }

        Arrays.sort(correctOrder);
        for (int i = 0; i < n; i++) {
            if (arr[i] != correctOrder[i]) {
                System.out.print(i + 1 + " ");
            }
        }
    }
}
```

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

```java
package 정렬_new.main7;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.Scanner;

class Point implements Comparable<Point> {
    int x;
    int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public int compareTo(Point o) {
        Point target = (Point) o;
        if (this.x == target.x) {
            return this.y - target.y;
        } else {
            return this.x - target.x;
        }
    }

    public void print() {
        System.out.println(this.x + " " + this.y);
    }
}

public class Main {
    public static void main(String[] args) {
        final Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();
        final List<Point> points = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            Point point = new Point(scanner.nextInt(), scanner.nextInt());
            points.add(point);
        }

        Collections.sort(points);
        for (Point p : points) {
            p.print();
        }
    }
}

```

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

```java
package 정렬_new.main8;

import java.util.Arrays;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();
        int target = scanner.nextInt();

        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = scanner.nextInt();
        }

        Arrays.sort(arr);
        int lt = 0;
        int rt = n - 1;
        int answer = 0;
        while (lt <= rt) {
            int mid = (lt + rt) / 2;
            int midValue = arr[mid];

            if (midValue == target) {
                answer = mid + 1;
                break;
            }

            if (midValue > target) {
                rt = mid - 1;
            } else {
                lt = mid + 1;
            }
        }

        System.out.println(answer);
    }
}
```

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

```java
package 정렬_new.main9;

import java.util.Arrays;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();
        int limitCount = scanner.nextInt();
        int[] arr = new int[n];

        for (int i = 0; i < n; i++) {
            arr[i] = scanner.nextInt();
        }

        int lt = Arrays.stream(arr).max().getAsInt();
        int rt = Arrays.stream(arr).sum();
        int answer = 0;
        while (lt <= rt) {
            int midValue = (lt + rt) / 2;

            if (count(arr, midValue) <= limitCount) {
                answer = midValue;
                rt = midValue - 1;
            } else {
                lt = midValue + 1;
            }
        }

        System.out.println(answer);
    }

    private static int count(int[] arr, int capacity) {
        int count = 1;
        int sum = 0;
        for (int i = 0; i < arr.length; i++) {
            if (sum + arr[i] > capacity) {
                count++;
                sum = arr[i];
            } else {
                sum += arr[i];
            }
        }

        return count;
    }
}
```

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

```java
package 정렬_new.main10;

import java.util.Arrays;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();
        int horseCount = scanner.nextInt();
        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = scanner.nextInt();
        }

        Arrays.sort(arr);
        int lt = 1;
        int rt = arr[n - 1];
        int answer = 0;
        while (lt <= rt) {
            int mid = (lt + rt) / 2;

            if (count(arr, mid) >= horseCount) {
                answer = mid;
                lt = mid + 1;
            } else {
                rt = mid - 1;
            }
        }

        System.out.println(answer);
    }

    private static int count(int[] arr, int gap) {
        int horseCount = 1;
        int currentGap = 0;
        for (int i = 1; i < arr.length; i++) {
            currentGap += arr[i] - arr[i - 1];
            if (currentGap >= gap) {
                horseCount++;
                currentGap = 0;
            }
        }

        return horseCount;
    }
}

```

