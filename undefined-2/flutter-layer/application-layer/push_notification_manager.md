# push\_notification\_manager

## 용도

백그라운드에서 푸시 하는 것은 쉬운데 포그라운드에서 푸시를 처리할때 안드로이드에서 추가적으로 해줘야 할 사항이 있다. 이 경우의 푸시 처리를 위해서 추가적인 코드 구현이 필요하고 push\_notification\_manager 는 이 처리를 담당하는 객체이다.



## 구현

&#x20;일단 구현했었던 코드를 그대로 붙인다. 아래 코드 말고도 권한 설정등 해줘야 할 내용들이 더 있다. 안드로이드의 경우 해주고 있는 처리를 간단히 설명하자면 알림 채널을 하나 생성해서 중요도가 높다고 커스텀하게 Importance 를 설정해준다.

이 채널을 통해서 로컬 알림을 포그라운드에서 내보내도록 처리한다.

```dart
import 'dart:io';

import 'package:firebase_messaging/firebase_messaging.dart';
import 'package:flutter_local_notifications/flutter_local_notifications.dart';

class PushNotificationManager {
  static Future<void> initializeNotification() async {
    if (Platform.isIOS) {
      await FirebaseMessaging.instance
          .setForegroundNotificationPresentationOptions(
        alert: true,
        badge: true,
        sound: true,
      );
      return;
    }

    const AndroidNotificationChannel androidNotificationChannel =
        AndroidNotificationChannel(
      'high_importance_channel', // 임의의 id
      'High Importance Notifications', // 설정에 보일 채널명
      importance: Importance.max,
    );

    final FlutterLocalNotificationsPlugin flutterLocalNotificationsPlugin =
        FlutterLocalNotificationsPlugin();
    await flutterLocalNotificationsPlugin
        .resolvePlatformSpecificImplementation<
            AndroidFlutterLocalNotificationsPlugin>()
        ?.createNotificationChannel(androidNotificationChannel);

    await flutterLocalNotificationsPlugin.initialize(
        const InitializationSettings(
            android: AndroidInitializationSettings('@mipmap/ic_launcher')));

    FirebaseMessaging.onMessage.listen((RemoteMessage remoteMessage) {
      RemoteNotification? notification = remoteMessage.notification;
      AndroidNotification? android = remoteMessage.notification?.android;

      if (notification != null && android != null) {
        flutterLocalNotificationsPlugin.show(
          0,
          notification.title,
          notification.body,
          const NotificationDetails(
            android: AndroidNotificationDetails(
              'high_importance_channel',
              'High Importance Notifications',
            ),
          ),
        );
      }
    });
  }
}

```

이것 외에 파이어베이스 콘솔에서 처리해줘야할 내용들이 있는데 그 부분은 다른 블로그에도 많이 나와있어서 생략하도록 한다.



## 사용

environment.dart 에서 run() 할 때 아래와 같이 호출해줘야한다.

```dart
    await PushNotificationManager.initializeNotification();
```
