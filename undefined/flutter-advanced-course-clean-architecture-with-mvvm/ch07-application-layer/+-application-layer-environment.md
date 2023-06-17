# (+) Application Layer - environment

## Environment 의 필요성

같은 종류이지만 빌드 환경에 따라서 값이 달라져야하는 것들이 있다. 가장 큰 예로 개발 서버, 운영 서버의 엔드포인트가 있다. 하지만 이를 위해서 프로덕션 코드와 개발 코드를 구분해서 개발하는 것은 잘못된 개발이다. 모든 테스트와 개발은 프로덕션을 기준으로 작동하고 구현되어야한다.

따라서 코드 내에서 프로덕션 코드와 개발 코드가 구분되지 않고 일관되게 프로덕션 코드로 구현되도록 하되 빌드 환경에 따라 달라져야할 값들은 다르게 사용 되도록 해야한다. 이를 위해서 환경에 대해서만 분기 처리를 하고 그 외에는 모두 동일한 프로덕션 코드에서 환경 별로 다른 값을 갖고 동작하도록 처리해야하는데 이를 Environment 가 처리하도록 했다.

VM 옵션을 활용해도 될 것 같았는데 일단 앱 기동시 진입 지점을 아예 다르게 설정하여 코드 내에 enum 으로 다르게 처리해두는 방식을 취했다.

## 패키지 구성 관점에서 바라본 Environment

![](<../../../.gitbook/assets/image (29) (1).png>)

## Environment 구현

#### development.dart(production.dart 도 동일한 방식)

```dart
void main() {
  final Environment environment =
      Environment.newInstance(EnvironmentType.development);
  environment.run();
}
```

#### environment.dart

```dart
import 'package:app_frame_sample/app.dart';
import 'package:flutter/cupertino.dart';

import '../application.dart';

class Environment {
  static Environment? _instance;
  final EnvironmentType environmentType;

  const Environment._internal(this.environmentType);

  static bool get _isDevelopment {
    return _instance!.environmentType == EnvironmentType.development;
  }

  static bool get _isProduction {
    return _instance!.environmentType == EnvironmentType.production;
  }

  static String get baseUrl {
    if (_isDevelopment) {
      return 'http://localhost:8080';
    }

    if (_isProduction) {
      return 'http://localhost:8090';
    }

    throw UnsupportedError(ErrorMessage.notFoundEnvironmentType);
  }

  factory Environment.newInstance(EnvironmentType environmentType) {
    _instance ??= Environment._internal(environmentType);
    return _instance!;
  }

  void run() async {
    WidgetsFlutterBinding.ensureInitialized();
    await Injector.prepareDependencies();
    runApp(const Application());
  }
}
```
