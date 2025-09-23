## File Purpose
This document describes general guidelines for a Flutter or Dart project. A project may additionally include other languages; in that case (if guidelines exist), use the supplementary documents for that specific language.

## Basic Rules
- Before deciding to “write code,” consider alternative solutions, choose the best one, and implement it.
- Do not edit code-generated files (those ending with `*.g.dart`). Ask the user to run `dart build_runner` after making changes to related source files.
- Public APIs must be documented with comments following Effective Dart doc style. Use `@template`, `@macro`, and `@docImport 'dart:async'` for documentation-only imports. If you are developing code of above-average complexity, explain in comments why the chosen approach was taken.
- Use the pub-sub pattern for feature design unless specified otherwise. See the “Feature Design Example” section in this document.
- For every data class, always implement: `hashCode`, `operator ==`, and `copyWith`. Implement `copyWith` with `ValueGetter<T?>?` so that you can set a value to `null` and distinguish “passed null” from “not provided.” Typically, projects provide an extension for working with `ValueGetter` so `copyWith` logic reduces to `newGetter.or(this.oldGetter)`. If such an extension is missing, you can implement it yourself; the full code is:
```dart
extension ValueGetterX<T> on ValueGetter<T>? {

  /// Used to choose between two values:
  /// either the result of this function (if provided) or [current].
  ///
  /// Commonly used in `copyWith` constructions involving [ValueGetter]:
  ///
  /// ```dart
  ///   SomeState copyWith({
  ///     ValueGetter<StateType>? stateType,
  ///     ValueGetter<Entity?>? entity,
  ///     ValueGetter<Object?>? error,
  ///   }) => SomeState(
  ///     stateType: stateType.or(this.stateType),
  ///     entity: entity.or(this.entity),
  ///     error: error.or(this.error),
  ///   );
  /// ```
  T or(T current) => this == null ? current : this!();
}
```
- After modifying a file, always run formatting and analysis: `dart format --line-length=120 . && dart analyze .`
- After all work is complete, along with your message to the person, propose a Git commit message that follows standard change-description conventions.
- If the `build` method becomes too long, extract parts of the UI into separate reusable widget classes.
- The `build` method is a pure function. All code inside it must concern UI construction only, with no side effects
- Encapsulate all business logic in separate services or state machines so that the UI layer remains as clean as possible.
- Explicitly use await for asynchronous functions. Otherwise, errors from such functions will escape to the zone and will be hard to handle correctly
- Avoid magic numbers
-	Add logging for critical actions. If code handles errors and suppresses them (which is usually incorrect), log such cases at warning level or higher.
- Do not handle errors merely for the sake of handling them. Errors are of two types: expected and unexpected. If an error is expected, the UI should have a specific error state; or the code should have dedicated, well-considered logic for that specific exception. If an error is unexpected, treat it as such—do not deliberately handle or suppress it. Prefer to propagate errors upward (using rethrow or Error.throwWithStackTrace). If your code boils down to `try { /* danger */ } catch (e) { print(e); }`, reconsider whether that exception handling is needed.


## Architecture
You can find the feature-based architecture description in [this document](feature_based_architecture.md).

## Feature Design Example
You can find the guidelines for designing a new feature in [this document](how_to_new_feature.md).
