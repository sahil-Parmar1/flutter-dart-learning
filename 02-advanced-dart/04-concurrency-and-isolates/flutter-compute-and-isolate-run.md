# Modern Concurrency: `Isolate.run` and Flutter's `compute()`

> **Module:** 02-advanced-dart  
> **Subsection:** 04-concurrency-and-isolates  
> **Prerequisites:** [isolate-spawning-and-ports.md](./isolate-spawning-and-ports.md)  
> **Official References:** [api.dart.dev/stable/dart-isolate/Isolate/run.html](https://api.dart.dev/stable/dart-isolate/Isolate/run.html) | [api.flutter.dev/flutter/foundation/compute.html](https://api.flutter.dev/flutter/foundation/compute.html)

---

## 1. Introduction

Prior to Dart 2.19, offloading a single heavy calculation to a background isolate required over 40 lines of boilerplate code: creating `ReceivePort`s, managing `SendPort` handshakes, and remembering to kill the isolate and close ports.

Dart 2.19 introduced **`Isolate.run()`**, revolutionizing isolate concurrency. With a single line of code, `Isolate.run()`:
1. Spawns an ephemeral background worker isolate.
2. Executes your heavy computation.
3. Automatically catches any thrown exceptions and surfaces them to the caller.
4. Returns the result to the main isolate.
5. Automatically terminates and cleans up the isolate.

In Flutter, the foundational library also provides **`compute()`**, which offers similar functionality and is deeply embedded across existing production codebases.

---

## 2. Core Concepts

```mermaid
flowchart TD
    Main["Main UI Isolate<br/>(Smooth 60/120 FPS Rendering)"]
    Main -->|await Isolate.run(() => heavyTask())| Background["Ephemeral Background Isolate<br/>• Spawns on available CPU core<br/>• Executes heavy computation<br/>• Returns result directly<br/>• Automatically terminates!"]
    Background -->|Returns Typed Result T| Main
```

### 1. `Isolate.run<R>(FutureOr<R> Function() computation)`
- Standard Dart SDK API (available in pure Dart and Flutter).
- Accepts a top-level function, static method, or closure.
- Automatically handles two-way port creation and teardown.
- **Fast Zero-Copy Optimization:** In many cases, Dart VM can optimize the return value using pointer transfer rather than full deep-copy serialization.

### 2. Flutter's `compute()`
- Provided by `package:flutter/foundation.dart`.
- Function signature: `compute<Q, R>(ComputeCallback<Q, R> callback, Q message)`.
- Historically powered one-off isolate execution in Flutter before `Isolate.run()` was standardized in the Dart SDK. In modern Flutter (Flutter 3.7+), `compute()` is internally implemented on top of `Isolate.run()`.

---

## 3. Syntax and Implementation

```dart
// bin/isolate_run_demo.dart

import 'dart:async';
import 'dart:convert';
import 'dart:isolate';

// 1. Heavy JSON Parsing Function
List<Map<String, dynamic>> parseHeavyJson(String rawJson) {
  print('--> [Background Isolate] Parsing JSON on thread: ${Isolate.current.debugName}...');
  
  // Decoding large JSON payload
  final List<dynamic> decoded = jsonDecode(rawJson) as List<dynamic>;
  
  // Transforming into domain structures
  return decoded.map((item) => item as Map<String, dynamic>).toList();
}

// 2. Heavy CPU Cryptographic / Math Function
int calculatePrimesCount(int limit) {
  int count = 0;
  for (int i = 2; i <= limit; i++) {
    bool isPrime = true;
    for (int j = 2; j * j <= i; j++) {
      if (i % j == 0) {
        isPrime = false;
        break;
      }
    }
    if (isPrime) count++;
  }
  return count;
}

void main() async {
  print('[Main Isolate] Application running on: ${Isolate.current.debugName}');

  // ==========================================
  // 1. Offloading Heavy Math via Isolate.run
  // ==========================================
  print('\n--- 1. Testing Isolate.run for CPU Work ---');
  final stopwatch = Stopwatch()..start();

  // Spawns isolate, runs computation, returns result, terminates isolate!
  final primeCount = await Isolate.run(() => calculatePrimesCount(200000));
  
  stopwatch.stop();
  print('Primes up to 200,000: $primeCount (computed in ${stopwatch.elapsedMilliseconds}ms)');

  // ==========================================
  // 2. Offloading JSON Parsing via Isolate.run
  // ==========================================
  print('\n--- 2. Testing Isolate.run for JSON Parsing ---');
  
  // Generating a mock 10,000-record JSON string
  final mockJson = jsonEncode(List.generate(10000, (i) => {'id': i, 'name': 'Item #$i'}));

  final parsedData = await Isolate.run(() => parseHeavyJson(mockJson));
  print('[Main Isolate] Received ${parsedData.length} parsed records without dropping UI frames!');
}
```

### Expected Output
```text
[Main Isolate] Application running on: main

--- 1. Testing Isolate.run for CPU Work ---
Primes up to 200,000: 17984 (computed in 142ms)

--- 2. Testing Isolate.run for JSON Parsing ---
--> [Background Isolate] Parsing JSON on thread: ...
[Main Isolate] Received 10000 parsed records without dropping UI frames!
```

### Common Mistakes
- **Mistake 1: Capturing `BuildContext` or Widget state inside `Isolate.run`.**  
  ```dart
  // RUNTIME CRASH:
  final data = await Isolate.run(() {
    return context.read<MyState>().expensiveMethod(); // ILLEGAL: BuildContext cannot be sent across isolates!
  });
  ```
  Only extract raw primitives or pure data before entering the isolate.
- **Mistake 2: Using `Isolate.run` in rapid high-frequency loops (e.g. 60 times a second).**  
  Spawning and destroying an isolate 60 times a second creates garbage collection thrashing. For high-frequency continuous work, use a persistent long-lived isolate with `Isolate.spawn` and ports.

---

## 4. Comparison: `Isolate.run` vs Flutter `compute()` vs `Isolate.spawn`

| Feature | `Isolate.run()` | Flutter `compute()` | `Isolate.spawn()` |
|:---|:---|:---|:---|
| **Platform Availability** | Pure Dart & Flutter (Dart 2.19+) | Flutter only (`foundation.dart`) | Pure Dart & Flutter |
| **Boilerplate** | **Minimal (1 line)** | **Minimal (1 line)** | High (manual ports & handshake) |
| **Closure Support** | Directly accepts closures | Requires top-level / static function | Requires top-level / static function |
| **Worker Lifespan** | Ephemeral (One-off) | Ephemeral (One-off) | **Persistent (Long-lived)** |
| **Recommended In** | **Modern Dart & Flutter** | Legacy / existing Flutter code | Audio, Streaming sockets, Video |

---

## 5. Interview Preparation

### Frequently Asked Questions

#### Q1: What makes `Isolate.run()` faster than legacy message passing?
- **Answer:** In traditional `Isolate.spawn()`, data is deeply copied between the two isolates' heaps. `Isolate.run()` leverages compiler optimizations: because the background isolate terminates immediately after returning the value, the Dart VM can often simply transfer ownership of the object's memory pointers to the calling isolate, avoiding a redundant deep-memory copy cycle.

#### Q2: When would you still choose `compute()` over `Isolate.run()`?
- **Answer:** In modern Flutter applications running Dart 2.19+, `Isolate.run()` is generally preferred because it is a native language feature that allows passing closures directly without strictly requiring top-level or static function declarations. However, `compute()` is widely used across existing codebases and provides helpful debugging names and compatibility flags across older Flutter versions.

---

## 6. Practice Exercises

### Exercise 1 (Intermediate): Background Image Inverter
Write a function `Future<List<int>> invertImageBytes(List<int> bytes)` that uses `Isolate.run()` to iterate through a simulated 1,000,000-byte array and inverts every byte (`255 - b`) without blocking the main isolate.

---

## 7. References
- [Dart API: Isolate.run](https://api.dart.dev/stable/dart-isolate/Isolate/run.html)
- [Flutter API: compute function](https://api.flutter.dev/flutter/foundation/compute.html)
