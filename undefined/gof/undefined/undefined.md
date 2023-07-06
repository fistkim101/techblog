# 싱글톤 패턴

## 대분류

객체생성



## 문제상황

### 1. 다수의 객체가 하나의 객체를 공유해야 하는 상황(공유하면 좋은 것이 아니고 반드시 하나의 유일한 객체를 두고 공유를 해야만 하는 상황)

첫번째 예로 캐시를 사용하는 상황이 있다. 예를 들어 캐시 저장소를 담당하는 특정 객체 '캐시 매니저'가 있다고 했을때 이 객체를 사용하는 객체 A, B, C가 있다고 치자. A, B, C 가 각자의 책임을 수행하는 과정에서 캐시된 데이터를 사용해야 할때 동일한 캐시 매니저를 사용해야만 클라이언트에게 데이터 일관성을 보장할 수 있다.

이러한 상황에서 데이터 일관성을 위해 반드시 A, B, C는 캐시 매니저로 동일하고 유일한 객체 하나를 사용해야 한다. 실제로 스프링의 캐시 매니저는 '대부분' 싱글톤 패턴으로 구현되었다고 한다.(이를코드 레벨로 다 확인하진 못했다)

두번째 예시로 설정을 공유하는 상황을 들 수 있다. 일반적으로 설정이라함은 전역적으로 동일한 값을 적용시켜야 하는 것이기 때문에 유일하게 하나만 존재하면서 여러 객체에 참조되어야 한다.



### 2. 다수의 객체가 하나의 객체를 공유하는 것이 자원 사용 효율 측면에서 유리한 상황

여러 객체에게 사용되어야 하지만 사용 되는 과정에 해당 객체의 상태값이 전혀 필요가 없는 경우에 사용될 때마다 만들어서 사용하기보다 계속 동일한 객체가 사용되는 것이 자원 사용 측면에서 효율적이다.

대표적으로 데이터베이스 연결시 사용되는 커넥션 풀이 이에 해당한다.



## 해결방안

하나의 객체가 여러 객체에게 공유될 수 있도록 한다. 이를 위해서 여러 객체가 이 객체를 함부로 만들 수 없도록 하며, 선언된 호출 인터페이스를 사용하면 항상 동일한 인스턴스가 반환되도록 처리한다.



## [실습코드](https://github.com/fistkim101/design-pattern-java/tree/master)



```java
public class Setting {

    private static Setting instance;

    private Setting() {
    }

    public static Setting getInstance() {
        if (instance == null) {
            instance = new Setting();
        }

        return instance;
    }

}
```

이 경우 멀티스레드 환경에서 싱글톤이 100% 보장되진 않는다. getInstance() 메소드에 두 스레드가 동시에 들어가서 if문을 동시에 들어가게되면 new Setting() 이 두 번 실행될 수 있다.



그래서 이를 스레드 세이프하게 처리하기 위해서 다음과 같이 메소드에 동기화를 적용할 수 있다.

```java
public class Setting {

    private static Setting instance;

    private Setting() {
    }

    public static synchronized Setting getInstance() {
        if (instance == null) {
            instance = new Setting();
        }

        return instance;
    }

}
```

하지만 이 방법도 단점이 있는데 synchronized 를 처리하는 비용 자체가 크다는 것이다. getInstance() 메소드가 얼마나 자주 많이 호출될지 모르는데 호출 될 때마다 싱글톤을 보장하여 동일한 객체를 주는 것은 보장이 되지만 매번 synchronized 를 처리하기 위해서 처리비용을 많이 치뤄야한다.

나는 이것이 처리중인 스레드가 점유를 하고 락을 걸고 처리가 끝나면 락을 풀어버림으로써 다른 스레드의 접근을 허락하는 동기화 처리 방식 자체의 비용처리를 의미하는 것으로  이해했다.

그래서 위 방식대로 synchronized 를 사용하되 성능 저하를 막을 수 있는 double checked locking 을 아래와 같이 구현할 수 있다.

```java
public class Setting {

    private static volatile Setting instance;

    private Setting() {
    }

    public static Setting getInstance() {
        if (instance == null) {
            synchronized (Setting.class) {
                instance = new Setting();
            }
        }

        return instance;
    }

}
```

