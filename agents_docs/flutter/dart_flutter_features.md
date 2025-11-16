## How to read this file

- Start with the version sections below; each one links to the upstream doc and gives an at-a-glance checklist plus a runnable snippet.
- Keep the original table (at the bottom) for fast link lookups. The prose above it should stay the source of truth for rollups and examples.
- When updating this document, prefer short subsections that pair a “why it matters” bullet list with a concrete code sample.
- When reading links, use `chrome devtools` mcp for opening pages; if you encounter error while starting chrome, stop and tell user to fix that.

## Release highlights

### Dart 3.10 — Ergonomics & tooling

#### Dot shorthands (`.foo`, `.new()`, `const .name()`)

- Lets you omit the type when referring to enum values, static members, or constructors as long as the surrounding code already has a concrete type context.
- Works in assignments, `switch` expressions, and other places where the analyzer can infer the exact type; it intentionally refuses to compile when the context becomes ambiguous (for example, after casting to `Object` or at the start of a statement).
- Keeps constant expressions valid by supporting `const .MyWidget()` as long as the context expects the same type you would normally write out fully.

```dart
enum Status { idle, running, paused }

class Point {
  const Point.origin();
}

Status state = .running;            // expands to Status.running
int port = .parse('8080');          // expands to int.parse('8080')
const Point origin = const .origin();
const bool traceLogging = const .fromEnvironment('TRACE');

void logStatus(Status next) {
  switch (next) {
    case .idle:
      // The type context from `next` keeps this valid.
      break;
    case .running:
      // ... handle active work
      break;
    case .paused:
      break;
  }
}
```

**Safety rails to remember:** provide an explicit context when chaining (e.g., `(.red == .blue)` fails if the context widens to `Object`), and never start a standalone expression with `.` because the parser cannot infer which type you meant.

#### Analyzer plugins (custom lint passes)

- Dart 3.10 ships the first stable analyzer plugin API so packages can contribute bespoke diagnostics that show up in IDEs and on `dart analyze`.
- Plugins are regular Dart packages that depend on `analyzer_plugin`, expose a `PluginStarter`, and are loaded through the new top-level `plugins:` stanza in `analysis_options.yaml`.
- Ideal for enforcing company- or package-specific rules (widget theming, localization coverage, async policies) without waiting for upstream lints.

```yaml
# analysis_options.yaml
include: package:flutter_lints/flutter.yaml

plugins:
  dart_code_metrics: ^6.0.0
  custom_localization_rules:
    enabled: true
    severity: warning

custom_localization_rules:
  requireLookup: true
```

```dart
// bin/plugin.dart
class LocalizationPlugin extends ServerPlugin {
  @override
  void contentChanged(List<String> _) {}

  @override
  AnalysisDriverGeneric createAnalysisDriver(ContextRoot root) {
    return LocalizationDriver(root);
  }
}
```

Wire the plugin into your package (or a separate repo), publish it, then point consuming apps at it via `plugins:`—no CLI wrappers required.

#### Build hooks (native assets, now stable)

- Build hooks are Dart scripts stored inside a package’s `hook/` directory. The SDK discovers them automatically and runs them alongside Dart compilation so you can compile/download native artifacts without wiring Gradle/CMake projects.
- Hooks receive strongly typed `BuildInput`/`BuildOutputBuilder` objects via the `package:hooks` API. That gives access to metadata such as the target platform plus a shared output directory for emitting assets.
- Because hooks run in dependency order, libraries can chain work: a leaf package can consume artifacts produced by its dependencies’ hooks.

```yaml
# pubspec.yaml
name: native_add_library
description: Sums two numbers with native code.
version: 0.1.0

environment:
  sdk: ^3.10.0

dependencies:
  hooks: any
  code_assets: any
  native_toolchain_c: any

dev_dependencies:
  ffigen: ^18.0.0
```

```dart
// hook/build.dart
import 'package:hooks/hooks.dart';
import 'package:logging/logging.dart';
import 'package:native_toolchain_c/native_toolchain_c.dart';

Future<void> main(List<String> args) async {
  await build(args, (input, output) async {
    final package = input.packageName;
    final builder = CBuilder.library(
      name: package,
      assetName: '$package.dart',
      sources: ['src/$package.c'],
    );

    await builder.run(
      input: input,
      output: output,
      logger: Logger(package)..onRecord.listen((record) => print(record.message)),
    );
  });
}
```

The builder writes compiled artifacts into the shared output directory exposed by `BuildInput`, and your Dart code can import them via the generated `code_assets` bindings.

### Dart 3.8 — Null-aware collection elements

- Any list/set/map literal can guard individual entries with `...?expression` or pattern guards so you stop sprinkling `if (x != null)` blocks around the literal.
- Works together with `if`/`for` elements and pattern matching, which keeps builder-style collection code declarative.

```dart
String? maybeLabel = fetchLabel();
List<String>? extras = fetchExtras();

final chips = [
  'Required',
  if (maybeLabel case final label?) label,
  ...?extras,
  for (final task in tasks) if (task.owner case final owner?) owner,
];
```

### Dart 3.3 — Extension types

- Extension types create lightweight wrappers (no runtime allocation) around an underlying representation so you can add APIs and static typing guarantees without paying the cost of a separate class.
- Perfect for “value object” concepts—IDs, validated strings, foreign handles—because you control the surface area and can implement interfaces/mixins just like a class.

```dart
extension type AccountId(String value) implements Comparable<AccountId> {
  const AccountId._(this.value);

  factory AccountId.parse(String raw) {
    if (!RegExp(r'^acct_[0-9a-f]{8}$').hasMatch(raw)) {
      throw FormatException('Bad account id: $raw');
    }
    return AccountId._(raw);
  }

  @override
  int compareTo(AccountId other) => value.compareTo(other.value);
}

void archive(AccountId id) {
  // `id` is guaranteed valid without leaking the underlying string.
}
```

## Quick link index (original table)

| SDK       | Feature             | Remote document                                          |
|-----------|---------------------|----------------------------------------------------------|
| Dart 3.10 | Dot shorthand       | https://dart.dev/language/dot-shorthands                 |
| Dart 3.10 | Analyzer plugin     | https://dart.dev/tools/analyzer-plugins                  |
| Dart 3.10 | Hooks               | https://dart.dev/tools/hooks                             |
| Dart 3.8  | Null-aware elements | https://dart.dev/language/collections#null-aware-element |
| Dart 3.3  | Extension type      | https://dart.dev/language/extension-types                |
