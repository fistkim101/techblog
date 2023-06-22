# (최종 검토 필요) JUnit 5







## 아래

#### 1. JUnit 5 소개 <a href="#1-junit-5" id="1-junit-5"></a>

\


**용어 정리**

![](http://localhost:4000/assets/images/spring/test-junit5-flow.png)



* JUnit : 자바 프로그래밍 언어용 유닛테스트 프레임워크. JUnitPlatform 위에서 Jupiter에 의해 구동.
  * [유닛테스트](https://ko.wikipedia.org/wiki/%EC%9C%A0%EB%8B%9B\_%ED%85%8C%EC%8A%A4%ED%8A%B8) : 특정 모듈이 의도된 대로 작동하는지 검증하는 절차
  * JUnitPlatform : main 메소드 없이 테스트가 실행될 수 있도록 런처를 제공
  * Jupiter: TestEngine API 구현체로 JUnit 5를 제공

\


**생각해볼 것**

* 유닛테스트의 ‘모듈’의 단위를 어떤 기준으로 정할 것인가? 기계적인 1메소드 - 1모듈이 아닌 모듈에 대한 적절한 ‘단위화’가 결국 의미있는 테스트 단위를 만드는 것이 아닐까. 그렇다면 적절한 ‘단위화’는 어떤 기준으로 해야할까.

\


***

#### 2. JUnit 5 시작하기 <a href="#2-junit-5" id="2-junit-5"></a>

\


**기본적인 annotation 정리 (JUnit 5 기준)**

* @BeforeAll / @AfterAll : 해당 메소드가 선언된 클래스의 모든 @Test 들이 시작되기 전과 후에 한 번씩 실행되며 **static 으로 만들어 줘야함.** 왜냐하면 나중에 다루겠지만 @TestInstance의 라이프사이클이 각각의 @Test 마다 새로 만들기 때문에 각각의 인스턴스에서 @BeforeAll / @AfterAll로 만들어준 메소드에 접근하려면 static이 필요하다.
* @BeforeEach / @AfterEach : 해당 메소드가 선언된 클래스의 모든 @Test 각각의 전과 후에 실행(@Test 메소드 수만큼 전과 후에 실행됨)
* @Disabled
* @Test

```
    @Test
    @Disabled
    void create() {
        User user = new User();
        assertNotNull(user);
    }

    @Test
    void first() {
        System.out.println("first");
    }

    @Test
    void second() {
        System.out.println("second");
    }

    @BeforeEach
    void beforeEach() {
        System.out.println("before each");
    }

    @AfterEach
    void afterEach() {
        System.out.println("after each");
    }

    @AfterAll
    static void afterAll() {
        System.out.println("After all");
    }

    @BeforeAll
    static void beforeAll() {
        System.out.println("Before all");
    }

```

\


***

#### 3. JUnit 5: 테스트 이름 표기하기 <a href="#3-junit-5" id="3-junit-5"></a>

* @DisplayNameGeneration : 해당 클래스 전체의 @Test에 대한 이름을 표기하는 전략을 설정할 수 있음
* @DisplayName : 개별 @Test의 이름을 설정(@DisplayNameGeneration보다 우선순위 높음)

```
@DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
class StudyTest {
}
```

\


**cf. @DisplayName을 적용하여도 지정한 테스트명이 테스트 콘솔창에 나타나지 않는 경우 테스트 환경 설정을 intellij로 잡아 준다.**

\


***

#### 4. JUnit 5 Assertion <a href="#4-junit-5-assertion" id="4-junit-5-assertion"></a>

\


* assertEquals(expected, actual) : 실제 값이 기대한 값과 같은지 확인(파라미터 순서가 무관하긴 하지만 좌측이 기대값, 우측이 출력값임을 알고 쓰자)
* assertNotNull(actual) : 출력값이 null이 아닌지 확인
* assertTrue(boolean) : 다음 조건이 참인지 확인
* assertAll(executables…) : 하나의 @Test에서 두 개 이상의 assert문을 사용했을 때 첫번째 assert에서 fail이 될 경우 두번째 assert를 실행하지 않음. 이 때 두 assert를 assertAll로 감싸주면 둘다 독립적으로 테스트가 가능하다.
* assertThrows(expectedType, executable) : 예외 발생 확인
* assertTimeout(duration, executable) : 특정 시간 안에 실행이 완료되는지 확인

\


```
    @Test
    @DisplayName("스터디 만들기")
    void createNewStudy() {
        Study study = Study.builder()
                .studyStatus(StudyStatus.DRAFT)
                .limit(20)
                .build();
        assumeTrue(study != null);
        assertAll(
                () -> assertEquals(StudyStatus.DRAFT, study.getStudyStatus(), "스터디를 처음 만들면 상태값이 DRAFT 여야 한다."),
                () -> assertTrue(study.getLimit() > 10, "스터디의 최소 인원은 10명을 초과해야 한다.")
        );
    }
```

테스트 자체가 하나라도 통과하지 못하면 문제가 있기 때문에 assertAll을 사용하지 않아도 결국 테스트가 통과하지 못하면 해당 @Test 자체가 실패한 것으로 인식할 수 있어서 문제될 것은 없다. 하지만 하나의 @Test 내에 두 개 이상의 assert 문이 있고, 테스트를 했을 때 어떤 것은 실패했고 어떤 것은 통과했는지를 구분하여 알 수 있다면 개발 자체가(=테스트 자체가) 더 용이할 수 있기 때문에 assertAll을 사용하는 것이다.

\


```
    @Test
    void statusTest() {
        Study study = new Study();
        assertEquals(StudyStatus.STARTED, study.getStudyStatus(), "처음 스터디를 만들면 상태는 DRAFT 여야 한다");
        assertEquals(StudyStatus.STARTED, study.getStudyStatus(), () -> "처음 스터디를 만들면 상태는 DRAFT 여야 한다");
    }
```

메세지를 그냥 그대로 넣어주는 것과 람다식으로 넣어주는 방법 둘다 메세지 처리에는 문제가 없는데, 람다식으로 할 경우 테스트가 실패했을 때에만 메세지 연산을 처리한다. 그래서 만약 처리할 메세지 연산의 처리비용이 부담이 될 수준이라면 람다식으로 넣어주는 것이 좋다. 그래서 결론적으로는 모든 메세지를 저렇게 람다식으로 만들어주는 습관을 들이는 것이 좋겠다.

\


```
    public Study(int limit) {
        if (limit < 0) {
            throw new IllegalArgumentException(Constant.studyLimitErrorMessage);
        }

        this.limit = limit;
    }
```

```
    @Test
    void limitTest() {
        IllegalArgumentException exception = assertThrows(IllegalArgumentException.class, () -> {
            Study study = new Study(-10);
        });
        assertEquals(exception.getMessage(), Constant.studyLimitErrorMessage);
    }
```

assertThrows를 단순히 원하는 에러 타입의 감지에만 사용하는 것 이상으로 에러 메세지의 일치성 여부까지 뽑아서 사용할 수 있다. 동일한 에러 유형 내에서 조건에 따라 에러 메세지를 다양하기 분기하는 경우 사용할 수 있겠다.

\


***

#### 5. JUnit5 조건에 따라 테스트 실행하기 <a href="#5-junit5" id="5-junit5"></a>

**조건 custum**

* assumeTrue(조건) : 조건이 만족할때 해당 @Test가 실행됨

\


**조건 종류에 따른 어노테이션 처리**

* Enabled(Disabled)OnOs/onJre : 특정 Os / 특정 자바 버전
* EnabledIfEnvironmentVariable
* EnabledIfSystemProperty

\


***

#### 6. JUnit 5 태깅과 필터링 <a href="#6-junit-5" id="6-junit-5"></a>

\


**태깅**

@Tag : 테스트 그룹을 만들고 원하는 테스트 그룹만 테스트를 실행할 수 있는 기능. intellij의 test configuration 조작(test kind = Tags, Tag expression = custom tag)을 통해서 특정 태그만 테스트 되도록 한다.

\


**필터링**

profile 별로 특정 태그만 달려있는 @Test 만 실행되도록 하는 방법. 강의에서는 [maven-surefile-plugin](https://maven.apache.org/guides/introduction/introduction-to-profiles.html) 을 다뤘고 gradle은 찾아보아야 한다. 이를 이용해 배포 환경에서 빌드시 특정 태그에 대한 테스트를 설정해두고 하나라도 실패한다면 이를 배포하지 않도록 하는 전략을 사용할 수 있다.

\


**커스텀 태그**

커스텀 태그를 사용하면 @Test에 부여해줄 여러가지 설정들을 단순화하고 중복을 제거하여 적용시킬 수 있다. 특히 @Tag를 사용해야 한다면 커스텀 태그를 이용하여 type safe하게 @Tag를 사용할 수 있다는 이점이 있다.\
\
다수의 @Tag에 매번 원하는 tag를 문자열로 입력하는 것 보다 설정해둔 tag가 적용된 커스텀 태그를 일괄로 적용하는 것이 곧 type safe한 방법이기 때문이다.\


```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Test
@Tag("development")
public @interface DevelopmentTest {
}
```

\


***

#### 7. JUnit 5: 테스트 반복하기 1부 <a href="#7-junit-5-1" id="7-junit-5-1"></a>

**@RepeatedTest**

특정 횟수만큼 테스트를 반복하여 하고 싶을 때 사용한다\
테스트 콘솔창에 나오는 테스트명과 반복 횟수 등을 커스텀하게 변경할 수 있다.\


```
    @RepeatedTest(value = 10, name = "{displayName}, {currentRepetition} / {totalRepetitions}")
    @DisplayName("스터디 반복 테스트")
    void repeatedTest(RepetitionInfo repetitionInfo) {
        System.out.println(repetitionInfo.getCurrentRepetition() + " / " + repetitionInfo.getTotalRepetitions());
    }
```

만약 placeholder 들이 제대로 작동하지 않는다면 Build, Execution, Deployment -> Build Tools -> Gradle로 이동한 다음 Run tests using 을 Gradle -> Intellij IDE 로 수정하면 된다.

\


**@ParameterizedTest**

@ValueSource를 이용해서 하나의 @Test에 여러 다른 매개변수(원하는 대로 선언된)를 대입해가며 반복 실행한다.\
\


추가적으로 사용 가능한 옵션

* @NullSource
* @EmptySource
* @NullAndEmptySource

```
    @DisplayName("parameterized Test")
    @ParameterizedTest(name = "{displayName} : {index} // {0}")
    @ValueSource(strings = {"first", "second", "third", "fourth", "fifth"})
    @NullSource
    @EmptySource
    //@NullAndEmptySource
    void parametersTest(String param) {
        System.out.println(param);
    }
```

\


**@ConvertWith + @ParameterizedTest**

커스텀한 컨버터를 이용한 테스트를 하고자 할 때 사용. 받은 인자값으로 바로 원하는 객체를 생성하여 사용할 수 있다.(받아서 내부에서 만들지 않고 바로 만들어진 객체를 파라미터로 받음)\
해당 예제는 SimpleArgumentConverter를 사용한 예제로 2개 이상의 파라미터를 사용할 수는 없다.\


```
    @ParameterizedTest
    @ValueSource(ints = {20, 30, 40, 50})
    void converterTest(@ConvertWith(StudyConverter.class) Study study) {
        System.out.println(study.getLimit());
    }

    static class StudyConverter extends SimpleArgumentConverter {

        @Override
        protected Object convert(Object source, Class<?> targetType) throws ArgumentConversionException {
            assertEquals(Study.class, targetType, "Can only convert Study");
            return Study.builder().limit(Integer.parseInt(source.toString())).build();
        }
    }
```

\


**@CsvSource + @AggregateWith + @ParameterizedTest**

복수의 파라미터와 이를 바로 객체로 받아 사용하고 싶을때 사용\


```
    @ParameterizedTest
    @CsvSource({"10, 자바", "20, 스프링"})
    void aggregateTest(@AggregateWith(StudyAggregator.class) Study study) {
        System.out.println("limit : " + study.getLimit());
        System.out.println("name : " + study.getName());
    }

    static class StudyAggregator implements ArgumentsAggregator {

        @Override
        public Object aggregateArguments(ArgumentsAccessor accessor, ParameterContext context) throws ArgumentsAggregationException {
            return Study.builder().limit(accessor.getInteger(0)).name(accessor.getString(1)).build();
        }
    }
```

\


***

#### 8. JUnit 5 테스트 인스턴스 <a href="#8-junit-5" id="8-junit-5"></a>

JUnit은 기본적으로 @Test 마다 해당 @Test가 속한 클래스의 인스턴스를 새로 만든다. 달리 말해서, 동일 class 내에서 특정 변수를 선언한 뒤 특정 @Test에서 이를 ++ 시켜주어도 다른 @Test에서는 여전히 그 변수가 ++되지 않은 상태라는 의미이기도 하다.\


왜냐하면 @Test 실행마다 이를 위한 독자적인 인스턴스를 늘 만들기 때문이다. 이는 @Test가 서로에게 주는 영향을 없애기 위해서 기본 전략으로 취하는 방법인데, JUnit 5에서는 이 기본 전략을 변경하는 것이 가능하다.

\


@TestInstance(Lifecycle.PER\_CLASS)

* 테스트 클래스당 인스턴스를 하나만 만들어 사용한다.
* 경우에 따라, 테스트 간에 공유하는 모든 상태를 @BeforeEach 또는 @AfterEach에서 초기화 할 필요가 있음
* @BeforeAll과 @AfterAll을 인스턴스 메소드 또는 인터페이스에 정의한 default 메소드로 정의할 수도 있다.

```
@DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class StudyTest {
}
```

\


***

#### 9. JUnit 테스트 순서 <a href="#9-junit" id="9-junit"></a>

한 클래스 내의 @Test 들은 특정한 순서에 의해 실행되지만 어떻게 그 순서를 정하는지는 의도적으로 분명히 하지 않는다. 이유는 테스트 인스턴스를 @Test 마다 독립적으로 생성하는 것과 같다. **즉, 제대로된 Unit 테스트라면 각 Unit test간에 누가 먼저 실행되든 서로에게 영향이 없어야 하기 때문이다.(멱등성의 개념과 어느정도 교집합이 있다고 생각된다)**

하지만, 경우에 따라서 특정한 순서대로 테스트를 실행하고 싶을 때가 있는데, 이 때에는 @TestInstance(TestInstance.Lifecycle.PER\_CLASS)과 함께 @TestMethodOrder를 통해서 메소드마다 순서를 정할 수 있다.(굳이 같이 쓸 필요는 없다. 다만, 순서를 정한다는 것이 클래스 내 공유할 자원이 있을 확률이 높기에 예제를 이렇게 다룬 것)

최근에 회사 일을 하면서도 실제로 회원가입을 먼저 해야 가능한 테스트 시나리오 내에서의 유닛 테스트를 위해서 따로 회원가입을 해주는 메소드를 따로 만들었다. 그리고 이를 @BeforeEach 처럼 특정 유닛 테스트전에 실행해서 회원가입을 한 뒤 거기서 토큰 정보를 추출하여 헤더에 이를 심어서 테스트를 하긴 했는데 뭐가 베스트인지는 아직도 잘 모르겠다. 내가 한 방법이 효율성은 떨어지긴 하지만 엄격한 의미에서 독립적인 유닛 테스트의 형태가 맞긴 하다고 생각하기 때문이다.\


```
@DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
@TestMethodOrder(MethodOrderer.OrderAnnotation.class) // @Order 라는 annotation으로 order를 정해주겠다고 선언하는 의미
class StudyTest {

// 각 메소드에 @Order(int) 로 순서를 지정해준다. int가 작을수록 먼저 실행.
}
```

\


***

#### 10. junit-platform.properties <a href="#10-junit-platformproperties" id="10-junit-platformproperties"></a>

junit-platform.properties는 JUnit 설정 파일로, 클래스패스 루트 (src/test/resources/)에 넣어두면 적용된다.\


* 테스트 인스턴스 라이프사이클 설정
  * junit.jupiter.testinstance.lifecycle.default = per\_class
* 확장팩 자동 감지 기능
  * junit.jupiter.extensions.autodetection.enabled = true
* @Disabled 무시하고 실행하기
  * junit.jupiter.conditions.deactivate = org.junit.\*DisabledCondition
* 테스트 이름 표기 전략 설정
  * junit.jupiter.displayname.generator.default = org.junit.jupiter.api.DisplayNameGenerator$ReplaceUnderscores

\


***

#### 11. JUnit 5 확장 모델 <a href="#11-junit-5" id="11-junit-5"></a>

JUnit 4의 확장 모델은 @RunWith(Runner), TestRule, MethodRule.\
[JUnit 5의 확장 모델](https://junit.org/junit5/docs/current/user-guide/#extensions) 은 단 하나, Extension.\


* 확장팩 등록 방법
  * 선언적인 등록 @ExtendWith
  * 프로그래밍 등록 @RegisterExtension
* 확장팩 만드는 방법
  * 테스트 실행 조건
  * 테스트 인스턴스 팩토리
  * 테스트 인스턴스 후-처리기
  * 테스트 매개변수 리졸버
  * 테스트 라이프사이클 콜백
  * 예외처리

```
@DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
@ExtendWith(ExecutionTimeExtension.class)
class StudyTest {
}
```

\
만약 위와 같이 THRESHOLD 와 같은 특정 값을 상수화 하지 않고 테스트 마다 다르게 설정하고 싶다면 @RegisterExtension을 이용한다.

```
@DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
class StudyTest {

    @RegisterExtension
    ExecutionTimeExtension executionTimeExtension = new ExecutionTimeExtension(1000L);
}
```

\


```

public class ExecutionTimeExtension implements BeforeTestExecutionCallback, AfterTestExecutionCallback {

    private final long threshold;

    public ExecutionTimeExtension(long threshold) {
        this.threshold = threshold;
    }

    @Override
    public void beforeTestExecution(ExtensionContext context) {
        ExtensionContext.Store store = this.getStore(context);
        store.put(START_TIME, System.currentTimeMillis());
    }

    @Override
    public void afterTestExecution(ExtensionContext context) throws Exception {
        ExtensionContext.Store store = this.getStore(context);
        long duration = System.currentTimeMillis() - store.remove(START_TIME, long.class);

        if (duration > threshold) {
            System.out.println("over threshold");
        }
    }

    private ExtensionContext.Store getStore(ExtensionContext context) {
        String testClassName = context.getRequiredTestClass().getName();
        String testMethodName = context.getRequiredTestMethod().getName();
        return context.getStore(ExtensionContext.Namespace.create(testClassName, testMethodName));
    }

}
```

\
