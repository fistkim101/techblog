# 팩토리 메소드 패턴

## 대분류

객체 생성



## 문제상황

Creator 가 만들어야 할 대상이 동일한 논리적 단위에 속하지만 구체적으로는 여러 갈래로 나뉘고, 이것이 확장될 가능성이 있는 상황.

예를 들어 피자 가게에서 피자를 만들어야 하는데 현재 메뉴 라인업이 페페로니 피자, 불고기 피자만 있는 상황이라 한다면 큰 틀에서 피자라는 논리적 단위에 속하지만 구체적으로는 저렇게 나뉘게 되고 구체적으로 다른 객체를 생성해야 하기 때문에 Creator 에서 생성에 관심을 둬야한다.

문제는 확장이 되면 점점 생성에 관한 복잡한 처리(분기문 등)가 더 늘어나게 되고 변경에 취약하고 가독성이 떨어지는 코드가 생겨나게 된다.



## 해결방안

클라이언트(Creator) 에게서 생성에 대한 관심사를 분리시켜서 변경에 닫혀있고 확장에 열려있도록(OCP 원칙을 준수) 하여 유연성과 확장성을 지니게 한다. 생성에 대한 관심사를 분리시킨다는 것은 곧 생성에 대한 로직을 다른 객체로 뺀다는 것을 의미한다.



<figure><img src="../../.gitbook/assets/image (9) (3).png" alt=""><figcaption></figcaption></figure>

위  UML 에 기반하여 코드레벨에서 설명하자면 예전에는 클라이언트가 각각의 구분된 Product 를 생성했다면 팩토리 메소드 패턴 적용시 큰 논리적 단위인 Product 를 구체적인 Creator 에 위임해서 생성하는 것이다.

강의 노트에 실린 UML 을 다시 보자.

<figure><img src="../../.gitbook/assets/image (12) (2).png" alt=""><figcaption></figcaption></figure>

팩토리 메소드 패턴 적용 전에는 ConcreteCreator(피자 가게)가 ConcreteProduct(페페로니 피자, 불고기 피자 등 구분된 객체 타입)을 각각 생성했었다.

하지만 팩토리 메소드 패턴을 적용하면 생성에 대한 관심을 분리하기 위해서 Creator 역할의 인터페이스를 만들고 생성 대상 역시 Product 라는 논리적 단위로 동일하게 취급하기 위해서 Creator 처럼 인터페이스로 정의한다. 이렇게하면 클라이언트는 생성을 오직 Creator 라는 인터페이스 하나에만 위임할 수 있고, 만들어야 하는 ConcreteProduct 에 따라 적절한 ConcreteCreator를 사용할 수 있다.



## [실습코드](https://github.com/fistkim101/design-pattern-java)

일단 아주 raw 레벨의 상황을 가정하여 코드로 보면 아래와 같다.

```java
public class Pizza {

    private String name;
    private int price;

    public Pizza(String name, int price) {
        this.name = name;
        this.price = price;
    }
    
}

public class PizzaStore {

    public Pizza makePizza(String pizzaKind) throws IllegalAccessException {

        if (pizzaKind.equals("cheese")) {
            return new Pizza("cheese", 15000);
        }

        if (pizzaKind.equals("meat")) {
            return new Pizza("meat", 17000);
        }

        throw new IllegalAccessException();
    }

}
```

여기서 피자의 종류가 늘어나면 클라이언트인 PizzaStore 의 코드가 변경되고 늘어난다. 만들 객체의 종류가 늘어나거나 생성시 필요한 로직이 지금처럼 단순히 프로퍼티를 할당해주는 것이 아니라 뭔가 다른 로직같은 것(생성시 외부 API 호출이 필요하다던가)이 필요하면 그만큼 코드가 더 복잡해진다.



생성될 대상 객체를 인터페이스로 처리하면 조금 더 깔끔해질 수 있다.

```java
public interface Pizza {

    String name();

    int price();

}

public class CheesePizza implements Pizza{
    @Override
    public String name() {
        return "cheese";
    }

    @Override
    public int price() {
        return 15000;
    }
}

public class MeatPizza implements Pizza {
    @Override
    public String name() {
        return "meat";
    }

    @Override
    public int price() {
        return 17000;
    }
}

public class PizzaStore {

    public Pizza makePizza(String pizzaKind) throws IllegalAccessException {

        if (pizzaKind.equals("cheese")) {
            return new CheesePizza();
        }

        if (pizzaKind.equals("meat")) {
            return new MeatPizza();
        }

        throw new IllegalAccessException();
    }

}
```

