# \[10] equals는 일반 규약을 지켜 재정의하라

## 핵심 내용

equals 를 정의 해야할 때와 하지 않아야 할 때를 잘 구분해야 한다. 이 말인즉, 명백한 이유 없이 기계적으로 오버라이딩 하지 말라는 의미이다. 왜냐하면 equals 를 잘못 정의하면 의도치 않은 결과가 발생하기 때문이다.

이번 아이템에서는 아래 내용들을 학습했다.

* equals 를 재정의 하지 말아야 하는 경우들은 어떤 경우들이 있는가
* equals 를 재정의 한다면 어떤 원칙들을 준수해야 하는가



## equals 를 재정의 하지 말아야 하는 경우들

### 각 인스턴스가 본질적으로 고유할때

책에서 Thread 를 예시로 들고 있다. 같고 다름을 비교할 필요조차 없는 본질적으로 각기 고유하게 존재하는 객체이다. 백기선님 강의에서 싱글톤 객체도 언급하고 있다.



### 인스턴스의 '논리적 동치성'을 검사할 일이 없을 때

굳이 같고 다름을 비교할 일이 없는 경우다. 이것도 마찬가지로 싱글톤 객체 같은 것들을 예로 들 수도 있겠다. 특정 객체에 대해서 무조건 절대적으로 이것과 다른 것을 비교할 일이 없는 경우를 의미한다.

강의에서 문자열을 예시로 드는데 이 또한 좋은 예시인 것 같다. "hi" 와 "hi" 는 다른 객체라 할 지라도 비교할 필요가 없는 당연히 같은 객체이다.



### 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞을때

상위 클래스에서 정의된 equals 가 현 상황에 잘 적용될 수 있다면 굳이 재정의 할 필요가 없다. 당연한 이야기이다.



### 클래스가 private 이거나 package-private 이고 equals 를 호출할 일이 없을때

이것도 너무 당연한 이야기이다. 노출될 일 자체가 없는데 비교하고 말고 할 일이 없으니까 재정의 할 필요가 없다. 쓸 일이 없는 메소드를 재정의하는 행동이다.



## equals 재정의시 지켜야 할 원칙

책에서 equals 를 재정의 해야한다면 반드시 안내하는 5가지 원칙을 지켜야 한다고 말하고 있다. equals 재정의시 지켜져야할 원칙들에 대해서 아래에 정리한다.



### null 이 아님

'null 이 아닌 모든 참조 값 x에 대해, x.equals(null) 은 false이다.'

이건 맨 마지막 원칙으로 안내되고 있었는데 내가 맨 위로 올렸다. 왜냐면 이것을 첫번째 원칙으로 넣어야 나머지 다른 원칙들에 대해서 'null이 아닌 참조값\~' 이라는 한정된 문구를 굳이 안넣어도 될 것 같아서이다.



### 반사성

'x.equals(x) 는 true 이다.'



### 대칭성

'x.equals(y)가 true 이면 y.equals(x)도 true 이다.'

이건 대칭성을 깨트리는 코드를 한번 살펴 보자.

```java
public class CaseInsensitiveString {
    final String value;

    public CaseInsensitiveString(String value) {
        this.value = value;
    }

    @Override
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString) {
            return value.equalsIgnoreCase(((CaseInsensitiveString) o).value);
        }

        // String 인 경우에도 대소문자 구분 없이 비교를 한다(대칭성을 깨트린다)
        if (o instanceof String) {
            return value.equalsIgnoreCase((String) o);
        }

        return false;
    }

}

public class Main {
    public static void main(String[] args) {
        CaseInsensitiveString example1 = new CaseInsensitiveString("hi");
        String example2 = "HI";

        System.out.println(example1.equals(example2)); // true
        System.out.println(example2.equals(example1)); // false
    }
}
```

```
> Task :Main.main()
true
false
```

극단적인 예시이긴 하지만 저렇게 equals 를 잘못 정의할 경우 대칭성이 깨져버린다.



### 추이성

'x.equals(y) 가 true 이고 y.equals(z) 가 true 이면 x.equals(z) 도 true 이다.'



위 논법에 의하면 x와 z는 같아야 한다. 이건 코딩 관련된 이야기가 아니라 논리적으로 당연히 그러해야한다. 하지만 아래 코드를 보면 대칭성이 깨지는 것을 확인 할 수 있다.

```java
public class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }

    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof Point)) {
            return false;
        }

        // Point 이긴 한데 ColorPoint 가 아닌 경우다
        if (!(obj instanceof ColorPoint)) {
            // Point 에서 정의한걸 쓰는데 x, y 좌표만 비교
            return super.equals(obj);
        }

        ColorPoint o = (ColorPoint) obj;
        return this.x == o.x && this.y == o.y && this.color == o.color;
    }
}

public enum Color {
    RED, BLUE;
}

public class Main {
    public static void main(String[] args) {
        ColorPoint colorPoint1 = new ColorPoint(1, 2, Color.RED);
        Point point = new Point(1, 2);
        ColorPoint colorPoint2 = new ColorPoint(1, 2, Color.BLUE);

        System.out.println(colorPoint1.equals(point)); // true
        System.out.println(point.equals(colorPoint2)); // true
        System.out.println(colorPoint1.equals(colorPoint2)); // false
    }
}
```



근본적으로 이 문제는 구체 클래스를 만들어서 필드를 추가하게 되어서 발생된 문제인데 책에서도 아래와 같이 나와있다.

> 구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다. 객체 지향적 추상화의 이점을 포기하지 않는 한은 말이다.

위 코드를 예시로 들면 ColorPoint 와 Point 간의 비교(유연성) 를 포기하고 ColorPoint 는 ColorPoint 끼리만 비교하게 하면 equals 규약을 지킬 수는 있다는 이야기이다.



이를 극복하기 위해서는 ColorPoint 가 Point 를 상속하는 것이 아니고 ColorPoint 의 필드로 Point 를 갖도록 하라고 하는데, 이건 당연한 이야기이고 다형성을 포기하게 된다.



### 일관성

'x.equals(y) 를 반복해서 호출하면 항상 true 를 반환하거나 항상 false 를 반환한다'

두 객체 모두가 수정되지 않는 한 비교 시점에 따라 값이 바뀌는 것이 아니라 항상 일관된 값을 반환해야 한다는 것이다.



## equals 정의시 속도를 고려한다면 어떻게 하면 좋을까

다를 가능성이 큰 필드를 비교의 순서상 앞에 두는 것이 좋다. 왜냐면 다를 경우 빨리 판단될 가능성이 크기 때문이다.



## equals 재정의시 구체적 방법(모범 답안(?) 이고 이 방법에 따라 만들더라도 위 원칙들을 잘 지키는지 살펴봐야한다)

1. \== 연산자로 입력이 자기 자신의 참조인지 확인한다.(반사성 지키는지를 확인하게 된다.)
2. instanceof 연산자로 올바른 타입인지 확인한다.(다른 타입 오면 바로 false 줘버리면 되니까 프로퍼티 비교까지 갈 필요도 없으면 빨리 거른다)
3. 올바른 타입으로 형변환 한다. (4 에서 프로퍼티 비교를 해야하니까 형변환을 해야한다)
4. 비교해야할 프로퍼티들을 일일이 하나씩 and 조건으로 비교하고, 속도를 고려해서 다를 가능성이 큰 필드를 앞에 배치한다.



매번 intelliJ 에 의존해서 재정의를 했었는데, 그렇게 만들지 않더라도 일일이 손으로 직접 만든다 했을때 제대로 만들 수 있어야 한다.
