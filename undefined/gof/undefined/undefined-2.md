# 추상 팩토리 패턴

## 팩토리 메소드 패턴과 추상 팩토리 패턴의 공통점

추상 팩토리는 다음 순서지만 지금 확실하게 짚어 두는 것이 팩토리 메소드 패턴 이해에도 도움이 될 것 같아 정리한다.

팩토리 메소드든 추상 팩토리든 공통적으로 **객체의 생성과 관련된 처리를 캡슐화해서 클라이언트 코드로부터 이를 분리해 내서 유지 보수성과 확장성을 갖는 것이 목적**이다.



## 팩토리 메소드 패턴과 추상 팩토리 패턴의 차이점

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

강의에서는 위와 같이 설명하고 있다. 제일 처음 짚어본 것처럼 팩토리 메소드든 추상 팩토리든 둘다 결국 객체의 생성 과정을 캡슐화 해서 이를 클라이언트로부터 디커플링 시키는 것이다. 하지만 팩토리 메소드의 경우 캡슐화 시키고자 하는 객체가 하나이고, 추상 팩토리의 경우 생성 과정을 캡슐화 시키고자 하는 객체'들'이 존재한다.

**같은 말이지만 다시 정리하자면, 팩토리 메소드는 생성 과정을 캡슐화 시키고자 하는 객체가 인터페이스 기준으로 하나 존재하고 추상 팩토리는 생성 과정을 캡슐화 시키고자 하는 객체가 인터페이스 기준으로 여러개 존재한다.**

팩토리 메소드는 특정 객체에 대한 생성을 캡슐화 하는 것인데, 여기서 만약 특정 종류로 구분되는 객체'들'의 생성을 캡슐화 해야하는 경우 팩토리 하나를 두고 해당 팩토리가 이 객체들을 생성하는 팩토리를 참조할 수도 있고, 해당 팩토리가 객체들을 전부 생성할 수도 있다.

결론적으로 클라이언트는 개별 생성 과정에 대해 알 필요도 없고 알 수도 없게 된다. 디커플링 되었기 때문이다.



## 대분류

객체 생성



## 문제상황

클라이언트가 생성해야 할 객체가 여러 군집이 존재하고 이것이 확장 가능성이 있을 때이다.

예를 들어 배를 만드는 조선소가 있다고 했을 때, 단순히 배의 종류가 확장 가능성이 있는 것은 팩토리 메소드 패턴의 적용이 가능하다. 하지만 배의 종류에 따라 각 배의 구성 부품 역시 달라져야 하는 경우에 특정 팩토리 내부에 각 부품의 생성을 위한 커스텀한 구현 과정이 들어가게 된다. 이때 부품이 추가가 되거나 부품 생성이 변경되게 되면 각 팩토리의 코드가 변경되어야 한다.

이 상황 자체가 잘못된 것은 아니지만 각 팩토리 역시 클라이언트라고 보고 여기서 각 부품의 생성을 캡슐화 할 수 있다면 더 좋은 설계가 될 수 있다.

다른 예를 들자면 가구 공장이 있다고 했을 때, 소파와 책상을 세트로 판다고 가정하자. 이 때 엔틱 스타일과 럭셔리 스타일이라는 디자인 종류가 존재하고 종류마다 소파와 책상이 종류가 달라져야한다. 이 때 종류가 추가가 되거나 기존 생성 로직에 변경이 발생하면 클라이언트 측에 많은 변경이 발생할 수 있다. 아예 각 종류마다 팩토리를 구분하고 각 팩토리에서 소파와 책상에 대한 생성 로직을 구현하게 하는 것이 추상 팩토리 메소드 패턴이다.

조선소 예시와 가구 예시 모두 사실 똑같다. 다만 조선소 예시에서는 특정 팩토리 내에서 여러 부품들에 대한 생성을 또 한번 팩토리로 캡슐화 하는 것이 추상 팩토리 패턴의 적용이고, 가구 예시에서는 각각의 부품(소파와 책상)  커스텀 구현 로직을 또 팩토리로 빼서 캡슐화 하지 않고 해당 팩토리 내에 구현했다는 것이다.

어떻게 구현하든 결국 위에서 짚어본대로 인터페이스 개수가 팩토리 메소드 패턴의 경우 하나, 추상 팩토리의 경우 여러개가 된다.



## 해결방안

문제 상황에 거의 해결책을 표현해놨긴 한데 다시 정리해보자. 결국 핵심은 클라이언트로부터 생성에 대한 관심을 제거하는 것이 주된 목적이고 추상 팩토리의 경우 생성 대상이 여러개가 존재한다.

따라서 아래와 같은 해결책이 있을 수 있다.

