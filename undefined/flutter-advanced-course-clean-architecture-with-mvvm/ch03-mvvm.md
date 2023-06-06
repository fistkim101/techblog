# CH03 MVVM

## MVVM 에 관한 이해 <a href="#mvvm" id="mvvm"></a>

사실 순서가 VVMM 인데, 왜 MVVM 으로 용어가 정립 되었는지 모르겠다. [이 포스팅](https://betterprogramming.pub/how-to-use-mvvm-in-flutter-4b28b63da2ca) 을 보고 학습했다.

정말 간단히 정리하자면 View는 말 그대로 View 다. 사용자와 상호작용 하면서 사용자의 action 을 ViewModel 로 전달하는 역할과, ViewModel 으로부터 응답을 받아 사용자에게 전달하는 것을 책임진다. 여기서 확실히 해야할 것은 View 는 상태관리를 하진 않는다는 것이다.

ViewModel 은 View 에 데이터를 제공하면서 상태관리도 담당한다.

Model 은 data access layer 와 동일한 역할이다.

## MVVM 기본 패턴

[유투브](https://www.youtube.com/watch?v=wh6jfZJelf0\&t=75s) 에 괜찮은 강의가 있어 참고했다. udemy 강의에서는 계층이 너무 분리가 되어서 일단 MVVM 자체에 대한 이해를 높히기 위해서 기본 패턴을 먼저 학습했다.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_clean_architecture/presentation/simple_mvvm/simple_view_model.dart';

class SimpleScreen extends StatefulWidget {
  @override
  _SimpleScreenState createState() => _SimpleScreenState();
}

class _SimpleScreenState extends State<SimpleScreen> {
  SimpleViewModel viewModel = SimpleViewModel();

  @override
  Widget build(BuildContext context) {
    return SafeArea(
      child: Scaffold(
        body: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          crossAxisAlignment: CrossAxisAlignment.center,
          mainAxisSize: MainAxisSize.max,
          children: [
            StreamBuilder(
              stream: viewModel.mvvmStream,
              builder: (context, snapshot) {
                print('StreamBuilder > build > snapshot : ${snapshot.data}');
                int count = 0;
                if (snapshot.data != null) {
                  count = snapshot.data!.count;
                }
                return Center(
                  child: Text(
                    count.toString(),
                    style: TextStyle(fontSize: 30),
                  ),
                );
              },
            ),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                IconButton(
                  onPressed: () {
                    viewModel.increaseCounter();
                  },
                  icon: Icon(Icons.exposure_plus_1),
                ),
                IconButton(
                  onPressed: () {
                    viewModel.decreaseCounter();
                  },
                  icon: Icon(Icons.exposure_minus_1),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

```dart
import 'dart:async';

import 'package:flutter_clean_architecture/presentation/simple_mvvm/simple_model.dart';

class SimpleViewModel {
  late SimpleModel _model;
  final StreamController<SimpleModel> _streamController =
      StreamController<SimpleModel>();

  Stream<SimpleModel> get mvvmStream => _streamController.stream;

  SimpleViewModel() {
    _model = SimpleModel();
  }

  void update() {
    _streamController.sink.add(_model);
  }

  void increaseCounter() {
    _model.count++;
    update();
  }

  void decreaseCounter() {
    _model.count--;
    update();
  }
}
```

```dart
class SimpleModel {
  int count = 0;

  @override
  String toString() {
    return 'SimpleModel{count: $count}';
  }
}
```

Provider 에서 notifyListeners(); 를 사용하는 방식과 유사하다. 결국 Stream 도 Publish, Subscribe 원리이며 이를 이용해서 해당 Stream 을 구독하는 StreamBuilder 를 통해서 변경이 있을때마다 이를 받아 rebuild 하는 방식이다.

유투브 강의에는 Provider 를 사용한 MVVM 도 있었는데, 사실 명칭을 이렇게 갖다 붙여서 다른 것 같지만 그냥 일반적인 Provider 였다. View 의 도메인 로직을 전부 Provider 에 위임하고 Provider 의 처리에 따라서 View 는 rebuild 가 필요할 때 알아서 rebuild 가 되도록(react) Provider 를 watch 하는 형태인 것이다. 이걸 기본형 맥락에서 보면 StreamBuilder 를 통해서 rebuild 되는 것과 똑같은 방식이다.

## 강의에서 사용된 MVVM 패턴 <a href="#mvvm" id="mvvm"></a>

[소스코드](https://github.com/minafarideleia/complete\_advanced\_flutter/tree/Lecture\_41\_How\_to\_Recive\_Data\_in\_View\_From\_Viewmodel) 가 너무 길어서 링크만 남긴다.

위 기본형과 비슷하지만 가장 크게 다른 부분은 계층이 더 세분화 되어 있다는 것이다. 대표적으로 아래 클래스가 있다.

```dart
import 'dart:async';

import 'package:complete_advanced_flutter/presentation/common/state_renderer/state_render_impl.dart';
import 'package:rxdart/rxdart.dart';

abstract class BaseViewModel extends BaseViewModelInputs
    with BaseViewModelOutputs {
  StreamController _inputStateStreamController =
  BehaviorSubject<FlowState>();

  @override
  Sink get inputState => _inputStateStreamController.sink;

  @override
  Stream<FlowState> get outputState =>
      _inputStateStreamController.stream.map((flowState) => flowState);

  @override
  void dispose() {
    _inputStateStreamController.close();
  }

// shared variables and functions that will be used through any view model.
}

abstract class BaseViewModelInputs {
  void start(); // will be called while init. of view model
  void dispose(); // will be called when viewmodel dies.

  Sink get inputState;
}

abstract class BaseViewModelOutputs {
  Stream<FlowState> get outputState;
}
```

BaseViewModel 라는 최상위 class 를 만들고 모든 ViewModel 이 이를 상속하도록 한다. 각 ViewModel 에서도 또 계층을 아래와 같이 나눈다.

```dart
import 'dart:async';

import 'package:complete_advanced_flutter/domain/model/model.dart';
import 'package:complete_advanced_flutter/presentation/base/baseviewmodel.dart';
import 'package:complete_advanced_flutter/presentation/resources/assets_manager.dart';
import 'package:complete_advanced_flutter/presentation/resources/strings_manager.dart';
import 'package:easy_localization/easy_localization.dart';

class OnBoardingViewModel extends BaseViewModel
    with OnBoardingViewModelInputs, OnBoardingViewModelOutputs {
  // stream controllers
  final StreamController _streamController =
      StreamController<SliderViewObject>();

  late final List<SliderObject> _list;

  int _currentIndex = 0;

  // inputs
  @override
  void dispose() {
    _streamController.close();
  }

  @override
  void start() {
    _list = _getSliderData();
    // send this slider data to our view
    _postDataToView();
  }

  @override
  int goNext() {
    int nextIndex = _currentIndex++; // +1
    if (nextIndex >= _list.length) {
      _currentIndex = 0; // infinite loop to go to first item inside the slider
    }
    return _currentIndex;
  }

  @override
  int goPrevious() {
    int previousIndex = _currentIndex--; // -1
    if (previousIndex == -1) {
      _currentIndex =
          _list.length - 1; // infinite loop to go to the length of slider list
    }
    return _currentIndex;
  }

  @override
  void onPageChanged(int index) {
    _currentIndex = index;
    _postDataToView();
  }

  @override
  Sink get inputSliderViewObject => _streamController.sink;

  // outputs
  @override
  Stream<SliderViewObject> get outputSliderViewObject =>
      _streamController.stream.map((slideViewObject) => slideViewObject);

  // private functions
  List<SliderObject> _getSliderData() => [
        SliderObject(
            AppStrings.onBoardingTitle1.tr(),
            AppStrings.onBoardingSubTitle1.tr(),
            ImageAssets.onboardingLogo1),
        SliderObject(
            AppStrings.onBoardingTitle2.tr(),
            AppStrings.onBoardingSubTitle2.tr(),
            ImageAssets.onboardingLogo2),
        SliderObject(
            AppStrings.onBoardingTitle3.tr(),
            AppStrings.onBoardingSubTitle3.tr(),
            ImageAssets.onboardingLogo3),
        SliderObject(
            AppStrings.onBoardingTitle4.tr(),
            AppStrings.onBoardingSubTitle4.tr(),
            ImageAssets.onboardingLogo4)
      ];

  _postDataToView() {
    inputSliderViewObject.add(
        SliderViewObject(_list[_currentIndex], _list.length, _currentIndex));
  }
}

// inputs mean the orders that our view model will recieve from our view
abstract class OnBoardingViewModelInputs {
  void goNext(); // when user clicks on right arrow or swipe left.
  void goPrevious(); // when user clicks on left arrow or swipe right.
  void onPageChanged(int index);

  Sink
      get inputSliderViewObject; // this is the way to add data to the stream .. stream input
}

// outputs mean data or results that will be sent from our view model to our view
abstract class OnBoardingViewModelOutputs {
  Stream<SliderViewObject> get outputSliderViewObject;
}

class SliderViewObject {
  SliderObject sliderObject;
  int numOfSlides;
  int currentIndex;

  SliderViewObject(this.sliderObject, this.numOfSlides, this.currentIndex);
}
```

이렇게 ViewModel 이 있는 .dart 에 위와 같이 abstract class 를 또 만들어서 BaseViewModel 을 상속하게 한뒤 저것들을 다 같이 mixin 한다.

goNext, goPrevious, onPageChanged 는 해당 ViewModel 특유의 것이라서 ViewModel 에 선언해도 되지만 아마도 명시적으로 \~Input 내에 넣어주어서 확실하게 분리시키려는 의도로 보인다.

\~Input 내에 Sink 로 설정해 준 것은 사실 결국 StreamController 의 sink 인데, 의미는 View -> ViewModel 로 데이터를 전달(상호작용)할때 데이터가 들어가는 입구라고 생각하면 된다. 개인적으로 따로 분리하지 말고 \~controller.sink 로 그대로 사용하는 것이 훨씬 해석이 단순하고 좋은 것 같다. 단순히 ViewModel 이 데이터를 받아서 stream 으로 publish 할는 과정에서 ‘받는’ input 개념이다.

## Stream 을 활용한 View, ViewModel 상호작용 이해 <a href="#stream-view-viewmodel" id="stream-view-viewmodel"></a>

아래 그림으로 만들었다. 강사의 샘플 코드가 중요한 것이 아니라 아래 그림이 골자라는 것을 알고 이해한다.

![](https://fistkim101.github.io/images/concept\_view\_viewmodel.png)

Stream 은 필요하다면 여러 개가 될 수 있다. 강의에서는 로그인시 name, password, validation, isLoginSuccess 각각을 위한 네 개의 StreamController 를 만들어 사용하고 있다. 강의 코드가 좀 깔끔하지 못하므로 View-ViewModel 간에 Stream 이 다수일 수 있다는 것만 기억하자. 다수인 상황이 오히려 많을 것 같다.
