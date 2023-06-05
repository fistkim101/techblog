# CH02 Provider Overview

<figure><img src="https://fistkim101.github.io/images/section-overview-page-001.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/section-overview-page-002.jpg" alt=""><figcaption></figcaption></figure>

&#x20;&#x20;

<figure><img src="https://fistkim101.github.io/images/section-overview-page-003.jpg" alt=""><figcaption></figcaption></figure>

\


## necessity of provider <a href="#necessity-of-provider" id="necessity-of-provider"></a>

<figure><img src="https://fistkim101.github.io/images/necessity-of-provider-page-001.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/necessity-of-provider-page-002.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/necessity-of-provider-page-003.jpg" alt=""><figcaption></figcaption></figure>

&#x20; &#x20;

<figure><img src="https://fistkim101.github.io/images/necessity-of-provider-page-004.jpg" alt=""><figcaption></figcaption></figure>

* Widget A 에서 counter 라는 데이터와 increment 라는 함수가 필요한 상황이고, Widget B 에서는 counter 라는 데이터만 필요한 상황.
* Widget A, Widget B 모두에서 동일한 데이터인 counter 가 필요하므로 공통된 부모 Widget C 에서 정의.
* counter 와 increment 의 경우 Widget C 에서 선언하지만 정작 제어는 Widget A 에서 하므로 Inversion of control 발생.
* Widget B 에 counter 를 넘겨주기 위해서 Widget C 와 Widget B 사이의 Widget 이 필요하지도 않은 counter 라는 데이터를 갖게 됨.

## managing state without provider <a href="#managing-state-without-provider" id="managing-state-without-provider"></a>

![](https://fistkim101.github.io/images/managing-state-without-provider-1-page-001.jpg)

* Counter B 만 봐서는 어디서 rebuilding 이 발생하는지 추적하기가 쉽지 않다.
* 공통 상단 부모 Widget 에서 setState() 호출되므로 쓸데없이 더 많은 Widget 이 rebuild 대상이 되어 퍼포먼스가 떨어질 수 있다.

&#x20;

<figure><img src="https://fistkim101.github.io/images/managing-state-without-provider-2-page-001.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/managing-state-without-provider-2-page-002.jpg" alt=""><figcaption></figcaption></figure>

결국 StateManagement 는 아래 두 가지 행위가 핵심이다.\


1. Dependency Injection (Object를 Widget Tree 상에서 쉽게 접근할 수 있도록 한다.)
2. Synchronizing data and UI

&#x20;

<figure><img src="https://fistkim101.github.io/images/managing-state-without-provider-2-page-003.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/managing-state-without-provider-2-page-004.jpg" alt=""><figcaption></figcaption></figure>

* Provider 는 Widget 에 Widget 이 아닌 데이터와 method 에 쉽게 접근할 수 있는 방법을 제공한다.
* 데이터가 변경 되었을 때 데이터가 변경되었다는 사실을 그 데이터가 필요한 Widget 에 제공해서 필요시 rebuild 될 수 있도록 한다.
* 결국 비즈니스 로직과 화면이 분리 되는 것이다.

\


## dependency injection using provider <a href="#dependency-injection-using-provider" id="dependency-injection-using-provider"></a>

![](https://fistkim101.github.io/images/dependency-injection-using-provider-page-001.jpg)

주목해야할 포인트는 아래 두 가지이다.

* Widget Tree 상에서 class Dog 로의 접근이 얼마나 쉽게 가능한지
* 이 방식이 생성자로 데이터를 넘겨주는 것보다 얼마나 더 간편한지

![](https://fistkim101.github.io/images/dependency-injection-using-provider-page-002.jpg)

* Provider 역시 Widget 이다.
* create 프로퍼티에서 Widget 이 필요로 하는 dog 인스턴스를 만든다.
* create 에 assign 되는 함수가 return 하는 object 에 Provider 하위 Widget 들의 접근이 가능하다.
* Provider 의 static 함수인 of 를 이용하면 Widget Tree 를 위로 traverse 하면서 원하는 Type 의 인스턴스를 찾을 수 있다.
  * `Provider.of<Dog>(context)` 와 같은 식인데, 여기서 context 를 주는 이유는 context 를 이용해서 Widget Tree 를 위로 탐색하기 때문이다.

![](https://fistkim101.github.io/images/dependency-injection-using-provider-page-003.jpg)

* `<T>` 가 같은 두 개 이상의 인스턴스가 있는 경우에는 가장 가까운 인스턴스를 가져온다.

```dart
class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Provider<Dog>(
      create: (context) => Dog(
        name: 'Sun',
        breed: 'Bulldog',
        age: 3,
      ),
      child: MaterialApp(
        title: 'Provider 02',
        debugShowCheckedModeBanner: false,
        theme: ThemeData(
          primarySwatch: Colors.blue,
        ),
        home: const MyHomePage(),
      ),
    );
  }
}

class MyHomePage extends StatelessWidget {
  const MyHomePage({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Provider 02'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          mainAxisSize: MainAxisSize.min,
          children: [
            Text(
              '- name: ${Provider.of<Dog>(context).name}',
              style: TextStyle(fontSize: 20.0),
            ),
            SizedBox(height: 10.0),
            BreedAndAge(),
          ],
        ),
      ),
    );
  }
}
```

\
ChangeNotifier & addListener
----------------------------

&#x20;

<figure><img src="https://fistkim101.github.io/images/changeNotifier-and-addListener-page-001.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/changeNotifier-and-addListener-page-002.jpg" alt=""><figcaption></figcaption></figure>

* addListener 는 자동으로 dispose 되지 않기 때문에 수동으로 직접 dispose 시켜줘야한다.
* 코드 내 Provider 제거 후 ChangeNotifier 와 addListener 를 같이 쓰면서 Listener 가 작동하는 것을 보고, 불편한 점을 살펴보았다.(Provider 삭제 후 생성자로 데이터를 넘겨주었다.)

\


## ChangeNotifierProvider <a href="#changenotifierprovider" id="changenotifierprovider"></a>

&#x20;

<figure><img src="https://fistkim101.github.io/images/ChangeNotifierProvider-page-001.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/ChangeNotifierProvider-page-002.jpg" alt=""><figcaption></figcaption></figure>

* `Provider.of<T>(context)` 로 인스턴스를 탐색 & 접근시 해당 데이터의 변경을 listen 해야할지의 필요성(데이터 변경에 따른 UI rebuild)이 있을때와 없을때 각각 옵션값이 다름을 인지한다.
  * `Provider.of<Dog>(context).age`
  * `Provider.of<Dog>(context, listen: false).age`

결론적으로 아래와 같다. 결국 아래 두 가지가 State Management 이다.

* ChangeNotifierProvider 는 데이터를 필요로 하는 Widget 이 dependency injection 을 받음으로써 데이터(인스턴스)에 쉽게 접근할 수 있게 해준다.
* 데이터의 변경이 발생했을 때 이 데이터의 변화에 맞춰서 선택적으로 Widget 을 rebuild 할 수 있게 해준다.

```dart
import 'package:flutter/foundation.dart';

class Dog with ChangeNotifier {
  final String name;
  final String breed;
  int age;
  Dog({
    required this.name,
    required this.breed,
    this.age = 1,
  });

  void grow() {
    age++;
    notifyListeners();
  }
}
```

ChangeNotifier 의 notifyListeners() 는 이를 구독하고 있는 모든 listener 들에게 변경 사실을 알리고 rebuild 하도록 만든다. [공식문서](https://api.flutter.dev/flutter/foundation/ChangeNotifier/notifyListeners.html)를 참고하자.

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import 'models/dog.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider<Dog>(
      create: (context) => Dog(name: 'dog04', breed: 'breed04'),
      child: MaterialApp(
        title: 'Provider 04',
        debugShowCheckedModeBanner: false,
        theme: ThemeData(
          primarySwatch: Colors.blue,
        ),
        home: const MyHomePage(),
      ),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({Key? key}) : super(key: key);

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Provider 04'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          mainAxisSize: MainAxisSize.min,
          children: [
            Text(
              '- name: ${Provider.of<Dog>(context).name}',
              style: TextStyle(fontSize: 20.0),
            ),
            SizedBox(height: 10.0),
            BreedAndAge(),
          ],
        ),
      ),
    );
  }
}

