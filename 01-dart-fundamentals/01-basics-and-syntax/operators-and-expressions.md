# Operators and Expressions in Dart

> **Module:** 01-dart-fundamentals  
> **Subsection:** 01-basics-and-syntax  
> **Prerequisites:** [variables-types-and-inference.md](./variables-types-and-inference.md)  
> **Official References:** [dart.dev/language/operators](https://dart.dev/language/operators)

---

## 1. Introduction

Operators are special symbols or keywords that perform operations on one, two, or three operands to produce a result. Dart offers a rich suite of operators, ranging from standard arithmetic calculations to specialized features like **cascade notation (`..`)**, **type test operators (`is`, `as`)**, and **spread operators (`...`)**.

### Why Do We Need Them?
1. **Expressive Power:** Operators allow writing compact, legible logic without verbosity.
2. **Object Manipulation:** Cascade notation lets you configure objects fluently without temporary variables.
3. **Collection Assembly:** Spread operators allow merging, transforming, and conditionally injecting items into lists and maps directly inside Flutter widget trees.

### Where Is It Used in Real Development?
- **Widget Tree Construction:** Merging lists of children widgets dynamically using spread operators (`...actions`).
- **Object Configuration:** Initializing paint styles, HTTP requests, or database queries using cascade notation (`..color = Colors.blue ..strokeWidth = 2.0`).
- **Data Validation & Parsing:** Type testing API payloads using `is` and `is!`.

---

## 2. Core Concepts

### 1. Arithmetic & Division Operators
- `+`, `-`, `*`: Standard addition, subtraction, multiplication.
- `/`: **Standard division** (always returns a `double`, even for integer operands: `5 / 2 = 2.5`).
- `~/`: **Integer division** (truncates the decimal part, returns an `int`: `5 ~/ 2 = 2`).
- `%`: Modulo (remainder: `5 % 2 = 1`).

### 2. Type Test Operators
- `is`: Evaluates to `true` if the object has the specified type.
- `is!`: Evaluates to `true` if the object does **not** have the specified type.
- `as`: Type cast. Forces the compiler to treat an object as a specific subtype. If the object is not of that type, throws a `TypeError` at runtime.

### 3. Cascade Notation (`..` and `?..`)
Cascade notation allows you to perform a sequence of operations (method calls, field assignments) on the **same object** without repeating the variable name.
- `target..method1()..field = value;`
- **Crucial Rule:** Cascade returns the **target object itself**, not the return value of the method called.
- `?..` is null-shorting cascade (performs cascades only if the target is non-null).

### 4. Spread Operators (`...` and `...?`)
Used inside collection literals (lists, sets, maps) to insert multiple elements into another collection.
- `...list`: Unpacks all elements from `list`.
- `...?nullableList`: Unpacks elements only if `nullableList` is non-null.

---

## 3. Syntax and Implementation

```dart
// bin/operators_demo.dart

void main() {
  // 1. Division: / vs ~/
  int a = 7;
  int b = 3;
  double standardDiv = a / b;
  int integerDiv = a ~/ b;
  int remainder = a % b;

  print('Standard Division (7 / 3): $standardDiv'); // 2.3333333333333335
  print('Integer Division  (7 ~/ 3): $integerDiv'); // 2
  print('Remainder         (7 % 3):  $remainder');  // 1

  // 2. Type Test Operators
  Object unknownValue = 'Flutter Specialist';
  if (unknownValue is String) {
    print('Type verified: ${unknownValue.toUpperCase()}');
  }

  // 3. Cascade Notation (..)
  final builder = StringBuilder()
    ..append('Line 1')
    ..append('Line 2')
    ..setUppercase(true);
  print('Built string: ${builder.build()}');

  // 4. Spread Operators (...)
  final initialItems = [1, 2, 3];
  List<int>? optionalItems; // null

  final combined = [
    0,
    ...initialItems,
    ...?optionalItems, // Does not crash!
    4,
  ];
  print('Combined list: $combined');
}

class StringBuilder {
  final List<String> _lines = [];
  bool _uppercase = false;

  void append(String line) => _lines.add(line);
  void setUppercase(bool value) => _uppercase = value;

  String build() {
    final result = _lines.join('\n');
    return _uppercase ? result.toUpperCase() : result;
  }
}
```

### Expected Output
```text
Standard Division (7 / 3): 2.3333333333333335
Integer Division  (7 ~/ 3): 2
Remainder         (7 % 3):  1
Type verified: FLUTTER SPECIALIST
Built string: LINE 1
LINE 2
Combined list: [0, 1, 2, 3, 4]
```

### Common Mistakes
- **Mistake 1: Confusing `/` with `~/`.**  
  Writing `int index = length / 2;` causes a compilation error because `/` returns `double`. Always use `length ~/ 2` when an integer result is required.
- **Mistake 2: Storing the result of a cascade expecting the method's return value.**  
  ```dart
  // WRONG:
  final lastItem = list..removeLast(); // 'lastItem' is the List, NOT the removed element!
  // CORRECT:
  final lastItem = list.removeLast();
  ```
- **Mistake 3: Unsafe casting with `as`.**  
  `final str = rawData as String;` will crash if `rawData` is an int. Always verify with `if (rawData is String)` first.

---

## 4. Practical Example: Building Dynamic Flutter Widget Lists

In Flutter UI design, conditional and dynamic widget generation frequently utilizes spread and cascade operators:

```dart
// lib/navigation_menu.dart

class MenuItem {
  final String title;
  final bool isAdminOnly;
  MenuItem(this.title, {this.isAdminOnly = false});
}

List<String> buildNavigationMenu({required bool isAdmin, required List<String>? extraItems}) {
  final defaultItems = ['Home', 'Profile', 'Settings'];
  final adminItems = ['User Management', 'Server Logs', 'Billing'];

  return [
    'App Logo',
    ...defaultItems,
    if (isAdmin) ...adminItems, // Conditional collection spread!
    ...?extraItems,             // Null-safe spread!
    'Logout',
  ];
}

void main() {
  final standardUserMenu = buildNavigationMenu(isAdmin: false, extraItems: null);
  print('Standard User Menu: $standardUserMenu');

  final adminMenu = buildNavigationMenu(isAdmin: true, extraItems: ['Feature Flags']);
  print('Admin Menu: $adminMenu');
}
```

---

## 5. Internal Working: Operator Overloading

In Dart, many operators are actually **instance methods** defined on classes with special syntax:

```dart
class Vector2D {
  final double x;
  final double y;

  const Vector2D(this.x, this.y);

  // Overloading the '+' operator
  Vector2D operator +(Vector2D other) => Vector2D(x + other.x, y + other.y);

  // Overloading the '==' operator
  @override
  bool operator ==(Object other) =>
      identical(this, other) ||
      other is Vector2D &&
          runtimeType == other.runtimeType &&
          x == other.x &&
          y == other.y;

  @override
  int get hashCode => Object.hash(x, y);

  @override
  String toString() => 'Vector2D($x, $y)';
}

void main() {
  final v1 = Vector2D(2.0, 3.5);
  final v2 = Vector2D(1.0, 1.5);
  final v3 = v1 + v2; // Calls v1.operator+(v2)
  print('Vector addition result: $v3'); // Vector2D(3.0, 5.0)
}
```

---

## 6. Operator Categories & Precedence Table

| Category | Operators | Associativity |
|:---|:---|:---|
| **Unary Postfix** | `expr++`, `expr--`, `()`, `[]`, `?.`, `.` | Left to right |
| **Unary Prefix** | `-expr`, `!expr`, `~expr`, `++expr`, `--expr`, `await` | Right to left |
| **Multiplicative** | `*`, `/`, `~/`, `%` | Left to right |
| **Additive** | `+`, `-` | Left to right |
| **Shift** | `<<`, `>>`, `>>>` (triple shift) | Left to right |
| **Relational** | `>=`, `>`, `<=`, `<`, `as`, `is`, `is!` | None |
| **Equality** | `==`, `!=` | None |
| **Bitwise** | `&` (AND), `^` (XOR), `|` (OR) | Left to right |
| **Logical** | `&&` (AND), `\|\|` (OR) | Left to right |
| **Null-Coalescing** | `??` | Left to right |
| **Conditional** | `condition ? expr1 : expr2` | Right to left |
| **Cascade** | `..`, `?..` | Left to right |
| **Assignment** | `=`, `+=`, `-=`, `*=`, `/=`, `~/=`, `??=`, etc. | Right to left |

---

## 7. Interview Preparation

### Frequently Asked Questions

#### Q1: What is the exact difference between `/` and `~/` in Dart?
- **Short Answer:** `/` performs floating-point division and returns a `double`. `~/` performs integer division and truncates the fractional portion, returning an `int`.
- **Detailed Answer:** Unlike languages like C or Java where `int / int` yields an integer, Dart's `/` operator always promotes the result to `double` (e.g. `4 / 2` evaluates to `2.0`). To obtain an integer result without rounding errors or explicit casting, Dart provides the dedicated integer division operator `~/` (`4 ~/ 2` evaluates to `2`).

#### Q2: How does cascade notation (`..`) work under the hood?
- **Answer:** Cascade notation performs a sequence of operations on the same receiver object. The expression evaluates the receiver, executes the cascade operation, discards any return value of that individual operation, and returns the receiver object itself. This enables fluent method chaining even on APIs that were not designed with a fluent builder interface (i.e. methods that return `void`).

#### Q3: Scenario Question: Why is `...` preferred over `List.addAll()` inside Flutter widget declarations?
- **Answer:** `List.addAll()` is an imperative method that returns `void` and modifies an existing list in-place. Flutter widget trees are constructed using declarative expressions. The spread operator (`...`) allows in-line, declarative composition of collections within widget children lists, enabling direct combination with `if` conditions (`if (condition) ...widgets`) and null safety (`...?optionalWidgets`).

---

## 8. Practice Exercises

### Exercise 1 (Beginner): Operator Calculation
What are the exact values and types of `val1` and `val2`?
```dart
void main() {
  final val1 = 9 / 2;
  final val2 = 9 ~/ 2;
  print('$val1 (${val1.runtimeType})');
  print('$val2 (${val2.runtimeType})');
}
```

### Exercise 2 (Intermediate): Custom Operator Overloading
Create a `Money` class with fields `double amount` and `String currency`.
- Overload the `+` operator to allow adding two `Money` objects of the same currency.
- Throw an `ArgumentError` if trying to add `Money` objects with different currencies.

---

## 9. References
- [Dart Language Tour: Operators](https://dart.dev/language/operators)
- [Effective Dart: Usage](https://dart.dev/effective-dart/usage)
