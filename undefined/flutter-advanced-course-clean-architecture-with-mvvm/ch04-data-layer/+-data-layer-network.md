# (+) Data Layer - Network

## [dio](https://pub.dev/packages/dio)

강의에서는 [retrofit](https://pub.dev/packages/retrofit)을 사용하고 있는데, dio 가 더 인기 있고 보편적인 라이브러리이다. 따라서 이 라이브러리를 사용하기로 했고 이 라이브러리 기준으로 설정한 것을 정리한다.

## 패키지 구성 관점에서 바라본 Network

![](<../../../.gitbook/assets/image (1).png>)

하나의 앱에서 각기 다른 서버와 통신할 일이 흔하고, 이 때 각 서버와의 통신을 책임지는 클라이언트도 구분을 해두는 것이 코드 확장성이나 유지보수 측면에서 유리하다.

그렇지만 한편으로는 각기 다른 서버와 통신한다고 해도 공통적으로 가져야할 역할 및 형태가 있을 수 밖에 없다. 그래서 이러한 부분을 고려해서 base\_http\_client 을 분리해두었고 각 클라이언트가 이를 상속해서 구현하도록 처리했다.

interceptor 도 용도별로 최대한 분리해서 따로 구현을 하여 필요한 클라이언트가 알아서 가져가서 사용하도록 처리했다.

## Network 구현

```dart
import 'package:dio/dio.dart';

abstract class BaseHttpClient with DioMixin implements Dio {
  final String clientBaseUrl;
  final List<Interceptor> customInterceptors;
  BaseOptions? customOptions;

  BaseHttpClient({
    required this.clientBaseUrl,
    required this.customInterceptors,
    this.customOptions,
  }) {
    if (customOptions != null) {
      options = customOptions!;
    }
    options.baseUrl = clientBaseUrl;
    // options.connectTimeout = Duration(seconds: 5);
    // options.receiveTimeout = Duration(seconds: 3);
    interceptors.add(LogInterceptor(requestBody: true, responseBody: true));
    interceptors.addAll(customInterceptors);
  }
}
```

```dart
import 'package:app_frame_sample/data/network/client/base_http_client.dart';

class TodoApiClient extends BaseHttpClient {
  TodoApiClient({
    required super.clientBaseUrl,
    required super.customInterceptors,
    super.customOptions,
  });
}
```

```dart
import 'dart:io';

import 'package:app_frame_sample/application/application.dart';
import 'package:device_info_plus/device_info_plus.dart';
import 'package:dio/dio.dart';
import 'package:package_info_plus/package_info_plus.dart';

class BaseHeaderInterceptor extends InterceptorsWrapper {
  @override
  void onRequest(
      RequestOptions options, RequestInterceptorHandler handler) async {
    await _setHeaderWithAccessToken(options.headers);
    await _setHeaderWithPackageInfo(options.headers);
    await _setHeaderWithDeviceInfo(options.headers);

    super.onRequest(options, handler);
  }

  Future<void> _setHeaderWithAccessToken(Map<String, dynamic> headers) async {
    headers[HttpHeaders.authorizationHeader] =
        await AccessManager.getAccessToken();
  }

  Future<void> _setHeaderWithPackageInfo(Map<String, dynamic> headers) async {
    final PackageInfo packageInfo = await PackageInfo.fromPlatform();
    headers['appName'] = packageInfo.appName;
    headers['packageName'] = packageInfo.packageName;
    headers['version'] = packageInfo.version;
    headers['buildNumber'] = packageInfo.buildNumber;
  }

  Future<void> _setHeaderWithDeviceInfo(Map<String, dynamic> headers) async {
    final DeviceInfoPlugin deviceInfo = Injector.deviceInfoPlugin;
    if (Platform.isAndroid) {
      final AndroidDeviceInfo androidInfo = await deviceInfo.androidInfo;
      headers['uuid'] = androidInfo.id;
      headers['operatingSystem'] = 'Android';
      headers['operatingSystemVersion'] = androidInfo.version.sdkInt.toString();
      return;
    }

    if (Platform.isIOS) {
      final IosDeviceInfo iOSInfo = await deviceInfo.iosInfo;
      headers['uuid'] = iOSInfo.identifierForVendor;
      headers['operatingSystem'] = 'iOS';
      headers['operatingSystemVersion'] = iOSInfo.systemVersion;
    }
  }
}
```

여기서 device 정보와 package 정보를 읽는데 사용된 패키지들은 아래와 같다.

* [device\_info\_plus](https://pub.dev/packages/device\_info\_plus)
* [package\_info\_plus](https://pub.dev/packages/package\_info\_plus)

위 코드상에 구현된 AccessManager 는 data 레이어보다는 application 레이어에 속하는게 맞는 것 같아서 일단은 application layer 에 구현해두었고 코드는 아래와 같다. [shared\_preferences](https://pub.dev/packages/shared\_preferences) 패키지를 사용했다.

```dart
import 'package:shared_preferences/shared_preferences.dart';

class AccessManager {
  static const String accessTokenKey = 'accessToken';

  static Future<void> saveAccessToken(String accessToken) async {
    final SharedPreferences preferences = await SharedPreferences.getInstance();
    await preferences.setString(accessTokenKey, accessToken);
  }

  static Future<String> getAccessToken() async {
    final SharedPreferences preferences = await SharedPreferences.getInstance();
    final String? accessToken = preferences.getString(accessTokenKey);
    if (accessToken == null) {
      return '';
    }
    
    return accessToken;
  }
}
```
