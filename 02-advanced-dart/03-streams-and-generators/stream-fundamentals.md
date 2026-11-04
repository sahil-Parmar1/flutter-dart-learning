# Stream Fundamentals in Dart: Single-Subscription vs Broadcast Streams

> **Module:** 02-advanced-dart  
> **Subsection:** 03-streams-and-generators  
> **Prerequisites:** [01-asynchronous-programming/](../01-asynchronous-programming/)  
> **Official References:** [dart.dev/tutorials/language/streams](https://dart.dev/tutorials/language/streams) | [api.dart.dev/stable/dart-async/Stream-class.html](https://api.dart.dev/stable/dart-async/Stream-class.html)

---

## 1. Introduction

While a `Future<T>` represents a **single asynchronous value** that arrives in the future, many real-world systems deal with **continuous sequences of asynchronous values over time** (e.g. real-time chat messages, GPS coordinate updates, WebSocket packets, or user touch drag events).

In Dart, an asynchronous sequence of events is represented by a **`Stream<T>`**.

### Why Do We Need Streams?
1. **Real-Time Data Pipelines:** Process data chunks as they arrive across a network without waiting for the entire file or response to finish downloading.
2. **Event-Driven Architectures:** Connect loosely coupled UI components to reactive state changes.
3. **Flutter Integration:** Power reactive widgets like `StreamBuilder` that automatically rebuild whenever new data arrives.

---

## 2. Core Concepts

```mermaid
flowchart LR
    Source["Stream Source (Network / Sensors)"] -->|Data Events| StreamPipe["Stream<T> Pipeline"]
    StreamPipe -->|onData(value)| Listener1["Listener 1 (UI View)"]
    StreamPipe -->|onError(error)| Listener2["Listener 2 (Logger)"]
    StreamPipe -->|onDone()| Complete["Completion Handler"]
```

### The Three Event Types in a Stream
A `Stream<T>` emits three kinds of events:
1. **Data Event:** A value of type `T`.
2. **Error Event:** An exception or failure notification.
3. **Done Event:** A signal indicating the stream has finished and no further events will be emitted.

### Single-Subscription vs. Broadcast Streams

#### 1. Single-Subscription Stream (Default)
- Allows **exactly one listener** during its entire lifespan.
- If a second listener attempts to subscribe, Dart throws a `StateError: Stream has already been listened to`.
- It **buffers events** until a listener attaches, ensuring no data is lost before consumption begins.
- Used for discrete operations: reading a file from disk, downloading an HTTP body.

#### 2. Broadcast Stream (`asBroadcastStream()`)
- Allows **multiple independent listeners** simultaneously.
- Events are delivered in real time ("fire-and-forget"). If no listeners are attached when an event occurs, that event is **dropped/discarded**.
- Used for continuous notifications: device orientation, button clicks, WebSocket chats, location coordinates.

---

## 3. Syntax and Implementation

```dart
// bin/streams_demo.dart

import 'dart:async';

// 1. Creating a Periodic Stream
Stream<int> createCounterStream(int maxCount) {
  return Stream.periodic(const Duration(milliseconds: 200), (count) => count + 1)
      .take(maxCount); // Completes after maxCount emissions
}

void main() async {
  // ==========================================
  // 1. Single-Subscription Stream
  // ==========================================
  print('--- 1. Single-Subscription Stream ---');
  final singleStream = createCounterStream(3);

  // Listening to the stream
  final StreamSubscription<int> subscription = singleStream.listen(
    (data) => print('Subscriber A received: $data'),
    onError: (err) => print('Error: $err'),
    onDone: () => print('Stream completed!'),
    cancelOnError: false,
  );

  // Demonstrate Pause and Resume
  await Future.delayed(const Duration(milliseconds: 300));
  print('Pausing subscription for 400ms...');
  subscription.pause();
  await Future.delayed(const Duration(milliseconds: 400));
  print('Resuming subscription...');
  subscription.resume();

  // Wait for singleStream to finish
  await Future.delayed(const Duration(milliseconds: 800));

  // ==========================================
  // 2. Broadcast Stream (Multiple Listeners)
  // ==========================================
  print('\n--- 2. Broadcast Stream ---');
  final broadcastStream = createCounterStream(3).asBroadcastStream();

  // Listener 1: Fast UI consumer
  broadcastStream.listen((val) => print('[Listener 1 - UI] Value: $val'));

  // Listener 2: Logger consumer
  broadcastStream.listen((val) => print('[Listener 2 - Logger] Telemetry: $val'));

  // Wait for broadcast to finish
  await Future.delayed(const Duration(milliseconds: 800));

  // ==========================================
  // 3. Consuming with await for
  // ==========================================
  print('\n--- 3. Consuming with "await for" ---');
  await for (final value in createCounterStream(3)) {
    print('Awaited stream value: $value');
  }
}
```

### Expected Output
```text
--- 1. Single-Subscription Stream ---
Subscriber A received: 1
Pausing subscription for 400ms...
Resuming subscription...
Subscriber A received: 2
Subscriber A received: 3
Stream completed!

--- 2. Broadcast Stream ---
[Listener 1 - UI] Value: 1
[Listener 2 - Logger] Telemetry: 1
[Listener 1 - UI] Value: 2
[Listener 2 - Logger] Telemetry: 2
[Listener 1 - UI] Value: 3
[Listener 2 - Logger] Telemetry: 3

--- 3. Consuming with "await for" ---
Awaited stream value: 1
Awaited stream value: 2
Awaited stream value: 3
```

### Common Mistakes
- **Mistake 1: Listening twice to a single-subscription stream.**  
  ```dart
  final stream = File('data.txt').openRead();
  stream.listen(processA);
  stream.listen(processB); // CRASH AT RUNTIME: StateError: Stream has already been listened to!
  ```
  Fix: Call `.asBroadcastStream()` if multiple listeners are needed.
- **Mistake 2: Forgetting to cancel `StreamSubscription` in Flutter.**  
  If a widget subscribes to a stream in `initState()` and fails to call `subscription.cancel()` inside `dispose()`, the listener remains alive in memory forever, leaking both the widget State and memory.

---

## 4. Practical Example: Real-Time Stock Telemetry Pipeline

```dart
// lib/stock_ticker.dart

import 'dart:async';

class StockQuote {
  final String symbol;
  final double price;
  StockQuote(this.symbol, this.price);
  @override
  String toString() => '$symbol: \$$price';
}

void main() async {
  // Simulating an active financial stream
  final rawPriceStream = Stream.periodic(
    const Duration(milliseconds: 100),
    (count) => StockQuote('FLTR', 150.0 + (count * 1.25)),
  ).take(5);

  // Applying Stream Transformations
  final alertStream = rawPriceStream
      .where((quote) => quote.price > 152.0) // Filter only prices above $152
      .map((quote) => 'ALERT: ${quote.symbol} surged to \$${quote.price}'); // Transform

  await for (final alert in alertStream) {
    print(alert);
  }
}
```

---

## 5. Comparison: Single-Subscription vs Broadcast Streams

| Feature | Single-Subscription Stream | Broadcast Stream |
|:---|:---|:---|
| **Listener Capacity** | **Strictly 1 listener** | **Multiple listeners (unlimited)** |
| **Second Listener Attempt** | Throws `Bad state: Stream has already been listened to` | Subscribes cleanly |
| **Event Buffering** | Buffers events until listener connects | **No buffering (Events dropped if no listener)** |
| **Cancellation** | Cancelling listener closes underlying source | Cancelling 1 listener does not affect others |
| **Conversion** | Base type | Derived via `stream.asBroadcastStream()` |
| **Primary Use Cases** | File I/O, Database reads, HTTP requests | WebSockets, GPS, User Gestures, Notifications |

---

## 6. Interview Preparation

### Frequently Asked Questions

#### Q1: What is the difference between `Stream.listen()` and `await for`?
- **Answer:**
  - `Stream.listen()` is non-blocking and callback-driven. It registers handlers (`onData`, `onError`, `onDone`) and returns a `StreamSubscription`, allowing code below it to execute immediately.
  - `await for` is sequential and suspends the surrounding `async` function until the stream completes its entire lifecycle (or until a `break` is encountered). It cannot be used if other code in that function needs to run concurrently while the stream is active.

#### Q2: What happens if an error occurs in a Stream and no `onError` callback is provided?
- **Answer:** If a stream emits an error and the listener did not provide an `onError` handler (or if the stream is consumed via `await for` without a surrounding `try-catch`), the error is treated as an **unhandled asynchronous exception** and will be routed to the enclosing `Zone` or crash the application.

---

## 7. Practice Exercises

### Exercise 1 (Beginner): Stream Transformation
Create a stream emitting integers from 1 to 10. Use `.where()` and `.map()` to emit only even numbers multiplied by 10 (`[20, 40, 60, 80, 100]`), and print them using `await for`.

### Exercise 2 (Intermediate): Broadcast Stream Listener Management
Create a broadcast stream using `StreamController.broadcast()`. Connect two listeners. Pause one listener for 1 second, verify that the second listener continues receiving events in real time without being paused.

---

## 8. References
- [Dart Official Tutorial: Asynchronous Programming - Streams](https://dart.dev/tutorials/language/streams)
- [Dart API Reference: Stream](https://api.dart.dev/stable/dart-async/Stream-class.html)
