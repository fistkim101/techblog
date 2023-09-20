# equals()와 hashCode()가 무엇이고 역할이 무엇인지

### equals() 가 무엇이며 기본 반환 값은 무엇인가

Object 가 가진 메소드로 두 객체가 동일한지 비교하는 메소드이다. 기본적으로 두 객체의 레퍼런스를 비교해서 불리언을 응답한다. 두 객체의 참조가 같은 메모리 주소를 가리키는지를 의미한다.



### hashCode() 가 무엇이며 기본 반환 값은 무엇인가

마찬가지로 Object 의 메소드이며 객체의 해시 코드 값을 반환한다. 기본 구현은 객체의 메모리 주소에 기반한 해시코드를 생성한다.



### 동일성과 동등성

동일성은 참조가 같은 경우를 의미한다. 동등성은 내용에 대한 동등함을 의미한다.



아래는 동등하지만 동일성이 다른 경우이다.

```java
String str1 = new String("hello");
String str2 = new String("hello");

System.out.println(str1 == str2);  // 출력: false (서로 다른 객체를 참조)
```



그러면 String 비교를 equals 로 하면 위에서 알아본 바와 같이 기본 값이 레퍼런스를 비교하는 것이니까 아래 코드는 false 가 나와야 할 것 같은데 true 이다.

```java
String str1 = new String("hello");
String str2 = new String("hello");

System.out.println(str1.equals(str2));  // 출력: true (내용이 같음)
```



왜? String 은 equals 가 오버라이딩 되어 있다. 내용을 검사해서 동등성 여부로 equals 에 대한 반환을 한다.

```java
public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
    // ... 생략 ...

    public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String) anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
    
    // ... 생략 ...
}
```



### equals()와 hashCode()를 꼭 같이 오버라이딩 해줘야하는가?

반드시 그래야 하는 것은 아니지만 같이 해주는 것이 좋다. 왜냐하면 일부 컬렉션(Collection(HashMap, HashSet, HashTable))에서 중복여부 판단을 hashCode() 를 사용하기 때문이다.

<figure><img src="../../.gitbook/assets/image (134).png" alt=""><figcaption><p><a href="https://tecoble.techcourse.co.kr/post/2020-07-29-equals-and-hashCode/">https://tecoble.techcourse.co.kr/post/2020-07-29-equals-and-hashCode/</a></p></figcaption></figure>

그래서 equals() 를 동등성 비교까지 고려해서 잘 정의해놨다고 해도 hashCode() 를 오버라이딩 해놓지 않으면 기본 처리인 메모리 주소값에 기반한 hashCode 만들 것이고 이것으로 동등성 여부를 판단하기 때문에 위 컬렉션 내에서 동등하지만 다른 객체로 판단될 수 있다.
