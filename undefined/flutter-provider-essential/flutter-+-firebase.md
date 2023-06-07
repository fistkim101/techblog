---
description: firebase_cli 이용한 firebase 와 flutter 연동
---

# (+) Flutter + Firebase 연동

## 1. firebase-cli 를 설치해야한다. mac 에서는 brew 로 설치가 가능하다. firebase 에 생성된 프로젝트 정보들을 가져오기 위해서 필요하다. <a href="#1-firebase-cli-mac-brew-firebase" id="1-firebase-cli-mac-brew-firebase"></a>

```
brew install firebase-cli
```

## 2. dart 로 global 하게 flutterfire\_cli 를 activate 한다. 하면 .zshrc 수정이 필요할 수 있다. <a href="#2-dart-global-flutterfire_cli-activate-zshrc" id="2-dart-global-flutterfire_cli-activate-zshrc"></a>

```
# Install the CLI if not already done so
dart pub global activate flutterfire_cli

Package flutterfire_cli is currently active at version 0.2.7.
The package flutterfire_cli is already activated at newest available version.
To recompile executables, first run `dart pub global deactivate flutterfire_cli`.
Installed executable flutterfire.
Warning: Pub installs executables into $HOME/.pub-cache/bin, which is not on your path.
You can fix that by adding this to your shell\'s config file (.bashrc, .bash_profile, etc.):

export PATH="$PATH":"$HOME/.pub-cache/bin"

Activated flutterfire_cli 0.2.7.
```

## 3. firebase 를 연결하고자 하는 프로젝트 경로에 들어가서 firebase\_core 의존성을 설치한다. firebase 와 app 연결을 책임지는 의존성이다. (firebase\_core plugin, which is responsible for connecting your application to Firebase.) <a href="#3-firebase-firebase_core-firebase-app-firebase_core-plugin-which-is-responsible-for-connecting-your" id="3-firebase-firebase_core-firebase-app-firebase_core-plugin-which-is-responsible-for-connecting-your"></a>

```
flutter pub add firebase_core
```

## 4. flutterfire configure 명령어를 통해서 연결 설정을 잡아준다. 기존 프로젝트 연결로 실습하지 않았고 create a new project 로 실습했다. <a href="#4-flutterfire-configure-create-a-new-project" id="4-flutterfire-configure-create-a-new-project"></a>

```
# Run the `configure` command, select a Firebase project and platforms
flutterfire configure
```

![](https://fistkim101.github.io/images/connection-firebase-flutter.png)

## 5. [공식문서](https://firebase.flutter.dev/docs/overview) 에 따라 아래 코드를 main 에 넣어준다. <a href="#5-main" id="5-main"></a>

```
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  runApp(MyApp());
}
```
