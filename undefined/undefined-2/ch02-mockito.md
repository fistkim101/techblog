# CH02 Mockito

## Mockito 에 대한 지금의 생각(결론: 최대한 쓰지 말자)

이 강의 자체를 꽤 예전에 들었던 것이고, 아래 노트도 그 때 정리했던 것이다. 당시에는 테스트를 할 때 필요한 부분에서 mock 객체 등을 활용해서 테스트를 더 쉽게 할 수 있어서 좋다고 생각했는데 사실 지금은 생각이 좀 바뀌었다.

mock 을 이용해서 어떠한 객체의 행동을 조작한다는 것 자체가 조작 비용이 너무 큰 일이다. 뭔가가 바뀌면 이 조작까지 모두 바꿔줘야하는데, 조작이 클 수록 수정 비용이 매우 크다. 내가 짜든, 다른 사람이 짜놓은 코드든 테스트 하고자 하는 모듈 혹은 메소드가 피치 못하게 꽤 많은 일들을 하도록 처리해 둔 경우에 어떤 의도된 테스트를 위해서 중간에 조작을 할 게 너무 많다.

그런데 내가 말하고자 하는 것은 모킹 자체가 수정비용이 크니까 안좋다는 이야기를 하려는 것이 아니고 애초에 모킹을 많이 해야할 정도로 테스트 단위가 커서는 안되고, 그러기 위해서는 설계시 객체의 책임과 역할 범위가 최대한 작아야한다는 것이다.

객체의 협력 구조를 최대한 쪼개서 설계를 해두면 뭔가 특정 단위의 테스트를 할 때 필요한 조작 행위가 매우 적어지거나 아예 필요가 없어진다. 그리고 Mockito 를 사용하지 않고 테스트 스코프에서 DIP 를 적용할 수도 있다. 그렇게 설계를 잘 해두면 된다.

앞으로 마주칠 기존 레거시 프로젝트 혹은 내가 만드는 새로운 프로젝트 모두 어떻게 될지는 모르겠고, 내 생각도 어떻게 바뀔지 모르겠지만 지금으로서는 Mockito 자체를 그냥 쓸 일이 없게끔 설계부터 최대한 신경쓰자는 것이 지금의 나의 생각이다.

## Mockito 소개

### mock

