# Interfaces and Abstract Classes in Dart

> **Module:** 01-dart-fundamentals  
> **Subsection:** 05-oop-and-type-system  
> **Prerequisites:** [inheritance-and-polymorphism.md](./inheritance-and-polymorphism.md)  
> **Official References:** [dart.dev/language/classes#implicit-interfaces](https://dart.dev/language/classes#implicit-interfaces) | [dart.dev/language/class-modifiers#interface](https://dart.dev/language/class-modifiers#interface)

---

## 1. Introduction

In software architecture, an **interface** is a formal contract specifying *what* operations a component provides, without dictating *how* those operations are implemented.

Unlike Java, TypeScript, or C#, which feature a distinct `interface` declaration keyword, Dart employs a uniquely elegant concept: **Implicit Interfaces**. In Dart, **every class implicitly defines an interface** containing all its instance members.

### Why Do We Need Interfaces and Abstract Classes?
1. **Decoupled Architecture:** Client code depends on abstract contracts rather than concrete implementations (Dependency Inversion Principle).
2. **Testing & Mocking:** In unit testing, real network services and databases can be swapped with mock implementations that implement the same interface.
3. **Multiple Implementation Contracts:** While Dart permits inheriting from only one class (`extends`), a class can implement **unlimited interfaces** (`implements A, B, C`).

---

## 2. Core Concepts

```mermaid
flowchart TD
    Contract["Abstract Contract: UserRepository"]
    Contract -->|extends (Single Inheritance)| BaseRepo["BaseRepository (Inherits Concrete Code)"]
    Contract -->|implements (Contract Only)| RemoteRepo["RemoteUserRepository (Implements API calls)"]
    Contract -->|implements (Contract Only)| LocalRepo["LocalUserRepository (Implements SQLite/Hive)"]
    Contract -->|implements (Contract Only)| MockRepo["MockUserRepository (Implements Test Stubs)"]
```

### 1. The Implicit Interface Rule
Every class in Dart—whether concrete or abstract—implicitly creates an interface containing:
- All instance methods.
- All instance fields (represented as getter/setter pairs).
- All explicit getters and setters.

### 2. `extends` vs. `implements`
- **`extends` (Inheritance):** You inherit the superclass's state, fields, and concrete method implementations. You can override only what you need. A class can extend only **one** superclass.
- **`implements` (Interface Contract):** You inherit **zero** implementations. You are bound by contract to implement **every single member** of the interface from scratch. A class can implement **multiple** interfaces.

### 3. Abstract Classes (`abstract class`)
A class that cannot be instantiated directly with a generative constructor (though it can provide `factory` constructors). It can contain both abstract method signatures (without bodies) and concrete utility methods.

---

## 3. Syntax and Implementation

```dart
// bin/interfaces_demo.dart

// 1. Defining an Abstract Class acting as an Interface Contract
abstract class UserRepository {
  // Abstract method signatures (the contract)
  Future<String> fetchUserEmail(int userId);
  Future<void> saveUserToken(int userId, String token);
  
  // Concrete method (optional utility for classes that extend this)
  void logRepositoryAction(String action) {
    print('[Repository Audit] $action at ${DateTime.now().toIso8601String()}');
  }
}

// 2. Concrete Implementation for Production (implements contract)
class RemoteHttpUserRepository implements UserRepository {
  final String apiEndpoint;

  RemoteHttpUserRepository(this.apiEndpoint);

  @override
  Future<String> fetchUserEmail(int userId) async {
    print('--> [Network] Querying $apiEndpoint/users/$userId...');
    return 'user_$userId@production-domain.com';
  }

  @override
  Future<void> saveUserToken(int userId, String token) async {
    print('--> [Network] POST $apiEndpoint/users/$userId/tokens');
  }

  @override
  void logRepositoryAction(String action) {
    // Because we used 'implements', we MUST provide this method too!
    print('--> [Remote Audit] $action');
  }
}

// 3. Mock Implementation for Unit Tests
class MockUserRepository implements UserRepository {
  final Map<int, String> _mockEmailStore = {
    101: 'test_alice@unit-test.com',
    102: 'test_bob@unit-test.com',
  };

  @override
  Future<String> fetchUserEmail(int userId) async {
    return _mockEmailStore[userId] ?? 'unknown@test.com';
  }

  @override
  Future<void> saveUserToken(int userId, String token) async {
    print('--> [Mock] Saved token in-memory: $token');
  }

  @override
  void logRepositoryAction(String action) {
    // No-op for tests
  }
}

// Client service depending strictly on the abstract interface!
class UserNotificationService {
  final UserRepository _repository;

  UserNotificationService(this._repository);

  Future<void> sendWelcomeNotification(int userId) async {
    final email = await _repository.fetchUserEmail(userId);
    print('Dispatched Welcome Email to: $email');
  }
}

void main() async {
  // Production Wiring
  print('--- 1. Production Execution ---');
  final prodRepo = RemoteHttpUserRepository('https://api.domain.com/v1');
  final prodService = UserNotificationService(prodRepo);
  await prodService.sendWelcomeNotification(101);

  // Unit Test Wiring (Zero network calls, fast in-memory mock!)
  print('\n--- 2. Automated Test Execution ---');
  final mockRepo = MockUserRepository();
  final testService = UserNotificationService(mockRepo);
  await testService.sendWelcomeNotification(101);
}
```

### Expected Output
```text
--- 1. Production Execution ---
--> [Network] Querying https://api.domain.com/v1/users/101...
Dispatched Welcome Email to: user_101@production-domain.com

--- 2. Automated Test Execution ---
Dispatched Welcome Email to: test_alice@unit-test.com
```

### Common Mistakes
- **Mistake 1: Using `implements` and expecting inherited superclass code.**  
  If you write `class MyWidget implements StatelessWidget`, your class will **not** inherit Flutter's element creation or lifecycle code; you will be forced to implement every single internal method of `StatelessWidget`. Always use `extends` for base classes and `implements` for pure contracts.
- **Mistake 2: Missing fields of an implemented class.**  
  Remember that fields in Dart define implicit getters and setters. If class `A` has `final int id;`, any class that `implements A` must declare an `id` field or write an explicit `int get id => ...;`.

---

## 4. Comparison: `extends` vs `implements`

| Dimension | `extends` (Inheritance) | `implements` (Interface Contract) |
|:---|:---|:---|
| **Code Reuse** | Inherits concrete fields and methods | **Inherits zero implementation** |
| **Number of Targets** | Single superclass only (`extends A`) | **Multiple interfaces (`implements A, B, C`)** |
| **Subclass Obligation** | Only override abstract or desired methods | **Must implement 100% of interface members** |
| **`super` Calls** | Can invoke `super.method()` | Cannot call `super.method()` |
| **Primary Architectural Use** | Base classes, UI Widgets | Repositories, Services, Mocks, Contracts |

---

## 5. Interview Preparation

### Frequently Asked Questions

#### Q1: Why does Dart not have a dedicated `interface` keyword like Java or TypeScript?
- **Answer:** In classic Dart, a separate `interface` keyword was deemed redundant because every class declaration already defines an implicit interface. Any class can serve as an interface when used with the `implements` keyword. (Note: Dart 3 introduced the `interface class` modifier, which restricts a class so that external libraries can only implement its interface, but cannot extend its implementation).

#### Q2: What happens if class `C` implements two interfaces `A` and `B` that share a method with the same signature?
- **Answer:** Class `C` only needs to provide **one single implementation** of that method that satisfies both signatures. As long as the parameter types and return type are compatible with both contracts, a single concrete method in `C` satisfies both interfaces simultaneously.

---

## 6. Practice Exercises

### Exercise 1 (Beginner): Multiple Interface Implementation
Create two abstract classes:
1. `abstract class Serializable { String toJson(); }`
2. `abstract class Printable { void printDetails(); }`
Create a concrete class `Invoice` that implements **both** interfaces.

---

## 7. References
- [Dart Official Language Guide: Implicit Interfaces](https://dart.dev/language/classes#implicit-interfaces)
- [Dart Class Modifiers: Interface](https://dart.dev/language/class-modifiers#interface)
