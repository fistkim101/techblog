# CH02 Array(1, 2 차원 배열)

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

```java
package array.array1;

import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int target = scanner.nextInt();
        int[] array = new int[target];
        for (int i = 0; i < target; i++) {
            array[i] = scanner.nextInt();
        }

        Main main = new Main();
        main.solution(target, array);
    }

    private void solution(int target, int[] array) {
        System.out.print(array[0]);
        for (int i = 1; i < array.length; i++) {
            if (array[i] > array[i - 1]) {
                System.out.print(" " + array[i]);
            }
        }
    }

}

```

<figure><img src="../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```java
package array.array2;

import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int count = scanner.nextInt();
        int[] array = new int[count];
        for (int i = 0; i < count; i++) {
            array[i] = scanner.nextInt();
        }

        int max = 0;
        int result = 0;
        for (int i : array) {
            if (i > max) {
                result++;
                max = i;
            }
        }

        System.out.println(result);
    }

}

```

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

```java
package array.array3;

import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        int count = scanner.nextInt();
        int[] aStrategy = new int[count];
        for (int i = 0; i < count; i++) {
            aStrategy[i] = scanner.nextInt();
        }

        int[] bStrategy = new int[count];
        for (int i = 0; i < count; i++) {
            bStrategy[i] = scanner.nextInt();
        }

        for (int i = 0; i < count; i++) {
            printResult(aStrategy[i], bStrategy[i]);
        }
    }

    private static void printResult(int aStrategy, int bStrategy) {
        if (aStrategy == bStrategy) {
            System.out.println("D");
        } else if ((aStrategy == 1 && bStrategy == 3) || (aStrategy == 2 && bStrategy == 1) || (aStrategy == 3 && bStrategy == 2)) {
            System.out.println("A");
        } else {
            System.out.println("B");
        }
    }

}

```



<figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

```java
package array.array4;

import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int count = scanner.nextInt();
        int[] array = new int[count];
        array[0] = 1;
        array[1] = 1;
        System.out.print(1 + " " + 1);
        for (int i = 2; i < count; i++) {
            array[i] = array[i - 2] + array[i - 1];
            System.out.print(" " + array[i]);
        }
    }

}

```

<figure><img src="../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

```java
package array.array5;

import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int limit = scanner.nextInt();
        int[] array = new int[limit + 1];

        int result = 0;
        for (int i = 2; i <= limit; i++) {
            if (array[i] == 0) {
                result++;
                for (int j = i; j <= limit; j = j + i) {
                    array[j] = 1;
                }
            }
        }

        System.out.println(result);
    }
}

```

<figure><img src="../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

```java
package array.array6;

import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int count = scanner.nextInt();
        int[] array = new int[count];

        for (int i = 0; i < count; i++) {
            array[i] = scanner.nextInt();
        }

        for (int number : array) {
            int reversedNumber = Integer.parseInt((new StringBuilder(String.valueOf(number)).reverse()).toString());
            if (isPrimeNumber(reversedNumber)) {
                System.out.print(reversedNumber + " ");
            }
        }
    }

    private static boolean isPrimeNumber(int number) {
        if (number < 2) {
            return false;
        }

        boolean result = true;
        for (int i = 2; i < number; i++) {
            if (number % i == 0) {
                return false;
            }
        }

        return result;
    }

}

```

<figure><img src="../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

```java
package array.array7;

import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int count = scanner.nextInt();
        int[] array = new int[count];
        for (int i = 0; i < count; i++) {
            array[i] = scanner.nextInt();
        }

        int weight = 0;
        int sum = 0;
        for (int result : array) {
            if (result == 0) {
                weight = 0;
            } else {
                sum += 1 + weight;
                weight++;
            }
        }

        System.out.println(sum);
    }

}

```



<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

```java
package array.array8;

import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int count = scanner.nextInt();

        int[] array = new int[count];
        for (int i = 0; i < count; i++) {
            array[i] = scanner.nextInt();
        }

        for (int i : array) {
            System.out.print(getOrder(array, i) + " ");
        }

    }

    private static int getOrder(int[] array, int targetNumber) {
        int order = 1;

        for (int i : array) {
            if (i > targetNumber) {
                order++;
            }
        }

        return order;
    }

}

