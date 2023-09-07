# Global 처리(시스템 점검, fore->back 등)

## 시스템 점검

### global dialog

이건 그냥 내가 지은 말이다. 전역적으로 다이얼로그 처리를 해야하는 경우가 있는데(적어도 나의 경우에는 있었다) 대표적인 경우가 시스템 점검중과 같은 경우인데 시스템 점검이 걸렸을때 사용자가 앱을 사용중이건 아니건 어느 라우트에 있건 간에 무조건 전역적으로 '현재 점검중'임을 알리는 다이얼로그와 함께 앱을 종료시키는 처리가 필요할 수 있다. 이런 경우에 전역적으로 특정 경우에 다이얼로그를 띄울 수 있어야 한다.



### global dialog 구현

큰 원리는 인터셉터를 활용해서 시스템 점검이 걸린 경우 글로벌 키를 활용해서 다이얼로그를 띄우는 것이다.

```dart
import 'package:flutter/cupertino.dart';
import 'package:go_router/go_router.dart';

import '../../barrel.dart';
import '../environments/environment.dart';

class RouterConfiguration {
  static GlobalKey<NavigatorState> navigatorKey = GlobalKey<NavigatorState>();

  static final GoRoute signIn = GoRoute(
    path: RouterLocation.signIn,
    name: RouterLocation.signIn,
    builder: (context, state) {
      return const SignInScreen();
    },
  );

  static final GoRouter router = GoRouter(
    navigatorKey: navigatorKey,
    initialLocation: RouterLocation.splash,
    debugLogDiagnostics: Environment.isProduction ? false : true,
    routes: [
      GoRoute(
        path: RouterLocation.splash,
        name: RouterLocation.splash,
        builder: (context, state) {
          return const SplashScreen();
        },
        routes: [
          signIn,
        ],
      ),
    ],
  );
}
```

RouterConfiguration 에 위와 같이 키를 생성해준다. 해당 키에는 전역적으로 접근해야 하므로 static 으로 할당한다.



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

이건 나중에 network 부분에서 다시 정리하겠지만 위와 같이 시스템 점검중인지 체크하는 인터셉터를 구현하고, 점검중으로 판단된 경우 다이얼로그를 띄운다.

이때 다이얼로그를 띄울 컨텍스트로 아까 선언한 글로벌 키를 참조하여 currentContext 를 호출한다. 이렇게 해주면 사용자가 어느 위젯트리에 있든지 간에 해당 위젯트리의 context 를 참조할 수 있다.



## background 에서 foreground 로의 전환

### 왜, 어떤 경우를 위해 이것을 인지하여야 하는가

사용자는 앱을 연속된 흐름으로 사용하지 않는다. 사용을 할 때마다 사용자는 앱이 진행될 수 있는 수많은 라우트 경우의 수 중 하나에 있을 뿐이다. 그래서 사용자가 사용을 하면 안되는 상황이 있을 경우에 개발자가 이를 대처하려면 강제로 개입을 해야한다.

예를 들어 중대한 업데이트가 있어서 반드시 업데이트를 하고 이용해야 한다던가 바로 위에 정리한 것처럼 시스템 점검을 해야해서 잠깐 이용을 막아야 하는 경우가 있다. 하지만 사용자는 현재 어느 라우트에서 무엇을 하고 있을지 알 수 없다. 따라서 위의 처리처럼 인터셉터를 걸어서 그 어떤 요청이든 간에 판단하고 개입을 할 수 있도록 처리를 하는 방법을 사용할 수 있다.

또 다른 경우로 background에서 foreground로의 전환 타이밍을 이용할 수도 있다. 위 인터셉터로 처리한 것처럼 매 요청에 대해서 다 개입하는 것은 아니지만 여러 용도로 활용 될 수 있다.

나는 앱 버전을 체크하는 목적으로 구현해두었다.

```dart
import 'package:flutter/material.dart';

import '../../barrel.dart';

class ProjectName extends StatefulWidget {
  const ProjectName({super.key});

  @override
  _ProjectNameState createState() => _ProjectNameState();
}

class _ProjectNameState extends State<ProjectName> with WidgetsBindingObserver {
  @override
  void didChangeAppLifecycleState(AppLifecycleState state) async {
    super.didChangeAppLifecycleState(state);
    if (state == AppLifecycleState.resumed) {
      // background to foreground 된 상태
    }
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      debugShowCheckedModeBanner: false,
      routerConfig: RouterConfiguration.router,
    );
  }

  @override
  void dispose() {
    WidgetsBinding.instance.removeObserver(this);
    super.dispose();
  }
}
```

WidgetsBindingObserver 를 활용하는 방식이다. WidgetsBindingObserver 는 앱의 생명주기를 감지하고 원하는 생명주기에 뭔가의 처리를 할때 사용된다.



## 폰트 크기 강제

이걸 해주지 않으면 예쁘게 앱 구현 해도 각자 시스템에서 정한 폰트 크기가 있어서 원치않는 줄바뀜이 발생할 수 있다. 그래서 '사용자가 시스템에서 기본 폰트 크기를 몇이라 지정했든 간에 무조건 이 앱은 이거로 해' 라고 강제할 수 있다.

```dart
  @override
  Widget build(BuildContext context) {
    return MediaQuery(
      data: MediaQuery.of(context).copyWith(textScaleFactor: 1.0),
      child: MaterialApp.router(
        debugShowCheckedModeBanner: false,
        routerConfig: RouterConfiguration.router,
      ),
    );
  }
```

MediaQuery 로 한 번 감싸준 뒤 data 파라미터에 위와 같이 textScaleFactor 를 지정해준다. 사용자의 자유도를 침해하지만 통일된 UI 를 더 우선시 한다면 위와 같이 처리해서 의도치 않은 줄바뀜 등을 방지할 수 있다.



