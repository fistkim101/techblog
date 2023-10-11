# \[14] Comparable 을 구현할지 고려하라

## Comparable 을 구현했다 = 해당 클래스에 자연적인 '순서'가 있다

Comparable 인터페이스를 구현한 구현체들은 컬렉션의 정렬 기능을 사용할 수 있다. 코딩테스트시 가끔 사용되곤 했다.



## compareTo 일반 규약

equals와 마찬기자로 규약이 존재한다.

```java
public class Main {
    public static void main(String[] args) {
        BigDecimal n1 = new BigDecimal("1");
        BigDecimal n2 = new BigDecimal("2");
        BigDecimal n3 = new BigDecimal("3");

        // 반사성
        System.out.println(n1.compareTo(n1)); // 0

        // 대칭성
        System.out.println(n1.compareTo(n2)); // -1
        System.out.println(n2.compareTo(n1)); // 1

        // 추이성
        if (n1.compareTo(n2) == -1 && n2.compareTo(n3) == -1) {
            System.out.println(n1.compareTo(n3)); // -1
        }
    }
}
```



## 정렬된 컬렉션은 동치성 비교시 equals 대신 compareTo 를 사용한다

```java
public class Main {
    public static void main(String[] args) {
        BigDecimal n1 = new BigDecimal("1.0");
        BigDecimal n2 = new BigDecimal("1.00");

        HashSet<BigDecimal> hashSet = new HashSet<>();
        hashSet.add(n1);
        hashSet.add(n2);
        System.out.println(hashSet.size()); // 2

        TreeSet<BigDecimal> treeSet = new TreeSet<>();
        treeSet.add(n1);
        treeSet.add(n2);
        System.out.println(treeSet.size()); // 1
    }
}
```

TreeSet 은 자체적으로 넣을때마다 순서를 고려해서 적재되는 컬렉션인데 이러한 '정렬된 컬렉션' 은 같고 다름의 동치성 판단을 equals 가 아니라 compareTo 로 판단한다.

1.0 과 1.00이 다른 해시코드를 반환하기 때문에 HashMap 에서는 다른 객체로 분류되어 사이즈가 2개가 나오는 것이고, TreeSet 에서는 compareTo 해보면 0가 나오니까 같다고 판단해서 사이즈가 1개가 되는 것이다.



## 상속을 한 클래스의 순서 비교를 위해서는 컴포지션을 활용하라

필드를 추가하고 컬렉션에 커스텀한 compare 를 설정해주면 규약이 깨진다. equals 때와 마찬가지로 규약을 지키기 위해서는 합성을 사용하는 것을 권장한다.



## > 또는 < 사용에 주의하자(기본 제공 compare 적극 활용하자)

```java
public class Main {
    public static void main(String[] args) {
        int min = Integer.MIN_VALUE;
        System.out.println(min); // -2147483648
        System.out.println(Integer.MIN_VALUE - 10); // 2147483638
        System.out.println(Integer.compare(min, -10)); // -1
    }
}
```

코드에서 확인 할 수 있듯이 의도치 않은 값이 나올 수 있다.
