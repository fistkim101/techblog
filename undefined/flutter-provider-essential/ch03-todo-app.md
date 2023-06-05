# CH03 TODO App

* TODO App 을 위 슬라이드에 나온 세 가지 방식으로 각각 총 세 번 구현한다.
* 하나의 폴더 내에 세 앱을 각각 만들고, 소스는 폴더 최상단에서 git init 해서 관리해야겠다.
* Independent State 의 예시로는 TODO Item 의 ‘완료여부’ 를 들 수 있다.
  * 불변하는 값이 아니고 완료가 될 경우 값이 변하는 성질이 있고, 이 변경의 여부에 따라 Widget 의 rebuild 가 필요하기 때문에 ChangeNotifierProvider 를 사용해야한다.
* Computed State 는 다른 것(것들) 에 의존된 Computed 된 값이다. 예시로는 ‘미완료 Item 수’ 를 들 수 있다.

&#x20;&#x20;

<figure><img src="https://fistkim101.github.io/images/TODO+App+Structure-page-001.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/ChangeNotifierProxyProvider_ProxyProvider-page-001.jpg" alt=""><figcaption></figcaption></figure>

최종적으로 StateNotifierProvider 로 앱을 구현했다. [깃허브](https://github.com/fistkim101/provider-sample-todo-app)에 푸시해두었다. 기록을 위해 Provider 주입 부분과 independent state, computed state 샘플만 코드를 남긴다.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_state_notifier/flutter_state_notifier.dart';
import 'package:provider/provider.dart';

import 'providers/providers.dart';
import 'screens/screens.dart';

void main() {
  runApp(const TodoApp());
}

class TodoApp extends StatelessWidget {
  const TodoApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: [
        StateNotifierProvider<SearchTerm, SearchTermState>(
          create: (_) => SearchTerm(),
        ),
        StateNotifierProvider<Filter, FilterState>(
          create: (_) => Filter(),
        ),
        StateNotifierProvider<Todos, TodosState>(
          create: (_) => Todos(),
        ),
        StateNotifierProvider<ActiveTodoCount, ActiveTodoCountState>(
          create: (_) => ActiveTodoCount(),
        ),
        StateNotifierProvider<FilteredTodos, FilteredTodosState>(
          create: (_) => FilteredTodos(),
        ),
        // ChangeNotifierProvider(
        //   create: (_) => SearchTerm(),
        // ),
        // ChangeNotifierProvider(
        //   create: (_) => Filter(initialFilterType: FilterType.all),
        // ),
        // ChangeNotifierProvider(
        //   create: (_) => Todos(initialTodos: []),
        // ),
        // ProxyProvider<Todos, ActiveTodoCount>(
        //   update: (
        //     _,
        //     Todos todos,
        //     __,
        //   ) =>
        //       ActiveTodoCount(todos: todos),
        // ),
        // ProxyProvider3<Todos, Filter, SearchTerm, FilteredTodos>(
        //   update: (
        //     _,
        //     Todos todos,
        //     Filter filter,
        //     SearchTerm searchTerm,
        //     __,
        //   ) =>
        //       FilteredTodos(
        //           todos: todos, filter: filter, searchTerm: searchTerm),
        // )
      ],
      child: const MaterialApp(
        debugShowCheckedModeBanner: false,
        home: Home(),
      ),
    );
  }
}
```

```dart
import 'package:equatable/equatable.dart';
import 'package:flutter_state_notifier/flutter_state_notifier.dart';

class SearchTermState extends Equatable {
  final String? searchTerm;

  const SearchTermState({
    required this.searchTerm,
  });

  @override
  bool get stringify => true;

  @override
  List<Object> get props => [searchTerm ?? ''];

  SearchTermState copyWith(String searchTerm) {
    return SearchTermState(searchTerm: searchTerm);
  }
}

class SearchTerm extends StateNotifier<SearchTermState> {
  SearchTerm() : super(const SearchTermState(searchTerm: ''));

  void searchTermChange(String searchTerm) {
    state = SearchTermState(searchTerm: searchTerm);
  }
}

// class SearchTerm with ChangeNotifier {
//   late SearchTermState _state;
//   final String? initialSearchTerm;
//
//   SearchTermState get state => _state;
//
//   SearchTerm({
//     this.initialSearchTerm,
//   }) {
//     _state = SearchTermState(searchTerm: initialSearchTerm);
//   }
//
//   void update(String searchTerm) {
//     _state = _state.copyWith(searchTerm);
//     notifyListeners();
//   }
// }
```

```dart
import 'package:equatable/equatable.dart';
import 'package:state_notifier/state_notifier.dart';

import '../models/model.dart';
import '../providers/providers.dart';

class ActiveTodoCountState extends Equatable {
  final int activeTodoCount;

  const ActiveTodoCountState({
    required this.activeTodoCount,
  });

  @override
  List<Object> get props {
    return [activeTodoCount];
  }

  @override
  bool get stringify => true;