class BreedAndAge extends StatelessWidget {
  const BreedAndAge({
    Key? key,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text(
          '- breed: ${Provider.of<Dog>(context).breed}',
          style: TextStyle(fontSize: 20.0),
        ),
        SizedBox(height: 10.0),
        Age(),
      ],
    );
  }
}

class Age extends StatelessWidget {
  const Age({
    Key? key,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text(
          '- age: ${Provider.of<Dog>(context).age}',
          style: TextStyle(fontSize: 20.0),
        ),
        SizedBox(height: 20.0),
        ElevatedButton(
          onPressed: () => Provider.of<Dog>(context, listen: false).grow(),
          child: Text(
            'Grow',
            style: TextStyle(fontSize: 20.0),
          ),
        ),
      ],
    );
  }
}
```

\


## read, watch, select extension methods <a href="#read-watch-select-extension-methods" id="read-watch-select-extension-methods"></a>

![](https://fistkim101.github.io/images/extension-methods-page-001.jpg)

* 4.1 부터 Provider 를 더 간편하게 쓸 수 있게 해주는 extension 이 도입됨.
* 일종의 shortCut 이라고 보면 되고, 그냥 쓰지 말고 원형들이 뭔지 이해하고 있는게 중요하다.
* `context.select` 는 다수의 property 를 가지고 있는 object 의 특정 property 의 변화만 listen 하고 싶을 때 사용한다.
  * `context.watch` 는 특정 property 하나만 바뀌어도 rebuild 를 하는 것에 반해 `context.select` 는 listen 하고 싶은 것만 선별적으로 listen 이 가능하다.(퍼포먼스 고려 측면에서 사용할 수 있다.)

### context.watch vs context.select

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import 'models/dog.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider<Dog>(
      create: (context) => Dog(name: 'dog05', breed: 'breed05', age: 3),
      child: MaterialApp(
        title: 'Provider 05',
        debugShowCheckedModeBanner: false,
        theme: ThemeData(
          primarySwatch: Colors.blue,
        ),
        home: const MyHomePage(),
      ),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({Key? key}) : super(key: key);

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Provider 05'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          mainAxisSize: MainAxisSize.min,
          children: [
            Text(
              '- name: ${context.watch<Dog>().name}',
              style: TextStyle(fontSize: 20.0),
            ),
            SizedBox(height: 10.0),
            BreedAndAge(),
          ],
        ),
      ),
    );
  }
}

class BreedAndAge extends StatelessWidget {
  const BreedAndAge({
    Key? key,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text(
          '- breed: ${context.select<Dog, String>((Dog dog) => dog.breed)}',
          style: TextStyle(fontSize: 20.0),
        ),
        SizedBox(height: 10.0),
        Age(),
      ],
    );
  }
}

class Age extends StatelessWidget {
  const Age({
    Key? key,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text(
          '- age: ${context.select<Dog, int>((Dog dog) => dog.age)}',
          style: TextStyle(fontSize: 20.0),
        ),
        SizedBox(height: 20.0),
        ElevatedButton(
          onPressed: () => context.read<Dog>().grow(),
          child: Text(
            'Grow',
            style: TextStyle(fontSize: 20.0),
          ),
        ),
      ],
    );
  }
}
```

\


## Multiple Provider <a href="#multiple-provider" id="multiple-provider"></a>

