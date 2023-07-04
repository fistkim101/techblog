# CH05 Firebase Authentication App

Provider 에서 구현한 앱과 동일했고, Bloc 활용 범위도 TODO 에서 다뤘던 범위 내에서 다뤘다.

단, WillPopScope 활용에 대해서 내가 예전에 이 앱을 만들때 다소 유심히 보지 않았던 것 같아서 따로 다시 정리해둔다.

> WillPopScope 위젯은 사용자가 뒤로 가기 버튼을 누를 때 발생하는 이벤트를 처리할 수 있는 위젯입니다. 예를 들어, 사용자가 스택 내에서 이전 화면으로 이동하려는 경우 뒤로 가기 버튼을 누릅니다. 이때 WillPopScope 위젯이 사용되면, 뒤로 가기 버튼 이벤트를 캐치하여 커스텀한 작업을 수행할 수 있습니다.
>
> WillPopScope 위젯은 onWillPop 속성을 가지며, 이 속성은 Future을 반환합니다. 만약 onWillPop이 true를 반환하면 이벤트는 무시되고 앱은 이전 화면으로 이동합니다. 반면에, false를 반환하면 이벤트가 취소되고 앱은 현재 화면에 머무르게 됩니다.
>
> WillPopScope는 일반적으로 Scaffold나 AppBar와 같은 위젯을 사용할 때 적용됩니다. 이러한 위젯을 사용하면 사용자가 뒤로 가기 버튼을 누르면 기본적으로 현재 화면이 종료되어야 하지만, WillPopScope를 사용하면 이벤트를 캐치하여 다른 작업을 수행할 수 있습니다. 예를 들어, 사용자가 뒤로 가기 버튼을 눌렀을 때 경고 메시지를 띄울 수 있습니다.



그리고 리스너에 다이얼로그를 설정해뒀을때 다이얼로그가 여러번 호출되는 문제도 아래와 같이 해결했다.

```dart
class SplashPage extends StatelessWidget {
  static const String routeName = '/';
  const SplashPage({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return BlocConsumer<AuthBloc, AuthState>(
      listener: (context, state) {
        if (state.authStatus == AuthStatus.unauthenticated) {
          Navigator.pushNamedAndRemoveUntil(
            context,
            SigninPage.routeName,
            (route) {
              print('route.settings.name: ${route.settings.name}');
              print('ModalRoute: ${ModalRoute.of(context)!.settings.name}');

              return route.settings.name ==
                      ModalRoute.of(context)!.settings.name
                  ? true
                  : false;
            },
          );
        } else if (state.authStatus == AuthStatus.authenticated) {
          Navigator.pushNamed(context, HomePage.routeName);
        }
      },
      builder: (context, state) {
        return Scaffold(
          body: Center(
            child: CircularProgressIndicator(),
          ),
        );
      },
    );
  }
}
```

로그인 성공 후 로그아웃을 한 뒤 로그인을 의도적으로 실패하면 원래 한 번 떠야할 다이얼로그가 중첩해서 두 개 뜨는 현상을 해결하는 것이다. 다이얼로그는 아래와 같이 리스너에 걸려있다.

```dart
child: BlocConsumer<SigninCubit, SigninState>(
  listener: (context, state) {
    if (state.signinStatus == SigninStatus.error) {
      errorDialog(context, state.error);
    }
  },
  ...
),  
```

1. 상태가 실제로 두 번 바뀌었거나
2. 리스너가 두 개 등록되어있거나

인데 이 경우 2번의 경우로 인해 다이얼로그가 두 개 뜨는 것이다. 왜냐하면 Splash 에서 pushNamed 로 Signin Page 로 보냈었기 때문에, 첫 로그인 진입시 하나와 Home 에서 로그아웃한 후 Splash 에서 분기를 타면서 Splash 로 보낸 것 두 개가 중첩되어 있기 때문이다.

이를 방지하고자 signin 에서 쌓인 라우트를 검사해서 splash 외에 모든 것을 없애는 작업을 해준 것이다.

Navigator.pushNamedAndRemoveUntil 에 대해서 잠깐 알아보자.



#### Navigator.pushNamedAndRemoveUntil <a href="#navigatorpushnamedandremoveuntil" id="navigatorpushnamedandremoveuntil"></a>

Navigator.pushNamedAndRemoveUntil은 현재 페이지에서 지정한 경로 이름의 페이지로 이동하면서, 이전 페이지 스택에서 특정 조건을 만족하는 모든 페이지를 제거하는 함수입니다.

Navigator.pushNamedAndRemoveUntil의 인자로는 1) 이동할 경로 이름과 2) 이전 페이지 스택에서 제거할 페이지를 결정하는 콜백 함수가 들어갑니다. 콜백 함수는 이전 페이지 스택에서 각 페이지를 검사하여 페이지를 제거할지 말지를 결정합니다.

예를 들어, 다음과 같은 코드를 사용하여 ‘/A’ 경로로 이동하면서 이전 페이지 스택에서 ‘/’ (루트) 경로를 제외한 모든 페이지를 제거할 수 있습니다.

```dart
Navigator.pushNamedAndRemoveUntil(
  context,
  '/A',
  (route) => route.settings.name == '/',
);
```



도큐먼트를 보면 아래와 같이 나와있다.

````dart
  /// Push the route with the given name onto the navigator, and then remove all
  /// the previous routes until the `predicate` returns true.
  ///
  /// {@macro flutter.widgets.navigator.pushNamedAndRemoveUntil}
  ///
  /// {@macro flutter.widgets.navigator.pushNamed.returnValue}
  ///
  /// {@macro flutter.widgets.Navigator.pushNamed}
  ///
  /// {@tool snippet}
  ///
  /// Typical usage is as follows:
  ///
  /// ```dart
  /// void _handleOpenCalendar() {
  ///   navigator.pushNamedAndRemoveUntil('/calendar', ModalRoute.withName('/'));
  /// }
  /// ```
  /// {@end-tool}
  ///
  /// See also:
  ///
  ///  * [restorablePushNamedAndRemoveUntil], which pushes a new route that can
  ///    be restored during state restoration.
  @optionalTypeArgs
  Future<T?> pushNamedAndRemoveUntil<T extends Object?>(
    String newRouteName,
    RoutePredicate predicate, {
    Object? arguments,
  }) {
    return pushAndRemoveUntil<T>(_routeNamed<T>(newRouteName, arguments: arguments)!, predicate);
  }
````

`Push the route with the given name onto the navigator, and then remove all the previous routes until the` predicate `returns true.`

이걸 보면 알 수 있듯이 ‘무조건 지운다. predicate 이 true 를 return 할 때 까지’ 로 동작한다. 그래서 pushNamedAndRemoveUntil 가 실행되면 predicate 에서 현재 스택에 쌓인 모든 route 를 순회하며 predicate 적용 여부를 검사하게 되는 것이다.
