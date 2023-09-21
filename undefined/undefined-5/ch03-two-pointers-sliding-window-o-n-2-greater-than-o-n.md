# CH03 Two pointers, Sliding window\[효율성: O(n^2)-->O(n)]

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```java
package 효율성.main1;

import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        int sizeA = scanner.nextInt();
        int[] arrayA = new int[sizeA];
        for (int i = 0; i < sizeA; i++) {
            arrayA[i] = scanner.nextInt();
        }

        int sizeB = scanner.nextInt();
        int[] arrayB = new int[sizeB];
        for (int i = 0; i < sizeB; i++) {
            arrayB[i] = scanner.nextInt();
        }

        int indexA = 0;
        int indexB = 0;

        List<Integer> result = new ArrayList<>();
        while (indexA < sizeA && indexB < sizeB) {
            int elementA = arrayA[indexA];
            int elementB = arrayB[indexB];

            if (elementA > elementB) {
                result.add(elementB);
                indexB++;
            } else if (elementA < elementB) {
                result.add(elementA);
                indexA++;
            } else {
                result.add(elementA);
                result.add(elementB);
                indexA++;
                indexB++;
            }
        }

        while (indexA < sizeA) {
            result.add(arrayA[indexA]);
            indexA++;
        }

        while (indexB < sizeB) {
            result.add(arrayB[indexB]);
            indexB++;
        }

        result.forEach(element -> {
            System.out.print(element + " ");
        });
    }
}

```

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```java
package 효율성.main2;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        int sizeA = scanner.nextInt();
        int[] arrayA = new int[sizeA];
        for (int i = 0; i < sizeA; i++) {
            arrayA[i] = scanner.nextInt();
        }
        Arrays.sort(arrayA);

        int sizeB = scanner.nextInt();
        int[] arrayB = new int[sizeB];
        for (int i = 0; i < sizeB; i++) {
            arrayB[i] = scanner.nextInt();
        }
        Arrays.sort(arrayB);

        int indexA = 0;
        int indexB = 0;

        List<Integer> result = new ArrayList<>();
        while (indexA < sizeA && indexB < sizeB) {
            int elementA = arrayA[indexA];
            int elementB = arrayB[indexB];

            if (elementA == elementB) {
                result.add(elementA);
                indexA++;
                indexB++;
            } else {
                if (elementA > elementB) {
                    indexB++;
                } else {
                    indexA++;
                }
            }
        }

        result.forEach(element -> {
            System.out.print(element + " ");
        });
    }
}

```

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```java
package 효율성.main3;

import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int totalDays = scanner.nextInt();
        int targetDays = scanner.nextInt();

        int[] sales = new int[totalDays];
        for (int i = 0; i < totalDays; i++) {
            sales[i] = scanner.nextInt();
        }

        // initialize sum
        int sum = 0;
        for (int i = 0; i < targetDays; i++) {
            sum += sales[i];
        }

        int max = sum;
        for (int i = targetDays; i <= totalDays - targetDays; i++) {
            int newElement = sales[i];
            int deleteTargetElement = sales[i - targetDays];
            sum = sum + newElement - deleteTargetElement;
            max = Math.max(max, sum);
        }

        System.out.println(max);
    }
}

```

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```java
package 효율성.main4;

import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int totalCount = scanner.nextInt();
        int targetValue = scanner.nextInt();

        int[] array = new int[totalCount];
        for (int i = 0; i < totalCount; i++) {
            array[i] = scanner.nextInt();
        }

        int count = 0;
        int leftIndex = 0;
        int sum = 0;
        for (int rightIndex = 0; rightIndex < totalCount; rightIndex++) {
            sum += array[rightIndex];
            if (sum == targetValue) count++;
            while (sum >= targetValue) {
                sum -= array[leftIndex];
                leftIndex++;
                if (sum == targetValue) count++;
            }
        }
        System.out.println(count);
    }
}

```

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();

        int limit = (n / 2) + 1;
        int[] arr = new int[limit];
        for (int i = 0; i < limit; i++) {
            arr[i] = i + 1;
        }

        int sum = 0;
        int count = 0;
        int lt = 0;
        for (int rt = 0; rt < limit; rt++) {
            sum += arr[rt];
            if (sum == n) {
                count++;
            }

            while (sum > n) {
                sum -= arr[lt];
                lt++;
                if (sum == n) {
                    count++;
                }
            }
        }

        System.out.println(count);
    }
//    public static void main(String[] args) {
//        Scanner scanner = new Scanner(System.in);
//
//        int targetValue = scanner.nextInt();
//        int[] array = new int[targetValue];
//        for (int i = 0; i < targetValue; i++) {
//            array[i] = i + 1;
//        }
//
//        int limit = (targetValue / 2) + 1;
//        int sum = 0;
//        int leftIndex = 0;
//        int count = 0;
//        for (int rightIndex = 0; rightIndex <= limit; rightIndex++) {
//            sum += array[rightIndex];
//            if (sum == targetValue) count++;
//            while (sum >= targetValue) {
//                sum -= array[leftIndex];
//                leftIndex++;
//                if (sum == targetValue) count++;
//            }
//        }
//
//        System.out.println(count);
//    }
}

```

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```java
package 효율성.main6;

import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int count = scanner.nextInt();
        int permitCount = scanner.nextInt();
        int[] array = new int[count];
        for (int i = 0; i < count; i++) {
            array[i] = scanner.nextInt();
        }

        int zeroCount = 0;
        int max = 0;
        int leftIndex = 0;
        for (int rightIndex = 0; rightIndex < count; rightIndex++) {
            if (array[rightIndex] == 0) {
                zeroCount++;
            }

            if (zeroCount <= permitCount) {
                max = Math.max(max, rightIndex - leftIndex + 1);
            }
            
            while (zeroCount > permitCount) {
                if (array[leftIndex] == 0) {
                    zeroCount--;
                }
                leftIndex++;

                if (zeroCount <= permitCount) {
                    max = Math.max(max, rightIndex - leftIndex + 1);
                }
            }
        }

        System.out.println(max);
    }
}

```