![](https://fistkim101.github.io/images/MultiProvider-page-001.jpg)

* 하나의 Provider 를 사용하더라도 MultiProvider 를 사용해놓으면 확장성이 있다.

\


## Future Provider <a href="#future-provider" id="future-provider"></a>

![](https://fistkim101.github.io/images/FutureProvider-page-001.jpg)

* 강사님은 개인적으로 쓸 일이 거의 없었다고 함.
* 만약 쓸 일이 있으면 FutureBuilder 를 쓸 것 같다고 함.
* Widget Tree 에는 빌드가 되었는데 사용하고자 하는 값이 아직 준비가 되지 않았을때 사용한다.
* Future 가 resolve 되지 않았을때, initialData 로 build 하고 Future 가 resolve 되면 rebuild 된다. 총 2번 build 가 된다는 것. (만약 여러번 build 를 원한다면 StreamProvider 사용한다.)

```dart
class Babies {
  final int age;
  Babies({
    required this.age,
  });

  Future<int> getBabies() async {
    await Future.delayed(Duration(seconds: 3));

    if (age > 1 && age < 5) {
      return 4;
    } else if (age <= 1) {
      return 0;
    } else {
      return 2;
    }
  }
}
```

```dart
class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: [
        ChangeNotifierProvider<Dog>(
          create: (context) => Dog(name: 'dog06', breed: 'breed06', age: 3),
        ),
        FutureProvider<int>(
          initialData: 0,
          create: (context) {
            final int dogAge = context.read<Dog>().age;
            final babies = Babies(age: dogAge);
            return babies.getBabies();
          },
        ),
      ],
      child: MaterialApp(
        title: 'Provider 06',
        debugShowCheckedModeBanner: false,
        theme: ThemeData(
          primarySwatch: Colors.blue,
        ),
        home: const MyHomePage(),
      ),
    );
  }
}
```

* 유심히 봐둬야할 부분은 FutureProvider 의 대상은 Future 이기만 하면 되는 것이라는 것이다. 즉, 인스턴스 내에 정의된 method 를 대상으로 하고 싶다면 인스턴스 전체를 return 할 필요가 없고 특정 메소드만 제공하는 형태로 사용해야 한다.
* FutureProvider 내에서 context.read 사용이 가능하다. 이유는 위에서 ChangeNotifierProvider 가 FutureProvider 보다 상위 Widget 으로 이미 사용되었기 때문이다.

\


## StreamProvider <a href="#streamprovider" id="streamprovider"></a>

![](https://fistkim101.github.io/images/StreamProvider-page-001.jpg)

* 강사님 개인적으로는 FutureProvider 보다 StreamProvider 를 사용할 일이 더 많았다고 함.
* 연속되는 Future value 를 타겟으로 하는 Provider

```dart
        StreamProvider<String>(
          initialData: 'Bark 0 times',
          create: (context) {
            final int dogAge = context.read<Dog>().age;
            final babies = Babies(age: dogAge * 2);
            return babies.bark();
          },
        ),
```

* create 는 한 번만 called 되기 때문에 watch 를 사용하면 에러가 발생한다. 논리적으로도 read 가 맞다.

```dart
class Babies {
  final int age;
  Babies({
    required this.age,
  });

  Future<int> getBabies() async {
    await Future.delayed(Duration(seconds: 3));

    if (age > 1 && age < 5) {
      return 4;
    } else if (age <= 1) {
      return 0;
    } else {
      return 2;
    }
  }

  Stream<String> bark() async* {
    for (int i = 1; i < age; i++) {
      await Future.delayed(Duration(seconds: 2));
      yield 'Bark $i times';
    }
  }
}
```

* async 와 async\* 의 차이는 [여기](https://stackoverflow.com/questions/55397023/whats-the-difference-between-async-and-async-in-dart)를 참고한다. 간략히 설명하면 return 타입이 `Stream<T>` 를 생산하면(+ method 를 떠나지 않으면서) async\* 를 사용한다.
* 이런 경우 return 을 사용하지 않는다. method 를 떠나지 않기 때문에 종결처리 하면 안된다. yield 를 사용하며, yield 의 사전적 의미는 ‘생산하다’ 라는 뜻이다.

\


## Consumer <a href="#consumer" id="consumer"></a>

&#x20;

<figure><img src="https://fistkim101.github.io/images/Consumer-page-001.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Consumer-page-002.jpg" alt=""><figcaption></figcaption></figure>

```dart
class _MyHomePageState extends State<MyHomePage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Provider 08'),
      ),
      body: Consumer<Dog>(
        builder: (BuildContext context, Dog dog, Widget? child) {
          return Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              mainAxisSize: MainAxisSize.min,
              children: [
                child!,
                SizedBox(height: 10.0),
                Text(
                  '- name: ${dog.name}',
                  style: TextStyle(fontSize: 20.0),
                ),
                SizedBox(height: 10.0),
                BreedAndAge(),
              ],
            ),
          );
        },
        child: Text(
          'I like dogs very much',
          style: TextStyle(fontSize: 20.0),
        ),
      ),
    );
  }
}
```

* Consumer 의 파라미터 (BuildContext context, Dog dog, Widget? child) 중 child 는 builder 내에서 rebuild 될 필요가 없는 Widget 이 있는 경우를 대비해서 위와 같이 사용한다. 위 예제에서는 어떤 경우든 rebuild 될 필요가 없는 Text Widget 을 child 로 빼주었다.
* 이해가 안가면 유투브에 남아있는 [강의](https://www.youtube.com/watch?v=BrzAZYExBBA) 에서 이 부분을 다시 보고 오자.

\


## Consumer, builder, ProviderNotFoundException <a href="#consumer-builder-providernotfoundexception" id="consumer-builder-providernotfoundexception"></a>

&#x20;   &#x20;

<figure><img src="https://fistkim101.github.io/images/Consumer-builder-ProviderNotFoundException-page-001.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Consumer-builder-ProviderNotFoundException-page-002.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Consumer-builder-ProviderNotFoundException-page-003.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Consumer-builder-ProviderNotFoundException-page-004.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Consumer-builder-ProviderNotFoundException-page-005.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Consumer-builder-ProviderNotFoundException-page-006.jpg" alt=""><figcaption></figcaption></figure>

* Consumer 를 쓰든 builder 를 쓰든 둘 중 편한 방법을 사용하자.

\


## Selector <a href="#selector" id="selector"></a>

![](https://fistkim101.github.io/images/Selector-page-001.jpg)

* Consumer 와 유사한데 Consumer 보다 더 세세한 컨트롤을 가능하게 해준다.
* 앞에서 학습한 `context.select<T, R>((R selector(T value))) => R` 과 유사한 개념이다.
* 지금 학습한 Selector 라는 Widget 과 `context.select<T, R>((R selector(T value))) => R` 는 공통적으로 특정 property 의 변경에 대해서 react 할 수 있도록 하는 것.

\


## ProviderNotFoundException 더 알아보기, Builder Widget <a href="#providernotfoundexception-builder-widget" id="providernotfoundexception-builder-widget"></a>

![](https://fistkim101.github.io/images/Selector-page-001.jpg)

* extension 의 context 는 build method 의 buildContext 임을 명심. 그래서 그걸 그대로 사용하면 ProviderNotFoundException 가 발생할 수 밖에 없다.
* 이를 해결하려면 ‘Consumer, builder, ProviderNotFoundException’ 에서 학습한 builder 프로퍼티를 사용하거나 child 로 Widget 을 따로 빼서 별도의 Widget 으로 사용한다.
* Builder Widget 의 context 는 조상 Widget 의 BuildContext 가 아니고 Builder Widget 자체의 BuildContext 이기 때문에 이를 사용해서 Widget Tree 를 위로 탐색하면 원하는 T(type)에 대한 Provider 를 찾을 수 있다.
* builder 프로퍼티도 이미 학습한 바와 같이 syntax sugar 로 Builder Widget 이 사용된 것과 동일하다.

\


## Provider Access - Anonymous route access <a href="#provider-access---anonymous-route-access" id="provider-access---anonymous-route-access"></a>

&#x20; &#x20;

<figure><img src="https://fistkim101.github.io/images/provider-access-and-value-constructor-page-001.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/provider-access-and-value-constructor-page-002.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/provider-access-and-value-constructor-page-003.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/provider-access-and-value-constructor-page-004.jpg" alt=""><figcaption></figcaption></figure>

* `Consumer, builder, ProviderNotFoundException` 에서 Error Message 2 의 경우를 학습함.
* `Consumer, builder, ProviderNotFoundException` 에서 Error Message 3 의 경우 parent-child 관계에서 child 로서 Widget Tree 를 위로 탐색하면서 Provider 를 찾아하는데 탐색시 사용하는 context 가 해당 child 의 context 가 아니라, Provider 위치와 같거나 혹은 그 이상의 level 의 context 일 경우 발생한 에러.
* Error Message 2 도 parent-child 관계에서 child 로서 위로 탐색이 발생되어야 하는데, 다른 route 에서 탐색했으므로(여기서 예제는 sibling 관계) 못찾는 문제가 발생.
* 이 때의 해결 방법은 value constructor 이다. 새로운 sub-tree 에 이미 존재하는 class 에 대한 access 를 제공할 때 사용. value constructor 는 class 를 자동으로 close 하지 않는다.

```
onPressed: () {
  Navigator.push(
    context,
    MaterialPageRoute(builder: (_) {
      return ChangeNotifierProvider.value(
        value: context.read<Counter>(),
        child: ShowMeCounter(),
      );
    }),
  );
},
```

* context.read 시 사용하는 context 는 Navigator.push 의 context 를 사용해야 한다. MaterialPageRoute 의 context 가 아님을 명심. 위로 탐색하여 Provider 를 찾아야 하는데, MaterialPageRoute 의 경우 sibling 이기 때문에 MaterialPageRoute 의 context 를 사용할 경우 Provider 를 여전히 못찾는다.

\


아래는 코드 전체.

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import 'counter.dart';
import 'show_me_counter.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Anonymous Route',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: ChangeNotifierProvider<Counter>(
        create: (context) => Counter(),
        child: const MyHomePage(),
      ),
    );
  }
}

class MyHomePage extends StatelessWidget {
  const MyHomePage({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              child: Text(
                'Show Me Counter',
                style: TextStyle(fontSize: 20.0),
              ),
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(builder: (_) {
                    return ChangeNotifierProvider.value(
                      value: context.read<Counter>(),
                      child: ShowMeCounter(),
                    );
                  }),
                );
              },
            ),
            SizedBox(height: 20.0),
            ElevatedButton(
              child: Text(
                'Increment Counter',
                style: TextStyle(fontSize: 20.0),
              ),
              onPressed: () {
                context.read<Counter>().increment();
              },
            )
          ],
        ),
      ),
    );
  }
}
```

\


## Provider Access - Named route access <a href="#provider-access---named-route-access" id="provider-access---named-route-access"></a>

계속해서 Provider Access 에 대해서 학습한다. 바로 위에서는 Anonymous route access 를 살펴보았다. 왜 위의 케이스가 Anonymous route 냐면 Navigator.push 를 사용했기 때문이다.

이번에는 Named route 에서의 Provider 에 대한 access 를 학습한다.

복습 차원에서 짚고 넘어가자면, Anonymous route 가 Anonymous 인 이유는 말 그대로 이름이 없어서다. 반대로 이름이 있다는 말은 MaterialApp 에서 routes property 내에 route 주소와 Screen 으로 사용할 Widget 이 등록되어 있다는 것을 의미한다. 바로 위 Anonymous route 는 Navigator.push 로 route 설정을 따로 해주지 않고 화면 Stack 을 바로 쌓아 올린 것이기 때문에 Anonymous 였다.

```dart
routes: {
  '/': (context) => ChangeNotifierProvider.value(
        value: _counter,
        child: MyHomePage(),
      ),
  '/counter': (context) => ChangeNotifierProvider.value(
        value: _counter,
        child: ShowMeCounter(),
      ),
},dart
```

위와 같이 routes 설정 부분에서 ChangeNotifierProvider.value 로 감싸주면 되는데, 주의할 점이 있다. create 를 사용해서 사용할 인스턴스를 생성할 경우 자동으로 dispose 를 해주는데 지금 위의 경우 create 로 인스턴스를 생성하지 않았고, 아래 전체 코드에 나와있다시피 state 에서 인스턴스를 직접 생성해줬다.

따라서 dispose 역시 아래와 같이 직접 해줘야한다.

```dart
  @override
  void dispose() {
    _counter.dispose();
    super.dispose();
  }
