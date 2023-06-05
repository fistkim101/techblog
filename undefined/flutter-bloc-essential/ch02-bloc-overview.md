# CH02 Bloc Overview

\
&#x20;                                          &#x20;

<figure><img src="https://fistkim101.github.io/images/Section+Overview-page-001.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Section+Overview-page-002.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Section+Overview-page-003.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Section+Overview-page-004.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Section+Overview-page-005.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Section+Overview-page-006.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Section+Overview-page-007.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Section+Overview-page-008.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Section+Overview-page-009.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Section+Overview-page-010.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Section+Overview-page-011.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Section+Overview-page-012.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Section+Overview-page-013.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/injecting+cubits+or+blocs-page-001.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/injecting+cubits+or+blocs-page-002.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/BlocBuilder-page-001.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/BlocListener+and+BlocConsumer-page-001.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/BlocListener+and+BlocConsumer-page-002.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/BlocListener+and+BlocConsumer-page-003.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/BlocListener+and+BlocConsumer-page-004.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/BlocListener+and+BlocConsumer-page-005.jpg" alt=""><figcaption></figcaption></figure>

다수의 Bloc 을 구독하여 이 결과들을 복합적으로 참조하여 새로운 결과를 만들어야할 경우 아래와 같이 MultiBlocListener를 사용할 수 있다.

```dart
import 'package:bloc_sample_todo_app/blocs/filtered_todos/filtered_todos_bloc.dart';
import 'package:bloc_sample_todo_app/blocs/search_term/search_term_bloc.dart';
import 'package:bloc_sample_todo_app/blocs/selected_filter/selected_filter_bloc.dart';
import 'package:bloc_sample_todo_app/blocs/todos/todos_bloc.dart';
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

import '../widgets/widgets.dart';

class TodoList extends StatefulWidget {
  const TodoList({Key? key}) : super(key: key);

  @override
  State<TodoList> createState() => _TodoListState();
}

class _TodoListState extends State<TodoList> {
  @override
  Widget build(BuildContext context) {
    return MultiBlocListener(
      listeners: [
        BlocListener<TodosBloc, TodosState>(listener: (context, state) {
          context.read<FilteredTodosBloc>().add(CalculateFilteredTodosEvent(
                currentTodos: state.todos,
                selectedFilter:
                    context.read<SelectedFilterBloc>().state.selectedFilter,
                searchTerm: context.read<SearchTermBloc>().state.searchTerm,
              ));
        }),
        BlocListener<SearchTermBloc, SearchTermState>(
            listener: (context, state) {
          context.read<FilteredTodosBloc>().add(CalculateFilteredTodosEvent(
                currentTodos: context.read<TodosBloc>().state.todos,
                selectedFilter:
                    context.read<SelectedFilterBloc>().state.selectedFilter,
                searchTerm: state.searchTerm,
              ));
        }),
        BlocListener<SelectedFilterBloc, SelectedFilterState>(
            listener: (context, state) {
          context.read<FilteredTodosBloc>().add(CalculateFilteredTodosEvent(
                currentTodos: context.read<TodosBloc>().state.todos,
                selectedFilter: state.selectedFilter,
                searchTerm: context.read<SearchTermBloc>().state.searchTerm,
              ));
        }),
      ],
      child: BlocBuilder<FilteredTodosBloc, FilteredTodosState>(
        builder: (context, state) {
          return ListView.separated(
            primary: false,
            shrinkWrap: true,
            separatorBuilder: (_, __) {
              return const Divider(color: Colors.grey);
            },
            itemCount: state.filteredTodos.length,
            itemBuilder: (_, index) =>
                TodoItem(todo: state.filteredTodos[index]),
          );
        },
      ),
    );
  }
}

```

