## Purpose

Use this playbook whenever you touch Flutter or pure Dart code. It now front-loads the prep work (so you always read the
latest Dart/Flutter feature notes), then routes you to deeper documents, and finally summarizes the non‑negotiable
standards.

---

## Prep Checklist (run before editing)

1. Open [`dart_flutter_features.md`](dart_flutter_features.md) every session. Those release highlights are part of the
   required context, not optional reading. Apply them proactively (dot shorthand in enum-heavy code, analyzer plugins
   for custom linting, hooks for native assets, null-aware collection builders, and extension types for value objects).
2. Skim the task-specific doc from the routing table below.
3. Only after steps 1–2 start changing code, then finish with `dart format`/`dart analyze` via the Dart MCP tool.

### Feature reminders from `dart_flutter_features.md`

- **Dot shorthands** keep enums/constructors terse; provide an explicit type context whenever inference could widen to
  `Object` or dynamic.
- **Analyzer plugins** live under the `plugins:` stanza in `analysis_options.yaml`. Prefer plugins over ad-hoc scripts
  when you need bespoke lint rules.
- **Build hooks** (`hook/build.dart`) are the default for compiling native assets. Chain work via `BuildInput`/
  `BuildOutput`.
- **Null-aware collection elements** (`...?`, guarded `if`/`for`) make builder-style literals declarative—use them
  instead
  of surrounding `if != null` blocks.
- **Extension types** wrap primitives without allocation. Reach for them when expressing IDs, handles, or validated
  strings so call sites stay type-safe.

---

## Route Yourself

Pick the scope that matches your task and open the linked module before editing:

| Scenario                             | Start Here                                                       | Notes                                                                      |
|--------------------------------------|------------------------------------------------------------------|----------------------------------------------------------------------------|
| Quickly orienting yourself           | **Core Engineering Standards** (below)                           | High-level rules that apply everywhere.                                    |
| Structuring features                 | [`feature_based_architecture.md`](feature_based_architecture.md) | Explains the feature-first folder layout, boundaries, and dependency flow. |
| Designing a new feature/API          | [`how_to_new_feature.md`](how_to_new_feature.md)                 | Requirements → scope controllers → state controllers → UI.                 |
| Need ValueGetter/copyWith semantics  | **Data Classes & `copyWith`** (below)                            | Includes reusable `ValueGetterX` extension.                                |
| Debugging UI composition issues      | **UI Construction Discipline** (below)                           | Covers `build()` purity, widget extraction, and magic-number policy.       |
| Error handling, logging, async flows | **Error & Async Rules** (below)                                  | Defines how to await, propagate, and log.                                  |

---

## Core Engineering Standards

1. Always explore alternative solutions before coding; pick the option with the cleanest architecture and lowest future
   cost.
2. Generated artifacts (especially `*.g.dart`) are read-only. Change source inputs, then ask the user to run
   `dart run build_runner build --delete-conflicting-outputs` (or their preferred build command).
3. Every public API needs Effective-Dart documentation. Use `@template`, `@macro`, and `@docImport` to avoid
   duplication. Explain the “why” when logic is non-trivial.
4. Format/analyze with the Dart MCP tool after each change; never hand-wave analyzer output.

---

## Data Classes & `copyWith`

- All state/data classes must implement `==`, `hashCode`, and a `copyWith` that distinguishes “not provided” from
  “explicit null”.
- Prefer `ValueGetter<T?>?` for every `copyWith` parameter and reuse this helper to keep the implementation terse:

```dart
extension ValueGetterX<T> on ValueGetter<T>? {
  /// Returns the provided getter's value if available; otherwise keeps [current].
  T or(T current) => this == null ? current : this!();
}
```

Add `ValueGetterX` to a shared utilities file before adopting the pattern if the project does not define it yet.

---

## UI Construction Discipline

- Keep widget trees declarative. When a `build` method grows unwieldy, extract widgets into Widget classes with
  descriptive names instead of burying logic inline.
- Treat `build` as a pure function: no mutation, side effects, logging, or calls that depend on previous UI frames.
- Encapsulate business logic in services, state-machines, or controllers. UI widgets should orchestrate state, not own
  it.
- Avoid magic numbers. Introduce constants or theme values with names that clarify intent.

---

## Error & Async Rules

- `await` every `Future` inside `async` functions so that exceptions surface locally. No “fire-and-forget” unless
  explicitly justified and documented.
- Classify errors early:
    - **Expected**: provide dedicated UI states or domain-specific handling.
    - **Unexpected**: let the exception bubble up (use `rethrow` or `Error.throwWithStackTrace`) and log at warning
      level
      or higher.
- Never write `try { /* danger */ } catch (e) { print(e); }`. Either remove the `try` or handle the exception properly.
- Add structured logging for critical actions. When logic intentionally suppresses an error path, log with context to
  aid debugging.

---

## Tooling & Architecture References

- Architecture overview: [`feature_based_architecture.md`](feature_based_architecture.md)
- Feature design workflow: [`how_to_new_feature.md`](how_to_new_feature.md)
- Always revisit [`dart_flutter_features.md`](dart_flutter_features.md) for SDK-level capabilities before coding.
- Need live scope examples? Compare [`inherited_model_example.md`](inherited_model_example.md) and
  [`inherited_notifier_example.md`](inherited_notifier_example.md) to choose the right pattern.