```

아래는 전체 코드이다.

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import 'counter.dart';
import 'show_me_counter.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatefulWidget {
  MyApp({Key? key}) : super(key: key);

  @override
  State<MyApp> createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  final Counter _counter = Counter();

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Anonymous Route',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      routes: {
        '/': (context) => ChangeNotifierProvider.value(
              value: _counter,
              child: MyHomePage(),
            ),
        '/counter': (context) => ChangeNotifierProvider.value(
              value: _counter,
              child: ShowMeCounter(),
            ),
      },
    );
  }

  @override
  void dispose() {
    _counter.dispose();
    super.dispose();
  }
}

class MyHomePage extends StatelessWidget {
  const MyHomePage({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              child: Text(
                'Show Me Counter',
                style: TextStyle(fontSize: 20.0),
              ),
              onPressed: () {
                Navigator.pushNamed(context, '/counter');
              },
            ),
            SizedBox(height: 20.0),
            ElevatedButton(
              child: Text(
                'Increment Counter',
                style: TextStyle(fontSize: 20.0),
              ),
              onPressed: () {
                context.read<Counter>().increment();
              },
            )
          ],
        ),
      ),
    );
  }
}
```

\


## Provider Access - Generated route access, Global access <a href="#provider-access---generated-route-access-global-access" id="provider-access---generated-route-access-global-access"></a>