```

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

```java
package array.array9;

import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int count = scanner.nextInt();

        int[][] row = new int[count][count];
        for (int i = 0; i < count; i++) {
            for (int j = 0; j < count; j++) {
                row[i][j] = scanner.nextInt();
            }
        }

        int max = 0;

        // row, column
        for (int i = 0; i < count; i++) {
            int rowSum = 0;
            int columSum = 0;
            for (int j = 0; j < count; j++) {
                rowSum += row[i][j];
                columSum += row[j][i];
            }
            int tmpMax = Math.max(rowSum, columSum);
            max = Math.max(max, tmpMax);
        }

        int leftToRight = 0;
        for (int i = 0; i < count; i++) {
            leftToRight += row[i][i];
        }
        max = Math.max(max, leftToRight);

        // x > 00,11,22 | 02,11,20
        int rightToLeft = 0;
        for (int i = 0; i < count; i++) {
            rightToLeft += row[i][count - 1 - i];
        }
        max = Math.max(max, rightToLeft);


        System.out.println(max);
    }

}

```

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

```java
package array.array10;

import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int count = scanner.nextInt();
        int[][] array = new int[count + 2][count + 2];
        for (int i = 1; i < count + 1; i++) {
            for (int j = 1; j < count + 1; j++) {
                array[i][j] = scanner.nextInt();
            }
        }

//        for (int i = 0; i < count + 2; i++) {
//            for (int j = 0; j < count + 2; j++) {
//                if (i == 0) {
//                    array[i][j] = 0;
//                } else if (i == count + 1) {
//                    array[i][j] = 0;
//                } else if (j == 0 || j == count + 1) {
//                    array[i][j] = 0;
//                } else {
//                    array[i][j] = scanner.nextInt();
//                }
//            }
//        }

        int resultCount = 0;
        for (int i = 1; i < count + 1; i++) {
            for (int j = 1; j < count + 1; j++) {
                int targetNumber = array[i][j];

                int top = array[i - 1][j];
                int bottom = array[i + 1][j];
                int left = array[i][j - 1];
                int right = array[i][j + 1];

                if (top < targetNumber && bottom < targetNumber && left < targetNumber && right < targetNumber) {
                    resultCount++;
                }
            }
        }

        System.out.println(resultCount);
    }

}

```

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

```java
package array.array11;

import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int memberCount = scanner.nextInt();
        int[][] array = new int[memberCount][5];
        for (int i = 0; i < memberCount; i++) {
            for (int j = 0; j < 5; j++) {
                array[i][j] = scanner.nextInt();
            }
        }

        int leader = 0;
        int maxCount = 0;
        for (int i = 0; i < memberCount; i++) {
            int sameCount = 0;
            for (int j = 0; j < memberCount; j++) {
                for (int k = 0; k < 5; k++) {
                    int currentGrade = array[i][k];
                    int targetGrade = array[j][k];
                    if (currentGrade == targetGrade) {
                        sameCount++;
                        break;
                    }
                }
            }

            if (maxCount < sameCount) {
                maxCount = sameCount;
                leader = i;
            }
        }

        System.out.println(leader + 1);
    }

}

```

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

```java
package array.array12;

import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int memberCount = scanner.nextInt();
        int testCount = scanner.nextInt();

        int[][] array = new int[testCount][memberCount];
        for (int i = 0; i < testCount; i++) {
            for (int j = 0; j < memberCount; j++) {
                array[i][j] = scanner.nextInt();
            }
        }

        int count = 0;
        for (int i = 1; i <= memberCount; i++) {
            for (int j = 1; j <= memberCount; j++) {
                if (i == j) {
                    continue;
                }

                boolean iIsMentor = true;
                for (int k = 0; k < testCount; k++) {
                    int iPosition = 0;
                    int jPosition = 0;
                    for (int p = 0; p < memberCount; p++) {
                        if (array[k][p] == i) {
                            iPosition = p;
                        }
                        if (array[k][p] == j) {
                            jPosition = p;
                        }
                    }

                    if (iPosition > jPosition) {
                        iIsMentor = false;
                        break;
                    }
                }

                if (iIsMentor) {
                    count++;
                }
            }
        }

        System.out.println(count);
    }

}

```