  ActiveTodoCountState copyWith(int activeTodoCount) {
    return ActiveTodoCountState(activeTodoCount: activeTodoCount);
  }
}

class ActiveTodoCount extends StateNotifier<ActiveTodoCountState>
    with LocatorMixin {
  ActiveTodoCount() : super(const ActiveTodoCountState(activeTodoCount: 0));

  @override
  void update(Locator watch) {
    final List<Todo> todos = watch<TodosState>().todos;
    final int newActiveTodoCount =
        todos.where((todo) => !todo.isCompleted).toList().length;
    state = ActiveTodoCountState(activeTodoCount: newActiveTodoCount);

    super.update(watch);
  }
}

// class ActiveTodoCount {
//   final Todos todos;
//
//   ActiveTodoCount({
//     required this.todos,
//   });
//
//   ActiveTodoCountState get state {
//     final int newActiveTodoCount =
//         todos.state.todos.where((todo) => !todo.isCompleted).toList().length;
//
//     return ActiveTodoCountState(activeTodoCount: newActiveTodoCount);
//   }
// }

```

## State 를 다룰때 immutable state 을 사용해야 하는 이유 (+ Equatable 원리) <a href="#state-immutable-state-equatable" id="state-immutable-state-equatable"></a>

Todo app 실습 중 todo 를 삭제하는 과정에서 rebuild 가 발생하지 않는 현상이 발생했다. 그래서 강사님 코드를 보고 수정했더니 rebuild 가 잘 작동했다. 덕분에 기계적으로 사용했던 Equatable 을 다시 살펴봤고, Notifier 의 state 비교 로직도 다시 살펴보았다.

### Object.dart 의 == 과 hashCode <a href="#objectdart-hashcode" id="objectdart-hashcode"></a>

dart 의 모든 객체들은 Object 를 상속하고 있으며, Object 의 == 은 아래와 같다.

```
  /// The equality operator.
  ///
  /// The default behavior for all [Object]s is to return true if and
  /// only if this object and [other] are the same object.
  ///
  /// Override this method to specify a different equality relation on
  /// a class. The overriding method must still be an equivalence relation.
  /// That is, it must be:
  ///
  ///  * Total: It must return a boolean for all arguments. It should never throw.
  ///
  ///  * Reflexive: For all objects `o`, `o == o` must be true.
  ///
  ///  * Symmetric: For all objects `o1` and `o2`, `o1 == o2` and `o2 == o1` must
  ///    either both be true, or both be false.
  ///
  ///  * Transitive: For all objects `o1`, `o2`, and `o3`, if `o1 == o2` and
  ///    `o2 == o3` are true, then `o1 == o3` must be true.
  ///
  /// The method should also be consistent over time,
  /// so whether two objects are equal should only change
  /// if at least one of the objects was modified.
  ///
  /// If a subclass overrides the equality operator, it should override
  /// the [hashCode] method as well to maintain consistency.
  external bool operator ==(Object other);
```

`The default behavior for all [Object]s is to return true if and only if this object and [other] are the same object.` 핵심은 이 문구다. 주소값이 같아야 Object 의 == 는 true 를 return 한다.

종합해서 보면 dart 에서 특별히 == 을 override 하지 않는 이상, 주소값이 같아야만 == 에서 true 를 받을 수 있고 그 외에는 모두 false 이다.

또 == 과 밀접한 연관이 있는(== 에 영향을 주는) hashCode 를 살펴보자.

```
  /// The hash code for this object.
  ///
  /// A hash code is a single integer which represents the state of the object
  /// that affects [operator ==] comparisons.
  ///
  /// All objects have hash codes.
  /// The default hash code implemented by [Object]
  /// represents only the identity of the object,
  /// the same way as the default [operator ==] implementation only considers objects
  /// equal if they are identical (see [identityHashCode]).
  ///
  /// If [operator ==] is overridden to use the object state instead,
  /// the hash code must also be changed to represent that state,
  /// otherwise the object cannot be used in hash based data structures
  /// like the default [Set] and [Map] implementations.
  ///
  /// Hash codes must be the same for objects that are equal to each other
  /// according to [operator ==].
  /// The hash code of an object should only change if the object changes
  /// in a way that affects equality.
  /// There are no further requirements for the hash codes.
  /// They need not be consistent between executions of the same program
  /// and there are no distribution guarantees.
  ///
  /// Objects that are not equal are allowed to have the same hash code.
  /// It is even technically allowed that all instances have the same hash code,
  /// but if clashes happen too often,
  /// it may reduce the efficiency of hash-based data structures
  /// like [HashSet] or [HashMap].
  ///
  /// If a subclass overrides [hashCode], it should override the
  /// [operator ==] operator as well to maintain consistency.
  external int get hashCode;
