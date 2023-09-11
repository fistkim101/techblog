# resources

<figure><img src="../../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

presentation 레이어에서 사용되는 모든 자원에 대해서 자원의 성격별로 나눠서 이를 관리하는 책임 수행하는 -manager 들이다.



### AssetManager

```dart
import 'package:fitsquad_app/application/types/emoji_type.dart';

class AssetManager {
  static const String splashMain1 = 'assets/images/splash_main_1.jpg';
  static const String splashMain2 = 'assets/images/splash_main_2.jpg';
  static const String splashMain3 = 'assets/images/splash_main_3.jpg';
  static const String signIn1 = 'assets/images/sign_in_1.jpg';
  static const String signIn2 = 'assets/images/sign_in_2.jpg';
  static const String signIn3 = 'assets/images/sign_in_3.jpg';
  static const String appleLogo = 'assets/images/apple_logo.png';
  static const String facebookLogo = 'assets/images/facebook_logo.png';
  static const String googleLogo = 'assets/images/google_logo.png';
  static const String kakaoLogo = 'assets/images/kakao_logo.png';
  static const String bottomMenuIconHome =
      'assets/images/bottom_menu_icon_home.png';
  static const String bottomMenuIconGroup =
      'assets/images/bottom_menu_icon_group.png';
  static const String bottomMenuIconLounge =
      'assets/images/bottom_menu_icon_lounge.png';
  static const String bottomMenuIconMy =
      'assets/images/bottom_menu_icon_my.png';
  static const String addRounded = 'assets/images/add_rounded.png';
  static const String calendar = 'assets/images/calendar.png';
  static const String forkAndKnife = 'assets/images/fork_and_knife.png';
  static const String emojiCountSmile = 'assets/images/emoji_count_smile.png';
  static const String commentCountChatBubble =
      'assets/images/comment_count_chat_bubble.png';
  static const String heart = 'assets/images/heart.png';
  static const String search = 'assets/images/search.png';
  static const String viewCountEyes = 'assets/images/view_count_eyes.png';
  static const String emojiSmile = 'assets/images/emoji_smile.png';
  static const String emojiSad = 'assets/images/emoji_sad.png';
  static const String emojiEmbarrassed = 'assets/images/emoji_embarrassed.png';
  static const String emojiOutrageous = 'assets/images/emoji_outrageous.png';
  static const String emojiPanicked = 'assets/images/emoji_panicked.png';
  static const String emojiPray = 'assets/images/emoji_pray.png';

  // lottie
  static const String loadingSpinner = 'assets/lotties/loading-utensils.json';

  static String getEmojiPath(EmojiType emojiType) {
    if (emojiType == EmojiType.smile) {
      return emojiSmile;
    }

    if (emojiType == EmojiType.sad) {
      return emojiSad;
    }

    if (emojiType == EmojiType.embarrassed) {
      return emojiEmbarrassed;
    }

    if (emojiType == EmojiType.outrageous) {
      return emojiOutrageous;
    }

    if (emojiType == EmojiType.panicked) {
      return emojiPanicked;
    }

    return '';
  }
}

```

asset 의 path 를 상수화해서 사용한다.



### ColorManager

```dart
import 'dart:ui';

import 'package:flutter/cupertino.dart';
import 'package:flutter/material.dart';

import '../../../barrel.dart';

class ColorManager {
  static Color primary = "#20C997".convertToColorFromHex(); // Teal-500
  static Color accentGreen = "#087F5B".convertToColorFromHex(); // Teal-500
  static Color teal100 = "#C3FAE8".convertToColorFromHex();
  static Color teal700 = "#0CA678".convertToColorFromHex();
  static Color teal800 = "#099268".convertToColorFromHex();
  static Color teal900 = "#087F5B".convertToColorFromHex();
  static Color blueGrey900 = "263238".convertToColorFromHex();
  static Color blueGrey600 = "546E7A".convertToColorFromHex();
  static Color blueGrey500 = "607D8B".convertToColorFromHex();
  static Color blueGrey400 = "78909C".convertToColorFromHex();
  static Color blueGrey300 = "90A4AE".convertToColorFromHex();
  static Color blueGrey200 = "B0BEC5".convertToColorFromHex();
  static Color blueGrey100 = "CFD8DC".convertToColorFromHex();
  static Color blueGrey50 = "ECEFF1".convertToColorFromHex();
  static Color subCommentBackground = "F4F4F4".convertToColorFromHex();
  static Color white = Colors.white;

  static Color kakaoMainColor = "FEE500".convertToColorFromHex();
  static Color facebookMainColor = "4267B2".convertToColorFromHex();
  static Color googleMainColor = "EA4335".convertToColorFromHex();
  static Color appleMainColor = "000000".convertToColorFromHex();
  static Color heartColor = "FF0000".convertToColorFromHex();
}
```

