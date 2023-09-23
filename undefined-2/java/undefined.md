# 함수형 인터페이스

## 함수형 인터페이스란

'오직 하나의 추상 메서드만 가지고 있는 인터페이스'를 함수형 인터페이스라고 한다. 이 경우 모두 함수형 인터페이스로 간주된다. @FunctionalInterface 어노테이션을 사용해주면 더 명시적으로 함수형 인터페이스 선언이 가능하다. 이를 붙여주지 않으면 두 개 이상의 함수를 정의해도 에러가 나지 않기 때문에 선언해주는 것이 좋다.



## 람다식을 활용한 함수형 인터페이스 구현

```java
@FunctionalInterface
public interface MyFunctionalInterface {
    void execute();
}
```

```java
MyFunctionalInterface mfi = () -> System.out.println("Executing...");
mfi.execute();
```

람다식을 이용하면 위와 같이 깔끔하게 구현이 가능하다.



## 자주 사용되는 표준 함수형 인터페이스

```java
public class Main {
    public static void main(String[] args) {
        Function<Integer, String> intToString1 = i -> i.toString();
        String result1 = intToString1.apply(1);
        System.out.println(result1); // 1

        Function<Integer, String> intToString2 = Object::toString;
        String result2 = intToString2.apply(1);
        System.out.println(result2); // 1

        Supplier<String> newString = () -> "hello";
        String result3 = newString.get(); // hello
        System.out.println(result3);

        Consumer<String> sayHello1 = (str) -> System.out.println(str);
        sayHello1.accept("sayHello1"); // sayHello1

        Consumer<String> sayHello2 = System.out::println;
        sayHello1.accept("sayHello2"); // sayHello1

        Predicate<Person> isAdult = (person) -> person.getAge() > 19;
        System.out.println(isAdult.test(new Person(20))); // true
        System.out.println(isAdult.test(new Person(18))); // false
    }
}
```
