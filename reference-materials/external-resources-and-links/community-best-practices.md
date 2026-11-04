# Flutter & Dart Community Best Practices, Linter Rules, and Standards

> **Module:** reference-materials  
> **Subsection:** external-resources-and-links  
> **Target Audience:** All Software Engineers Contributing to Production Codebases

---

## 1. Effective Dart: Core Coding Style Rules

The Dart team maintains the official style guide at [dart.dev/effective-dart](https://dart.dev/effective-dart).

### 1. Naming Conventions:
- **`UpperCamelCase`:** Classes, enums, typedefs, type parameters (`class UserProfile`, `enum PriorityLevel`).
- **`lowerCamelCase`:** Class members, methods, local variables, parameters (`void fetchUserData()`, `final totalCount = 10;`).
- **`lowercase_with_underscores`:** File names, library names, package names (`user_repository.dart`, `app_theme.dart`).
- **Avoid Prefixes:** Avoid Hungarian notation or prefixing classes with `I` (e.g. use `AuthRepository`, **not** `IAuthRepository`).

---

## 2. Recommended Linter Rules (`analysis_options.yaml`)

Every production Flutter project should use `package:flutter_lints` or `package:very_good_analysis`:

```yaml
include: package:flutter_lints/flutter.yaml

analyzer:
  language:
    strict-casts: true
    strict-inference: true
    strict-raw-types: true
  errors:
    missing_required_param: error
    missing_return: error
    todo: ignore

linter:
  rules:
    - prefer_const_constructors
    - prefer_const_declarations
    - prefer_const_literals_to_create_immutables
    - avoid_print
    - unawaited_futures
    - cancel_subscriptions
    - close_sinks
    - use_super_parameters
    - prefer_final_locals
    - prefer_single_quotes
    - sort_child_properties_last
```

---

## 3. Top 5 Architectural Best Practices

1. **Keep `build()` Methods Pure and Synchronous:** Never initiate asynchronous operations, HTTP calls, or database writes inside `build()`. `build()` should be a pure mathematical projection of state: $\text{UI} = f(\text{State})$.
2. **Preserve Immutability:** Model your business state using immutable classes and `final` fields. Mutating existing objects in place causes subtle bugs where state management frameworks fail to detect changes.
3. **Always Clean Up Resources:** Implement `dispose()` on `TextEditingController`, `AnimationController`, `ScrollController`, `FocusNode`, and cancel all active `StreamSubscription` objects.
4. **Prefer Composition Over Deep Inheritance:** In Flutter, inheritance hierarchies for widgets are an anti-pattern. Assemble rich user interfaces by nesting smaller, single-purpose widgets.
5. **Decouple Business Logic from UI Frameworks:** Ensure your core domain use cases and entities never import `package:flutter`. This guarantees fast, deterministic pure Dart unit tests.
