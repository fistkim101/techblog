# (+) Data Layer - response to model

별 내용은 없는데 일반적으로 자주 쓰이는 형태라서 정리를 해둔다. API클라이언트를 이용해서 서버와 통신을 하고 response 를 수신 받게 되면 이를 가지고 내부에서 사용하는 model 로 컨버팅을 해줘야 하는데 이 때 일반적으로 처리하는 방식에 대한 정리이다.



## 1. 의존성으로 [build\_runner](https://pub.dev/packages/build\_runner) 를 추가한다.

build\_runner 는 코드 생성 및 빌드 관련 작업을 편리하게 해주는 툴이다. 여기서 추가해주는 이유는 어노테이션을 기반으로 응답으로 받아온 데이터를 모델에 매핑해주는 과정에 필요한 코드를 자동 생성해주기 위함이다.

> `build_runner`는 플러터(Flutter) 프로젝트에서 코드 생성과 빌드 관련 작업을 자동화하기 위한 도구입니다. 일반적으로 코드 생성 작업은 반복적이고 번거로운 작업일 수 있습니다. `build_runner`는 이러한 작업을 자동화하여 개발자가 더 효율적으로 작업할 수 있도록 도와줍니다.
>
> `build_runner`는 주로 플러터의 코드 생성과 관련된 작업에 사용됩니다. 예를 들어, 애노테이션(annotation)을 기반으로 모델 클래스, 라우트 설정, 리소스 파일 등의 코드를 자동으로 생성할 수 있습니다. 이를 통해 개발자는 반복적인 작업을 수동으로 처리하지 않고, 자동화된 방식으로 코드를 생성하고 업데이트할 수 있습니다.
>
> `build_runner`는 `build.yaml` 파일을 통해 구성되며, 주요 명령어는 다음과 같습니다:
>
> * `build_runner build`: 코드를 생성하고 빌드합니다.
> * `build_runner watch`: 코드를 생성하고 변경 사항을 감지하여 자동으로 빌드합니다.
> * `build_runner clean`: 이전 빌드에서 생성된 코드를 삭제합니다.
>
> `build_runner`는 주로 코드 생성, 리소스 번들링, 다국어 지원 등과 관련된 작업에 사용됩니다. 코드 생성 도구인 `Freezed`, `json_serializable`, `injectable` 등과 함께 사용하여 플러터 애플리케이션의 개발 생산성을 향상시킬 수 있습니다.



## 2. 의존성으로 [json\_serializable](https://pub.dev/packages/json\_serializable), [json\_annotation ](https://pub.dev/packages/json\_annotation)을 추가한다.

json\_serializable 은응답으로 받아온 데이터 json 에서 모델로 컨버팅을 하는 함수를 자동으로 생성해주기 위해서&#x20;

```
@JsonSerializable()
```

을 사용할 것인데 이 때 필요한 의존성이다. 사용법은 [공식문서](https://pub.dev/packages/json\_serializable)에 잘 나와있는데 아래에 짧게 정리한다. 그리고 json\_serializable 을 사용하기 위해서 json\_annotation 도 의존성으로 추가해 준다.



## 3. build\_runner 를 이용해서 build 명령어를 수행한다.

아래와 같이 @JsonSerializable() 을 달아주고 build\_runner 로 명령어를 수행하면 원하는 코드가 생성이 되지 않는다.

```dart
import 'package:json_annotation/json_annotation.dart';

@JsonSerializable()
class SignInResponse {
  final String accessToken;
  final String refreshToken;
  final bool isProfileCompleted;

  const SignInResponse({
    required this.accessToken,
    required this.refreshToken,
    required this.isProfileCompleted,
  });

}
```

```
sign_in_response.g.dart must be included as a part directive in the input library with:
    part 'sign_in_response.g.dart';
```

그래서 part 'sign\_in\_response.g.dart'; 를 아래와 같이 추가해주고 아래 명령어를 실행해야 .g 파일이 만들어진다.

```dart
import 'package:json_annotation/json_annotation.dart';
part 'sign_in_response.g.dart';

@JsonSerializable()
class SignInResponse {
  final String accessToken;
  final String refreshToken;
  final bool isProfileCompleted;

  const SignInResponse({
    required this.accessToken,
    required this.refreshToken,
    required this.isProfileCompleted,
  });

}
```

```dart
// GENERATED CODE - DO NOT MODIFY BY HAND

part of 'sign_in_response.dart';

// **************************************************************************
// JsonSerializableGenerator
// **************************************************************************

SignInResponse _$SignInResponseFromJson(Map<String, dynamic> json) =>
    SignInResponse(
      accessToken: json['accessToken'] as String,
      refreshToken: json['refreshToken'] as String,
      isProfileCompleted: json['isProfileCompleted'] as bool,
    );

Map<String, dynamic> _$SignInResponseToJson(SignInResponse instance) =>
    <String, dynamic>{
      'accessToken': instance.accessToken,
      'refreshToken': instance.refreshToken,
      'isProfileCompleted': instance.isProfileCompleted,
    };

```

```bash
dart run build_runner build
```

자동 생성된 코드를 사용해서 아래와 같이 factory 메서드를 선언해주자.

```dart
import 'package:json_annotation/json_annotation.dart';

part 'sign_in_response.g.dart';

@JsonSerializable()
class SignInResponse {
  final String accessToken;
  final String refreshToken;
  final bool isProfileCompleted;

  const SignInResponse({
    required this.accessToken,
    required this.refreshToken,
    required this.isProfileCompleted,
  });

  factory SignInResponse.fromJson(Map<String, dynamic> json) {
    return _$SignInResponseFromJson(json);
  }
}
```