[mock](https://en.dict.naver.com/#/entry/enko/b933178ef279482db2f7a569a102361e) 단어 자체가 '가짜의' 와 같은 뜻을 가지고 있다. 테스트에서 '목 객체' 와 같은 단어로 자주 사용되는 개념이다. 테스트를 위한 더미의 개념이다.

### [Mockito](https://site.mockito.org/)

공식 홈페이지 소개 그대로를 인용한다.

> #### Tasty mocking framework for unit tests in Java

## Mock 객체 만드는 방법 <a href="#2-mock" id="2-mock"></a>

### Mockito.mock() 메소드로 만드는 방법

```java
MemberService memberService = mock(MemberService.class);
StudyRepository studyRepository = mock(StudyRepository.class);
```

### @Mock 애노테이션으로 만드는 방법

* JUnit 5 extension으로 MockitoExtension을 사용해야 한다.
* 메소드 매개변수에서 선언하여도 되고 클래스의 멤버 필드로도 선언하여 사용이 가능하다.

```java
@ExtendWith(MockitoExtension.class)
class SocialLoginServiceTest {

    // @Mock SocialLoginService socialLoginService  <- 클래스의 멤버 필드로도 가능

    @DisplayName("어노테이션을 이용한 mock 객체 생성 테스트")
    @Test
    void createSocialLoginWithAnnotation(@Mock SocialLoginService socialLoginService) {
        assertNotNull(socialLoginService);
    }
}
```

### Mock 객체 조작하기

인터페이스와 같이 다형성을 위해서 추상화된 클래스를 테스트에서 사용해야 할 경우가 있다. 뿐만 아니라 외부 api와 연동해야 할 경우, 원하는 결과값의 수신이 되었다고 가정하고 그 다음의 행동들을 하나의 유닛으로 보고 이를 테스트 할 때, 값을 제대로 받았다고 가정해야 할 필요가 있다. 이럴 때, Mock객체를 조작하면 원하는 테스트 시나리오대로 유닛 테스트를 하기가 용이해진다.

```java
    @DisplayName("social login 시 외부 api통한 토큰 수신")
    @Test
    void socialLoginGetToken(@Mock SocialLoginService socialLoginService) {
        String token = "userToken";
        Mockito.when(socialLoginService.getToken()).thenReturn(token);
        assertEquals(token, socialLoginService.getToken());
    }
```

원리는 Mockito.when\~ 에서 정의 해둔대로 결과값이 미리 세팅이되고 실제로 해당 코드가 실행이 되지만 결과값은 미리 세팅된 결과값을 받도록 하는 것이다. 즉, 실제로 socialLoginService의 getToken()가 실행이 되긴 하는데 그것의 결과값으로 미리 정의해둔 String token = “userToken”; 값을 return 받게 되는 것이다.

이렇듯 미리 정의한 약속대로 들어올 경우 미리 정의한 값을 return 하는 방식이기 때문에, 파라미터가 사전에 정의하지 않은 다른 것이 오면 실제로 로직대로 처리한 결과가 return 된다. 만약 파라미터에 귀속되지 않고 mock 객체를 사용하고 싶다면 아래와 같이 any()를 사용하면 된다.

추가로, doThrow()를 이용하면 미리 정의한 값을 return 받는 것이 아니라 미리 정의한 에러가 발생하게끔 mock 객체를 활용할 수도 있다.

```java
doThrow(new RuntimeException()).when(mockedList).clear();
```

아래는 mokito의 [ArgumentMatchers](https://javadoc.io/doc/org.mockito/mockito-core/2.2.7/org/mockito/ArgumentMatchers.html) 가 가진 any()를 통해서 더 편하게 테스트가 가능하다.\


```java
    @DisplayName("social login 시 외부 api통한 토큰 수신")
    @Test
    void socialLoginGetToken(@Mock SocialLoginService socialLoginService) {
        String token = "userToken";
        Mockito.when(socialLoginService.getToken(any())).thenReturn(token);

        Object userInfo = new Object();
        assertEquals(token, socialLoginService.getToken(userInfo));
        assertEquals(token, socialLoginService.getToken("anything"));
    }
```

## BDD 스타일 Mockito API

[BDD](https://en.wikipedia.org/wiki/Behavior-driven\_development) 스타일로 Mokito를 사용할 수 있다. BDD의 기본 철학은 개발자와 비개발자간의 collaboration 을 적극 응원한다는 것인데 어플리케이션의 ‘행동’을 소통의 기준으로 삼는다. 즉, ‘어플리케이션이 어떻게 행동해야 한다’ 라는 측면에서 여러가지를 규정할 수 있고 이 ‘행동’을 바탕으로 현재 혹은 개발중인 어플리케이션이 적절하게 ‘행동’하고 있는지를 논의할 수 있는 것이다.

그래서 BDD에 입각하여 어플리케이션의 행동을 규정할 때 ‘어떤 상황에서, 어떤 경우에, 이렇게 행동해야한다’ 라고 어플리케이션의 행동을 기대할 수 있다. 그리고 이러한 기대를 Mokito를 이용해서 코드로 표현하자면 대표적인 BDD 포맷으로 given, when, then 을 사용할 수 있다. 아래는 그 예시이다.

```java
given(memberService.findById(1L)).willReturn(Optional.of(member));
given(studyRepository.save(study)).willReturn(study);
```



아래는 강의 소스에서 사용된 given, when, then 예시이다.

```java
    @Test
    void createNewStudy() {
        System.out.println("========");
        System.out.println(port);

        // Given
        StudyService studyService = new StudyService(memberService, studyRepository);
        assertNotNull(studyService);

        Member member = new Member();
        member.setId(1L);
        member.setEmail("keesun@email.com");

        Study study = new Study(10, "테스트");

        given(memberService.findById(1L)).willReturn(Optional.of(member));

        // When
        studyService.createNewStudy(1L, study);

        // Then
        assertEquals(1L, study.getOwnerId());
        then(memberService).should(times(1)).notify(study);
        then(memberService).shouldHaveNoMoreInteractions();
    }
```