생성 대상 객체를 인터페이스(추상 클래스로 처리해도 무관하다)로 만들고 각각이 구분되는 다른 객체에게 필요한 각자의 처리를 해당 객체들에게 위임했다. 이로써 해당 객체의 생성시 커스텀하게 처리해줘야할 로직을 클라이언트로부터 분리할 수 있다.



저기서 아래 코드와 같이 조금만 변경을 해주면 SimpleFactory method 패턴을 구현할 수 있다.

```java
public class SimplePizzaFactory {

    public Pizza makePizza(String pizzaKind) throws IllegalAccessException {
        if (pizzaKind.equals("cheese")) {
            return new CheesePizza();
        }

        if (pizzaKind.equals("meat")) {
            return new MeatPizza();
        }

        throw new IllegalAccessException();
    }

}

public class PizzaStore {

    SimplePizzaFactory simplePizzaFactory = new SimplePizzaFactory();

    public Pizza makePizza(String pizzaKind) throws IllegalAccessException {
        return simplePizzaFactory.makePizza(pizzaKind);
    }

}
```

클라이언트에게서 생성에 대한 관심을 완전하게 분리한 상태가 되었다.



강의에서 설명한 완전한 형태의 팩토리 메소드 패턴으로 구현하려면 각각의 피자 종류를 생성하는 Creator 가 각각 존재해야한다. 그렇게 각각 분리하면 각 객체를 생성할 때 필요한 처리들을 모두 책임으로 부여할 수 있다.



<figure><img src="../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

```java
public interface PizzaFactory {
    Pizza makePizza();

}

public class CheesePizzaFactory implements PizzaFactory {
    @Override
    public Pizza makePizza() {
        return new CheesePizza();
    }
}

public class MeatPizzaFactory implements PizzaFactory {
    @Override
    public Pizza makePizza() {
        return new MeatPizza();
    }
}

public interface Pizza {

    String name();

    int price();

}

public class CheesePizza implements Pizza{
    @Override
    public String name() {
        return "cheese";
    }

    @Override
    public int price() {
        return 15000;
    }

    @Override
    public String toString() {
        return "CheesePizza";
    }
}

public class MeatPizza implements Pizza {
    @Override
    public String name() {
        return "meat";
    }

    @Override
    public int price() {
        return 17000;
    }

    @Override
    public String toString() {
        return "MeatPizza";
    }
}

public class PizzaStore {

    final List<PizzaFactory> pizzaFactories = List.of(new CheesePizzaFactory(), new MeatPizzaFactory());

    public Pizza makePizza(String pizzaKind) {
        return this.pizzaFactoryFinder(pizzaKind).makePizza();
    }

    private PizzaFactory pizzaFactoryFinder(String pizzaKind) {
        if (pizzaKind.equals("cheese")) {
            return pizzaFactories.get(0);
        }

        if (pizzaKind.equals("meat")) {
            return pizzaFactories.get(1);
        }

        throw new IllegalArgumentException();
    }

}

```

피자 종류를 enum 으로 선언하면 더 깔끔한 코드가 될 것이다. 그리고 Factory 를 찾는 부분도 저렇게 인덱스 번호로 할 것이 아니고 클래스 명 같은 것으로 찾아주는게 좋을 것 같다. 특히 스프링에서 저러한 상황이 있다면 Factory 각각을 Bean 으로 지정하고 Bean 이름을 가지고 ApplicationContext 에서 찾는 형태로 가면 좋을 것 같다.

&#x20;하지만 지금은 팩토리 메소드 패턴이 어떤 것인지에 대한 이해가 주 목적이어서 이정도로 일단 구현했다. 핵심은 **생성을 책임지는 객체도 인터페이스로 선언하고 생성이 되는 생성 대상 객체도 인터페이스로 선언한 뒤, 생성할 각각의 구현체를 생성하는 것을 책임지는 각각의 Creator 를 구현하고 최종적으로 클라이언트는 생성의 구체적인 처리에 대해서 관심을 두지 않는 것이다.**

위 상황에서 새로운 특정한 피자가 추가가 된다고 가정해보자. 예를 들어 쉬림프 피자가 추가된다고 가정하자. 확장이 되어야하는 상황이다. 이 때 추가되어야 할 것은 쉬림프 피자라는 Pizza 라는 인터페이스를 따르는 구체적인 객체 타입을 추가해야할 것이고 이를 생성하는 책임을 가지는 PizzaFactory 인터페이스를 구현하는 쉬림프피자Factory 가 추가가 되어야 한다.

클라이언트 쪽에서는 단지 Factory 가 추가만 되면 된다. 이 부분이 핵심이다. 이렇기 때문에 변경에 닫혀있고 확장에는 열려 있다고 말할 수 있기 때문이다.
