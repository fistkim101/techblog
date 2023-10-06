# \[13] clone 재정의는 주의해서 진행하라

## 핵심 내용

* Cloneable 인터페이스
* clone() 규약
* clone() 과 얕은 복사, 깊은 복사
* 그래서 결론적으로 clone() 을 써야할까? 써도 되는 경우와 안되는 경우
* clone() 의 대안

책 내용만 봐서는 왠만해선 쓰지 않는게 좋은 것 같고, 실제로 저자도 사용을 권하고 있지는 않다.(clone 자체를) 실무에서도 Cloneable 을 자주 만나진 못했던거 같다. 딥카피가 필요했던 경우가 있긴 했지만 성능을 고려하면 생성자로 새로 만드는 것이 좋은 것 같고 책에서도 이를 권하고 있다. 아무튼 clone 에 대해서 짚어보는 시간이었다.



## Cloneable 인터페이스

### 마커인터페이스다

자바에서 제공하는 마커 인터페이스(Marker Interface) 중 하나다. 마커 인터페이스는 메서드를 정의하지 않고, 단지 해당 클래스가 특정 기능을 지원하거나 특정 상태에 있음을 나타내는 데 사용된다. 즉, 정의된 메소드도 하나도 없고 그냥 마킹 하려고 만든 인터페이스이다.

### 역할이 무엇일까

Object 의 clone() 을 사용하기 위해서는 해당 인터페이스를 따르고 clone() 을 재정의 해줘야 한다. clone() 의 규칙이 그러하다. 반대로 말하면 clone() 을 쓰려면 반드시 상속을 해야하는 인터페이스이다.



## clone() 규약

### x.clone() != x 반드시 true

리얼 복제가 되었다는 것이다. 얕은 복사건 뭐건 어쨌든 실제 복사된 객체 자체의 참조값은 다르다.(다른 객체라는 뜻)



### x.clone().getClass() == x.getClass() 반드시 true

복제 되었으니 타입이 같아야 한다.



### x.clone().equals(x) true가 아닐 수도 있다.

이게 조금 혼란스러울 수 있는데 정리하자면 '복사를 했다고 해서 같다는 것은 아니다' 라는 것이다. 왜냐면 재정의를 하는 과정에서 id 값과 같은 유니크 해야하는 값은 똑같지 않도록 처리할 수 있기 때문이다.



### clone() 과 얕은 복사, 깊은 복사

얕은 복사로 인해서 원치 않게 카피본의 객체를 변경 시키다가 원본을 변경 시킬 수도 있다.

```java
public class Coin {
    private int value;

    public Coin(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }
}

public class Coins implements Cloneable {
    public final Stack<Coin> values = new Stack<>();

    public Coins() {
    }

    @Override
    protected Coins clone() throws CloneNotSupportedException {
        return (Coins) super.clone();
    }
}

public class Main {
    public static void main(String[] args) throws CloneNotSupportedException {
        Coin coin1 = new Coin(100);
        Coin coin2 = new Coin(200);
        Coin coin3 = new Coin(300);

        Coins coinsOriginal = new Coins();
        coinsOriginal.values.push(coin1);
        coinsOriginal.values.push(coin2);
        coinsOriginal.values.push(coin3);

        Coins coinsClone = coinsOriginal.clone();

        System.out.println(coinsOriginal); // item12.Coins@7e9e5f8a
        System.out.println(coinsClone); // item12.Coins@8bcc55f

        System.out.println(coinsOriginal.values); // [item12.Coin@14dad5dc, item12.Coin@18b4aac2, item12.Coin@764c12b6]
        System.out.println(coinsClone.values); // [item12.Coin@14dad5dc, item12.Coin@18b4aac2, item12.Coin@764c12b6]
        
        
        while (!coinsOriginal.values.isEmpty()) {
            coinsOriginal.values.pop();
        }

        System.out.println(coinsClone.values.size()); // 0
    }
}
```

복사한 객체 자체는 다른 객체인 것이 맞지만(규약 1을 만족) 내부의 참조형 변수는 똑같은 값임을 알 수 있다. 얕은 복사가 이뤄진다는 것이다.



## 그래서 결론적으로 clone() 을 써야할까? 써도 되는 경우와 안되는 경우

* 내부 프로퍼티가 모두 원시형이다
* 불변 객체이다

두 경우라면 clone 을 써도 무방하다. 왜냐하면 원시형이라면 얕은 복사에 대한 우려를 할 필요가 없고(값 객체로서 사용된다는 측면에서) 불변 객체라면 변경에 의한 사이드 이펙트를 걱정할 필요가 없기 때문이다.



쓰면 안되는 경우는 위와 같이 얕은 복사로 인해서 문제가 생길 여지가 있는 경우이다. 쓰면 안된다기 보다는 지양 해야한다는 표현이 맞는 것 같다.



## clone() 의 대안

복사 생성자 또는 복사 팩터리를 이용한다.

```java
class Student
{
    private String name;
    private int age;
    private Set<String> subjects;

    public Student(String name, int age, Set<String> subjects)
    {
        this.name = name;
        this.age = age;
        this.subjects = subjects;
    }

    // 복사 생성자
    public Student(Student student)
    {
        this.name = student.name;
        this.age = student.age;

        // 얕은 복사
        // this.subjects = student.subjects;

        // 깊은 복사 – `HashSet`의 새 인스턴스 생성
        this.subjects = new HashSet<>(student.subjects);
    }

    @Override
    public String toString()
    {
        return Arrays.asList(name, String.valueOf(age),
                subjects.toString()).toString();
    }

    public Set<String> getSubjects() {
        return subjects;
    }
}
```

```java
    // 복사 생성자
    public Student(Student student)
    {
        this.name = student.name;
        this.age = student.age;
        this.subjects = new HashSet<>(student.subjects);    // 딥 카피
    }
 
    // 팩토리 복사
    public static Student newInstance(Student student) {
        return new Student(student);
    }
```

책에서도 '복사 생성자와 복사 팩터리라는 더 나은 객체 복사 방식을 제공할 수 있다' 고 '더 나은' 이라고 평하고 있다. 성능면에서도 굳이 직렬화, 역직렬화를 한다던가 할 것 없이 생성자를 이용해서 수작업처리(?) 를 해주는 것이 더 좋다.