```dart
import 'package:bloc/bloc.dart';
import 'package:bloc_sample_todo_app/domain/models/models.dart';
import 'package:bloc_sample_todo_app/enums/enums.dart';
import 'package:equatable/equatable.dart';

part 'filtered_todos_event.dart';
part 'filtered_todos_state.dart';

class FilteredTodosBloc extends Bloc<FilteredTodosEvent, FilteredTodosState> {
  FilteredTodosBloc() : super(FilteredTodosState.initial()) {
    on<CalculateFilteredTodosEvent>((event, emit) {
      final List<TodoModel> _currentTodos = event.currentTodos;
      final FilterType _selectedFilter = event.selectedFilter;
      final String _searchTerm = event.searchTerm;

      List<TodoModel> _filteredTodos;

      switch (_selectedFilter) {
        case FilterType.active:
          _filteredTodos = _currentTodos
              .where((TodoModel todo) => !todo.isCompleted)
              .toList();
          break;
        case FilterType.completed:
          _filteredTodos = _currentTodos
              .where((TodoModel todo) => todo.isCompleted)
              .toList();
          break;
        case FilterType.all:
        default:
          _filteredTodos = _currentTodos;
          break;
      }

      if (_searchTerm.isNotEmpty) {
        _filteredTodos = _filteredTodos
            .where((TodoModel todo) => todo.description
                .toLowerCase()
                .contains(_searchTerm.toLowerCase()))
            .toList();
      }

      emit(state.copyWith(filteredTodos: _filteredTodos));
    });
  }
}

```



<figure><img src="https://fistkim101.github.io/images/BlocListener+and+BlocConsumer-page-006.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/BuildContext+extension+methods-page-001.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/BuildContext+extension+methods-page-002.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/BuildContext+extension+methods-page-003.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/BuildContext+extension+methods-page-004.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/BuildContext+extension+methods-page-005.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/BuildContext+extension+methods-page-006.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Bloc+Access+-+context-page-001.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Bloc+Access+-+context-page-002.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Bloc+Access+-+context-page-003.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/19+Observing+Cubit+and+Blocs(new)-page-001.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/19+Observing+Cubit+and+Blocs(new)-page-002.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/19+Observing+Cubit+and+Blocs(new)-page-003.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/19+Observing+Cubit+and+Blocs(new)-page-004.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/19+Observing+Cubit+and+Blocs(new)-page-005.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/19+Observing+Cubit+and+Blocs(new)-page-006.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/19+Observing+Cubit+and+Blocs(new)-page-007.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/19+Observing+Cubit+and+Blocs(new)-page-008.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/19+Observing+Cubit+and+Blocs(new)-page-009.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Event+Transformer-page-001.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Event+Transformer-page-002.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Event+Transformer-page-003.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Event+Transformer-page-004.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Event+Transformer-page-005.jpg" alt=""><figcaption></figcaption></figure>

```dart
import 'package:bloc/bloc.dart';
import 'package:bloc_concurrency/bloc_concurrency.dart';
import 'package:equatable/equatable.dart';

part 'counter_event.dart';
part 'counter_state.dart';

class CounterBloc extends Bloc<CounterEvent, CounterState> {
  CounterBloc() : super(CounterState.initial()) {
    // on<IncrementCounterEvent>(
    //   _handleIncrementCounterEvent,
    //   transformer: sequential(),
    // );

    // on<DecrementCounterEvent>(
    //   _handleDecrementCounterEvent,
    //   transformer: sequential(),
    // );

    on<CounterEvent>(
      (event, emit) async {
        if (event is IncrementCounterEvent) {
          await _handleIncrementCounterEvent(event, emit);
        } else if (event is DecrementCounterEvent) {
          await _handleDecrementCounterEvent(event, emit);
        }
      },
      transformer: sequential(),
    );
  }

  Future<void> _handleIncrementCounterEvent(event, emit) async {
    await Future.delayed(Duration(seconds: 4));
    emit(state.copyWith(counter: state.counter + 1));
  }

  Future<void> _handleDecrementCounterEvent(event, emit) async {
    await Future.delayed(Duration(seconds: 2));
    emit(state.copyWith(counter: state.counter - 1));
  }
}
```

&#x20;           &#x20;

<figure><img src="https://fistkim101.github.io/images/24+Hydrated+Bloc+(9.0)-page-001.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/24+Hydrated+Bloc+(9.0)-page-002.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/24+Hydrated+Bloc+(9.0)-page-003.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/24+Hydrated+Bloc+(9.0)-page-004.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/24+Hydrated+Bloc+(9.0)-page-005.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/24+Hydrated+Bloc+(9.0)-page-006.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/24+Hydrated+Bloc+(9.0)-page-007.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/RepositoryProvider-page-001.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/RepositoryProvider-page-002.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/RepositoryProvider-page-003.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/RepositoryProvider-page-004.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/RepositoryProvider-page-005.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Cubit+vs+Bloc-page-001.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/Cubit+vs+Bloc-page-002.jpg" alt=""><figcaption></figcaption></figure>

## Bloc 에서 언제 state 가 바뀌나

