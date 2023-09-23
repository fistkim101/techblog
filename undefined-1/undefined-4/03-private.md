# \[03] private 생성자나 열거 타입으로 싱글턴임을 보증하라

## 핵심 내용

이 아이템은 싱글톤을 어떻게 만들지에 관한 내용이었다. priavte 생성자 선언으로 외부의 생성을 막고 싱글톤을 만드는 방식 두 가지(public static 프로퍼티로 내부에 가지고 있기, private 프로퍼티로 두고 public static 으로 접근시키기)를 소개하고 있는게 내용의 전부이다.

하지만 책에서 언급하고 있는 중요한 부분들을 아래에서 짚어보자.



## 싱글톤 객체를 추상화 시켜두지 않으면 테스트가 어려워진다

싱글톤 객체가 만약 유일한 타입(가장 상위 단일 객체)로 존재하면 클라이언트 코드를 테스트 할때 이 의존성 때문에 테스트 하기가 어려워 진다. 테스트하기가 어렵다는 말은 클라이언트의 테스트가 불가능하다는 이야기가 아니고, 클라이언트 코드를 테스트 할때 해당 싱글톤 객체를 무조건 호출해서 사용해야 한다는 것을 의미한다.

아래 코드를 보자.

```java
package item3;

public class Singer {
    public static final Singer INSTANCE = new Singer();
    public void sing(){
        System.out.println("sing ~~");
    }
}
```

```java
package item3;

public class Concert {
    private final Singer singer;

    public Concert(Singer singer) {
        this.singer = singer;
    }

    public void start(){
        singer.sing();
    }
}
```

```java
package item3;

public class Client {
    public static void main(String[] args) {
        Concert concert = new Concert(Singer.INSTANCE);
        concert.start();
    }
}
```

여기서 만약에 Singer의 sing()이 외부 API 를 태워야 한다던가, 개발 환경에서 실행이 되면 안된다던가, 호출 비용이 비싸다던가 하는 등의 문제가 있으면 결론적으로 Concert 의 start() 를 테스트 하는것이 어려워진다.

따라서 Concert 는 의존하는 Singer 를 싱글톤으로 사용하지만 테스트는 용이하게 해줄 수 있는 것이 이상적일 것이다. 그래서 아래와 같이 Singer 위에 추상화를 두고 Concert 가 이 추상화된 객체를 의존하게 함으로써 mock 객체를 주입 시킬 수 있다.

```java
package item_new;

public interface ISinger {
    void sing();
}
```

```java
public class Concert {
    private final ISinger singer;

    public Concert(ISinger singer) {
        this.singer = singer;
    }

    public void start(){
        singer.sing();
    }
}
```

```java
public class MockSinger implements ISinger{
    @Override
    public void sing() {
        System.out.println("Mock sing ~~");
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        Concert concert = new Concert(new MockSinger());
        concert.start();
    }
}
```

이렇게 해주면 클라이언트 코드에 필요한 Singer 를 주입해서 사용할 수 있고 테스트 때는 위와 같이 mock 을 주입해서 사용하면 된다.



## 정적 팩터리 방식의 장점

### 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다

정적 팩터리 방식으로 싱글톤 객체를 얻는 방식을 사용하면 제네릭 싱글턴 팩터리로 사용할 수 있다. 이게 실질적으로&#x20;

```java
public class Social3rdParty<T> {
    private static final Social3rdParty<Object> INSTANCE = new Social3rdParty<>();

    private Social3rdParty() {
    }

    public static <E> Social3rdParty<E> getInstance() {
        return (Social3rdParty<E>) INSTANCE;
    }
}

public class Client {
    public static void main(String[] args) {
        Social3rdParty<KaKao> social3rdParty1 = Social3rdParty.getInstance();
        Social3rdParty<Naver> social3rdParty2 = Social3rdParty.getInstance();
    }
}
```



### 정적 팩터리의 메서드 참조를 공급자로 사용할 수 있다

```java
public class ConcertNew {
    private ISinger singer;

    public ConcertNew(Supplier<ISinger> singer) {
        this.singer = singer.get();
    }

    public void perform() {
        singer.sing();
    }
}

public class Singer implements ISinger {
    public static final Singer INSTANCE = new Singer();

    public static Singer getInstance() {
        return INSTANCE;
    }

    public void sing() {
        System.out.println("sing ~~");
    }
}


public class Client {
    public static void main(String[] args) {
        ConcertNew concertNew = new ConcertNew(Singer::getInstance);
    }
}
```



## 제일 안전한 싱글톤 생성 방법은 열거(ENUM)

싱글톤을 깨트릴 수 있는 방법으로 리플렉션과 직렬화/역직렬화를 언급하고 있는데, 열거형으로 싱글톤을 구현하게 되면 이 방법들에 대해서 방어가 된다.

리플렉션으로 싱글톤을 깨트릴 때는 노출되지 않은 생성자를 꺼내와서 생성하는 방법이 일반적인데 열거형으로 클래스를 선언할 경우 리플렉션으로 생성자를 가져 올 수 없다.

그리고 직렬화 되었다가 역직렬화가 되면 힙에 새로운 객체가 할당되면서 새로운 객체가 만들어지는데 열거형으로 선언시 동일한 객체가 반환된다.

심지어 열거형도 인터페이스 구현이 가능해서 테스트의 유연함도 가져갈 수 있다.









