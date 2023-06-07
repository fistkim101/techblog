# Flutter 가 위젯을 그리는 원리

## 참고자료 <a href="#undefined" id="undefined"></a>

{% embed url="https://youtu.be/996ZgFRENMs" %}

## Widget Tree, Element Tree, Render Object Tree 정리 <a href="#undefined" id="undefined"></a>

Flutter에서는 위젯 트리 (Widget Tree), 엘리먼트 트리 (Element Tree), 렌더 오브젝트 트리 (Render Object Tree)라는 세 가지 트리 개념을 사용하여 UI를 구성한다. 각각의 트리는 Flutter의 다른 단계에서 사용되며, UI의 구조와 렌더링을 관리하는 역할을 한다.

아래에서 다룰 것이지만 세 트리간 참조관계를 나타내는 이미지를 첨부한다.

![](https://fistkim101.github.io/images/flutter-three-tree.png)

1. 위젯 트리 (Widget Tree):
   * 위젯 트리는 Flutter 앱의 UI 구조를 나타내는 위젯들의 계층 구조이다. (위젯의 Configuration 이라고 보면 된다)
   * 위젯 트리는 위젯들을 포함하는 부모-자식 관계를 표현한다.
   * 위젯 트리는 불변성을 가지며, 위젯들은 재사용 가능(const)하다. 생성 비용이 저렴하다.
   * 위젯 트리는 위젯을 구성하고, 엘리먼트 트리를 생성하기 위한 기반이 된다.
2. 엘리먼트 트리 (Element Tree):
   * 엘리먼트 트리는 위젯 트리를 기반으로 생성되는 가변적인 개체.
   * 엘리먼트는 위젯과 연결된 실행 및 라이프사이클 정보를 가지고 있다.
   * 엘리먼트 트리는 위젯 트리와 거의 동일한 구조를 가지지만, 위젯 트리보다 세밀한 제어를 가능하게 한다.
   * 엘리먼트 트리는 위젯의 변경 사항을 추적하고, 필요한 경우 UI를 업데이트한다.
3. 렌더 오브젝트 트리 (Render Object Tree):
   * 렌더 오브젝트 트리는 엘리먼트 트리에서 실제 화면에 그려지는 렌더링 정보를 가지는 개체이다.
   * 렌더 오브젝트는 화면에 표시되는 위젯의 실제 픽셀 위치와 속성을 정의한다.
   * 렌더 오브젝트 트리는 엘리먼트 트리와 유사한 구조를 가지지만, 실제로 화면에 그려지는 렌더링 정보를 포함한다.
   * 렌더 오브젝트 트리는 GPU와 통신하여 실제 화면에 UI를 그린다.

이러한 트리들은 Flutter의 내부 작동 방식을 나타내며, UI의 구조를 관리하고 업데이트하는 데 중요한 역할을 한다.

위젯 트리는 앱의 UI 구성을 정의하고, 엘리먼트 트리는 위젯의 변경과 라이프사이클을 관리하며, 렌더 오브젝트 트리는 화면에 실제로 그려지는 UI를 정의한다.

## Flutter가 위젯을 그리는 원리 <a href="#undefined" id="undefined"></a>

![](https://fistkim101.github.io/images/how-flutter-render-widget-01.png)

> A widget is an immutable description of part of a user interface. widget describes the configuration for an element.

widget은 불변이다. widget 이 그림과 같이 교체될 수는 있지만 노드(위젯) 단위로 보면 불변이다. widget은 element 에 대한 configuration 이다.

{% hint style="info" %}
Element: an instantiation of a Widget at a particular location in the tree.
{% endhint %}

{% hint style="info" %}
RenderObject: handles size, layout, painting.
{% endhint %}

flutter가 UI 를 렌더링 하는 것은 widget을 바라보는게 아니라 RenderObject 를 바라보는 것이다.

&#x20;

<figure><img src="https://fistkim101.github.io/images/how-flutter-render-widget-02.png" alt=""><figcaption></figcaption></figure>

간단한 샘플로 어떻게 flutter가 최초로 렌더링을 하는지 그 과정을 살펴본다. 세 tree 를 어떤 순서로 어떤 과정에 의해서 만들게 되는지 보는게 포인트다.\
&#x20;

<figure><img src="https://fistkim101.github.io/images/how-flutter-render-widget-03.png" alt=""><figcaption></figcaption></figure>

먼저 widget tree 를 완성한다.\
&#x20;&#x20;

<figure><img src="https://fistkim101.github.io/images/how-flutter-render-widget-04.png" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/how-flutter-render-widget-05.png" alt=""><figcaption></figcaption></figure>

그리고 element tree 를 widget 을 configuration 삼아 만든다.\


<figure><img src="https://fistkim101.github.io/images/how-flutter-render-widget-06.png" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/how-flutter-render-widget-07.png" alt=""><figcaption></figcaption></figure>

renderObject 를 만든다.\
\
여기서 widget 이 바뀌면 어떻게 되는지 살펴본다.

<figure><img src="https://fistkim101.github.io/images/how-flutter-render-widget-08.png" alt=""><figcaption></figcaption></figure>

위와 같은 케이스때 tree 들의 참조관계가 어떻게 변하는지 살펴보자.&#x20;

<figure><img src="https://fistkim101.github.io/images/how-flutter-render-widget-09.png" alt=""><figcaption></figcaption></figure>

위에서 쭉 설명한 순서로 세 tree가 만들어지고 두 번째 runApp 이 실행된 단계다.&#x20;

<figure><img src="https://fistkim101.github.io/images/how-flutter-render-widget-10.png" alt=""><figcaption></figcaption></figure>

flutter는 이 때 위 로직에 의해서 element update 를 판단한다. 같은 element 를 사용할지에 관해서 판단하게 된다.

<figure><img src="https://fistkim101.github.io/images/how-flutter-render-widget-11.png" alt=""><figcaption></figcaption></figure>

&#x20;다음으로 renderObject 를 업데이트 하게 된다.

<figure><img src="https://fistkim101.github.io/images/how-flutter-render-widget-12.png" alt=""><figcaption></figcaption></figure>

최종적으로 위와 같이 render가 된다.

책에 따르면 렌더 객체는 ‘복잡하고 비싸서’ 플러터 내부적으로 비싼 렌더 객체를 재사용하면서 동시에 비싸지 않은 위젯을 마음껏 파괴해서 결과적으로 성능 최적화를 이뤄낸다.

## 세 tree 간 참조 관계 <a href="#tree" id="tree"></a>

![](https://fistkim101.github.io/images/flutter-three-tree.png)

element 는 widget 을 참조하고, element 가 renderObject 를 만든다. renderObject 는 dart:ui 를 이용해서 화면을 그린다.

## element 가 상태 객체를 관리한다 <a href="#element" id="element"></a>

&#x20;말 그대로 element가 상태 객체를 관리한다.

<figure><img src="https://fistkim101.github.io/images/flutter-element-manage-state.png" alt=""><figcaption></figcaption></figure>

그래서 위 그림에서 widget tree 내 A, B의 위치를 바꿔도 element tree 의 참조는 변하지 않는다. 참조를 갱신하는 기준은 ‘런타임의 형식’, ‘위젯의 키’ 인데 둘이 변하지 않았다면 참조를 갱신하지 않는다.&#x20;

<figure><img src="https://fistkim101.github.io/images/how-flutter-render-widget-10.png" alt=""><figcaption></figcaption></figure>

runTimeType 이 클래스라고 (지금으로서는) 확인이 된다. 이 예제에서 아래와 같이 FancyButtonSecond 로 클래스를 구분해주니 의도대로 색깔이 같이 바뀌는 것을 확인할 수 있었다.

```dart
    final incrementButton = FancyButton(
      child: Text(
        "Increment",
        style: TextStyle(color: Colors.white),
      ),
      onPressed: _incrementCounter,
    );

    final decrementButton = FancyButtonSecond(
      child: Text(
        "Decrement",
        style: TextStyle(color: Colors.white),
      ),
      onPressed: _decrementCounter,
    );
```

