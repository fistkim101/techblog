# connection\_manager

## 용도

네트워크 요청시 인터넷 연결 여부를 확인하고 인터넷 자체가 연결이 되어 있지 않은 경우 네트워크 요청을 아예 날리지 않고 바로 에러 처리를 하기 위함이다.



## 구현

[connectivity\_plus](https://pub.dev/packages/connectivity\_plus) 를 사용한다.

```dart
flutter pub add connectivity_plus
```

```dart
import 'package:connectivity_plus/connectivity_plus.dart';

class ConnectionManager {
  final Connectivity connectivity;

  ConnectionManager({
    required this.connectivity,
  });

  Future<bool> get isConnected async {
    final ConnectivityResult connectivityResult =
    await connectivity.checkConnectivity();
    return ((connectivityResult == ConnectivityResult.wifi) ||
        (connectivityResult == ConnectivityResult.mobile));
  }
}
```



## 사용

프로젝트에 사용했던 실제 코드를 발췌해왔다. 현재 템플릿 코드를 구현중인 레포지토리에는 위 코드(ConnectionManager) 까지만 구현해두었다.

```dart
  @override
  Future<Either<AppException, MealSummaryModel>> getMealSummary(
      int mealId) async {
    bool isConnected = await connectionManager.isConnected;
    if (!isConnected) {
      return Left(AppException(TextManager.internetNotConnected.tr()));
    }

    try {
      final response = await apiClient.get('/meal/summary/$mealId');
      MealSummaryModel mealSummary =
          MealSummaryModel.fromResponse(response.data);

      return Right(mealSummary);
    } on DioException catch (_) {
      return Left(AppException(TextManager.mealFetchFailedMessage.tr()));
    }
  }

```
