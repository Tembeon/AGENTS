# Overview
This document contains an example implementation of a feature using InheritedModel. It’s useful when an InheritedWidget needs to store multiple values that widgets can subscribe to independently.

Architecture:
```
├── controllers
│   ├── example_scope_controller.dart
│   └── example_state_controller.dart
├── components                            # additional widgets if needed
├── example_scope.dart
└── example_screen.dart
```

ExampleScope
```dart
class ExampleScope extends StatefulWidget {
  const ExampleScope({
    super.key,
    required ExampleStateController stateController,
    required this.child,
  }) : _stateController = stateController;

  final ExampleStateController _stateController;

  final Widget child;

  static IExampleScopeController scopeControllerOf(BuildContext context, {bool listen = false}) =>
      _ExampleInherited.of(context, aspect: _ExampleAspect.scopeController, listen: listen).exampleScopeController;

  static ExampleState stateOf(BuildContext context, {bool listen = true}) =>
      _ExampleInherited.of(context, aspect: _ExampleAspect.exampleState, listen: listen).exampleState;

  static ExampleStateController stateControllerOf(BuildContext context, {bool listen = false}) =>
      _ExampleInherited.of(context, aspect: _ExampleAspect.stateController, listen: listen).exampleStateController;

  @override
  State<ExampleScope> createState() => _ExampleScopeState();
}

class _ExampleScopeState extends State<ExampleScope> implements IExampleScopeController {
  @override
  Widget build(BuildContext context) {
    return StateConsumer<ExampleStateController, ExampleState>( // some kind of ListenableBuilder
      controller: widget._stateController,
      child: widget.child,
      builder: (context, state, child) {
        return _ExampleInherited(
          exampleState: state,
          exampleScopeController: this,
          exampleStateController: widget._stateController,
          child: child!,
        );
      },
    );
  }

  @override
  Future<void> requestLoad() {
    return widget._stateController.create(id: 1);
  }
}

enum _ExampleAspect {
  exampleState,
  scopeController,
  stateController,
}

class _ExampleInherited extends InheritedModel<_ExampleAspect> {
  const _ExampleInherited({
    // ignore: unused_element, unused_element_parameter
    super.key,
    required super.child,
    required this.exampleState,
    required this.exampleScopeController,
    required this.exampleStateController,
  });

  final ExampleState exampleState;

  final IExampleScopeController exampleScopeController;

  final ExampleStateController exampleStateController;

  static _ExampleInherited of(BuildContext context, {_ExampleAspect? aspect, bool listen = true}) {
    final _ExampleInherited? result = switch (listen) {
      true => context.dependOnInheritedWidgetOfExactType<_ExampleInherited>(aspect: aspect),
      false => context.getElementForInheritedWidgetOfExactType<_ExampleInherited>()?.widget as _ExampleInherited?,
    };

    if (result == null) {
     throw FlutterError.fromParts(<DiagnosticsNode>[
        ErrorSummary('No ExampleScope widget ancestor found.'),
        ErrorDescription('${context.widget.runtimeType} widgets require a ExampleScope widget ancestor.'),
        context.describeWidget('The specific widget that could not find a ExampleScope ancestor was'),
        context.describeOwnershipChain('The ownership chain for the affected widget is'),
      ]);
    }
    return result;
  }

  @override
  void debugFillProperties(DiagnosticPropertiesBuilder properties) {
    super.debugFillProperties(
      properties
        ..add(DiagnosticsProperty<ExampleState>('exampleState', exampleState))
        ..add(DiagnosticsProperty<IExampleScopeController>('exampleScopeController', exampleScopeController))
        ..add(DiagnosticsProperty<ExampleStateController>('exampleStateController', exampleStateController))
        ,
    );
  }

  @override
  bool updateShouldNotify(_ExampleInherited old) =>
      exampleState != old.exampleState || exampleScopeController != old.exampleScopeController
      || exampleStateController != old.exampleStateController;

  @override
  bool updateShouldNotifyDependent(covariant _ExampleInherited oldWidget, Set<_ExampleAspect> dependencies) =>
    dependencies.any(
        (element) => switch (element) {
          _ExampleAspect.scopeController => oldWidget.exampleScopeController != exampleScopeController,
          _ExampleAspect.stateController => oldWidget.exampleStateController != exampleStateController,
          _ExampleAspect.exampleState => oldWidget.exampleState != exampleState,
        },
      );
}
```

ExampleScreen
```dart
class ExampleScreen extends StatefulWidget {
  const ExampleScreen({super.key});

  @override
  State<ExampleScreen> createState() => _ExampleScreenState();
}

class _ExampleScreenState extends State<ExampleScreen> {
  late final _stateController = ExampleStateController();

  @override
  Widget build(BuildContext context) {
    return ExampleScope(
      stateController: _stateController,
      child: const _ExampleUI(),
    );
  }
}
```

ExampleScopeController
Purpose — either general service methods, or methods that encapsulate private data internally while exposing a convenient external API for interacting with the StateController.
```dart
abstract interface class IExampleScopeController {
  Future<void> requestLoad();
}
```

ExampleStateController
```dart
final class ExampleState {
  const ExampleState({this.stateType = StateType.idle, this.data, this.error});

  final StateType stateType;

  final Object? data;

  final Object? error;
}

base class ExampleStateController extends StateController<ExampleState> with SequentialControllerHandler {
  ExampleStateController({
    super.initialState = const ExampleState(),
  });

  Future<void> create({
    required final int id,
  }) => handle(
    () async {
      setState(state.copyWith(stateType: () => StateType.loading));
      // some async logic
      setState(state.copyWith(data: ()=> _asyncData.body));
    },
    done: () async => setState(state.copyWith(stateType: () => StateType.idle)),
    error: (error, stack) async => setState(state.copyWith(error: () => error)),
    name: 'create $id',
  );
}

```