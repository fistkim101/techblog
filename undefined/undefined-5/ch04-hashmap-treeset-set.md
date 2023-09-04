# CH04 HashMap, TreeSet (해쉬, 정렬지원 Set)

<figure><img src="../../.gitbook/assets/image (113).png" alt=""><figcaption></figcaption></figure>

```java
package 해쉬트리셋.hash1;

import java.util.HashMap;
import java.util.Map;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int count = scanner.nextInt();
        String vote = scanner.next();
        char[] array = vote.toCharArray();

        HashMap<Character, Integer> voteSet = new HashMap<>();
        for (Character key : array) {
            voteSet.put(key, voteSet.getOrDefault(key, 0) + 1);

//            Integer currentValue = voteSet.get(key);
//            if (currentValue == null) {
//                voteSet.put(key, 1);
//            } else {
//                Integer addedValue = currentValue + 1;
//                voteSet.put(key, addedValue);
//            }
        }

        int max = 0;
        Character leader = null;
        for (Map.Entry<Character, Integer> entry : voteSet.entrySet()) {
            if (max < entry.getValue()) {
                leader = entry.getKey();
                max = entry.getValue();
            }
        }

        System.out.println(leader);
    }
}

```

<figure><img src="../../.gitbook/assets/image (114).png" alt=""><figcaption></figcaption></figure>

```java
package 해쉬트리셋.hash2;

import java.util.HashMap;
import java.util.Map;
import java.util.Objects;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        String first = scanner.next();
        String second = scanner.next();

        Map<Character, Integer> firstMap = convertToMap(first);
        Map<Character, Integer> secondMap = convertToMap(second);

        boolean isSame = true;
        for (Map.Entry<Character, Integer> entry : firstMap.entrySet()) {
            if (!secondMap.containsKey(entry.getKey()) || !Objects.equals(secondMap.get(entry.getKey()), entry.getValue())) {
                isSame = false;
                break;
            }
        }

        if (isSame) {
            System.out.println("YES");
        } else {
            System.out.println("NO");
        }
    }


    private static Map<Character, Integer> convertToMap(String str) {
        Map<Character, Integer> converted = new HashMap<>();
        for (char c : str.toCharArray()) {
            converted.put(c, converted.getOrDefault(c, 0) + 1);
        }

        return converted;
    }
}

```

```java
package 해쉬트리셋.hash2;

import java.util.HashMap;
import java.util.Map;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        String first = scanner.next();
        String second = scanner.next();

        Map<Character, Integer> targetMap = new HashMap<>();
        for (char c : first.toCharArray()) {
            targetMap.put(c, targetMap.getOrDefault(c, 0) + 1);
        }

        for (char c : second.toCharArray()) {
            if (!targetMap.containsKey(c) || targetMap.get(c) == 0) {
                System.out.println("NO");
                return;
            }

            targetMap.put(c, targetMap.get(c) - 1);
        }

        System.out.println("YES");
    }

//    public static void main(String[] args) {
//        Scanner scanner = new Scanner(System.in);
//        String first = scanner.next();
//        String second = scanner.next();
//
//        Map<Character, Integer> firstMap = convertToMap(first);
//        Map<Character, Integer> secondMap = convertToMap(second);
//
//        boolean isSame = true;
//        for (Map.Entry<Character, Integer> entry : firstMap.entrySet()) {
//            if (!secondMap.containsKey(entry.getKey()) || !Objects.equals(secondMap.get(entry.getKey()), entry.getValue())) {
//                isSame = false;
//                break;
//            }
//        }
//
//        if (isSame) {
//            System.out.println("YES");
//        } else {
//            System.out.println("NO");
//        }
//    }
//
//
//    private static Map<Character, Integer> convertToMap(String str) {
//        Map<Character, Integer> converted = new HashMap<>();
//        for (char c : str.toCharArray()) {
//            converted.put(c, converted.getOrDefault(c, 0) + 1);
//        }
//
//        return converted;
//    }
}

```

<figure><img src="../../.gitbook/assets/image (115).png" alt=""><figcaption></figcaption></figure>

```java
package 해쉬트리셋.hash3;

import java.util.HashMap;
import java.util.Map;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int totalDays = scanner.nextInt();
        int period = scanner.nextInt();
        int[] sales = new int[totalDays];
        for (int i = 0; i < totalDays; i++) {
            sales[i] = scanner.nextInt();
        }

        Map<Integer, Integer> kinds = new HashMap<>();
        for (int i = 0; i < period - 1; i++) {
            kinds.put(sales[i], kinds.getOrDefault(sales[i], 0) + 1);
        }

        int leftIndex = 0;
        StringBuilder result = new StringBuilder();
        for (int rightIndex = period - 1; rightIndex < totalDays; rightIndex++) {
            kinds.put(sales[rightIndex], kinds.getOrDefault(sales[rightIndex], 0) + 1);
            result.append(kinds.size()).append(" ");

            kinds.put(sales[leftIndex], kinds.get(sales[leftIndex]) - 1);
            if (kinds.get(sales[leftIndex]) == 0) {
                kinds.remove(sales[leftIndex]);
            }
            leftIndex++;
        }

        System.out.println(result);
    }
}

```



<figure><img src="../../.gitbook/assets/image (116).png" alt=""><figcaption></figcaption></figure>

```java
package 해쉬트리셋.hash4;

import java.util.HashMap;
import java.util.Map;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        String first = scanner.next();
        String second = scanner.next();

        Map<Character, Integer> targetMap = new HashMap<>();
        for (char c : second.toCharArray()) {
            targetMap.put(c, targetMap.getOrDefault(c, 0) + 1);
        }

        char[] firstArray = first.toCharArray();
        Map<Character, Integer> loopMap = new HashMap<>();
        for (int i = 0; i < second.length() - 1; i++) {
            loopMap.put(firstArray[i], loopMap.getOrDefault(firstArray[i], 0) + 1);
        }

        int leftIndex = 0;
        int count = 0;
        for (int rightIndex = second.length() - 1; rightIndex < first.length(); rightIndex++) {
            loopMap.put(firstArray[rightIndex], loopMap.getOrDefault(firstArray[rightIndex], 0) + 1);
            if (loopMap.equals(targetMap)) count++;

            loopMap.put(firstArray[leftIndex], loopMap.get(firstArray[leftIndex]) - 1);
            if (loopMap.get(firstArray[leftIndex]) == 0) {
                loopMap.remove(firstArray[leftIndex]);
            }
            leftIndex++;
        }

        System.out.println(count);
    }
}

```

<figure><img src="../../.gitbook/assets/image (117).png" alt=""><figcaption></figcaption></figure>

```java
package 해쉬트리셋.hash5;

import java.util.Collections;
import java.util.Scanner;
import java.util.TreeSet;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int totalCount = scanner.nextInt();
        int targetIndex = scanner.nextInt();
        int[] array = new int[totalCount];
        for (int i = 0; i < totalCount; i++) {
            array[i] = scanner.nextInt();
        }

        TreeSet<Integer> tSet = new TreeSet<>(Collections.reverseOrder());
        for (int i = 0; i < totalCount; i++) {
            for (int j = i + 1; j < totalCount; j++) {
                for (int k = j + 1; k < totalCount; k++) {
                    tSet.add(array[i] + array[j] + array[k]);
                }
            }
        }

        int count = 0;
        for (int sum : tSet) {
            count++;
            if (count == targetIndex) {
                System.out.println(sum);
            }
        }

        if (count < targetIndex) {
            System.out.println("-1");
        }
    }
}
```
