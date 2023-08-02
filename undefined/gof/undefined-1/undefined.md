# 어댑터 패턴

## 대분류

구조



## 문제상황

클라이언트가 특정 객체를 사용하고 싶은데 해당 객체가 클라이언트가 사용하는 인터페이스의 구현체가 아닌 경우 이를 사용하기 위해서 어댑터를 중간에 두는 것



## 해결방안

가장 간단하게는 클라이언트가 사용하고자 하는 그 객체가 클라이언트가 사용하는 인터페이스를 구현하면 된다. 하지만 이렇게 되면 변경점이 더 커지므로 중간에 어댑터를 두고 어댑터 객체가 클라이언트가 사용하는 인터페이스를 구현한 뒤 어댑터가 사용되면서 어댑터는 내부에서 타겟 객체를 호출한다.

<figure><img src="../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>



## 실습코드

```java
public class Main {
    public static void main(String[] args) {

        // before
        final TargetObject targetObject = new TargetObject();
        Client client = new Client();
        // client.call(targetObject); // <-- 쓸 수가 없다.

        // after
        final Adapter adapter = new Adapter(targetObject);
        client.call(adapter);
    }
}

public class Client {

    public void call(OriginalInterface originalInterface){
        System.out.println("Client called");
        originalInterface.call();
    }

}

public interface OriginalInterface {

    void call();
}

public class Adapter implements OriginalInterface {

    private final TargetObject targetObject;

    public Adapter(TargetObject targetObject) {
        this.targetObject = targetObject;
    }

    @Override
    public void call() {
        System.out.println("Adapter called");
        targetObject.call();
    }
}

public class TargetObject {

    public void call(){
        System.out.println("TargetObject called");
    }

}
```

이번 것은 예시로 비유를 뭔가 하려다가 있는 그대로 그냥 명시를 했다. 오히려 더 알아보기가 좋은 것 같아서 다음 디자인 패턴도 이런 식으로 할까 싶다.

핵심을 다시 정리하자.

1. 어댑터는 클라이언트가 사용하는 인터페이스를 구현해야한다.
2. 1 덕분에 어댑터는 클라이언트가 사용할 수 있게 되었다. 따라서 최종적인 목표로 클라이언트가 타겟 객체를 사용할 수 있도록 어댑터가 타겟 객체를 의존하고 이를 클라이언트의 call 흐름 속에서 타겟 객체를 어댑터 객체가 call 한다.
