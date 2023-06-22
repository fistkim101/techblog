# (최종 검토 필요) Mockito





## 아래

#### 1. Mockito 소개 <a href="#1-mockito" id="1-mockito"></a>

mock 객체는 인터페이스만 나와있고 구현체는 없는 상황에서 테스트를 해야 하는 상황이거나, 구현체가 있긴 하지만 그걸 쓰지 않고 mock객체를 써서 when \~ 과 같은 식으로 상황을 조작해서 테스트를 하고 싶을 때 사용한다.

* Mock: 진짜 객체와 비슷하게 동작하지만 프로그래머가 직접 그 객체의 행동을 관리하는 객체.
* Mockito: Mock 객체를 쉽게 만들고 관리하고 검증할 수 있는 방법을 제공한다.

cf. [테스트를 작성하는 자바 개발자 50%+ 사용하는 Mock 프레임워크](https://www.jetbrains.com/lp/devecosystem-2019/java/)

cf. [현재 최신 버전 3.1.0 단위 테스트에 고찰](https://martinfowler.com/bliki/UnitTest.html)

\


***

#### 2. Mock 객체 만드는 방법 <a href="#2-mock" id="2-mock"></a>

*   Mockito.mock() 메소드로 만드는 방법

    ```
    MemberService memberService = mock(MemberService.class);
    StudyRepository studyRepository = mock(StudyRepository.class);
    ```

    \

* @Mock 애노테이션으로 만드는 방법
  * JUnit 5 extension으로 MockitoExtension을 사용해야 한다.
  * 메소드 매개변수에서 선언하여도 되고 클래스의 멤버 필드로도 선언하여 사용이 가능하다.

```
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

\


***

#### 3. Mock 객체 조작하기 <a href="#3-mock" id="3-mock"></a>

인터페이스와 같이 다형성을 위해서 추상화된 클래스를 테스트에서 사용해야 할 경우가 있다. 뿐만 아니라 외부 api와 연동해야 할 경우, 원하는 결과값의 수신이 되었다고 가정하고 그 다음의 행동들을 하나의 유닛으로 보고 이를 테스트 할 때, 값을 제대로 받았다고 가정해야 할 필요가 있다. 이럴 때, Mock객체를 조작하면 원하는 테스트 시나리오대로 유닛 테스트를 하기가 용이해진다

\


```
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

```
doThrow(new RuntimeException()).when(mockedList).clear();
```

\


아래는 mokito의 [ArgumentMatchers](https://javadoc.io/doc/org.mockito/mockito-core/2.2.7/org/mockito/ArgumentMatchers.html) 가 가진 any()를 통해서 더 편하게 테스트가 가능하다.\


```
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

\
\


#### 4. BDD 스타일 Mockito API <a href="#4-bdd-mockito-api" id="4-bdd-mockito-api"></a>

[BDD](https://en.wikipedia.org/wiki/Behavior-driven\_development) 스타일로 Mokito를 사용할 수 있다. BDD의 기본 철학은 개발자와 비개발자간의 collaboration 을 적극 응원한다는 것인데 어플리케이션의 ‘행동’을 소통의 기준으로 삼는다. 즉, ‘어플리케이션이 어떻게 행동해야 한다’ 라는 측면에서 여러가지를 규정할 수 있고 이 ‘행동’을 바탕으로 현재 혹은 개발중인 어플리케이션이 적절하게 ‘행동’하고 있는지를 논의할 수 있는 것이다.

그래서 BDD에 입각하여 어플리케이션의 행동을 규정할 때 ‘어떤 상황에서, 어떤 경우에, 이렇게 행동해야한다’ 라고 어플리케이션의 행동을 기대할 수 있다. 그리고 이러한 기대를 Mokito를 이용해서 코드로 표현하자면 대표적인 BDD 포맷으로 given, when, then 을 사용할 수 있다. 아래는 그 예시이다.

```
given(memberService.findById(1L)).willReturn(Optional.of(member));
given(studyRepository.save(study)).willReturn(study);
```

\


아래는 강의 소스에서 사용된 given, when, then 예시이다.

```

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

\
