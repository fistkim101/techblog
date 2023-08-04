# 컴포짓 패턴

## 대분류

구조



## 문제상황

클라이언트가 사용하고자 하는 타겟 오브젝트의 위계를 전혀 신경 쓰지 않고 통일된 어떠한 기능을 수행하고자 한다. 부분이든 전체든 동일한 기능을 수행해준다.



## 해결방안

위계에 속한 모든 컴포넌트들이 동일한 인터페이스를 구현하도록 하고, 클라이언트에서는 이 인터페이스를 통해서 메세지를 전달한다.

<figure><img src="../../../.gitbook/assets/image (94).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (95).png" alt=""><figcaption></figcaption></figure>



## 실습코드

```java
public interface Employee {
    void work();
}

public class Junior implements Employee{
    @Override
    public void work() {
        System.out.println("Junior is working");
    }
}

public class Team implements Employee {
    private final List<Employee> employees;

    public Team(List<Employee> employees) {
        this.employees = employees;
    }

    @Override
    public void work() {
        System.out.println("team is working start ---");
        this.employees.forEach(Employee::work);
        System.out.println("team is working end ---");
    }
}

public class Client {
    public static void main(String[] args) {
        Employee junior1 = new Junior();
        junior1.work();
        Employee junior2 = new Junior();
        junior2.work();

        Employee team = new Team(List.of(junior1, junior2));
        team.work();
    }
}
```

사실 이건 [일급컬렉션](https://jojoldu.tistory.com/412)과 크게 다르지 않다. 일급컬렉션 개념에 통일된 인터페이스의 구현을 강제한 것이 컴포짓 패턴이다.