결론부터 말하자면 emit 과정에서 state 를 갈아치운다. (기존 state와 비교하여 달라진 state 가 들어온 경우에만) Bloc\<Event, State>가 BlocBase\<State>를 상속하고 있고 BlocBase\<State>가 state를 필드로 갖고 있는 상태이다. &#x20;

코드에서 확인할 수 있듯이 Bloc\<Event, State>의 emit 은 결국 super.emit(state)로 결국 BlocBase\<State>의 emit 을 호출하는 것이며 여기서 필드로 관리중인 기존 state 와 비교 후 다르면 새로운 state 을 할당해주는 방식으로 동작한다.

```dart
abstract class Bloc<Event, State> extends BlocBase<State>
    implements BlocEventSink<Event> {
  
  ...
      
  @visibleForTesting
  @override
  void emit(State state) => super.emit(state);

  ...
    
  }
```

```dart
abstract class BlocBase<State>
    implements StateStreamableSource<State>, Emittable<State>, ErrorSink {
 
 ...
 
   /// Updates the [state] to the provided [state].
  /// [emit] does nothing if the [state] being emitted
  /// is equal to the current [state].
  ///
  /// To allow for the possibility of notifying listeners of the initial state,
  /// emitting a state which is equal to the initial state is allowed as long
  /// as it is the first thing emitted by the instance.
  ///
  /// * Throws a [StateError] if the bloc is closed.
  @protected
  @visibleForTesting
  @override
  void emit(State state) {
    try {
      if (isClosed) {
        throw StateError('Cannot emit new states after calling close');
      }
      if (state == _state && _emitted) return;
      onChange(Change<State>(currentState: this.state, nextState: state));
      _state = state;
      _stateController.add(_state);
      _emitted = true;
    } catch (error, stackTrace) {
      onError(error, stackTrace);
      rethrow;
    }
  }

...
       
}    
```

## Bloc 에서 언제 구독 위젯이 state 변화를 알게 되는가

state 변화에 대해서 이를 구독하고 있는 위젯에 변화를 알리는 것은 핸들러 등록 부분에서 발생한다. 아래 코드를 보자.

````dart
  /// Register event handler for an event of type `E`.
  /// There should only ever be one event handler per event type `E`.
  ///
  /// ```dart
  /// abstract class CounterEvent {}
  /// class CounterIncrementPressed extends CounterEvent {}
  ///
  /// class CounterBloc extends Bloc<CounterEvent, int> {
  ///   CounterBloc() : super(0) {
  ///     on<CounterIncrementPressed>((event, emit) => emit(state + 1));
  ///   }
  /// }
  /// ```
  ///
  /// * A [StateError] will be thrown if there are multiple event handlers
  /// registered for the same type `E`.
  ///
  /// By default, events will be processed concurrently.
  ///
  /// See also:
  ///
  /// * [EventTransformer] to customize how events are processed.
  /// * [package:bloc_concurrency](https://pub.dev/packages/bloc_concurrency) for an
  /// opinionated set of event transformers.
  ///
  void on<E extends Event>(
    EventHandler<E, State> handler, {
    EventTransformer<E>? transformer,
  }) {
    assert(() {
      final handlerExists = _handlers.any((handler) => handler.type == E);
      if (handlerExists) {
        throw StateError(
          'on<$E> was called multiple times. '
          'There should only be a single event handler per event type.',
        );
      }
      _handlers.add(_Handler(isType: (dynamic e) => e is E, type: E));
      return true;
    }());

    final _transformer = transformer ?? _eventTransformer;
    final subscription = _transformer(
      _eventController.stream.where((event) => event is E).cast<E>(),
      (dynamic event) {
        void onEmit(State state) {
          if (isClosed) return;
          if (this.state == state && _emitted) return;
          onTransition(Transition(
            currentState: this.state,
            event: event as E,
            nextState: state,
          ));
          emit(state);
        }

        final emitter = _Emitter(onEmit);
        final controller = StreamController<E>.broadcast(
          sync: true,
          onCancel: emitter.cancel,
        );

        void handleEvent() async {
          void onDone() {
            emitter.complete();
            _emitters.remove(emitter);
            if (!controller.isClosed) controller.close();
          }

          try {
            _emitters.add(emitter);
            await handler(event as E, emitter);
          } catch (error, stackTrace) {
            onError(error, stackTrace);
            rethrow;
          } finally {
            onDone();
          }
        }

        handleEvent();
        return controller.stream;
      },
    ).listen(null);
    _subscriptions.add(subscription);
  }
````
