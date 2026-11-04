# StreamControllers and StreamTransformers in Dart

> **Module:** 02-advanced-dart  
> **Subsection:** 03-streams-and-generators  
> **Prerequisites:** [stream-fundamentals.md](./stream-fundamentals.md)  
> **Official References:** [api.dart.dev/stable/dart-async/StreamController-class.html](https://api.dart.dev/stable/dart-async/StreamController-class.html) | [api.dart.dev/stable/dart-async/StreamTransformer-class.html](https://api.dart.dev/stable/dart-async/StreamTransformer-class.html)

---

## 1. Introduction

While utility methods like `Stream.periodic()` create simple automated streams, real-world development requires **manually pushing events into a stream** from arbitrary user interactions, background sockets, or database triggers.

In Dart, this is managed by the **`StreamController<T>`**. A `StreamController` acts as a bridge between an event producer and event consumers:
- The **producer** pushes events into the controller's **`Sink`**.
- The **consumers** listen to the controller's **`Stream`**.

To cleanly intercept, validate, or transform stream events without cluttering business logic, Dart provides **`StreamTransformer<S, T>`**.

### Why Do We Need StreamControllers and Transformers?
1. **Producer-Consumer Decoupling:** Consumers only see a read-only `Stream<T>` and cannot tamper with event emission.
2. **Lifecycle Management:** Notifies producers when the first listener subscribes (`onListen`) or when all listeners detach (`onCancel`).
3. **Reusable Data Pipelines:** `StreamTransformer` allows packaging complex event mapping (decoding bytes, parsing lines, debouncing) into reusable components.

---

## 2. Core Concepts

```mermaid
flowchart LR
    Producer["Producer Code"] -->|sink.add(data)| Sink["StreamSink<T>"]
    subgraph Controller["StreamController<T>"]
        Sink
        Pipeline["StreamTransformer<br/>(Transforms / Filters / Decodes)"]
        StreamObj["Stream<T>"]
        Sink --> Pipeline --> StreamObj
    end
    StreamObj -->|listen()| Consumer["Consumer (UI / State Manager)"]
```

### 1. The Anatomy of a `StreamController`
- **`StreamSink<T> get sink`:** The entry point for writing data. Methods include:
  - `sink.add(T event)`: Pushes a new data value.
  - `sink.addError(Object error)`: Pushes an error event.
  - `sink.close()`: Closes the stream, signaling `onDone` to all listeners.
- **`Stream<T> get stream`:** The exit point for reading data. Only consumers should see this.

### 2. Architectural Best Practice: Encapsulation
Never expose a `StreamController` directly to outside callers. Always keep the controller and its sink `_private`, exposing only the public read-only `stream`:
```dart
// GOOD ARCHITECTURE:
final _controller = StreamController<String>();
Stream<String> get onMessage => _controller.stream; // Read-only!
```

### 3. `StreamTransformer<S, T>`
A transformer takes a stream of type `S` and converts it into a stream of type `T`:
```dart
final transformedStream = rawStream.transform(myTransformer);
```
Standard SDK examples include `utf8.decoder` (converting byte buffers to strings) and `LineSplitter` (splitting raw text streams by newline characters).

---

## 3. Syntax and Implementation

```dart
// bin/stream_controller_demo.dart

import 'dart:async';

class ChatRoomService {
  // 1. Private StreamController (Broadcast so multiple UI widgets can listen)
  final _messageController = StreamController<String>.broadcast();

  // 2. Exposing ONLY the read-only stream publicly
  Stream<String> get messageStream => _messageController.stream;

  // 3. Method to push data into the sink
  void postMessage(String username, String message) {
    if (_messageController.isClosed) {
      throw StateError('Cannot post message: ChatRoomService is closed');
    }
    
    if (message.trim().isEmpty) {
      _messageController.addError('Message cannot be blank');
      return;
    }

    _messageController.sink.add('[$username]: $message');
  }

  // 4. Teardown / Disposal
  Future<void> dispose() async {
    print('--> Closing chat stream controller...');
    await _messageController.close();
  }
}

void main() async {
  final chat = ChatRoomService();

  // Custom StreamTransformer: Filters out profanity and uppercases messages
  final sanitizingTransformer = StreamTransformer<String, String>.fromHandlers(
    handleData: (message, sink) {
      if (message.contains('SPAM')) {
        sink.addError('Rejected spam message');
      } else {
        sink.add(message.toUpperCase());
      }
    },
    handleError: (error, stackTrace, sink) {
      sink.add('[ERROR SANITIZED] $error');
    },
  );

  // Applying transformer to the public stream
  final sanitizedStream = chat.messageStream.transform(sanitizingTransformer);

  // Subscriber 1: Terminal Logger
  final subscription = sanitizedStream.listen(
    (msg) => print('[Client 1] $msg'),
    onError: (err) => print('[Client 1 Error] $err'),
    onDone: () => print('[Client 1] Chat session terminated.'),
  );

  // Pushing messages through service
  chat.postMessage('Alice', 'Hello team!');
  chat.postMessage('Bob', 'SPAM Buy crypto now!');
  chat.postMessage('Charlie', ''); // Triggers blank error
  chat.postMessage('Alice', 'Let us start the Flutter review.');

  await Future.delayed(const Duration(milliseconds: 100));

  // Clean shutdown
  await chat.dispose();
  await subscription.cancel();
}
```

### Expected Output
```text
[Client 1] [ALICE]: HELLO TEAM!
[Client 1 Error] Rejected spam message
[Client 1] [ERROR SANITIZED] Message cannot be blank
[Client 1] [ALICE]: LET US START THE FLUTTER REVIEW.
--> Closing chat stream controller...
[Client 1] Chat session terminated.
```

### Common Mistakes
- **Mistake 1: Adding events to a closed controller.**  
  Calling `controller.add(value)` after calling `controller.close()` throws `Bad state: Cannot add event after closing`. Always check `controller.isClosed` or guard the service lifecycle.
- **Mistake 2: Leaking controllers in Flutter State objects.**  
  Every `StreamController` created inside a `StatefulWidget` or BLoC/Cubit **must be closed** inside `dispose()`. Failing to close it keeps the stream pipeline and listeners alive, causing severe memory leaks.

---

## 4. Practical Example: Reading and Transforming a Text Stream

```dart
// lib/line_transformer_example.dart

import 'dart:async';
import 'dart:convert';

void main() async {
  // Simulating incoming byte chunks from an HTTP network connection
  final byteController = StreamController<List<int>>();

  // Transforming: Bytes -> String -> Individual Lines
  final lineStream = byteController.stream
      .transform(utf8.decoder) // StreamTransformer<List<int>, String>
      .transform(const LineSplitter()); // StreamTransformer<String, String>

  lineStream.listen((line) {
    print('Received Line: "$line"');
  });

  // Producer emits byte chunks
  byteController.add(utf8.encode('Header: Application/JSON\nContent-Length: 42\n'));
  byteController.add(utf8.encode('Status: 200 OK\n'));

  await Future.delayed(const Duration(milliseconds: 50));
  await byteController.close();
}
```

---

## 5. Comparison: `StreamController` vs `StreamController.broadcast`

| Feature | `StreamController()` | `StreamController.broadcast()` |
|:---|:---|:---|
| **Max Subscribers** | Strictly 1 | Unlimited |
| **Backpressure / Buffering** | Buffers events before listener attaches | **Zero buffering (Drops events if no listeners)** |
| **Listener Callbacks** | `onListen`, `onCancel`, `onPause`, `onResume` | `onListen`, `onCancel` |
| **Primary Use Cases** | Reading files, discrete requests | UI events, event buses, real-time sync |

---

## 6. Interview Preparation

### Frequently Asked Questions

#### Q1: Why should you never expose a `StreamController` directly as a public property?
- **Answer:** Exposing the `StreamController` violates encapsulation by allowing external code to access the `sink`. Any external caller could arbitrarily push events (`add`), inject errors (`addError`), or close the stream (`close`), corrupting the internal state of your service. Expose only `controller.stream`.

#### Q2: What is the purpose of the `onListen` and `onCancel` callbacks in a `StreamController`?
- **Answer:** They enable lazy resource management (cold streams). When the first subscriber attaches, `onListen` fires, allowing you to open a network socket, start a hardware sensor (e.g. GPS, accelerometer), or initiate timer ticks. When all subscribers detach, `onCancel` fires, allowing you to pause or disconnect the underlying expensive hardware/network resources immediately.

---

## 7. Practice Exercises

### Exercise 1 (Intermediate): Temperature Sensor Controller
Create a `TemperatureSensor` class with a private `StreamController<double>`. Expose a `Stream<double>` and a method `void recordReading(double temp)`. In the `StreamController` constructor, use `onListen` and `onCancel` to print diagnostic logs when listeners connect and disconnect.

---

## 8. References
- [Dart API: StreamController](https://api.dart.dev/stable/dart-async/StreamController-class.html)
- [Dart API: StreamTransformer](https://api.dart.dev/stable/dart-async/StreamTransformer-class.html)