학습한 Provider Access 에 대해서 무엇을 했고, 무엇이 남았는지 다시 짚어본다.

* Provider Access
  * Anonymous route access\
    Navigator.push 내에서 사용할 screen 을 value constructor 로 감싸주되, context 를 Navigator.push 의 것을 사용한다.
  * Named route access\
    routes 설정 부분에서 value constructor 로 감싸주되, 사용할 인스턴스가 create 로 만들어진게 아니라 자동으로 dispose 되지 않으므로 직접 해당 인스턴스를 dispose 처리 해준다.
  * Generated route access\
    Named route access 과 거의 동일하다. 아래 코드를 보자.

```dart
class _MyAppState extends State<MyApp> {
  final Counter _counter = Counter();

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Anonymous Route',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      onGenerateRoute: (RouteSettings settings) {
        switch (settings.name) {
          case '/':
            return MaterialPageRoute(
              builder: (context) => ChangeNotifierProvider.value(
                value: _counter,
                child: MyHomePage(),
              ),
            );
          case '/counter':
            return MaterialPageRoute(
              builder: (context) => ChangeNotifierProvider.value(
                value: _counter,
                child: ShowMeCounter(),
              ),
            );
          default:
            return null;
        }
      },
    );
  }

  @override
  void dispose() {
    _counter.dispose();
    super.dispose();
  }
}
```

Global access 의 경우 아래와 같이 최 상단에 Provider 로 감싸두면 접근이 가능하다.

```dart
return Provider<T>(
  create: (_) => T(),
  child: MaterialApp(
      ...
  ),    
);
```

하지만 분업 환경 및 유지보수 등을 고려할 때 적절지 못한 방식이다. 실무에서 이렇게 사용할 인스턴스가 있을까 싶다. 전에 로딩스피너를 이렇게 global access 가능하게 처리해서 사용했던 것 같기도 하고.. 오픈소스 프로젝트들을 탐구할 때 확인해보도록 하자.

\


## ProxyProvider - 개요 1 <a href="#proxyprovider-1" id="proxyprovider-1"></a>

