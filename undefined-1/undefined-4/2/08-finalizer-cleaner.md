# \[08] finalizer 와 cleaner 사용을 피하라

## 핵심 내용

1. finalizer 와 cleaner 모두 그냥 사용하지 마라.
2. 반환을 꼭 시켜야하는 자원을 사용한다면 AutoCloseable과 try with resource를 사용하는 것을 권장한다.
3. 굳이 cleaner 를 사용해서 반환이 보장되도록 하는 방어 코딩을 할 수는 있다.



## finalizer 와 cleaner 가 뭐지?

둘 다 객체정리 매커니즘인데 finalizer 의 경우 5에 공개되고 9에 deprecated 되었다. 왜냐하면 실행 시점이 제어가 안되기 때문이다. (=언제 실행 될지 알 수가 없고 원하는 시점에 실행 시키지도 못한다) 그리고 성능 오버헤드도 발생 시켜서 성능 면에서도 부정적이다.

finalizer 는 보안 문제까지 있다고 하는데 하위 클래스가 직렬화 과정에서 예외가 발생하면 생성이 멈추는 것이 아니라 생성 되다 만 객체에서 finalizer 의 동작이 수행될 수 있다고 한다.(막으려면 상속 자체를 막아버리면 된다)

cleaner 의 경우도 객체 정리 매커니즘인데, 이번 아이템에서 이것 역시 사용을 권하지 않고 있다. cleaner 역시 실행 시점을 제대로 보장할 수는 없다고 한다. 다만 예제 코드에서는 동작을 시키긴 한다. System.gc() 에 의해서 동작할 확률이 올라가는 것이지 실제로 동작이 무조건 되진 않는다.



## Cleaner 사용 방법(+ 예시 코드)

```java
package item8;

import java.util.List;

public class Room {
    private List<Object> objects;

    public Room(List<Object> objects) {
        this.objects = objects;
    }

    static class RoomCleaner implements Runnable {
        private List<Object> objects;

        public RoomCleaner(List<Object> objects) {
            this.objects = objects;
        }

        @Override
        public void run() {
            System.out.println("RoomCleaner : room clean ...");
            objects = null;
        }
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Cleaner cleaner = Cleaner.create();
        List<Object> objects = List.of("a", "b", "c");
        Room room = new Room(objects);

        cleaner.register(room, new Room.RoomCleaner(objects));
        room = null;
        System.gc();
        System.out.println("gc started");
    }
}
```

```bash
> Task :Main.main()
gc started
RoomCleaner : room clean ...
```

위와 같이 명시적으로 등록을 해주고 gc 동작시 어떤 식으로 리소스를 정리할 지 내부에서 Runnable 로 정의를 해두었다. 주의할 점은 단순히 내부 클래스로 구현하면 참조값을 가지게 되므로 '정적 내부'클래스로 구현해야하고, 해당 클래스에서 바깥 클래스를 참조해서는 안된다는 것이다. 그럴 경우 gc 동작과 함께 바깥 클래스가 부활할 수 있다. 이 부분은 아이템 24 에서 또 다룬다고 하니 일단 이정도로 하고 넘어간다.



## AutoCloseable 사용 방법(+ 예시 코드)

```java
package item8_new;

import java.util.List;

public class Room implements AutoCloseable {
    private List<Object> objects;

    public Room(List<Object> objects) {
        this.objects = objects;
    }

    @Override
    public void close() throws Exception {
        objects = null;
        System.out.println("room cleaned up");
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        List<Object> objects = List.of("a", "b", "c");
        Room room1 = new Room(objects);

        List<Object> objects = List.of("a", "b", "c");
        try (Room room2 = new Room(objects)) {

        } catch (Exception exception) {

        }
    }
}
```

위 코드의 경우 당연한 것이지만 room1 은 close() 가 동작하지 않고, room2 에서는 try 문이 끝나면서 바로 동작한다. 그래서 메모리 반환을 꼭 해야하는 자원들이 있다면 close() 에 구현해주고 해당 객체를 위와 같이 try with resource 에서 생성하여 사용하면 된다.



### 만약 close() 에서 CheckedException 처리를 해야하는 경우

throw 를 한다는 것은 클라이언트에 예외 처리 책임을 전가하는 것이고 close() 내에서 예외처리를 한다는 것은 책임을 전가하지 않겠다는 것이다. 이건 취사선택을 하는 것. 지금 굳이 당연한 것을 짚고 넘어가는 이유는 이것의 멘탈 모델을 인지하기 위함이다.



## (+) 정적이 아닌 중첩 클래스는 자동으로 바깥 객체의 참조를 갖는다

위 cleaner 예시 코드에서 코드 관리(관심사를 분리하지 않고 해당 객체 내에 모아둠)를 위해서 Clean 을 위한 로직을 해당 객체 내부에 선언해두었는데, 이 경우 정적 클래스로 선언을 해줘야 한다. 왜냐하면 정적 클래스로 선언해주지 않을 경우에 외부 클래스에 대한 참조값을 가지게 되기 때문에 GC 의 동작에 외부 클래스가 활성 객체로 분류되어 메모리 반환이 이뤄지지 않을 수 있기 때문이다. 아래 코드를 보자.

```java
package item8_new;

import java.lang.reflect.Field;

public class OuterClass {

    class InnerClass {

        public void printAllFields() {
            Class<InnerClass> innerClass = InnerClass.class;
            for (Field field : innerClass.getDeclaredFields()) {
                System.out.println("Name : " + field.getName());
                System.out.println("Type : " + field.getType());
            }
        }

    }

}
```

```java
package item8_new;

public class Main {
    public static void main(String[] args) {
        OuterClass outerClass = new OuterClass();
        OuterClass.InnerClass innerClass = outerClass.new InnerClass();
        innerClass.printAllFields();

    }
}
```

```bash
> Task :Main.main()
Name : this$0
Type : class item8_new.OuterClass
```

보면 InnerClass 에 어떤 필드도 만들어주지 않았는데도 OuterClass 에 대한 참조를 가지고 있음을 알 수 있다.

결론적으로 굳이 클리닝 작업을 내가 선언해서 사용하겠다(AutoCloseable 사용 하지 않겠다)고 한다면 클리닝 대상 객체 내부에 선언하지 말고 밖에 선언하는 것도 좋은 방법이다.
