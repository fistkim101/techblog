# (+) Spy vs Mock

## Spy와 Mock 은 다르다

굳이 조작 비용을 치르고 테스트를 위한 조작을 한다면 Spy 도 활용하기 좋다. Spy는 실제 객체의 실제 메소드 호출이 발생되고 Mock 은 리얼한 껍데기다.

[스택오버플로우의 답변](https://stackoverflow.com/questions/28295625/mockito-spy-vs-mock)중 직관적인 설명을 해둔 것이 있어 옮겨왔다.

> Technically speaking both "mocks" and "spies" are a special kind of "test doubles".
>
> Mockito is unfortunately making the distinction weird.
>
> **A mock in mockito is a normal mock** in other mocking frameworks (allows you to stub invocations; that is, return specific values out of method calls).
>
> **A spy in mockito is a partial mock** in other mocking frameworks (part of the object will be mocked and part will use real method invocations).

mock 은 mock 이고, spy 는 부분적인 mock 이다.



## 예제코드

아래 샘플 코드를 보자. 물론 전부 통과하는 코드다.

```java
@SpringBootTest
public class SampleTest {

    @Test
    void spyAndMockTest() {
        User spyUser = Mockito.spy(new User("kim", 1));
        Mockito.doReturn("king").when(spyUser).getName();
        Assertions.assertThat(spyUser.getName()).isEqualTo("king");
        Assertions.assertThat(spyUser.getGrade()).isEqualTo(1);

        User mockUser = Mockito.mock(User.class);
        Assertions.assertThat(mockUser).isNotNull();
        Assertions.assertThat(mockUser.getName()).isNull();
    }

}


public class User {

    private String name;

    private int grade;

    public User(String name, int grade) {
        this.name = name;
        this.grade = grade;
    }

    public String getName() {
        return name;
    }

    public int getGrade() {
        return grade;
    }
}
```



아래 코드에서도 알 수 있듯이 spy 로 한번 감싼 mockList가 실제로 add(100), add(200) 을 호출 했음을 알 수 있다. 하지만 Stubbing 역시 잘 동작하고 있음을 보여주고 있다.

```java
@Test
public void SPY_SAMPLE_TEST(){
    List<Integer> list = new ArrayList<Integer>();
    List<Integer> mockList = Mockito.spy(list);

    mockList.add(100);
    assertEquals(mockList.size(),1); // PASS

    doReturn(5).when(mockList).size(); // Stubbing
    assertEquals(mockList.size(),5); // PASS

    mockList.add(200);
    assertEquals(mockList.size(),5); // PASS

    System.out.println(mockList); // [100, 200]
    verify(mockList,times(2)).add(anyInt()); // PASS
}
```
