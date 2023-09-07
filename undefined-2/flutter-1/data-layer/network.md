# network

## 용도

서버와 통신하기 위함이다. 통신을 할 때 기본적으로 헤더에 실어서 전달해야할 정보들도 있고, 인터셉터를 활용해서 서버의 응답에 따라 전처리가 필요한 경우도 있다. 이 모든 사항을 대응하기 위한 모듈이다.



## 구현

[dio](https://pub.dev/packages/dio) 를 사용한다.

```bash
$ flutter pub add dio
```

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

data 레이어 내에 network 말고 response, request, repository 를 둘 수 있다. 나중에 domain 영역에서도 정리하겠지만 repository 의 경우 domain 레이어에 인터페이스를 두고 data 레이어에서 구현체를 만들어서 사용하는 것이 엄격하게 클린 아키텍처를 구현하는 것이다. 하지만 프로젝트를 해보니 데이터 소싱 전략이 전체적으로 바뀐다던가 하지 않는 이상 불필요하게 계층을 나누는 행위라고 느껴졌다. 결론적으로 repository 는 data 레이어에만 바로 구현체를 두는 것이 좋은 것 같다.



```dart
import 'base_http_client.dart';

class ApiClient extends BaseHttpClient {
  ApiClient({
    required super.clientBaseUrl,
    required super.customInterceptors,
    super.customOptions,
  });
}
```

```dart
import 'package:dio/dio.dart';

import '../../../application/environments/environment.dart';

abstract class BaseHttpClient with DioMixin implements Dio {
  final String clientBaseUrl;
  final List<Interceptor> customInterceptors;
  BaseOptions? customOptions;

  BaseHttpClient({
    required this.clientBaseUrl,
    required this.customInterceptors,
    this.customOptions,
  }) : super() {
    if (customOptions != null) {
      options = customOptions!;
    }
    httpClientAdapter = HttpClientAdapter();
    options = BaseOptions(
      baseUrl: clientBaseUrl,
      connectTimeout: const Duration(seconds: 10),
      receiveTimeout: const Duration(seconds: 10),
    );

    if (Environment.isDevelopment) {
      interceptors.add(LogInterceptor(requestBody: true, responseBody: true));
    }
    interceptors.addAll(customInterceptors);
  }
}
```

위와 같이 작업해주면 클라이이언트 정의는 끝이 난다. 아래는 내가 사용한 인터셉터다. 기본적인 것들이다.



BaseHeaderInterceptor 는 헤더에 기본적으로 담을 내용들을 세팅한다.

```dart
import 'dart:io';

import 'package:device_info_plus/device_info_plus.dart';
import 'package:dio/dio.dart';
// import 'package:firebase_messaging/firebase_messaging.dart';
import 'package:package_info_plus/package_info_plus.dart';

import '../../../barrel.dart';

class BaseHeaderInterceptor extends InterceptorsWrapper {
  @override
  void onRequest(
      RequestOptions options, RequestInterceptorHandler handler) async {
    await _setHeaderWithAccessToken(options.headers);
    await _setHeaderWithPackageInfo(options.headers);
    await _setHeaderWithDeviceInfo(options.headers);
    // await _setHeaderWithPushToken(options.headers);

    super.onRequest(options, handler);
  }

  Future<void> _setHeaderWithAccessToken(Map<String, dynamic> headers) async {
    final String? accessToken = await AccessManager.getAccessToken();
    if (accessToken == null) {
      return;
    }

    const String ACCESS_TOKEN_PREFIX = 'Bearer';
    const String SPACE = ' ';
    headers['access-token'] = ACCESS_TOKEN_PREFIX + SPACE + accessToken;
  }

  Future<void> _setHeaderWithPackageInfo(Map<String, dynamic> headers) async {
    final PackageInfo packageInfo = await PackageInfo.fromPlatform();
    headers['app-name'] = packageInfo.appName;
    headers['package-name'] = packageInfo.packageName;
    headers['version'] = packageInfo.version;
    headers['build-number'] = packageInfo.buildNumber;
  }

  Future<void> _setHeaderWithDeviceInfo(Map<String, dynamic> headers) async {
    final DeviceInfoPlugin deviceInfo = Injector.deviceInfoPlugin;
    if (Platform.isAndroid) {
      final AndroidDeviceInfo androidInfo = await deviceInfo.androidInfo;
      headers['uuid'] = androidInfo.id;
      headers['operating-system'] = 'Android';
      headers['operating-system-version'] =
          androidInfo.version.sdkInt.toString();
      return;
    }

    if (Platform.isIOS) {
      final IosDeviceInfo iOSInfo = await deviceInfo.iosInfo;
      headers['uuid'] = iOSInfo.identifierForVendor;
      headers['operating-system'] = 'iOS';
      headers['operating-system-version'] = iOSInfo.systemVersion;
    }
  }

// Future<void> _setHeaderWithPushToken(Map<String, dynamic> headers) async {
//   String? pushToken = await FirebaseMessaging.instance.getToken();
//   headers['push-token'] = pushToken ?? '';
// }
}

```



TokenInterceptor 는 토큰 만료시 바로 에러를 내는 것이 아니라 리프레시 토큰을 이용해서 다시 토큰을 재발급받아서 기존의 요청을 그대로 수행하는 것이다. 서버에서 토큰이 만료일때 아래와 같이 약속된 코드와 status를 내려주고 두 조건이 모두 만족할 때 재발급 처리를 했다. 나는 서버에서도 리프레시 토큰을 보관하도록 해서 보안을 강화 하는 쪽으로 구현했다.

```dart
import 'package:dio/dio.dart';

import '../../../barrel.dart';

class TokenInterceptor extends InterceptorsWrapper {
  TokenInterceptor();

  @override
  Future<void> onError(
      DioException err, ErrorInterceptorHandler handler) async {
    if (err.response?.statusCode == 401 &&
        err.response?.data['code'] == 'AU01') {
      final String? refreshToken = await AccessManager.getRefreshToken();

      if (refreshToken != null) {
        try {
          Response response = await Injector.apiClient
              .put('/user/token/refresh?refreshToken=$refreshToken');

          final String newAccessToken = response.data['accessToken'];
          final String newRefreshToken = response.data['refreshToken'];
          await AccessManager.replaceTokens(
              accessToken: newAccessToken, refreshToken: newRefreshToken);

          // 원래의 요청을 재실행
          RequestOptions requestOptions = err.requestOptions;
          requestOptions.headers['access-token'] = 'Bearer $newAccessToken';

          Response newResponse = await Injector.apiClient.request(
            requestOptions.path,
            queryParameters: requestOptions.queryParameters,
            options: Options(
              method: requestOptions.method,
              headers: requestOptions.headers,
              responseType: requestOptions.responseType,
              contentType: requestOptions.contentType,
              extra: requestOptions.extra,
            ),
            data: requestOptions.data,
          );

          handler.resolve(newResponse);
        } catch (exception) {
          // 토큰 재발급 실패시 로직 처리
          handler.next(err);
        }
      }
    } else {
      handler.next(err);
    }
  }
}
```



아래는 시스템 점검중일때 시스템 점검중이라는 다이얼로그를 띄우고 선택지 없이 앱을 종료시키도록 하는 인터셉터이다. 토큰 인터셉터와 마찬가지로 서버에서 정의된 코드와 status 를 내려주게 했다.

```dart
import 'dart:io';

import 'package:dio/dio.dart';
import 'package:easy_localization/easy_localization.dart';

import '../../../barrel.dart';

class ServerMaintenanceInterceptor extends Interceptor {
  ServerMaintenanceInterceptor();

  @override
  Future<void> onError(
      DioException err, ErrorInterceptorHandler handler) async {
    if (err.response?.statusCode == 503 &&
        err.response?.data['code'] == 'SY01') {
      BasicDialog.show(
          context: RouterConfiguration.navigatorKey.currentContext!,
          title: TextManager.serviceMaintenanceTitle.tr(),
          contents: TextManager.serviceMaintenanceMessage.tr(),
          confirmAction: () {
            exit(0);
          });
    } else {
      handler.next(err);
    }
  }
}
```



## 사용

```dart
import 'package:device_info_plus/device_info_plus.dart';
import 'package:dio/dio.dart';
import 'package:flutter_template/barrel.dart';
import 'package:get_it/get_it.dart';

class Injector {
  Injector._();

  static DeviceInfoPlugin get deviceInfoPlugin =>
      GetIt.instance.get<DeviceInfoPlugin>();

  static Dio get apiClient => GetIt.instance.get<Dio>();

  static Future registerDependencies() async {
    _registerUtils();
    _registerNetworks();
    _registerRepositories();
  }

  static _registerUtils() async {
    GetIt.instance
        .registerLazySingleton<DeviceInfoPlugin>(() => DeviceInfoPlugin());
  }

  static _registerNetworks() async {
    GetIt.instance.registerLazySingleton<Dio>(
      () => ApiClient(
        clientBaseUrl: Environment.baseUrl,
        customInterceptors: [
          BaseHeaderInterceptor(),
          // TokenInterceptor(),
        ],
      ),
    );
  }

  static _registerRepositories() async {}
}
```

사용은 위와 같이 Injector 에 정의해두고 필요할 때(= Bloc 에 주입 또는 Repository 에 주입) 꺼내서 사용한다.
