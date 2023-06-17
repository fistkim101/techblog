# (+) Application Layer - l10n

## [easy\_localization](https://pub.dev/packages/easy\_localization)

기존에 사용했던 패키지에서 이것으로 바꾸었다. 더 많이 쓰이는 것으로 보이고 사용이 더 편리하다.



## usage

거의 공식 도큐먼트에 정리되어 있는 내용과 똑같아서 별달리 정리할게 없긴 하다. 도큐먼트에서 지시하는대로 똑같이 하면 되긴 하는데 주의할 점은 EasyLocalization 위젯은 MaterialApp 과 동일 계층(아래 코드에서 보면 Application 내) 에 설정하니까 에러가 났다.

그래서 아래와 같이 runApp 내에 위치시켰다. 도큐먼트에 나온대로 await EasyLocalization.ensureInitialized(); 처리는 반드시 해주어야한다.

```dart
void run() async {
  WidgetsFlutterBinding.ensureInitialized();
  await EasyLocalization.ensureInitialized();
  await Injector.prepareDependencies();
  runApp(
    EasyLocalization(
      supportedLocales: const [
        LocaleManager.koLocale,
        LocaleManager.enLocale,
      ],
      path: 'assets/translations',
      fallbackLocale: LocaleManager.koLocale,
      child: Application(),
    ),
  );
}
```

LocalManager 라고 해둔 것은 별 것이 없다. 아래와 같다. Locale 생성시 넣은 언어에 대한 파라미터 값과 파일명.json 에서 파일명이 일치해야한다.

```dart
class LocaleManager {
  static const Locale enLocale = Locale('en');
  static const Locale koLocale = Locale('ko');
}
```



그 외에 아래와 같이 json 파일을 만들어줘야  하고, 파일 내에 아무것도 없으면 launch 시 에러가 난다. 파일이 없거나 파일 내용이 empty 라는 메세지가 뜬다. 당장 뭔가 쓸 일이 없어도 내용을 채워둬야한다.

<figure><img src="../../../.gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure>

다양한 사용법은 [공식문서](https://pub.dev/packages/easy\_localization)를 참고하자. 따로 연습한 샘플 프로젝트는 [여기](https://github.com/fistkim101/flutter\_i18n\_sample/tree/master/assets/translations)에 있다.
