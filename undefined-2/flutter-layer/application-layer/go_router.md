# go\_router

## go\_router 용도(=라우팅 패키지 사용 목적)

사실 따로 이렇게 라이브러리를 사용하지 않아도 Navigator 만으로 화면 라우팅은 가능하다. 하지만 url based 된 이동 방식, 여러 플랫폼에서 사용될 수 있다는 점, Named routes 등 많은 부가기능이 있어서(=사실 그만큼 순정이 제대로 역할을 못하고 있다는 이야기가 된다) 따로 라이브러리를 사용한다.



## 왜 go\_router 사용하는가

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Flutter Favorite 이고 아주 보편적인 라이브러리다. 라우터 이동 관련해서 여러 라이브러리가 있지만 장단을 따질 것 없이 제일 보편적이어서 일단 채택했다.



## go\_router 간략한 소개

{% embed url="https://www.youtube.com/watch?v=b6Z885Z46cU&t=6s" %}

어차피 아래에 샘플 코드를 쓰긴 할건데 보통 새로운 위젯을 쓸 때 나는 플러터팀에서 만든 짧은 위젯 소개 영상을 먼저 본다. 숲을 먼저 보고 나무를 볼 수 있어서 좋다. 대부분의 위젯 소개 영상들이 깔끔하게 시각화 되어 있어서 어떤 용도인지, 간략하게나마 어떤식으로 쓰는지 등을 빠른 시간 내에 파악할 수 있다.



## go\_router 구현

```bash
flutter pub add go_router
```

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```dart
class RouterLocation {
  static const String splash = '/';
  static const String signIn = 'singIn';
}
```

```dart
import 'package:go_router/go_router.dart';

import '../../barrel.dart';
import '../environments/environment.dart';

class RouterConfiguration {
  static final GoRoute signIn = GoRoute(
    path: RouterLocation.signIn,
    name: RouterLocation.signIn,
    builder: (context, state) {
      return const SignInScreen();
    },
  );

  static final GoRouter router = GoRouter(
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



그리고 environment 에서 호출하는 위젯에 아래와 같이 구현한다.

```dart
  run() async {
    await Injector.registerDependencies();
    runApp(const ProjectName());
  }
```

```dart
import 'package:flutter/material.dart';

import '../../barrel.dart';

class ProjectName extends StatefulWidget {
  const ProjectName({super.key});

  @override
  _ProjectNameState createState() => _ProjectNameState();
}

class _ProjectNameState extends State<ProjectName> {
  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      debugShowCheckedModeBanner: false,
      routerConfig: RouterConfiguration.router,
    );
  }
}
```



## go, push 차이 확실히 알기

[공식문서](https://pub.dev/documentation/go\_router/latest/topics/Get%20started-topic.html)에 이것저것 정리가 잘 되어 있어서 핵심만 간단히 정리한다. 개인적으로 매우 기본적인 내용이면서도 핵심적인 사항이라 생각하는 부분이다.

화면은 스택구조로 되어있다. push 로 이동하면 현재 화면 스택에 신규 화면을 쌓는 형식이다. 반대로 go 로 이동하면 현재의 화면을 교체하는 것이다.

이 원리와 연결하여 GoRoute의 routes\[] 에 대해서 알아둬야한다. go 로 이동하는 것은 스택에 쌓는 것이 아니라 교체하는 것이기 때문에 독립적인 루트레벨로 라우트가 존재해도 된다. 하지만 push 로 쌓는 것은 반드시 상위 위젯이 하위 위젯을 가지고 있는 경우만 쌓을 수 있다.

예를 들어 A위젯에서 B위젯으로 go 로 이동하는 경우 독립적인 루트를 가지고 있어도 상관이 없다. 하지만 A에서 push 로 B를 쌓아 올리고 싶은 경우에 A는 반드시 하위 위젯으로 B를 가지고 있어야 한다. routes 배열 내에 B가 들어가 있어야 한다는 뜻이다.



## go\_router 활용하여 데이터 넘기기

단순한 사용법 수준이라서 프로젝트에 실제 구현했던 코드를 예제로 남긴다.



### path 를 통한 데이터 전달

```dart
              onPressed: () async {
                String path = _currentSubMenuType == SubMenuType.normalLounge
                    ? RouterLocation.createPost
                    : '${RouterLocation.createPostGroup}/:id';

                final refreshRequired = await context.pushNamed(
                  path,
                  pathParameters: _selectedGroupId != null
                      ? {'id': _selectedGroupId.toString()}
                      : {},
                  extra: _currentSubMenuType == SubMenuType.normalLounge
                      ? PostType.lounge
                      : PostType.group,
                );

                if ((refreshRequired != null) &&
                    ((refreshRequired as List<bool>).first)) {
                  if (_currentSubMenuType == SubMenuType.normalLounge) {
                    Future.microtask(() => context
                        .read<PostBloc>()
                        .add(NormalLoungeSubMenuRequested.initial()));
                  } else {
                    Future.microtask(() => context
                        .read<PostBloc>()
                        .add(GroupLoungeSubMenuRequested.initial()));
                  }
                }
              },
```

```dart
  static final GoRoute createPostGroup = GoRoute(
    path: '${RouterLocation.createPostGroup}/:id',
    name: '${RouterLocation.createPostGroup}/:id',
    builder: (context, state) {
      final String groupId = state.pathParameters['id']!;
      final PostType postType = state.extra as PostType;

      return BlocProvider<PostBloc>(
        create: (_) => PostBloc(
          loadingSpinnerBloc: Injector.loadingSpinnerBloc,
          postRepository: Injector.postRepository,
          groupRepository: Injector.groupRepository,
        ),
        child: CreatePostScreen(
          postType: postType,
          groupId: postType == PostType.group ? int.parse(groupId) : null,
        ),
      );
    },
  );// Some code
```



### queryParameter 통한 데이터 전달

```dart
                      onTap: () async {
                        final result = await context.pushNamed(
                          RouterLocation.subComment,
                          extra: widget.comment,
                          queryParameters: {
                            'postId': widget.postId.toString(),
                            'commentId': widget.comment.id.toString(),
                          },
                        );
                        final refreshRequired = ((result != null) &&
                            ((result as List<bool>).first));

                        if (refreshRequired) {
                          widget.commentInfiniteModel!.initialize();
                          _refreshPost();
                        }
                      },
```

```dart
  static final GoRoute subComment = GoRoute(
    path: RouterLocation.subComment,
    name: RouterLocation.subComment,
    builder: (context, state) {
      final int postId = int.parse(state.queryParameters['postId']!);
      final int commentId = int.parse(state.queryParameters['commentId']!);
      return BlocProvider<CommentBloc>(
        create: (_) => CommentBloc(
          loadingSpinnerBloc: Injector.loadingSpinnerBloc,
          commentRepository: Injector.commentRepository,
        ),
        child: SubCommentScreen(postId: postId, commentId: commentId),
      );
    },
  );
```



### extra 를 통한 데이터 전달

```dart
    final rotatedResult = await context.pushNamed(
      RouterLocation.imageRotation,
      extra: File(imageFile.path),
    );
```

```dart
  static final GoRoute imageRotation = GoRoute(
    path: RouterLocation.imageRotation,
    name: RouterLocation.imageRotation,
    builder: (context, state) {
      final File image = state.extra as File;
      return ImageRotationScreen(image: image);
    },
  );
```
