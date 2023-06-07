# StatefulWidget 생명주기

flutter basic 페이지에 이미 정리해두긴 했는데 중요한 내용이라 subPage 를 따로 만들어서 중복 정리한다.

## StatelessWidget vs StatefulWidget <a href="#statelesswidget-vs-statefulwidget" id="statelesswidget-vs-statefulwidget"></a>

### StatelessWidget

StatelessWidget 은 프레임워크에 그 어떤 것도 알리지 않는다. 프레임워크에서 위젯에 언제 리빌드 해야하는지 알려준다. 철저하게 외부(=프레임워크)에 의해서 생명이 결정(‘생명이 결정’ 된다는 말은 위젯의 생명주기가 수동적으로 발생한다는 의미이다.)된다.

수동적으로 철저히 정의된 기능을 수행한다. 즉, 상태가 없는 위젯으로서 새로운 정보에 반응(react)한다.

### StatefulWidget

StatefulWidget 은 항상 State 객체를 갖는다. State 객체는 setState()란는 메서드를 갖고 있는데, 위젯을 다시 그려야함을 플러터에 알리는 기능을 수행한다. 이 포인트에서 StatelessWidget 와 크게 구별된다. StatelessWidget 는 프레임워크에 의사표현을 할 수 없는데 반해, StatefulWidget 는 언제 위젯을 다시 그려야하는지에 대해서 프레임워크에 위젯이 발언권을 가지고 있는 것이다.

setState() 이 발생할 경우 해당 위젯 뿐만 아니라 해당 위젯에 의존하는 모든 위젯을 다시 그려야한다고 프레임워크에 지시한다.

## StatefulWidget 위젯 생명 주기 <a href="#statefulwidget" id="statefulwidget"></a>

![](https://fistkim101.github.io/images/concept-flutter-stateful-lifecycle.png)

출처: https://betterprogramming.pub/stateful-widget-lifecycle-a01c44dc89b0

[StatefulWidget 위젯 생명 주기 이미지 출처이기도 한 포스팅](https://betterprogramming.pub/stateful-widget-lifecycle-a01c44dc89b0)에 아주 자세히 각 생명주기 메서드에 대해 정리가 되어 있다. 이 중 didUpdateWidget(), setState() 이 같은 계층에 있어 조금 더 확실하게 구분 해두면 좋을 것 같아서 따로 아래에 적어둔다.

> **didUpdateWidget()**\
> Gets called if the parent widget changes its configuration and has to rebuild the widget.\
> The framework gives you the old widget as an argument that you can use to compare it with the new widget.\
> Flutter will call the build() method after it.\
> _Tip: Use this method if you need to compare the new widget to the old one._

```dart
@override
void didUpdateWidget(covariant MyHomePage oldWidget) {
  super.didUpdateWidget(oldWidget);
  // TODO: implement didUpdateWidget
}
```

> **setState()**\
> This method is often called from the Flutter framework itself and from the developer.\
> The setState() method notifies the framework that the internal state of the current object is “dirty”,\
> which means that it has been changed in a way that might impact the UI.\
> After this notification, the framework will call the build() method to update and rebuild a widget.\
> _Tip: Whenever you change the internal state of a State object, make that change in the setState() method._

```dart
setState(() {
        // implement setState
        });
```

