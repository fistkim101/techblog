# (+) Spy vs Mock

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
