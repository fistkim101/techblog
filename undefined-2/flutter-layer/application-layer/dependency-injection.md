# dependency injection

## 앱에서의 의존성 주입

사실 여기서 의존성 주입이라는 단어를 써도 될지 개인적으로 애매하다는 생각은 든다. 왜냐하면 의존성 주입은 사용하는 쪽에서 필요한 것을 생성하여 사용하도록 하는 것이 아니라 받은대로 사용하게 되는 '제어의 역전' 이라는 현상을 발생시키는 행위라고 생각한다. 그리고 이 행위의 전제는 추상화된 인터페이스 아래에서 여러 구현체가 경우에 따라 다른 구현체가 필요한 상황이라는 것이다.

물론 이것 외에도 효율성 측면에서 두 개 이상의 객체가 필요 없기에 싱글톤 으로 하나만 만들어놓고 계속 재사용을 한다는 경제성 측면도 있긴 하지만, 스프링에서의 의존성 주입은 제어의 역전을 통해서 테스트도 용이하고 설계상 더 유연한 설계(객체간의 유연한 협력 구조)를 달성하기 위함인 측면이 더 큰 것 같다.

그런데 앱에서의 의존성 주입은 사실 싱글톤으로 만들어진 하나의 객체에 대한 수월한 사용이 가장 큰 사용 목적인 것 같아서 사실 이걸 의존성 주입이라고 표현하기 보다는 '누구나 접근하기 쉬운 최상단 위젯' 이라는 표현이 적절하지 않을까 싶다.



## [get\_it 패키지](https://pub.dev/packages/get\_it)

의존성 주입을 쉽게 할 수 있도록 도와주는 패키지이다.&#x20;

> This is a simple **Service Locator** for Dart and Flutter projects with some additional goodies highly inspired by [Splat](https://github.com/reactiveui/splat). It can be used instead of `InheritedWidget` or `Provider` to access objects e.g. from your UI.
>
> Typical usage:
>
> * Accessing service objects like REST API clients or databases so that they easily can be mocked.
> * Accessing View/AppModels/Managers/BLoCs from Flutter Views

InheritedWidget 과 Provider 모두 핵심 원리는 context 참조로 위젯트리를 탐색하여 쭉 올라가서 원하는 위젯에 access 하는 것이며 get\_it 역시 비슷한 원리로 만들어져 있다. 공식 문서를 보면 아래와 같이 자랑하고 있다.

> * Extremely fast (O(1))
> * Easy to learn/use
> * Doesn't clutter your UI tree with special Widgets to access your data like, Provider or Redux does.

확실히 Provider 보다는 참조하는 코드가 조금 더 간결하다고 느꼈다.



## get\_it 을 이용한 의존성 주입 구현

패키지를 일단 설치한다.

```bash
flutter pub add get_it
```



글로벌하게 참조되어 사용되므로 dependency\_injection은 application layer 에 위치한다.

<figure><img src="../../../.gitbook/assets/image (119).png" alt=""><figcaption></figcaption></figure>

```dart
import 'package:device_info_plus/device_info_plus.dart';
import 'package:get_it/get_it.dart';

class Injector {
  Injector._();

  static DeviceInfoPlugin get deviceInfoPlugin =>
      GetIt.instance.get<DeviceInfoPlugin>();

  static Future registerDependencies() async {
    _registerUtils();
    _registerNetworks();
    _registerRepositories();
  }

  static _registerUtils() async {
    GetIt.instance
        .registerLazySingleton<DeviceInfoPlugin>(() => DeviceInfoPlugin());
  }

  static _registerNetworks() async {}

  static _registerRepositories() async {}
}

```

1. 기본 생성자를 private 처리해서 외부에서 Injector 를 생성하지 못하게 했다.
2. registerDependencies() 함수 내에서 용도별로 구분된 메소드를 다시 호출하게 해서 코드 가독성을 높혔다. 단순히 나열식으로 필요한걸 쭉 등록하는게 아니라 어떤 디펜던시가 무슨 용도인지를 쉽게 파악하게 했다는 의미이다.
3. getter 를 선언해줘야 다른 위젯에서 참조 할 수 있다.
4. 위 코드는 레이지로딩 방식으로 구현 한 것인데 레이지하지 않게 바로 즉시 등록시키는 방법도 있다. [공식 문서](https://pub.dev/packages/get\_it)에 자세히 나와있고 정리할 가치가 있을 만큼 복잡하진 않아서 생략한다.
5. 일단 샘플 구현을 위해서 어떤 프로젝트든 내가 쓸 것으로 예상되는 [device\_info\_plus](https://pub.dev/packages/device\_info\_plus)를 등록했다.



```dart
  run() async {
    await Injector.registerDependencies();
    runApp(const AppName());
  }
```

environment 에서 구현했던 run() 함수에서 runApp() 을 하기 전에 위와 같이 처리해준다. 저 시점에 타입을 기반으로 객체를 생성 및 생성 준비(아마도 프록시 패턴으로 일단 등록을 해놓고 내부 참조 변수를 비워뒀다가 최초 참조시 메모리에 할당해주는 방식일 것 같은데 더 들여다 보진 않았다)을 해놓을 것으로 생각 된다.(get\_it 의 내부 구동 원리가 이것으로 추측되는데 학습의 가성비를 위해 여기서 일단 호기심을 끊는다.)



```dart
  Future<void> _showPlatformInfo() async {
    final DeviceInfoPlugin deviceInfoPlugin = Injector.deviceInfoPlugin;
    if (Platform.isAndroid) {
      final AndroidDeviceInfo androidInfo = await deviceInfoPlugin.androidInfo;
      print(androidInfo.toString());
    }

    if (Platform.isIOS) {
      final IosDeviceInfo iosDeviceInfo = await deviceInfoPlugin.iosInfo;
      print(iosDeviceInfo.toString());
    }
  }
```

사용은 위와 같이 선언해둔 getter 를 통해서 access 해서 사용한다.
