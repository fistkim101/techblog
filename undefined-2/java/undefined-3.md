# 메소드  참조

## 메소드 참조가 무엇인가

메소드 참조를 자주 사용해 왔지만 명확하게 정리해본 적은 없어서 이번에 깔끔하게 정리하고자 한다. 메소드 참조를 한 문장으로 표현하자면 '특정 메소드의 이름을 참조해서 그 메소드를 호출하는 데 사용하는 구문'이다. 특히 람다식을 만들어야 하는 경우에 메소드 참조를 사용해서 더 쉽게 만들 수 있다.

핵심은 메소드 참조의 반환 타입은 결국 functionalInterface 라는 것이다.



## 메소드 참조 사용 케이스

### 정적 메소드 참조

static 메소드에 대한 참조이므로 해당 객체를 생성할 필요가 없이 바로 호출이 가능하다.

```java
Integer::parseInt
```



### 생성자 참조

```java
ArrayList::new
```



### 특정 인스턴스에 대한 메소드 참조

```java
String myString = "hello";
Supplier<String> supplier = myString::toUpperCase; 
```
