# repository

## 용도

network 에서 만들어준 클라이언트를 통해서 서버와 통신해서 데이터를 직접 가져오는 책임을 가진 객체다. 도메인 별로 나눠서 각 도메인에 대해서 서버로부터 데이터를 가져와 처리하는 등의 책임을 수행한다.



## 구현

클린 아키텍처 강의에서 [dartz](https://pub.dev/packages/dartz) 를 사용했는데 처음엔 좀 이상하다 생각했는데 막상 프로젝트에 써보니 에러 났을때와 정상 처리를 더 분명하게 구분할 수 있어서 좋았다.

```dart
flutter pub add dartz
```

```dart
import 'package:dartz/dartz.dart';
import 'package:dio/dio.dart';
import 'package:easy_localization/easy_localization.dart';

import '../../../../barrel.dart';

class UserRepositoryImpl extends UserRepository {
  final Dio apiClient;
  final ConnectionManager connectionManager;

  UserRepositoryImpl({
    required this.apiClient,
    required this.connectionManager,
  });

  @override
  Future<Either<AppException, bool>> replaceIsPushPermitted(
      bool isPushPermitted) async {
    final togglePushPermittedRequest =
        apiClient.put('/user/push-permitted', queryParameters: {
      'isPushPermitted': isPushPermitted,
    });
    try {
      Response response = await togglePushPermittedRequest;
      return Right(response.data['isPushPermitted'] as bool);
    } catch (error) {
      return Left(
          AppException(TextManager.notificationToggleFailedMessage.tr()));
    }
  }

  @override
  Future<Either<AppException, void>> replacePushToken(String pushToken) async {
    final replacePushTokenRequest = apiClient.put('/user/push-token');
    try {
      await replacePushTokenRequest;
      return const Right(null);
    } catch (error) {
      return Left(AppException.basicError());
    }
  }

  @override
  Future<Either<AppException, bool>> appVersionCheck() async {
    final appVersionCheck = apiClient.get('/app/version');

    try {
      final Response response = await appVersionCheck;
      return Right(response.data['isForceUpdate']);
    } catch (error) {
      return Left(AppException(TextManager.appVersionCheckFailedMessage.tr()));
    }
  }

  @override
  Future<Either<AppException, void>> updateProfile(
      UpdateProfileCommand command) async {
    bool isConnected = await connectionManager.isConnected;
    if (!isConnected) {
      return Left(AppException(TextManager.internetNotConnected.tr()));
    }

    final Map<String, dynamic> jsonData = command.toJson();
    final FormData formData;
    if (command.profileImage != null) {
      formData = FormData.fromMap({
        'profileImage':
            await MultipartFile.fromFile(command.profileImage!.path),
      });

      jsonData.forEach((key, value) {
        formData.fields.add(MapEntry(key, value.toString()));
      });
    } else {
      formData = FormData.fromMap(jsonData);
    }

    try {
      await apiClient.put(
        '/user/profile',
        data: formData,
        options: Options(
          contentType: 'multipart/form-data',
        ),
      );
      return const Right(null);
    } on DioException catch (error) {
      return Left(AppException(TextManager.updateProfileFailedMessage.tr()));
    }
  }

  @override
  Future<Either<AppException, void>> quitService() async {
    bool isConnected = await connectionManager.isConnected;
    if (!isConnected) {
      return Left(AppException(TextManager.internetNotConnected.tr()));
    }

    final minimumLatency = Future.delayed(const Duration(milliseconds: 700));
    final quitServiceRequest = apiClient.delete('/user/quit');
    try {
      await Future.wait([minimumLatency, quitServiceRequest]);
      return const Right(null);
    } catch (error) {
      return Left(AppException(TextManager.quitServiceFailedMessage.tr()));
    }
  }

  @override
  Future<Either<AppException, UserDetailModel>> getUserDetail() async {
    bool isConnected = await connectionManager.isConnected;
    if (!isConnected) {
      return Left(AppException(TextManager.internetNotConnected.tr()));
    }

    final minimumLatency = Future.delayed(const Duration(milliseconds: 700));
    final getUserDetailRequest = apiClient.get('/user/profile');
    try {
      await Future.wait([minimumLatency, getUserDetailRequest]);
      Response response = await getUserDetailRequest;
      UserDetailModel result = UserDetailModel.fromJson(response.data);

      return Right(result);
    } catch (error) {
      return Left(AppException(TextManager.userDetailFailedMessage.tr()));
    }
  }

  @override
  Future<Either<AppException, SignInResponse>> signIn() async {
    bool isConnected = await connectionManager.isConnected;
    if (!isConnected) {
      return Left(AppException(TextManager.internetNotConnected.tr()));
    }

    final minimumLatency = Future.delayed(const Duration(milliseconds: 700));
    final signInRequest = apiClient.post('/user/token');
    try {
      await Future.wait([minimumLatency, signInRequest]);
    } catch (error) {
      return Left(AppException(TextManager.signInFailed.tr()));
    }
    Response signInResponse = await signInRequest;

    return Right(SignInResponse.fromJson(signInResponse.data));
  }

  @override
  Future<Either<AppException, SignInResponse>> continueWithSocial(
      ContinueWithSocialQuery continueWithSocialQuery) async {
    bool isConnected = await connectionManager.isConnected;
    if (!isConnected) {
      return Left(AppException(TextManager.internetNotConnected.tr()));
    }

    final minimumLatency = Future.delayed(const Duration(milliseconds: 700));
    final continueWithSocialRequest =
        apiClient.post('/user', data: continueWithSocialQuery.toJson());

    try {
      await Future.wait([minimumLatency, continueWithSocialRequest]);
    } on DioException catch (error) {
      return Left(AppException(TextManager.continueWithSocialFailed.tr()));
    }
    final Response continueWithSocialResponse = await continueWithSocialRequest;

    return Right(SignInResponse.fromJson(continueWithSocialResponse.data));
  }

  @override
  Future<Either<AppException, void>> completeOnBoarding(
      CompleteOnBoardingCommand command) async {
    bool isConnected = await connectionManager.isConnected;
    if (!isConnected) {
      return Left(AppException(TextManager.internetNotConnected.tr()));
    }

    final Map<String, dynamic> jsonData = command.toJson();
    final FormData formData;
    if (command.profileImage != null) {
      formData = FormData.fromMap({
        'profileImage':
            await MultipartFile.fromFile(command.profileImage!.path),
      });

      jsonData.forEach((key, value) {
        formData.fields.add(MapEntry(key, value.toString()));
      });
    } else {
      formData = FormData.fromMap(jsonData);
    }

    try {
      await apiClient.post(
        '/user/profile',
        data: formData,
        options: Options(
          contentType: 'multipart/form-data',
        ),
      );
      return const Right(null);
    } on DioException catch (error) {
      return Left(
          AppException(TextManager.onboardingFailedExceptionContents.tr()));
    }
  }
}

```

다양한 데이터 통신 케이스가 있어서 일단 프로젝트에 사용한 코드를 다 가져왔다. 템플릿 코드에서는 인터페이스, 구현체를 나누지 않고 data 레이어에 바로 구현체 하나만 사용할 예정이다.



## 사용

```dart
  static _registerRepositories() async {
    GetIt.instance.registerLazySingleton<UserRepository>(
      () => UserRepository(
        apiClient: apiClient,
        connectionManager: connectionManager,
      ),
    );
  }
```

다른 것들과 마찬가지로 Injector 에 위와 같이 선언한 후 필요한 때(= Bloc 에 주입) 사용한다.

