# environment

## environment 의 책임

변수명에서 추론할 수 있듯이 환경 분리를 위해 만든 객체다. 핵심은 앱의 진입시 사용되는 main 에서 enum 으로 타입을 정적으로 구분해서 사용하여 원하는 환경의 앱을 실행 시키도록 한다.

달리 말해, 엔트리 포인트에 코드를 아예 정해두고 엔트리 포인트를 원하는 환경에 따라 다르게 설정하여 앱을 빌드하는 것이다.



## 모바일에서의 환경분리는 무엇을 의미하는가

서버에서 환경 분리의 목적과 같다. 큰 틀에서 결국 원하는 환경에 따라 설정값이 달라지는 것이다. 스프링으로 치자면 프로퍼티를 구분해주는 것과 같다.

구체적으로 예를 들자면 앱도 당연히 환경마다 바라봐야할 엔드포인트가 다를 것이고, 사용해야할 외부 서비스에 대한 토큰 값 등도 다를 수 있다. 이런 모든 요소들을 동일한 코드베이스에서 환경마다 다르게 세팅할 수 있도록 하는 것이 '환경분리' 라는 행위의 목적이다.

이러한 맥락에서 environment 객체는 정해진 환경에 따라 다른 설정값을 가지는 configuration 성격의 정적 객체(객체간 커뮤니케이션을 한다기보다 일방적으로 설정값을 제공해주는) 라고 볼 수 있다.



## 패키지 구조

현재 템플릿 코드의 초반이라서 다른 레이어들도 들어갔는데 application > environments 만 보면 된다.

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>



## environment 구현 설명

#### local.dart

```dart
import '../../barrel.dart';
import 'environment.dart';

void main() {
  final Environment environment =
      Environment.newInstance(EnvironmentType.LOCAL);
  environment.run();
}
```

```dart
enum EnvironmentType {
  LOCAL,
  DEVELOPMENT,
  PRODUCTION,
}
```

위와 같이 각 엔트리 포인트(local.dart, development.dart, production.dart) 마다 enum 으로 환경을 다르게 설정해두고 엔트리 포인트를 선택해서 빌드하는 원리이다.



```dart
import 'package:flutter/material.dart';

import '../../barrel.dart';

class Environment {
  Environment._(this._environmentType);

  static Environment? _instance;
  final EnvironmentType _environmentType;

  static get isLocal => _instance!._environmentType == EnvironmentType.LOCAL;

  static get isDevelopment =>
      _instance!._environmentType == EnvironmentType.DEVELOPMENT;

  static get isProduction =>
      _instance!._environmentType == EnvironmentType.PRODUCTION;

  static newInstance(EnvironmentType environmentType) {
    _instance ??= Environment._(environmentType);
    return _instance!;
  }

  static get baseUrl {
    switch (_instance!._environmentType) {
      case EnvironmentType.LOCAL:
        return 'http://localhost:8080';
      case EnvironmentType.DEVELOPMENT:
        return 'https://dev.example.com';
      case EnvironmentType.PRODUCTION:
        return 'https://example.com';
      default:
        return 'http://localhost:8080';
    }
  }

  run() {
    runApp(const AppName());
  }
}

```

전형적인 싱글톤 패턴이긴 한데 멀티 스레드가 감안되어 있지는 않다. 왜냐면 감안할 필요가 없기 때문이다. 앱 실행시 main 이 최초로 단건으로 실행 되는 것이므로 newInstance() 에 두 스레드가 같이 진입할 일은 없다.

여기서 다국어 처리나 플러터 엔진에 대한 바인딩 초기화 등을 해줘야하는데 이건 그걸 정리할때 같이 정리하는 편이 좋을 것 같다. 왜냐하면 지금은 environment 의 용도만 구현된 코드가 있는 것이 더 내용의 요지가 드러나는 데에 좋을 것 같아서이다.
