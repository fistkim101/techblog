# String interning

## String interning(문자열 인터닝) 이란

> In computer science, string interning is a method of storing only one copy of each distinct [string](https://en.wikipedia.org/wiki/String\_\(computer\_science\)) value, which must be [immutable](https://en.wikipedia.org/wiki/Immutable\_object).[\[1\]](https://en.wikipedia.org/wiki/String\_interning#cite\_note-1) Interning strings makes some string processing tasks more time- or space-efficient at the cost of requiring more time when the string is created or interned. The distinct values are stored in a string intern pool.

[위키](https://en.wikipedia.org/wiki/String\_interning)에 위와 같이 잘 설명되고 있다. 핵심만 아래에 쓰자면 아래와 같다.

> 문자열 이터닝이란 불변의 유일 문자열로 각 문자열 copy 를 보관하기 위한 방법이다. 문자열을 인터닝 하는 것은 문자열이 새로 생성됨에 있어서 시간 복잡도, 공간 복잡도 측면에서 훨씬 효율적으로 만들어준다. 각각의 구분되는 값은 문자열 풀에 저장된다.



쉽게 말해서 문자열은 내부적으로 문자열 풀에 캐싱처럼 저장해놨다가 동일한 문자열이 필요할 때 이미 만들어진 문자열에 그것이 있으면 그 객체를 반환한다는 것이다.

```java
        String hi1 = "hi";
        String hi2 = new String("hi");
        String hi3 = "hi";

        System.out.println(hi1 == hi2); // false
        System.out.println(hi2 == hi3); // false
        System.out.println(hi3 == hi1); // true
```

new String 을 사용한 경우 문자열 인터닝을 무시하게 된다. 그래서 hi1 과 hi2 은 객체는 동등하지만 다른 참조값을 가지게 된다. 동일성이 깨지는 것이다. 하지만 hi3의 경우 "hi" 로 결과적으로 h1 과 동일하고 h1 생성시 인터닝 된 문자열을 바라보게 되어서 h3과 h1의 참조값은 같아진다.



## 문자열 인터닝이 주는 성능 효율을 확인해보자

아래 코드로는 76\~77ms 이 나오고 있다. 문자열 풀에서 꺼내 쓰는 것을 피하고 새로운 객체를 일부러 만들게 했다.

```java
public class Main {
    public static void main(String[] args) {

        long startTime = System.currentTimeMillis();
        String hi1 = "hi";
        for (int i = 0; i < Integer.MAX_VALUE; i++) {
            String hi2 = new String("hi");
        }
        long endTime = System.currentTimeMillis();

        System.out.println(endTime - startTime);
    }
}
```



반면에 아래 코드는 1ms 이 찍힌다. 그냥 숫자만으로 속도가 70배 이상 차이가 난다.

```java
public class Main {
    public static void main(String[] args) {

        long startTime = System.currentTimeMillis();
        String hi1 = "hi";
        for (int i = 0; i < Integer.MAX_VALUE; i++) {
            String hi2 = "hi";
        }
        long endTime = System.currentTimeMillis();

        System.out.println(endTime - startTime);
    }
}
```