1. 하나의 팩토리를 두고 해당 팩토리가 여러 객체의 생성을 담당하는 또 다른 팩토리에 그 객체들의 생성을 일임한다.(이 팩토리에서도 각 객체들의 생성에 대한 처리가 디커플링 되는 것)
2. 하나의 팩토리를 두고 해당 팩토리가 여러 객체의 생성 로직을 캡슐화 한다.

1번의 상황이 강의에서 소개 및 구현된 추상 팩토리 패턴이다.

<figure><img src="../../../.gitbook/assets/image (85).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (81).png" alt=""><figcaption></figcaption></figure>



2번 상황이 [이 사이트](https://refactoring.guru/ko/design-patterns/abstract-factory)에 소개된 추상 팩토리 패턴의 구현이다.

<figure><img src="../../../.gitbook/assets/image (79).png" alt=""><figcaption></figcaption></figure>



## [실습코드](https://github.com/fistkim101/design-pattern-java)

일단 바로 위 예시 중 1번 상황에 대한 완성된 결과물 부터 보자.

<figure><img src="../../../.gitbook/assets/image (82).png" alt=""><figcaption></figcaption></figure>

```java
public class Client {
    public static void main(String[] args) {
        ShipPartsFactory blackShipPartsFactory = new BlackShipPartsFactory();
        ShipFactory blackShipFactory = new BlackShipFactory(blackShipPartsFactory);
        Ship blackShip = blackShipFactory.makeShip();
        System.out.println(blackShip.toString());
        blackShip.speedUp();

        ShipPartsFactory whiteShipPartsFactory = new WhiteShipPartsFactory();
        ShipFactory whiteShipFactory = new WhiteShipFactory(whiteShipPartsFactory);
        Ship whiteShip = whiteShipFactory.makeShip();
        System.out.println(whiteShip.toString());
        whiteShip.speedUp();
    }
}
```

```
Ship{wheel=com.fistkim.designpatternjava.creation.abstract_factory.version1.after.BlackWheel@87aac27, anchor=com.fistkim.designpatternjava.creation.abstract_factory.version1.after.BlackAnchor@3e3abc88}
----- speed up BlackShip
Ship{wheel=com.fistkim.designpatternjava.creation.abstract_factory.version1.after.WhiteWheel@eed1f14, anchor=com.fistkim.designpatternjava.creation.abstract_factory.version1.after.WhiteAnchor@7229724f}
----- speed up WhiteShip
```



인터페이스인 각 팩토리의 정의를 보자. ShipFactory 가 ShipPartsFactory 를 사용할 것이다.

```java
public interface ShipPartsFactory {

    Wheel makeWheel();

    Anchor makeAnchor();

}

public interface ShipFactory {
    Ship makeShip();
}
```



예시로 White 만 보면 아래와 같다. WhiteShipFactory  는 WhiteShipPartsFactory 를 사용하게 된다.

```java
public class WhiteShipFactory implements ShipFactory {

    private final ShipPartsFactory shipPartsFactory;

    public WhiteShipFactory(ShipPartsFactory shipPartsFactory) {
        this.shipPartsFactory = shipPartsFactory;
    }

    @Override
    public Ship makeShip() {
        Wheel wheel = shipPartsFactory.makeWheel();
        Anchor anchor = shipPartsFactory.makeAnchor();
        return new WhiteShip(wheel, anchor);
    }
}

public class WhiteShipPartsFactory implements ShipPartsFactory {
    @Override
    public Wheel makeWheel() {
        return new WhiteWheel();
    }

    @Override
    public Anchor makeAnchor() {
        return new WhiteAnchor();
    }
}
```

만약 White와 Black 말고 GreenShip 이 추가가 된다면 GreenShipPartsFactory와 GreenShipFactory 만 만들어주면 된다. GreenShip 생성과 이 부품들의 생성에 대한 구체적인 처리는 모두 GreenShipFactory 와 GreenShipPartsFactory에 책임을 부여해서 거기서 처리해주면 된다. 클라이언트는 단지 GreenShip 가 필요한 경우 GreenShipFactory 를 사용해서 GreenShip을 만들기만 하면 된다.

위에서 든 상황 2를 여기 적용하자면 ShipFactory 가 ShipPartsFactory 에 의존해서 부품들을 만드는 것이 아니라 각 부품의 생성에 대해서 ShipFactory 가 직접 관여해서 처리하는 것이다.

상황1이든 상황2든 클라이언트가 각 개별 객체의 생성에 일체 관여하지 않고 이를 Factory 객체에 위임해서 처리하고 클라이언트는 단지 Factory 가 어떻게 만드는진 모르겠지만 Factory 가 만들어 준걸 쓰기만 하면 되도록 하는 것이 핵심이다.
