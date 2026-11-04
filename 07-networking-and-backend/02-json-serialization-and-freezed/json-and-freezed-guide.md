# JSON Serialization, Code Generation, and Freezed

> **Module:** 07-networking-and-backend  
> **Subsection:** 02-json-serialization-and-freezed  
> **Prerequisites:** [01-dart-fundamentals/05-oop-and-type-system/classes-and-objects.md](../../01-dart-fundamentals/05-oop-and-type-system/classes-and-objects.md)  
> **Official References:** [docs.flutter.dev/data-and-backend/serialization/json](https://docs.flutter.dev/data-and-backend/serialization/json) | [pub.dev/packages/freezed](https://pub.dev/packages/freezed)

---

## 1. Introduction

Because Dart is a statically typed language with Ahead-of-Time (AOT) compilation and no runtime reflection (`dart:mirrors` is disabled in Flutter for binary size and performance), parsing JSON strings into strongly typed Dart objects requires explicit mapping contracts.

In professional Flutter development, data modeling evolves through three distinct tiers:
1. **Tier 1: Manual Serialization:** Writing `fromJson` and `toJson` methods by hand using `Map<String, dynamic>`.
2. **Tier 2: Code Generation (`json_serializable`):** Automating JSON boilerplate using annotations and `build_runner`.
3. **Tier 3: Immutable Data Architecture (`Freezed`):** Adding structural value equality (`==`), deep immutable cloning (`copyWith`), and union types (sealed classes) on top of JSON serialization.

---

## 2. Architectural Comparison

| Feature | Tier 1: Manual Mapping | Tier 2: `json_serializable` | Tier 3: `@freezed` |
|:---|:---:|:---:|:---:|
| **Boilerplate** | High (Every field written 4x) | Low | Ultra-Low (Single line constructor) |
| **Typo Safety** | None (String key errors at runtime) | Guaranteed by Dart compiler | Guaranteed by Dart compiler |
| **Value Equality (`==`)** | By reference (Must override manually) | By reference | Built-in deep structural equality |
| **Immutable `copyWith()`** | Must be hand-written | Must be hand-written | Generated automatically |
| **Union / Sealed Types** | Manual class hierarchies | Manual | First-class union support |

---

## 3. Syntax and Implementation

### 1. Tier 2: `json_serializable`

```dart
import 'package:json_annotation/json_annotation.dart';

part 'user_profile.g.dart';

@JsonSerializable(explicitToJson: true)
class UserProfile {
  final String id;

  @JsonKey(name: 'full_name') // Custom JSON key mapping
  final String fullName;

  @JsonKey(defaultValue: false)
  final bool isVerified;

  final DateTime createdAt;

  UserProfile({
    required this.id,
    required this.fullName,
    required this.isVerified,
    required this.createdAt,
  });

  // Factory constructor connecting to generated code
  factory UserProfile.fromJson(Map<String, dynamic> json) =>
      _$UserProfileFromJson(json);

  Map<String, dynamic> toJson() => _$UserProfileToJson(this);
}
```

---

### 2. Tier 3: Modern Data Architecture with `@freezed`

`Freezed` transforms Dart classes into fully functional immutable domain entities:

```dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'order_status.freezed.dart';
part 'order_status.g.dart';

@freezed
class OrderStatus with _$OrderStatus {
  // Union Cases (Sealed Class Hierarchy)
  const factory OrderStatus.pending({required DateTime orderDate}) = _Pending;
  const factory OrderStatus.shipped({
    required String trackingNumber,
    required String carrier,
  }) = _Shipped;
  const factory OrderStatus.delivered({required DateTime deliveryDate}) = _Delivered;
  const factory OrderStatus.cancelled({required String reason}) = _Cancelled;

  factory OrderStatus.fromJson(Map<String, dynamic> json) =>
      _$OrderStatusFromJson(json);
}
```

#### Utilizing Generated Features in UI:
```dart
void displayOrderStatus(OrderStatus status) {
  // Pattern matching over union types
  final message = status.when(
    pending: (date) => 'Order placed on $date. Awaiting fulfillment.',
    shipped: (tracking, carrier) => 'In transit with $carrier (Tracking: $tracking)',
    delivered: (date) => 'Delivered successfully on $date!',
    cancelled: (reason) => 'Order cancelled: $reason',
  );
  print(message);
}

// Deep immutable cloning with copyWith
void exampleCopyWith() {
  final original = const OrderStatus.shipped(
    trackingNumber: 'TRACK_9921',
    carrier: 'FedEx',
  );

  // copyWith produces a brand new immutable instance
  final updated = original.copyWith(carrier: 'DHL Express');
}
```

---

## 4. The Build Runner Workflow

To generate `*.freezed.dart` and `*.g.dart` files, execute:

```bash
# One-time build:
dart run build_runner build --delete-conflicting-outputs

# Continuous file watcher during active development:
dart run build_runner watch --delete-conflicting-outputs
```

---

## 5. Common Pitfalls & Anti-Patterns

### Pitfall 1: Mutating Freezed Entities
`Freezed` classes are deeply immutable (`final`). You cannot do:
```dart
user.fullName = 'Jane Doe'; // ❌ COMPILE ERROR
```
Always create updated instances via `.copyWith()`:
```dart
final updatedUser = user.copyWith(fullName: 'Jane Doe'); // ✅ CORRECT
```

### Pitfall 2: Forgetting `part 'filename.freezed.dart';`
Dart requires the `part` directive to attach generated files to the library. If missing, the compiler throws unresolved identifier errors for `_$MyClass`.

---

## 6. Interview & Assessment Takeaways

| Interview Question | Concise Verbal Answer |
|:---|:---|
| **Why is runtime reflection (`dart:mirrors`) disabled in Flutter?** | Flutter disables reflection to allow Ahead-of-Time (AOT) tree-shaking and dead code elimination, producing tiny native binaries and lightning-fast execution. Consequently, JSON mapping must rely on compile-time code generation (`build_runner`). |
| **What does `@freezed` provide beyond `json_serializable`?** | While `json_serializable` only creates `fromJson`/`toJson` mapping methods, `@freezed` adds deep structural value equality (`==`), hashcode calculation, immutable `copyWith()` methods, and union type pattern matching (`when`/`maybeWhen`). |
| **What is the difference between Dart 3 pattern matching and Freezed `.when()`?** | Freezed `.when()` is a callback-based pattern matching method generated on union classes. Dart 3 pattern matching (`switch (status) { case Shipped(:final carrier): ... }`) is built natively into the Dart language grammar and provides compile-time exhaustiveness checking without method invocation overhead. |
