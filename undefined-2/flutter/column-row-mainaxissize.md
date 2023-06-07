# Column 과 Row 의 MainAxisSize

Column 은 레이아웃 위젯이므로 다른 위젯과의 상관관계를 생각해야한다. 결국 MainAxisSize 라는 프로퍼티의 속성으로 정해지는 것인데, 기본값이 max다.

```dart
  Flex({
    super.key,
    required this.direction,
    this.mainAxisAlignment = MainAxisAlignment.start,
    this.mainAxisSize = MainAxisSize.max,
    this.crossAxisAlignment = CrossAxisAlignment.center,
    this.textDirection,
    this.verticalDirection = VerticalDirection.down,
    this.textBaseline, // NO DEFAULT: we don't know what the text's baseline should be
    this.clipBehavior = Clip.none,
    super.children,
  })
```

```dart
  /// Maximize the amount of free space along the main axis, subject to the
  /// incoming layout constraints.
  ///
  /// If the incoming layout constraints have a small enough
  /// [BoxConstraints.maxWidth] or [BoxConstraints.maxHeight], there might still
  /// be no free space.
  ///
  /// If the incoming layout constraints are unbounded, the [RenderFlex] will
  /// assert, because there would be infinite remaining free space and boxes
  /// cannot be given infinite size.
```

free space 를 모두 차지하는 것이다. 기본적으로 shrink 가 아니다. 내부 위젯들을 딱 감싸려면 min 옵션으로 설정해줘야한다.

{% hint style="info" %}
Column, Row 를 사용할때 아래 세 가지는 기본값 쓰지말고 무조건 명시해주자.\
mainAxisAlignment, crossAxisAlignment, mainAxisSize
{% endhint %}
