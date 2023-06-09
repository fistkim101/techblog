# CH02 AOP

## AOP

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

cross cutting 이라는 용어가 '횡단적인'의 의미를 가진다. AOP 에서의 cross cutting concern 은 결국 위 그림에서 볼 수 있듯이 여러 Class 들을 두고 '횡단으로 봤을때의 (공통된) 관심사' 를 말한다. 여러 모듈 혹은 객체를 가로지르는 공통된 관심사이며 이것에 대한 처리를 분산시켜 놓을 것이 아니라 한 곳에서 모듈화해서 처리하도록 하는 것이 AOP 라고 할 수 있다.

## AOP 적용방법

### 컴파일 타임(Compile-time)

컴파일 타임에 바이트코드 자체에 원하는 처리가 반영된다. 컴파일러가 컴파일 단계에서 AOP 를 인식하여 의도에 맞게 이를 반영한 바이트코드를 만들어낸다.

바이트코드 자체에 이미 반영이 되었기 때문에 실행 시점에 추가적인 부하가 발생하진 않는다. 하지만 컴파일 단계에 추가적인 작업이 필요하다.

### 로드 타임(Load-time)

클래스 로딩 시점에 AOP를 적용하는 방식이다. 클래스 로딩 중에 바이트 코드를 수정해서 AOP 를 적용시킨다. 로드 타임 위빙(Load-time Weaving) 기능이 사용된다. 로드 타임에 '끼워넣는' 것이다. 컴파일 이후에도 적용될 수 있지만 실행 시 오버헤드가 있다.

### 런타임(Runtime)

스프링 프레임워크가 사용하는 방식이다. Bean을 생성할때 프록시 방식으로 객체를 감싸서 프록시 객체를 만든다. 초기에 Bean 을 생성할때 약간의 부하가 발생하지만 실행 시점에는 오버헤드가 없다.

예를 들어 ClassA 가 있을때, 이것을 바이트코드로 만들때와  클래스 로드하는 시점에 모두 아무 행위도 하지 않지만 초기에 Bean 을 생성하여 컨테이너에 담을때 프록시 방식으로 객체를 감싸서 원하는 의도가 반영된 프록시 객체를 만들어서 Bean 을 생성한다.

## 스프링의 AOP

위에서 살펴본 것처럼 런타임 방식에 프록시 객체를 만들어서 bean 을 생성한다.

```java
public abstract class AbstractAutoProxyCreator extends ProxyProcessorSupport
		implements SmartInstantiationAwareBeanPostProcessor, BeanFactoryAware {
```

AbstractAutoProxyCreator 가 이 역할을 담당한다.&#x20;

<figure><img src="../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

주석을 보면 아래와 같이 적혀있다.

```java
 * {@link org.springframework.beans.factory.config.BeanPostProcessor} implementation
 * that wraps each eligible bean with an AOP proxy, delegating to specified interceptors
 * before invoking the bean itself.
```

정리하자면 [BeanPostProcessor ](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/config/BeanPostProcessor.html)(Factory hook that allows for custom modification of new bean instances) 의 구현체인 AbstractAutoProxyCreator 가 Bean 생성 후처리에 관여하여 이를 modify 해서 프록시 객체 생성을 하면서 부가 기능을 의도에 맞게 넣어주는 것이다.

이 과정에 대해서 더 자세히는 토비의 스프링에서 다루고 있다고 하는데 나중에 토비의 스프링 정리할 때 자세히 봐야겠다.
