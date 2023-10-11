# \[11] equals 를 재정의하려거든 hashCode도 재정의하라

## 핵심 내용

* HashMap 에 데이터를 넣고 꺼낼때 어떤 일이 일어나는지
* equals를 재정의 했는데 hashCode를 재정의 하지 않을 경우 무슨 문제가 생기는지



## hashCode 규약

1. equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을 호출해도 일관되게 같은 값을 반환해야한다.(equals 판단에 사용되는 프로퍼티가 그대로 hashCode 에 똑같이 사용되어서 멱등성이 지켜져야 한다는 것)
2. **equals 로 두 객체가 같다고 판단 했으면, 두 객체의 hashCode 는 똑같은 값을 반환해야 한다.**
3. equals(Object)가 두 객체를 다르게 판단해도, 해시코드가 서로 다른 값을 반환할 필요는 없으나 다른 객체에 대해서 다른 값을 반환해야 해시테이블 성능이 좋아진다.



## HashMap 에 데이터를 넣고 꺼낼때 어떤 일이 일어나는지

HashMap 에 데이터를 넣을때 내부에서 해당 객체의 키값으로 사용할 해시 코드를 계산해서 적재한다. 해시 코드가 키값으로서 탐색시 인덱스로 사용된다. 찾을때도 해당 객체를 가지고 해시코드를 만들어서 인덱스 처럼 사용해서 값을 꺼내온다.

만약 해시 충돌이 발생하면 해당 인덱스에 연결 리스트로 데이터를 저장하여 충돌을 처리한다.



## equals를 재정의 했는데 hashCode를 재정의 하지 않을 경우 무슨 문제가 생기는지

```java
package item11;

import java.util.Objects;

public class User {
    int age;
    String name;
    String phoneNumber;

    public User(int age, String name, String phoneNumber) {
        this.age = age;
        this.name = name;
        this.phoneNumber = phoneNumber;
    }

//    @Override
//    public boolean equals(Object o) {
//        if (this == o) return true;
//        if (o == null || getClass() != o.getClass()) return false;
//        User user = (User) o;
//        return age == user.age && Objects.equals(name, user.name) && Objects.equals(phoneNumber, user.phoneNumber);
//    }
//
//    @Override
//    public int hashCode() {
//        return Objects.hash(age, name, phoneNumber);
//    }

}
```

```java

public class Main {
    public static void main(String[] args) {
        final Map<User, String> userMap = new HashMap<>();

        User user1 = new User(20, "홍길동1", "010-1234-1234");
        User user2 = new User(20, "홍길동2", "010-1234-1235");

        userMap.put(user1, "hello");
        userMap.put(user2, "world");
        System.out.println(userMap.size());

        System.out.println(userMap.get(user1)); // hello
        System.out.println(userMap.get(user2)); // world
        System.out.println(userMap.get(new User(20, "홍길동2", "010-1234-1235"))); // null
    }
}
```

해시코드가 구현되어 있지 않으므로 user2 가 들어가 있음에도 마지막 줄에서 찾는 값은 존재하지 않아서 null 이 발생한다.

HashMap, HashSet 에서 삽입시 및 조회시 키값으로 해시코드 함수로 만든 해시값이 키값으로 사용되므로 이를 감안하여 equals() 에 사용되는 필드와 동일한 필드들을 이용해서 hashCode 를 만들도록 함수를 정의해야한다.



```java
        User user1 = new User(20, "홍길동1", "010-1234-1234");
        User user2 = new User(20, "홍길동2", "010-1234-1235");
        User user3 = new User(20, "홍길동2", "010-1234-1235");

        userMap.put(user1, "hello");
        userMap.put(user2, "world");
        userMap.put(user3, "world");
        System.out.println(userMap.size());
```

이 경우에 user2, user3 이 같음에도 size 는 3이 나온다.