> `volatile`은 Java에서 사용되는 키워드로, 변수에 대한 가시성(visibility)과 순서(orderedness)를 보장하기 위해 사용됩니다. `volatile` 변수를 사용하면 다중 스레드 환경에서 발생하는 문제들을 해결할 수 있습니다.
>
> `volatile` 변수의 주요 특징은 다음과 같습니다:
>
> 1. 가시성 (Visibility): `volatile` 변수는 스레드 간에 값을 읽거나 쓸 때 항상 메인 메모리의 최신 값을 읽거나 쓰도록 보장합니다. 이는 변수의 변경 내용이 다른 스레드에 즉시 반영되어 동기화 문제를 해결합니다.
> 2. 순서 (Orderedness): `volatile` 변수를 통해 발생한 쓰기와 읽기 연산은 항상 동일한 순서로 수행됩니다. 다시 말해, `volatile` 변수에 대한 연산은 다른 스레드가 참조하는 변수의 순서를 보장합니다.
>
> `volatile` 키워드는 주로 다음과 같은 상황에서 사용됩니다:
>
> 1. 상태 플래그: 여러 스레드가 공유하는 상태 플래그 변수는 `volatile`로 선언하여 상태 변경이 다른 스레드에 즉시 알려질 수 있도록 보장합니다. 이를 통해 다른 스레드는 상태 변화를 즉시 감지하여 처리할 수 있습니다.
> 2. 무한 루프 제어: 스레드가 무한 루프를 실행하는 경우, 다른 스레드에서 해당 루프를 중지시키기 위해 `volatile` 변수를 사용할 수 있습니다. 변수의 값을 변경하여 루프 조건을 변경하고, 스레드가 변경을 인식하여 루프를 종료하게 할 수 있습니다.
> 3. Double-checked locking: 멀티스레드 환경에서 싱글톤 패턴을 구현할 때 `volatile` 변수를 사용하여 인스턴스의 초기화 과정을 안전하게 처리할 수 있습니다. `volatile` 변수를 사용하면 다른 스레드가 인스턴스를 사용하기 전에 초기화가 완료됨을 보장할 수 있습니다.
>
> `volatile` 키워드는 가시성과 순서를 보장해주지만, 원자성(atomicity)은 보장하지 않습니다. 즉, 여러 스레드에서 동시에 `volatile` 변수를 읽고 쓰는 경우 원자적인 연산이 보장되지 않을 수 있습니다. 원자성을 보장하기 위해서는 `volatile` 키워드와 함께 `synchronized` 또는 `java.util.concurrent.atomic` 패키지의 원자적인 연산 클래스를 사용해야 합니다.

하지만 이 방식은 구현 자체가 다른 방법에 비해서 복잡하고 이해해야할 개념도 추가적으로 존재한다.



그래서 그냥 필요할 때 만드는 Lazy Loading 방식이 아니라 아래와 같이 Eager initialization 방식으로 바로 만들어두는 방식이 있다.

```java
public class Setting {

    private static final Setting instance = new Setting();

    private Setting() {
    }

    public static Setting getInstance() {
        return instance;
    }

}
```

하지만 이 방식도 극단적으로 Setting 이 사용되지 않는 경우를 가정했을 때 굳이 만들어두게 된다는 단점이 있다. 그래서 아래와 같은 방식으로 구현하면 이 문제가 해결된다.



```java
public class Setting {

    private Setting() {
    }

    public static class InstanceHolder {
        public static final Setting instance = new Setting();
    }

}
```

왜 이것이 해결이 되는 걸까? inner class 의 클래스 로딩 타이밍은 호출이 된 시점이기 때문이다. 그리고 static 프로퍼티에 new Setting() 이 할당되는 것은 inner class 가 클래스 로딩 되는 시점이다.

다시 정리해서 말하면 Setting 클래스의 프로퍼티로 뒀을 때는 Setting 이 클래스로딩 되면서 바로 new Setting() 이 실행되기 때문에 Lazy Loading 이 되지 않았지만, inner class 로 두는 경우에는 Setting 이 클래스 로딩 되는 시점에는 inner class자체가 클래스 로딩 되지 않고 이후 이것이 호출 될 때 inner class가 클래스 로딩 되면서 이의 내부 프로퍼티 instance 에 new Setting() 이 할당되기에 결과적으로 Lazy Loading 이 실현되는 것이다.



## 싱글톤 패턴 깨트리는 방법 1 - 리플렉션

굳이 깨트릴 필요는 없겠지만 아무튼 강의에서 이러한 방법에 대해서 알아봤다. 리플렉션을 이용해서 생성자를 가져와서 객체를 생성하는 것이 가능하기 때문에 당연히 소스코드 레벨에서 싱글톤으로 구현해놨다고 해도 이를 깨트릴 수는 있다.

```java
package com.fistkim.designpatternjava;

import com.fistkim.designpatternjava.creation.singleton.Setting;
import com.fistkim.designpatternjava.old.creation.prototype.GitIssue;
import com.fistkim.designpatternjava.old.creation.prototype.GitRepository;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;

@SpringBootApplication
public class DesignPatternJavaApplication implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
        Setting setting1 = Setting.InstanceHolder.instance;
        Constructor<Setting> declaredConstructor = Setting.class.getDeclaredConstructor();
        declaredConstructor.setAccessible(true);
        Setting settings2 = declaredConstructor.newInstance();

        System.out.println(setting1); // com.fistkim.designpatternjava.creation.singleton.Setting@2b960a7
        System.out.println(settings2); // com.fistkim.designpatternjava.creation.singleton.Setting@31dfc6f5

        System.out.println(setting1 == settings2); // false
    }

    public static void main(String[] args) {
        SpringApplication.run(DesignPatternJavaApplication.class, args);
    }

}
```



## 싱글톤 패턴 깨트리는 방법 2 - 직렬화, 역직렬화

파일에 객체를 직렬화 해서 저장하고, 이를 역직렬화해서 꺼내서 변수에 할당하는 경우 새로운 객체로 메모리에 적재하게 된다. 이 과정에서 싱글톤이 깨지게 된다.
