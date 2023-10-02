# (부록) 람다(lambda)

나온지 꽤 오래된 책이라서 당시에는 람다가 보편화 되기 전이었던 것 같다. 람다를 약간 핫한(?) 컨셉으로 다루고 있다. 책에서 다루고 있는 내용과 직접 다룬 것은 아니지만 연관해서 필요하다 싶은 내용을 정리한다.



## 함수형 인터페이스란

자바 8 이상에서 제공되는 것으로 단 하나의 추상메서드를 가지고 있는 인터페이스이다.

```java
// 함수형 인터페이스 정의
@FunctionalInterface
public interface MyFunctionalInterface {
    void myMethod();
}
```

함수형 인터페이스를 먼저 짚는 이유는 이후에 다룰 람다와 연관되기 때문이다.



## 람다(lambda)란

람다라는 표현을 자주 써서 막상 람다가 무엇인지, 어원이 뭔지에 대해서는 고민을 덜 해보았다. 그냥 '스트림 쓸때 사용하는 함수 표현하는 방식이 람다' 라고만 이해하고 있었다. 큰 틀에서 틀리진 않는데 다시 한번 정리를 하자.



### 람다 어원

람다는 수학에 있는 람다 대수이다. 람다 대수는 함수를 표현함에 있어 추상화시켜 표현하는 일종의 '함수 표현 기법'이다. 자바의 람다 말고 '람다' 자체가 '함수를 추상화 해서 표현하는 방법' 인 것이다.



### 자바의 람다

그래서 람다를 '람다 표현식' 이라고도 할 수 있다. 람다를 이용해서 익명 함수를 정의해서 스트림 등에서 쉽게 활용할 수 있다. 또 람다를 활용 함으로써 함수 자체를 변수처럼 전달하거나 반환하게 할 수 있다.

```java
// 매개변수가 없고, "Hello, World!"를 출력
Runnable r = () -> System.out.println("Hello, World!");

// 매개변수가 하나인 경우, 타입을 명시적으로 지정할 수 있음
Consumer<String> c = (String x) -> System.out.println(x);

// 타입 추론이 가능하면 타입을 생략할 수 있음
Consumer<String> c2 = x -> System.out.println(x);

// 두 개의 매개변수를 받아서 더하는 함수
BinaryOperator<Integer> add = (a, b) -> a + b;

// 두 개의 매개변수를 받아서 더하는 함수 (타입 명시)
BinaryOperator<Integer> addExplicit = (Integer a, Integer b) -> a + b;
```



아래 코드는 내가 NEXTSTEP 코틀린 수업을 들을때 만든 코드인데, 람다를 사용하고 있어서 가져왔다.

{% embed url="https://github.com/fistkim101/kotlin-racingcar/blob/fistkim101/src/main/kotlin/step2/Operation.kt" %}

```kotlin
class OperationConstant {
    companion object {

        val PLUS = "+"
        val MINUS = "-"
        val MULTIPLE = "*"
        val DIVIDE = "/"
        val SYMBOLS = listOf(PLUS, MINUS, MULTIPLE, DIVIDE)
        val SPACE = " "
        val EMPTY = ""
    }
}

enum class Operation(val symbol: String, val calculation: (Long, Long) -> Long) {

    PLUS(OperationConstant.PLUS, { num1: Long, num2: Long -> num1 + num2 }),
    MINUS(OperationConstant.MINUS, { num1: Long, num2: Long -> num1 - num2 }),
    MULTIPLE(OperationConstant.MULTIPLE, { num1: Long, num2: Long -> num1 * num2 }),
    DIVIDE(OperationConstant.DIVIDE, { num1: Long, num2: Long -> num1 / num2 });

    companion object {
        fun findCalculation(symbol: String?): Operation {
            require(symbol != null && symbol != OperationConstant.SPACE && symbol != OperationConstant.EMPTY) {
                "null, 빈 문자열, 공백은 허용되지 않습니다."
            }

            require(values().map { it.symbol }.contains(symbol)) {
                "사칙연산 기호가 아닙니다."
            }

            return values().first() { it.symbol == symbol }
        }
    }
}
```



자바에 만들어져 있는 함수형 인터페이스들은 아래와 같다. 실무에서도 당연히 쓰인다.

> 자바의 표준 라이브러리에는 다양한 함수형 인터페이스가 포함되어 있습니다. 아래는 몇몇 대표적인 예입니다:
>
> #### `java.util.function` 패키지의 주요 함수형 인터페이스:
>
> 1. **`Function<T, R>`**: `T` 타입의 매개변수를 받아 `R` 타입을 반환하는 함수를 나타냅니다.
>    * `R apply(T t);`
> 2. **`Consumer<T>`**: `T` 타입의 매개변수를 받아서 아무 것도 반환하지 않는 함수를 나타냅니다.
>    * `void accept(T t);`
> 3. **`Supplier<T>`**: 매개변수 없이 `T` 타입의 결과를 제공하는 함수를 나타냅니다.
>    * `T get();`
> 4. **`Predicate<T>`**: `T` 타입의 매개변수를 받아서 `boolean`을 반환하는 함수를 나타냅니다.
>    * `boolean test(T t);`
> 5. **`UnaryOperator<T>`**: `T` 타입의 매개변수를 받아 동일한 `T` 타입을 반환하는 함수를 나타냅니다. `Function<T, T>`의 특수한 형태입니다.
>    * `T apply(T t);`
> 6. **`BinaryOperator<T>`**: 같은 타입 `T`의 두 매개변수를 받아 `T` 타입을 반환하는 함수를 나타냅니다.
>    * `T apply(T t1, T t2);`
> 7. **`BiFunction<T, U, R>`**: `T`와 `U` 타입의 매개변수를 받아 `R` 타입을 반환하는 함수를 나타냅니다.
>    * `R apply(T t, U u);`
> 8. **`BiConsumer<T, U>`**: `T`와 `U` 타입의 매개변수를 받아서 아무 것도 반환하지 않는 함수를 나타냅니다.
>    * `void accept(T t, U u);`
> 9. **`BiPredicate<T, U>`**: `T`와 `U` 타입의 매개변수를 받아 `boolean`을 반환하는 함수를 나타냅니다.
>    * `boolean test(T t, U u);`
>
> 이 외에도 더 많은 함수형 인터페이스와 변형이 있습니다. 이들 함수형 인터페이스는 자바 8 이상에서 다양한 상황에서 유용하게 사용됩니다, 예를 들어 스트림 API, Optional 클래스, CompletableFuture 클래스 등에서 많이 활용됩니다.

개인적으로 굳이 함수형 인터페이스를 쓰려고 고집하기 보다는 상황에 맞게 객체의 메소드로 선언하고 이를 활용하는게 설계나 가독성 측면에서 더 좋을지, 아니면 함수형 인터페이스를 활용하는게 좋을지 잘 판단하는게 중요한 것 같다.