![](https://fistkim101.github.io/images/ProxyProvider-Overview-1-page-001.jpg)

* Provider 에서 다른 Provider 의 값이 필요한 경우에 ProxyProvider 를 사용한다.
* 다른 Provider 에 기반하여 value 를 만드는 Provider (A provider that builds a value based on other provider)
* 다른 Provider 의 값에 의존해서 value 를 만드는 Provider 인 것이다.
* 꼭 다른 Provider 에 의존하지 않더라도, 어떤 변하는 값에 의존해야 한다면 ProxyProvider 를 사용한다.

![](https://fistkim101.github.io/images/ProxyProvider-Overview-1-page-002.jpg)

* ProxyProvider 도 종류가 많다.

&#x20;

<figure><img src="https://fistkim101.github.io/images/ProxyProvider-Overview-1-page-003.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/ProxyProvider-Overview-1-page-004.jpg" alt=""><figcaption></figcaption></figure>

* create 는 optional 이고 update 가 required 이다.
* ProxyProvider 가 자체적으로 만들고 관리해야할 value 가 있다면 create 가 필요하지만, 그런 경우가 아니라면 만들 필요가 없기 때문이다.
* 그래서 create 는 한 번만 called, 되고 update 는 여러번 called 된다.
* ProxyProvider 는 다른 Provider 가 제공하는 값에 의존하는데, 다른 Provider 가 제공하는 value 가 바뀌면 이를 바라보고 있는(의존하고 있는) ProxyProvider 가 제공하는 값이 바뀌어야 하므로 update 가 여러번 called 되는 것은 매우 당연한 일이다.

&#x20;

<figure><img src="https://fistkim101.github.io/images/ProxyProvider-Overview-1-page-005.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/ProxyProvider-Overview-1-page-006.jpg" alt=""><figcaption></figcaption></figure>

update 가 호출되는 경우들은 아래와 같다.

* ProxyProvider 가 가장 처음으로 의존하고 있는 다른 Provider 의 값을 얻은 경우
* ProxyProvider 가 의존하고 있는 다른 Provider 가 제공하는 값이 변경된 경우
* ProxyProvider 가 rebuild 되는 경우

\


## ProxyProvider - 개요 2 <a href="#proxyprovider-2" id="proxyprovider-2"></a>

&#x20; &#x20;

<figure><img src="https://fistkim101.github.io/images/ProxyProvider-Overview-2-page-001.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/ProxyProvider-Overview-2-page-002.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/ProxyProvider-Overview-2-page-003.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/ProxyProvider-Overview-2-page-004.jpg" alt=""><figcaption></figcaption></figure>

* The instance of MyChangeNotifier is updated whenever myModel changes.
* The same instance of MyChangeNotifier is used again and again, not created again. (새로 만들어지지 않고 한번 만들어진 MyChangeNotifier 가 계속 사용된다.)

![](https://fistkim101.github.io/images/ProxyProvider-Overview-2-page-005.jpg)

\


## ProxyProvider - 실습 & 예제 <a href="#proxyprovider" id="proxyprovider"></a>

```dart

import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

class Translations {
  const Translations(this._value);
  final int _value;

  String get title => 'You clicked $_value times';
}

class WhyProxyProv extends StatefulWidget {
  const WhyProxyProv({Key? key}) : super(key: key);

  @override
  _WhyProxyProvState createState() => _WhyProxyProvState();
}

class _WhyProxyProvState extends State<WhyProxyProv> {
  int counter = 0;

  void increment() {
    setState(() {
      counter++;
      print('counter: $counter');
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Why ProxyProvider'),
      ),
      body: Center(
        child: Provider<Translations>(
          create: (_) => Translations(counter),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              ShowTranslations(),
              SizedBox(height: 20.0),
              IncreaseButton(increment: increment),
            ],
          ),
        ),
      ),
    );
  }
}

class ShowTranslations extends StatelessWidget {
  const ShowTranslations({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    final title = Provider.of<Translations>(context).title;

    return Text(
      title,
      style: TextStyle(fontSize: 28.0),
    );
  }
}

class IncreaseButton extends StatelessWidget {
  final VoidCallback increment;
  const IncreaseButton({
    Key? key,
    required this.increment,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: increment,
      child: Text(
        'INCREASE',
        style: TextStyle(fontSize: 20.0),
      ),
    );
  }
}
```

* create 는 한 번만 실행되므로 increment 에 의해서 counter 값이 증가해도 결국 `create: (_) => Translations(counter)` 에서 딱 한번 만들어진 인스턴스는 만들어지던 당시의 counter 로 만들어졌기 때문에 `final title = Provider.of<Translations>(context).title;` 의 값은 create 때 만들어진 인스턴스 그대로다.

\


```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

class Translations {
  const Translations(this._value);
  final int _value;

  String get title => 'You clicked $_value times';
}

class ProxyProvUpdate extends StatefulWidget {
  const ProxyProvUpdate({Key? key}) : super(key: key);

  @override
  _ProxyProvUpdateState createState() => _ProxyProvUpdateState();
}

class _ProxyProvUpdateState extends State<ProxyProvUpdate> {
  int counter = 0;

  void increment() {
    setState(() {
      counter++;
      print('counter: $counter');
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('ProxyProvider update'),
      ),
      body: Center(
        child: ProxyProvider0<Translations>(
          update: (_, __) => Translations(counter),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              ShowTranslations(),
              SizedBox(height: 20.0),
              IncreaseButton(increment: increment),
            ],
          ),
        ),
      ),
    );
  }
}

class ShowTranslations extends StatelessWidget {
  const ShowTranslations({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    final title = Provider.of<Translations>(context).title;

    return Text(
      title,
      style: TextStyle(fontSize: 28.0),
    );
  }
}

class IncreaseButton extends StatelessWidget {
  final VoidCallback increment;
  const IncreaseButton({
    Key? key,
    required this.increment,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: increment,
      child: Text(
        'INCREASE',
        style: TextStyle(fontSize: 20.0),
      ),
    );
  }
}
```

* 제공해야할 값이 가변적인 값인 counter 에 의존하는 인스턴스이므로 ProxyProvider 를 사용해준 코드이다.
* 이미 counter 값이 0 으로 초기화 되어 있어서, create 가 필요가 없다. 그리고 제공해야할 가변 인스턴스가 하나다. 그래서 ProxyProvider0 를 사용했다.([공식문서](https://pub.dev/documentation/provider/latest/provider/ProxyProvider0-class.html) 참고)

\


```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

class Translations {
  late int _value;

  void update(int newValue) {
    _value = newValue;
  }

  String get title => 'You clicked $_value times';
}

class ProxyProvCreateUpdate extends StatefulWidget {
  const ProxyProvCreateUpdate({Key? key}) : super(key: key);

  @override
  _ProxyProvCreateUpdateState createState() => _ProxyProvCreateUpdateState();
}

class _ProxyProvCreateUpdateState extends State<ProxyProvCreateUpdate> {
  int counter = 0;

  void increment() {
    setState(() {
      counter++;
      print('counter: $counter');
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('ProxyProvider create/update'),
      ),
      body: Center(
        child: ProxyProvider0<Translations>(
          create: (_) => Translations(),
          update: (_, Translations? translations) {
            translations!.update(counter);
            return translations;
          },
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              ShowTranslations(),
              SizedBox(height: 20.0),
              IncreaseButton(increment: increment),
            ],
          ),
        ),
      ),
    );
  }
}

class ShowTranslations extends StatelessWidget {
  const ShowTranslations({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    final title = context.watch<Translations>().title;

    return Text(
      title,
      style: TextStyle(fontSize: 28.0),
    );
  }
}

class IncreaseButton extends StatelessWidget {
  final VoidCallback increment;
  const IncreaseButton({
    Key? key,
    required this.increment,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: increment,
      child: Text(
        'INCREASE',
        style: TextStyle(fontSize: 20.0),
      ),
    );
  }
}
```

* create 에서 생성을 먼저 한 케이스다.
* update 에서 create 에서 생성된 객체를 받아 .update() 를 call 하고 return 한다.
* 여기서 또 update 발생하면 처음 create 발생시 만들어진 인스턴스(= 처음 update 때 사용된)를 계속 참조하여 재활용한다. (update 한다고 새로 만들어서 return 하는 것이 아니라는 것을 명심)

\


```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

class Translations {
  const Translations(this._value);
  final int _value;

  String get title => 'You clicked $_value times';
}

class ProxyProvProxyProv extends StatefulWidget {
  const ProxyProvProxyProv({Key? key}) : super(key: key);

  @override
  _ProxyProvProxyProvState createState() => _ProxyProvProxyProvState();
}

class _ProxyProvProxyProvState extends State<ProxyProvProxyProv> {
  int counter = 0;

  void increment() {
    setState(() {
      counter++;
      print('counter: $counter');
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('ProxyProvider ProxyProvider'),
      ),
      body: Center(
        child: MultiProvider(
          providers: [
            ProxyProvider0<int>(
              update: (_, __) => counter,
            ),
            ProxyProvider<int, Translations>(
              update: (_, value, __) => Translations(value),
            ),
          ],
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              ShowTranslations(),
              SizedBox(height: 20.0),
              IncreaseButton(increment: increment),
            ],
          ),
        ),
      ),
    );
  }
}

class ShowTranslations extends StatelessWidget {
  const ShowTranslations({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    final title = context.watch<Translations>().title;

    return Text(
      title,
      style: TextStyle(fontSize: 28.0),
    );
  }
}

class IncreaseButton extends StatelessWidget {
  final VoidCallback increment;
  const IncreaseButton({
    Key? key,
    required this.increment,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: increment,
      child: Text(
        'INCREASE',
        style: TextStyle(fontSize: 20.0),
      ),
    );
  }
}
```

* 두 개의 ProxyProvider 를 사용하는 케이스.
* ProxyProvider0 는 가변적인 int value 를 return
* ProxyProvider 는 ProxyProvider0 가 제공하는 가변적인 int 를 받아서 새로운 인스턴스를 return
* 이 경우에 ProxyProvider 가 항상 새로운 인스턴스를 만들어서 return 하고 있다는 것을 명심. 매번 새로운 Translations 를 만들어서 return 하고 있다.

\


```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

class Counter with ChangeNotifier {
  int counter = 0;

  void increment() {
    counter++;
    notifyListeners();
  }
}

class Translations with ChangeNotifier {
  late int _value;

  void update(Counter counter) {
    _value = counter.counter;
    notifyListeners();
  }

  String get title => 'You clicked $_value times';
}

class ChgNotiProvChgNotiProxyProv extends StatefulWidget {
  const ChgNotiProvChgNotiProxyProv({Key? key}) : super(key: key);

  @override
  _ChgNotiProvChgNotiProxyProvState createState() =>
      _ChgNotiProvChgNotiProxyProvState();
}

class _ChgNotiProvChgNotiProxyProvState
    extends State<ChgNotiProvChgNotiProxyProv> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('ChangeNotifierProvider ChagneNotifierProxyProvider'),
      ),
      body: Center(
        child: MultiProvider(
          providers: [
            ChangeNotifierProvider<Counter>(
              create: (_) => Counter(),
            ),
            ChangeNotifierProxyProvider<Counter, Translations>(
              create: (_) => Translations(),
              update: (
                BuildContext _,
                Counter counter,
                Translations? translations,
              ) {
                translations!..update(counter);
                return translations;
              },
            ),
          ],
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              ShowTranslations(),
              SizedBox(height: 20.0),
              IncreaseButton(),
            ],
          ),
        ),
      ),
    );
  }
}

class ShowTranslations extends StatelessWidget {
  const ShowTranslations({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    final title = context.watch<Translations>().title;

    return Text(
      title,
      style: TextStyle(fontSize: 28.0),
    );
  }
}

class IncreaseButton extends StatelessWidget {
  const IncreaseButton({
    Key? key,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () => context.read<Counter>().increment(),
      child: Text(
        'INCREASE',
        style: TextStyle(fontSize: 20.0),
      ),
    );
  }
}
```

* 정말 여러가지 방식으로 동일한 결과를 구현할 수 있다.
* Counter 가 Value Object 로 사용 되었다.
* IncreaseButton 에서는 단지 Counter 에 access 만 했다. increment() 를 실행하기만 할 목적이기 때문.

\


```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

class Counter with ChangeNotifier {
  int counter = 0;

  void increment() {
    counter++;
    notifyListeners();
  }
}

class Translations {
  const Translations(this._value);
  final int _value;

  String get title => 'You clicked $_value times';
}

class ChgNotiProvProxyProv extends StatefulWidget {
  const ChgNotiProvProxyProv({Key? key}) : super(key: key);

  @override
  _ChgNotiProvProxyProvState createState() => _ChgNotiProvProxyProvState();
}

class _ChgNotiProvProxyProvState extends State<ChgNotiProvProxyProv> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('ChangeNotifierProvider ProxyProvider'),
      ),
      body: Center(
        child: MultiProvider(
          providers: [
            ChangeNotifierProvider<Counter>(
              create: (_) => Counter(),
            ),
            ProxyProvider<Counter, Translations>(
              update: (_, counter, __) => Translations(counter.counter),
            ),
          ],
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              ShowTranslations(),
              SizedBox(height: 20.0),
              IncreaseButton(),
            ],
          ),
        ),
      ),
    );
  }
}

class ShowTranslations extends StatelessWidget {
  const ShowTranslations({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    final title = context.watch<Translations>().title;

    return Text(
      title,
      style: TextStyle(fontSize: 28.0),
    );
  }
}

class IncreaseButton extends StatelessWidget {
  const IncreaseButton({
    Key? key,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () => context.read<Counter>().increment(),
      child: Text(
        'INCREASE',
        style: TextStyle(fontSize: 20.0),
      ),
    );
  }
}
```

* 직전에 살펴본 ChangeNotifierProvider, ChangeNotifierProxyProvider 조합은 기존의 Translations 를 mutation 시켜서 재활용 하는 반면 이 방식은 Counter 가 변할 때마다 매번 새로운 Translations 인스턴스를 return 한다.
* 강사님 이야기로는 단순히 computed state 를 만들어 내는 경우 ChangeNotifierProvider 를 쓰지 않고 ProxyProvider 를 사용하는 것이 매뉴얼에도 나와있는 preferred way 라고 한다.

단순히 computed state 를 만들어 내는 경우 ChangeNotifierProvider 를 쓰지 않고 ProxyProvider 를 사용하는 것이 매뉴얼에도 나와있는 preferred way 라고 한다.

\


## Errors & addPostFrameCallback - 1 <a href="#errors--addpostframecallback---1" id="errors--addpostframecallback---1"></a>

&#x20;&#x20;

<figure><img src="https://fistkim101.github.io/images/errors-and-addPostFrameCallback-1-page-001.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/errors-and-addPostFrameCallback-1-page-002.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/errors-and-addPostFrameCallback-1-page-003.jpg" alt=""><figcaption></figcaption></figure>

\


## Errors & addPostFrameCallback - 2 <a href="#errors--addpostframecallback---2" id="errors--addpostframecallback---2"></a>

![](https://fistkim101.github.io/images/errors-and-addPostFrameCallback-2-page-001.jpg)

* 위의 에러는 아래와 같은 코드에서 발생되었다.
* flutter 가 Widget 들을 그리고 있는 중에 다시 build 요청을 할 수 없다는 것.

```dart
class Counter with ChangeNotifier {
  int counter = 0;

  void increment() {
    counter++;
    notifyListeners();
  }
}
```

```dart
@override
void initState(){
  super.initState();
  context.read<Counter>().increament();
  myCounter = context.read<Counter>().counter + 10;
}
```

&#x20;

<figure><img src="https://fistkim101.github.io/images/errors-and-addPostFrameCallback-2-page-002.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/errors-and-addPostFrameCallback-2-page-003.jpg" alt=""><figcaption></figcaption></figure>

```dart
WidgetsBinding.instance!.addPostFrameCallback((_) {
  context.read<Counter>().increment();
  myCounter = context.read<Counter>().counter + 10;
});
```

* addPostFrameCallback 은 현재의 Frame 이 완성된 후 등록된 callback 을 실행시키도록 한다.
* add(추가) + PostFrameCallback(Frame 다 그려진 후 불려질 Callback)
* UI에 영향을 미치는 action 의 실행을 현재의 Frame 이후로 지연시킬 수 있다. ‘현재의 Frame 이후’ 라는 뜻은 결국 현재 싸이클의 build 가 끝난 후를 의미한다.
* 다시 말하면 해당 Widget 의 build 가 끝난 후 실행될 action 을 예약할 수 있는 것이다.

addPostFrameCallback 외에 아래와 같은 처리도 동일한 원리이다.

```dart
Future.delayed(Duration(seconds: 0), () {
  context.read<Counter>().increment();
  myCounter = context.read<Counter>().counter + 10;
});
```

```dart
Future.microtask(() {
  context.read<Counter>().increment();dartdddd
  myCounter = context.read<Counter>().counter + 10;
});
```

![](https://fistkim101.github.io/images/errors-and-addPostFrameCallback-2-page-004.jpg)

* 위 error 는 `context.read` 에서 read 를 watch 로 바꾸면 볼 수 있는 에러이다.
* Widget Tree 밖에서 listen 을 하려 했다는 것.
* 시간차를 두고 현재 Frame 끝나고 이걸 실행해달라는 예약 행위에서 listen 을 한다는 것이 논리적으로 맞지 않다. 이렇게 이해하자.

\


## Errors & addPostFrameCallback - 3 <a href="#errors--addpostframecallback---3" id="errors--addpostframecallback---3"></a>

`Errors & addPostFrameCallback - 3` 에서는 특정 화면에 진입했을때 혹은 가변적인 값이 특정 조건에 걸렸을 때 Dialog 를 띄우는 경우에 있어서 처리 방식 들을 살펴본다.

\


![](https://fistkim101.github.io/images/errors-and-addPostFrameCallback-3-page-001.jpg)

* Dialog 는 해당 Screen 이 다 그려지고 나서 그 위에 그려지는 overlay Widget.
* 따라서 initState 에서 그냥 호출시 위와 같은 에러가 발생하는 것이 당연하다.
* 이 경우도 아래와 같이 해당 Frame 이 다 그려지고 나서 그 이후에 그려지도록 처리해야 한다.

```dart
  @override
  void initState() {
    super.initState();
    WidgetsBinding.instance!.addPostFrameCallback((_) {
      showDialog(
        context: context,
        builder: (context) {
          return AlertDialog(
            content: Text('Be careful!'),
          );
        },
      );
    });
  }
```

\


* 같은 맥락에서 가변적인 값을 바라보고 특정 조건에서 Dialog 를 띄우고 싶을때도 아래와 같이 addPostFrameCallback 로 처리해줘야한다.
* 해당 Frame 이 다 그려지고 나서 이후에 조건 검사를 한 후 overlay 하는 것이기 때문이다.
* 조건식을 포함한 Dialog call 자체를 모두 addPostFrameCallback 로 예약할 순 있으나 조건에 부합하지도 않는 경우에도 불필요하게 action 이 register 되므로 좋지 않은 코드가 된다.

```dart
  @override
  Widget build(BuildContext context) {
    if (context.read<Counter>().counter == 3) {
      WidgetsBinding.instance!.addPostFrameCallback((_) {
        showDialog(
          context: context,
          builder: (context) {
            return AlertDialog(
              content: Text('Count is 3'),
            );
          },
        );
      });
    }

    return Scaffold(
      appBar: AppBar(
        title: Text('Handle Dialog'),
      ),
      body: Center(
        child: Text(
          '${context.watch<Counter>().counter}',
          style: TextStyle(fontSize: 40.0),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        child: Icon(Icons.add),
        onPressed: () {
          context.read<Counter>().increment();
        },
      ),
    );
  }
```

&#x20; &#x20;

<figure><img src="https://fistkim101.github.io/images/errors-and-addPostFrameCallback-3-page-002.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/errors-and-addPostFrameCallback-3-page-003.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/errors-and-addPostFrameCallback-3-page-004.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/errors-and-addPostFrameCallback-3-page-005.jpg" alt=""><figcaption></figcaption></figure>

\


## Errors & addPostFrameCallback - 4 <a href="#errors--addpostframecallback---4" id="errors--addpostframecallback---4"></a>

`Errors & addPostFrameCallback - 4` 에서는 State 가 변할 때 Navigate 하는 방법에 대해 살펴본다.

![](https://fistkim101.github.io/images/errors-and-addPostFrameCallback-4-page-001.jpg)

* Navigate 하는 것도 결국 Stack 위에 올리는 Overlay 행위이기 때문에 아래와 같이 예약을 걸어 주어야 한다.
* 만약 addPostFrameCallback 를 사용하지 않으면 build 하는 와중에 Navigator.push 가 처리 되는 것이고 이는 본 화면이 다 그려지기도 전에(정확히는 그려지는 중간에) Overlay 요청을 한 것과 같다. 그래서 위와 같은 에러를 만나게 된다.

```dart
    if (context.read<Counter>().counter == 3) {
      WidgetsBinding.instance!.addPostFrameCallback((_) {
        Navigator.push(
          context,
          MaterialPageRoute(
            builder: (context) => OtherPage(),
          ),
        );
      });
    }
```

![](https://fistkim101.github.io/images/errors-and-addPostFrameCallback-4-page-002.jpg)

* Dialog, Navigate, BottomSheet 모두 Overlay Widget 이므로 해당 Widget 을 사용할 경우 이번에 학습한 내용들을 잘 활용하도록 한다.

\


## ChangeNotifier 의 addListener 를 이용한 action 처리 <a href="#changenotifier-addlistener-action" id="changenotifier-addlistener-action"></a>

&#x20;  &#x20;

<figure><img src="https://fistkim101.github.io/images/Performing-actions-using-addListener-of-ChangeNotifier-1-page-001.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Performing-actions-using-addListener-of-ChangeNotifier-1-page-002.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Performing-actions-using-addListener-of-ChangeNotifier-1-page-003.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Performing-actions-using-addListener-of-ChangeNotifier-1-page-004.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Performing-actions-using-addListener-of-ChangeNotifier-4-page-001.jpg" alt=""><figcaption></figcaption></figure>

* 강사님은 두번째 혹은 세번째 방식을 추천.
* 두번째 방식만 아래에 코드로 정리.

```dart
  Future<void> getResult(String searchTerm) async {
    _state = AppState.loading;
    notifyListeners();

    await Future.delayed(Duration(seconds: 1));

    try {
      if (searchTerm == 'fail') {
        throw 'Something went wrong';
      }

      _state = AppState.success;
      notifyListeners();
    } catch (e) {
      _state = AppState.error;
      notifyListeners();
      rethrow;
    }
  }
```

```dart
  void submit() async {
    setState(() {
      autovalidateMode = AutovalidateMode.always;
    });

    final form = formKey.currentState;

    if (form == null || !form.validate()) return;

    form.save();
    try {
      await context.read<AppProvider>().getResult(searchTerm!);
      Navigator.push(context, MaterialPageRoute(
        builder: (context) {
          return SuccessPage();
        },
      ));
    } catch (e){
      showDialog(
        context: context,
        builder: (context) {
          return AlertDialog(
            content: Text('Something went wrong'),
          );
        },
      );
    }
  }
```

\
