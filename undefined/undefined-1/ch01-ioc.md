# CH01 IOC 컨테이너

## IOC(Inversion of Control)

IOC 는 말 그대로 제어의 역전을 의미한다. 제어는 뭘 제어한다는 말이고, 역전은 뭐가 거꾸로 되었단 말인지 이해를 해야한다. 일단 IOC 자체의 분류는 '디자인 패턴' 이다.

> In [software engineering](https://en.wikipedia.org/wiki/Software\_engineering), inversion of control (IoC) is a [design pattern](https://en.wikipedia.org/wiki/Software\_design\_pattern) in which custom-written portions of a [computer program](https://en.wikipedia.org/wiki/Computer\_program) receive the [flow of control](https://en.wikipedia.org/wiki/Control\_flow) from a generic [framework](https://en.wikipedia.org/wiki/Software\_framework).

### '제어의 역전'에 있어 대전제

일단 '제어의 역전' 을 이해하기에 앞서서 대전제는 '제어의 권한은 항상 사용하는 측이 가진다'는 것이다. 내가 밥을 먹더라도 숫가락으로 먹을지 젓가락으로 먹을지 내가 수단을 선택(제어)하는  것이 당연하다. 뭔가를 수행하는 주체가 그 과정에서 주도권을 가지고 알아서 처리(제어) 하는 것이 기본적인 전제이다. 이 기본적인 전제가 있어야 '역전' 되었다는 말이 성립된다.

### 예시 코드

아래 예시 코드를 보자.

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

```java
public class UserService {

    // 무슨 Repository 사용할지 주도적으로 판단하여 생성 및 사용중(직접 제어중)
    public void save() {
        User user = new User();
        UserRepository userRepository1 = new UserMongoRepository();
        userRepository1.save(user);

        UserRepository userRepository2 = new UserMysqlRepository();
        userRepository2.save(user);
    }

    // 제어의 주도권이 없는 상태로 수동적으로 처리(제어가 역전된 상황)
    public void save(UserRepository userRepository) {
        User user = new User();
        userRepository.save(user);
    }

}
```

### 제어(control) 권한을 가지고 있는 상태

첫번째 save 가 이에 해당하는데 UserService 가 save 메소드 내에서 무슨 레포지토리를 사용할지에 대해서 제어 권한을 가지고 있다. 실제 로직에서는 UserMongoRepository 또는 UserMysqlRepository 둘 중 하나만 사용할 것이고, 개발자의 의지에 따라(상황에 따라) 개발자가 선택한 Repository 를 UserService가 사용하게 될 것이다.

### 제어(Control) 가 역전(Inversion) 된 상태

두번째 save 에 해당하는 상태이다. UserService 는 save 를 실행할때 UserMongoRepository 혹은 UserMysqlRepository 둘 중 무엇이 주어질지 모르고 '수동적으로 주는대로 받아서' 그걸 가지고 save 로직을 수행하게 된다.

첫번째 save 와 다르게 '제어가 역전' 되어있는 것이다.

## [DI(Dependency Injection)](https://ko.wikipedia.org/wiki/%EC%9D%98%EC%A1%B4%EC%84%B1\_%EC%A3%BC%EC%9E%85)

DI(의존성 주입)은 의존성을 주입하는 프로그래밍 테크닉이다.

> In [software engineering](https://en.wikipedia.org/wiki/Software\_engineering), dependency injection is a [programming technique](https://en.wikipedia.org/w/index.php?title=Software\_programming\_technique\&action=edit\&redlink=1) in which an [object](https://en.wikipedia.org/wiki/Object\_\(computer\_science\)) or [function](https://en.wikipedia.org/wiki/Subroutine) receives other objects or functions that it depends on.

이미 알아본 IOC 에서 UserService가 Repository 를 외부로부터 받아(주입받아)서 사용한 것이 이미 DI 가 된 상태이다. '의존성' 이라는 것 자체가 프로그래밍 용어이고 UserService 가 UserRepository 에 의존하고 있는 것을 보면 결국 UserMongoRepository 를 받든 UserMysqlRepository 를 받든 상관없이 결국 UserRepository(의존하고 있는 객체) 를 주입 받는 것이다.

위에 위키에도 나와있지만 핵심은 아래와 같다.

> 의존성 주입의 의도는 객체의 생성과 사용의 관심을 분리하는 것이다.

## DI 장점

### 테스트가 용이하다.

테스트가 용이다는 의미는 상당히 중의적인데 이걸 풀어보자면 '테스트를 큰 수고를 들이지 않고 할 수 있다', '테스트 스코프를 내가 원하는 대로 조절 가능하다', '테스트 속도가 빨라진다' 등으로 해석할 수 있다. DI를 통해 테스트가 쉬워지는 것은 이 예시들 모두에 해당한다.

위 코드 예시에서 보면 DI가 안되는 경우 UserService 의 save 를 테스트 하기 위해서는 UserMongoRepository 또는 UserMysqlRepository이 실제로 구현이 되어야 한다. 왜냐하면 내부에서 생성까지 해서 사용하기 때문에 테스트에서 이를 간섭할 수 없기 때문이다. 그래서 구현이 안된 경우 테스트 자체가 불가능하다. 하지만 DI 가 가능한 경우 테스트에서 save 의 파라미터로 Mock 객체를 전달해주는 방식으로 처리하면 테스트가 바로 가능해진다.

그리고 이 Mock 객체를 실제DB가 아니라 InMemoryRepository 로 실제로 구현을 하거나 혹은 H2와 같은 것을 활용하면 테스트 속도도 빨라질 수 있다.

### 유지보수가 용이하다.

유지보수가 용이하다는 것도 풀어서 이해를 해야한다. 유지보수가 용이하다는 것은 '변경 사항이 있을때 수정 비용이 적다.'는 것이다. 수정 비용이 적으려면 객체간 결합도가 낮아야한다. 왜? 이거 바꾸면 저거도 연관되어 있어서 바꿔야하는 식으로 강하게 결합되어 있으면 하나 바꾸려면 많은 것을 바꿔야하기 때문이다.

그럼 결론적으로 DI 가 유지보수를 용이하게 한다는 말은 객체간 결합도를 낮추(루즈 커플링)어 준다는 의미인데 어떻게 이게 된다는 말일까? 위 예제 코드에서도 볼 수 있듯이 DI가 가능할 경우 UserService 가 UserRepository 의 생성에 관해서는 관심을 하나도 두지 않아도 되고 UserMongoRepository 또는 UserMysqlRepository 무엇이 주어지든 받는대로 그거로 사용할 수 있는 상황인데 이 상황 자체가 '결합도가 낮은' 상황이다.

즉, 여기서 상황에 따라 구현체를 다르게 써도 UserService 는 변경할 일이 전혀 없는 것이다. 반대로 DI 가 되고 있지 않은 상황이라면 구현체를 바꿔야할 경우에 UserService 까지 변경을 해줘야한다. UserRepository 와 UserService가 강하게 결합되어 있는 것이다.

### 코드 재사용성이 높아진다.

유지보수가 용이하다는 것과 맥락이 비슷하다. UserService 입장에서 보면 DI 적용시 수정사항이 발생하면 구현체만 바뀌면 되기 때문에(=주입을 다른걸로 해주기만 하면 되기 때문에) UserService 는 오랫동안 사용(=수정하지 않고 재사용 계속 됨) 할 가능성이 커진다.

## IOC 컨테이너

IOC 컨테이너는 위에서 자세히 정리한 IOC 를 위해 필요한 '컨테이너' 이다. IOC 는 결국 특정 객체가 특정 로직을 수행할 때 이를 위해 필요한 수단을 스스로가 정하지 못하고 '외부로부터 받아서' 처리하는데 이 과정에서 수동적으로 전달 받는 객체들을 모아둔 곳이 IOC 컨테이너인 것이다.

결과적으로 DI를 수행하기 위해서 IOC 컨테이너 역할을 하는 것이 필요한 것이다. Spring 에서는 BeanFactory 가 핵심적으로 이러한 역할을 정의하고 있는 interface 이고, 이 핵심 기능에 덧붙여 여러 다른 기능들을 확장한 ApplicationContext 가 사용된다.

### BeanFactory와 ApplicationContext

<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

위 간단한 클래스 다이어그램에서도 알 수 있듯이 최상위에 BeanFactory 가 존재하며 ApplicationContext는 BeanFactory 이외에 여러 자원들을 상속하며 기능을 확장, 제공한다.