figma 등 디자인 툴에서 정해진 명칭과 매칭시켜서 간편하게 사용할 수 있도록 처리했다.



### FontManager

```dart
import 'package:flutter/material.dart';

class FontConstants {
  static const String ManropeFontFamily = "Manrope";
  static const String NotoSansKrFontFamily = "NotoSansKR";
}

class FontWeights {
  static const FontWeight light = FontWeight.w300;
  static const FontWeight regular = FontWeight.w400;
  static const FontWeight medium = FontWeight.w500;
  static const FontWeight semiBold = FontWeight.w600;
  static const FontWeight bold = FontWeight.w700;
}
```



### ThemeManager

```dart
import 'package:flutter/material.dart';

import '../../barrel.dart';

class ThemeManager {
  static ThemeData getThemeData({
    required Locale locale,
    required bool isDarkMode,
  }) {
    final String defaultFontFamily = locale == LocaleManager.koLocale
        ? FontConstants.NotoSansKrFontFamily
        : FontConstants.ManropeFontFamily;

    ThemeData lightTheme = ThemeData(
      scaffoldBackgroundColor: ColorManager.white,
      primaryColor: ColorManager.primary,
      disabledColor: ColorManager.blueGrey50,
      textTheme: TextTheme(
        titleLarge: getDefaultTextStyle(defaultFontFamily, ValueManager.s34),
        titleMedium: getDefaultTextStyle(defaultFontFamily, ValueManager.s30),
        titleSmall: getDefaultTextStyle(defaultFontFamily, ValueManager.s26),
        bodyLarge: getDefaultTextStyle(defaultFontFamily, ValueManager.s22),
        bodyMedium: getDefaultTextStyle(defaultFontFamily, ValueManager.s18),
        bodySmall: getDefaultTextStyle(defaultFontFamily, ValueManager.s12),
      ),
    );

    ThemeData darkTheme = ThemeData();

    return isDarkMode ? darkTheme : lightTheme;
  }

  static TextStyle getDefaultTextStyle(
    String defaultFontFamily,
    double fontSize,
  ) {
    return TextStyle(
      fontSize: fontSize,
      fontFamily: defaultFontFamily,
    );
  }
}

```

```dart
  @override
  Widget build(BuildContext context) {
    return MultiBlocProvider(
      providers: [
        BlocProvider(create: (context) => Injector.loadingSpinnerBloc),
      ],
      child: MediaQuery(
        data: MediaQuery.of(context).copyWith(textScaleFactor: 1.0),
        child: MaterialApp.router(
          theme: ThemeManager.getThemeData(
            locale: context.fallbackLocale!,
            isDarkMode: false,
          ),
          localizationsDelegates: context.localizationDelegates,
          supportedLocales: context.supportedLocales,
          locale: context.locale,
          debugShowCheckedModeBanner: false,
          routerConfig: RouterConfiguration.router,
        ),
      ),
    );
  }
```



### ValueManager

```dart
class ValueManager {
  static const double half = 0.5;
  static const double s1 = 1.0;
  static const double s2 = 2.0;
  static const double s3 = 3.0;
```

수치를 상수처리 한 것인데 사실 이러면 안된다. xxs, xs 등으로 나눠서 써야하는데 디자인 요구조건을 만족시키려다 보면 결국 1을 1로 상수화해서 쓰는 어처구니 없는 코드가 나온다. 디자인 요구조건 부터 크기같은 것을 특정 수치로 다양화 하지 않고 의미적으로 좀 묶어서 나와야한다.



### TextManager

다국어 처리시 사용된 키값을 상수화 해서 가지고 있는 객체이다.
