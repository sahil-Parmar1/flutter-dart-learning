# Junior and Mid-Level Flutter & Dart Interview Questions

> **Module:** 13-interview-and-certification  
> **Subsection:** 02-junior-mid-level-qa  
> **Target Audience:** Junior to Mid-Level Flutter Developers Preparing for Technical Interviews

---

## 1. What is the difference between `final` and `const` in Dart?

### 🎙️ Concise Verbal Answer (30 Seconds):
> "`const` variables represent compile-time constants whose values must be known before the program runs; identical `const` instances are canonicalized in memory. `final` variables are runtime constants: their values can be calculated dynamically at runtime (such as reading the current time or an API response), but once assigned, they can never be reassigned."

### 🔍 Deep Technical Explanation:
- `const` values are baked directly into the Ahead-of-Time (AOT) compiled binary. Dart canonicalizes `const` objects: `const [1, 2] == const [1, 2]` evaluates to `true` via `identical()` because they share the exact same memory address.
- `final` is evaluated when program execution reaches the statement. If you write `final date = DateTime.now();`, it works because `DateTime.now()` is computed at runtime. Writing `const date = DateTime.now();` fails with a compile error because the current time cannot be known at compile time.
- In Flutter widgets, using `const MyWidget()` tells the engine: *"This widget's configuration cannot change; skip calling its constructor and re-allocating memory during parent rebuilds."*

---

## 2. How does Hot Reload work compared to Hot Restart?

### 🎙️ Concise Verbal Answer (30 Seconds):
> "Hot Reload injects updated source code files directly into the running Dart VM without restarting the app or destroying its current state. It immediately triggers a `build()` pass across the widget tree. Hot Restart tears down the entire Dart VM isolate, wipes all in-memory state, and restarts the application from `main()`, which is necessary when modifying initializers or global static state."

### 🔍 Deep Technical Explanation:
- **Hot Reload:** Preserves the Element and State trees. Flutter scans for modified `.dart` files, sends the changes to the Dart VM's Just-In-Time (JIT) compiler, and invokes `reassemble()` on all mounted Elements, causing them to re-run their `build()` methods.
- **When Hot Reload Fails:** Modifying `initState()`, changing static fields, switching a `StatelessWidget` to a `StatefulWidget`, or changing the app's `main()` entrypoint requires a **Hot Restart**.

---

## 3. Explain the complete `StatefulWidget` Lifecycle in order.

### 🎙️ Concise Verbal Answer (30 Seconds):
> "The lifecycle begins with `createState()`, followed by `initState()` for one-time initialization, `didChangeDependencies()` when inherited ancestors change, and `build()` to return the widget tree. If parent properties update, `didUpdateWidget()` executes. When removed from the tree, `deactivate()` is called, culminating in `dispose()` to permanently release memory and cancel listeners."

```mermaid
sequenceDiagram
    autonumber
    participant FW as Framework
    participant State as State<MyWidget>

    FW->>State: 1. createState()
    FW->>State: 2. initState() (One-time controller & subscription setup)
    FW->>State: 3. didChangeDependencies() (InheritedWidget / Theme / Locale lookups)
    loop Active Rendering Cycle
        FW->>State: 4. build(context) (Returns Widget configuration)
        Note over State: Parent rebuilds with new props? -> didUpdateWidget()
        Note over State: setState() called? -> Schedules build()
    end
    FW->>State: 5. deactivate() (Removed from current tree location)
    FW->>State: 6. dispose() (Permanent teardown; release memory & streams)
```

### ⚠️ Common Gotchas to Mention:
- Never call `BuildContext.dependOnInheritedWidgetOfExactType` (e.g. `Theme.of(context)` or `MediaQuery.of(context)`) inside `initState()`! The framework has not fully wired up inherited dependencies yet. Do this inside `didChangeDependencies()` instead.
- Always call `super.initState()` at the start of your method, and `super.dispose()` at the very end of `dispose()`.

---

## 4. What is `BuildContext` and why is it so important?

### 🎙️ Concise Verbal Answer (30 Seconds):
> "`BuildContext` is not a data container; it is an abstract interface representing the location of a widget within Flutter's Element Tree. Every `Element` implements `BuildContext`. It is used by widgets to look up inherited ancestors (like `Navigator.of(context)` or `Theme.of(context)`) by traversing upward through the tree hierarchy."

### 🔍 Deep Technical Explanation:
Because `BuildContext` is literally the `Element` itself, passing a context from a parent widget to a lookup like `Scaffold.of(context)` will fail if the Scaffold is created inside the *same* `build()` method! The context passed must be a **descendant** of the Scaffold. Using the `Builder` widget solves this by providing a fresh child context below the target ancestor.

---

## 5. What are Keys in Flutter, and when are they required?

### 🎙️ Concise Verbal Answer (30 Seconds):
> "Keys preserve the identity and state of widgets when they are reordered, inserted, or removed among siblings of the same runtime type. Flutter's element reconciliation algorithm checks `Widget.canUpdate(oldWidget, newWidget)` by comparing runtime type and key. If keys are omitted in a reorderable list of stateful widgets, the framework reuses the old elements with the wrong state."

### 🔍 Key Categories:
- **`ValueKey<T>`:** Keyed by a string or integer (e.g. `ValueKey(user.id)`).
- **`ObjectKey`:** Keyed by object instance identity.
- **`UniqueKey`:** Guaranteed unique random identity; forces recreation of an element.
- **`GlobalKey`:** Globally unique across the entire app; allows state access (`currentState`) and reparenting widgets across different locations in the tree without losing state.

---

## 6. How does `setState()` work internally?

### 🎙️ Concise Verbal Answer (30 Seconds):
> "`setState()` takes a synchronous callback, executes it, and marks the widget's corresponding `Element` as 'dirty'. During the next frame tick of the engine, Flutter's pipeline rebuilds all dirty elements by executing their `build()` methods and updating the RenderObject tree."

### ⚠️ What Developers Get Wrong:
The callback passed to `setState(() { _count++; })` does not magically trigger the update; calling `_count++; setState(() {});` produces the identical result. However, placing code inside the callback makes your intent explicit and guarantees the mutation occurs synchronously before the dirty flag is set.

---

## 7. What causes a `RenderFlex overflowed` error and how do you resolve it?

### 🎙️ Concise Verbal Answer (30 Seconds):
> "A `RenderFlex` overflow occurs inside a `Row` or `Column` when the cumulative size of its children exceeds the maximum available space along the flex layout's main axis. To resolve it, wrap the overflowing child in an `Expanded` or `Flexible` widget so it shrinks/fills available space, or replace the `Row`/`Column` with a scrollable container (`ListView` / `SingleChildScrollView`) or a `Wrap` widget."

---

## 8. What is the difference between `FutureBuilder` and `StreamBuilder`?

### 🎙️ Concise Verbal Answer (30 Seconds):
> "`FutureBuilder` handles a single, one-shot asynchronous operation that completes once with either data or an error (e.g. an HTTP GET request). `StreamBuilder` listens to an ongoing, continuous sequence of asynchronous events emitted over time (e.g. a WebSocket channel, Firebase live query, or sensor stream) and rebuilds on every new emission."

### ⚠️ Critical Implementation Trap:
Never instantiate the `Future` directly inside the `future:` property of `FutureBuilder`:
```dart
// ❌ WRONG: Re-fires network call on every single rebuild!
FutureBuilder(future: api.fetchUser(), ...)

// ✅ CORRECT: Store future in a variable in initState()
late final _userFuture = api.fetchUser();
FutureBuilder(future: _userFuture, ...)
```
