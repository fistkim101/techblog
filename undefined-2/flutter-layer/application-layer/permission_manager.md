# permission\_manager

## 용도

앱에서 앨범 혹은 카메라 등 네이티브 자원을 사용할 경우 반드시 사용자의 권한을 획득 해야한다. 이 때 사용자가 바로 권한을 주면 깔끔한데, 만약 거부 혹은 '다시 묻지 않기' 등으로 영원이 거부를 해놓고는 정작 기능은 사용하고 싶어하는 경우가 있다.

이런 여러 가지 경우가 있기에 권한에 대한 세부적인 처리는 매우 필수적이다. 예를 들어 앨범 및 카메라 접근 권한을 거절해놓고 앱 상에서 카메라를 사용하는 기능을 실행한 경우 권한을 확인한뒤 사용자가 거부를 했다면 다이얼로그 등을 띄워서 '권한을 줘야 사용할 수 있다'는 내용을 고지하고 설정으로 바로 이동처리를 해주는 등의 조치가 필요하다.&#x20;

즉, 매끄러운 앱 사용 흐름을 위해서 권한 상태에 대한 검사와 각 상태에 대한 적절한 분기 처리가 필요하다.



## 구현

실제 프로젝트에서 구현한 코드를 옮겨둔다. 템플릿 코드에는 구현할 앱이 어떤 권한을 사용할지 알 수 없어서 일단은 제외시켰다. 아래 코드는 앨범과 카메라를 사용하는 앱에서 각 기능의 사용 전에 권한 체크를 할 때 사용한 코드이다.

[permission\_handler](https://pub.dev/packages/permission\_handler) 를 사용한다.

```dart
flutter pub add permission_handler
```

```dart
import 'dart:io';

import 'package:device_info_plus/device_info_plus.dart';
import 'package:easy_localization/easy_localization.dart';
import 'package:fitsquad_app/barrel.dart';
import 'package:flutter/material.dart';
import 'package:permission_handler/permission_handler.dart';

class PermissionManager {
  static Future<void> checkPhotoPermission({
    required BuildContext currentBuildContext,
    required Function grantedAction,
  }) async {
    PermissionStatus status = await _checkStatus();

    if (status.isDenied) {
      status = await _requestPermission();

      if (status.isGranted) {
        grantedAction();
      } else if (status.isPermanentlyDenied) {
        Future.microtask(() {
          BasicDialog.show(
            context: currentBuildContext,
            title: TextManager.photoPermissionTitle.tr(),
            contents: TextManager.photoPermissionMessage.tr(),
            confirmAction: () {
              openAppSettings();
            },
          );
        });
      }
    } else if (status.isPermanentlyDenied) {
      Future.microtask(() {
        BasicDialog.show(
          context: currentBuildContext,
          title: TextManager.photoPermissionTitle.tr(),
          contents: TextManager.photoPermissionMessage.tr(),
          confirmAction: () {
            openAppSettings();
          },
        );
      });
    } else if (status.isGranted) {
      grantedAction();
    }
  }

  static Future<PermissionStatus> _checkStatus() async {
    if (Platform.isAndroid) {
      final androidInfo = await DeviceInfoPlugin().androidInfo;
      if (androidInfo.version.sdkInt <= 32) {
        return await Permission.storage.status;
      } else {
        return await Permission.photos.status;
      }
    }

    return await Permission.photos.status;
  }

  static Future<PermissionStatus> _requestPermission() async {
    if (Platform.isAndroid) {
      final androidInfo = await DeviceInfoPlugin().androidInfo;
      if (androidInfo.version.sdkInt <= 32) {
        return await Permission.storage.request();
      } else {
        return await Permission.photos.request();
      }
    }

    return await Permission.photos.request();
  }

  static Future<void> checkCameraPermission({
    required BuildContext currentBuildContext,
    required Function grantedAction,
  }) async {
    PermissionStatus status = await Permission.camera.status;

    if (status.isDenied) {
      status = await Permission.camera.request();

      if (status.isGranted) {
        grantedAction();
      } else if (status.isPermanentlyDenied) {
        Future.microtask(() {
          BasicDialog.show(
            context: currentBuildContext,
            title: TextManager.cameraPermissionTitle.tr(),
            contents: TextManager.cameraPermissionMessage.tr(),
            confirmAction: () {
              openAppSettings();
            },
          );
        });
      }
    } else if (status.isPermanentlyDenied) {
      Future.microtask(() {
        BasicDialog.show(
          context: currentBuildContext,
          title: TextManager.cameraPermissionTitle.tr(),
          contents: TextManager.cameraPermissionMessage.tr(),
          confirmAction: () {
            openAppSettings();
          },
        );
      });
    } else if (status.isGranted) {
      grantedAction();
    }
  }
}

```



## 사용

```dart
      onTap: () async {
        if (imageSource == ImageSource.camera) {
          await PermissionManager.checkCameraPermission(
              currentBuildContext: context,
              grantedAction: () async {
                await _takePicture(context, imageSource);
              });
        } else {
          await PermissionManager.checkPhotoPermission(
              currentBuildContext: context,
              grantedAction: () async {
                await _takePicture(context, imageSource);
              });
        }
      },
```



## 주의사항

permission\_handler는 각 OS 별로 구현 해줘야 할 것들이 추가적으로 존재한다. 이는 어떤 권한을 줘야 할지(어떤 네이티브 자원을 끌어다 써야 하는지)에 따라 달라서 여기서 정리는 생략한다.

자세한 것은 [공식문서의 SetUp](https://pub.dev/packages/permission\_handler#setup) 부분에 각 OS 별로 정리되어 있다. 반드시 처리를 해줘야한다.
