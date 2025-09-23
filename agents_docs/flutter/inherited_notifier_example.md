# Overview
This document contains an example implementation of a feature using InheritedNotifier. It’s useful when an InheritedWidget only needs to store a T value coming from a single Listenable<T>.

Architecture:
```
├── bottom_bar_scope.dart
├── bottom_bar_screen.dart
├── components                                    # additional widgets if needed
└── controllers
    └── bottom_bar_state_controller.dart
```

BottomBarScope
```dart
class BottomBarScope extends InheritedNotifier<BottomBarStateController> {
  const BottomBarScope({
    super.key,
    required super.child,
    required BottomBarStateController super.notifier,
  });

  static int indexOf(BuildContext context, {bool listen = true}) => of(context, listen: listen).notifier!.index;

  static void setIndex(BuildContext context, int newIndex) => of(context, listen: false).notifier!.setIndex(newIndex);

  static final navigators = <AppNavigator>[
    AppNavigator(pages: [Routes.example.page()]),
    AppNavigator(pages: [Routes.example.page()]),
    AppNavigator(pages: [Routes.example.page()]),
    AppNavigator(pages: [Routes.example.page()]),
  ];

  static BottomBarScope of(BuildContext context, {bool listen = true}) {
    final BottomBarScope? result = listen
        ? context.dependOnInheritedWidgetOfExactType<BottomBarScope>()
        : context.getInheritedWidgetOfExactType<BottomBarScope>();
    if (result == null) throwScopeError(context, name: 'BottomBar');
    return result;
  }
}
```

BottomBarScreen
```dart

class BottomBarScreen extends StatefulWidget {
  const BottomBarScreen({super.key});

  @override
  State<BottomBarScreen> createState() => _BottomBarScreenState();
}

class _BottomBarScreenState extends State<BottomBarScreen> {
  final _controller = BottomBarStateController();

  @override
  Widget build(BuildContext context) {
    return BottomBarScope(
      notifier: _controller,
      child: const _BottomBarUI(),
    );
  }
}

class _BottomBarUI extends StatelessWidget {
  const _BottomBarUI({
    // ignore: unused_element_parameter
    super.key,
  });

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Stack(
        children: BottomBarScope.navigators.indexed.map<Widget>(
          (item) {
            final (index, navigator) = item;
            final isActive = index == BottomBarScope.indexOf(context);

            return Offstage(
              offstage: !isActive,
              child: TickerMode(
                enabled: isActive,
                child: navigator,
              ),
            );
          },
        ).toList(),
      ),
      bottomNavigationBar: NavigationBar(
        selectedIndex: BottomBarScope.indexOf(context),
        onDestinationSelected: (value) => BottomBarScope.setIndex(context, value),
        destinations: const [
          NavigationDestination(
            label: 'Example',
            icon: Icon(Icons.apps_outlined),
          ),
          NavigationDestination(
            label: 'Example',
            icon: Icon(Icons.apps_outlined),
          ),
          NavigationDestination(
            label: 'Example',
            icon: Icon(Icons.apps_outlined),
          ),
          NavigationDestination(
            label: 'Example',
            icon: Icon(Icons.apps_outlined),
          ),
        ],
      ),
    );
  }
}

```

BottomBarStateController
```dart
final class BottomBarStateController extends ValueNotifier<int> {
  BottomBarStateController() : super(0);

  void setIndex(int newIndex) => value = newIndex;

  int get index => value;
}
```