```

`If a subclass overrides [hashCode], it should override the [operator ==] operator as well to maintain consistency.` 라는 것을 보면 == 를 override 할 경우 반드시 hashCode 도 override 해줘야 하는 것을 알 수 있다.

\


### Equatable 에서는 == 과 hashCode 를 override 한다 <a href="#equatable-hashcode-override" id="equatable-hashcode-override"></a>

아래는 Equatable 내에 있는 == 와 hashCode 이다. 이를 보면 Equatable 를 상속할 경우 Object 의 ==, hashCode 를 사용하지 않고, 아래의 Equatable 의 ==, hashCode 를 사용하게 된다는 것을 알 수 있다.

```dart
  @override
  bool operator ==(Object other) =>
      identical(this, other) ||
      other is Equatable &&
          runtimeType == other.runtimeType &&
          equals(props, other.props);

  @override
  int get hashCode => runtimeType.hashCode ^ mapPropsToHashCode(props);
```

이를 바탕으로 Equatable 을 상속하고 있는 TodosState 을 살펴보자.

```dart
    Todo todo1 = Todo(id: '1', description: 'test');
    Todo todo2 = Todo(id: '2', description: 'test');
    List<Todo> todos1 = [todo1, todo2];
    List<Todo> todos2 = [todo1, todo2];
    print(todos1.hashCode); // 807548389
    print(todos2.hashCode); // 639492342

    TodosState todosState1 = TodosState(todos: todos1);
    TodosState todosState2 = TodosState(todos: todos2);
    print(todosState1.hashCode); // 556324449
    print(todosState2.hashCode); // 556324449
    print(todosState1 == todosState2); // true
```

위와 같이 todos 자체는 다른 주소값을 가지고 있어도 해당 todos 의 구성이 todo1, todo2로 같기에 todosState1, todosState2 의 hashCode 는 같은 값을 return 하고 비교 역시 true 가 return 된다.

이번엔 원소의 구성에 변화를 줘보자.

```dart
    Todo todo1 = Todo(id: '1', description: 'test');
    Todo todo2 = Todo(id: '2', description: 'test');
    List<Todo> todos = [todo1, todo2];

    print(todos.length); // 2
    print(todos.hashCode); // 960134229
    TodosState todosState1 = TodosState(todos: todos);
    print(todosState1.hashCode); // 592161286

    todos.removeWhere((element) => element.id == '1');
    print(todos.length); // 1
    print(todos.hashCode); // 960134229
    TodosState todosState2 = TodosState(todos: todos);
    print(todosState1.hashCode); // 680645052
    print(todosState2.hashCode); // 680645052

    print(todosState1 == todosState2); // true
```

결국 최종적으로 todosState1, todosState2 각각의 원소는 todos 라는 똑같은 인스턴스다. 그래서 todos 가 어떻게 변하든지간에 해당 주소값은 동일하다. 그래서 todosState1, todosState2 는 모두 똑같은 todos 라는 인스턴스를 원소로 갖고 있으므로 항상 같을 수 밖에 없다.

내가 범한 실수는 이 포인트에서 나오는데, old state 과 new state 을 지금의 예시에서 todosState1, todosState2 로 뒀다. 즉, StateNotifier 에서 custom method 를 통해 `state 에 변화를 줘야` 하는데 state 자체는 그대로 두고 state 내의 원소만 바꾼 것이다.

다시 말해, `state 에 변화` 를 준다는 것은 identical 판정에 대해 완전히 변화를 주기 위해서 주소값 변경까지 고려했어야 하는데 동일한 객체를 두고 내부 원소만 바꿔버리니 위 예시처럼 변화를 줬다한들 결국 같은 object 가 된 것이다. todosState1 에서 todosState2 와 같이 바꿨지만 결국 같은 object 인 것이다.

그래서 state 를 다룰때는 완전히 immutable object 로 새로 만들어내야 한다. 그것이 생각하기도 편하고 버그를 줄이는 방법이다. 그럼 내가 실수한 코드를 보자.

```dart
// state 가 변경되었다고 인식되지 않음
void removeTodo(String removeTargetTodoId) {
  print('before : ${state.todos.length}');
  state.todos.removeWhere((todo) => todo.id == removeTargetTodoId);
  print('after : ${state.todos.length}');

  final List<Todo> todos = [...state.todos];
  state = state.copyWith(todos);
}

// 잘 작동하는 코드
void removeTodo(String removeTargetTodoId){
final List<Todo> todos = [...state.todos.where((todo) => todo.id != removeTargetTodoId).toList()];
state = state.copyWith(todos);
}
```

잘못된 코드를 보면 결국 removeTodo() 함수가 동작하기 전과 후는 `state 가 가진 todos 의 내용은 바뀌었을지언정, todos 자체는 그대로`이기 때문에 결국 state 의 변화는 발생하지 않았다고 판정된다. 고쳐진 코드에서는 removeWhere 이 아니라 where 을 통해서 필요한 원소들을 찾은 후 toList() 로 완전히 다른 todos 를 만들어서 결국 state 를 바꾸고 있다.

추가로 Equatable 과 관련하여 참고하기 좋았던 [포스팅](https://coflutter.com/dart-how-to-compare-2-objects/) 링크를 남긴다.
