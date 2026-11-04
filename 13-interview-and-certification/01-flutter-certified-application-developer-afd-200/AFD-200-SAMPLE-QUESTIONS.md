# AFD-200 Exam Practice: 45 Comprehensive Sample Questions & Solutions

> **Module:** 13-interview-and-certification  
> **Subsection:** 01-flutter-certified-application-developer-afd-200  
> **Official Exam Code:** **AFD-200** | Passing Score: 70% (32/45 correct)

---

## Part 1: Dart Fundamentals and OOP (Questions 1–7)

### Question 1
**What is the output of the following Dart code?**
```dart
void main() {
  const a = [1, 2];
  final b = [1, 2];
  print(identical(a, const [1, 2]));
  print(identical(b, [1, 2]));
}
```
- A) `true`, `true`
- B) `true`, `false`
- C) `false`, `false`
- D) `false`, `true`

**Correct Answer:** **B**  
**Explanation:** `const` collections are canonicalized at compile time; Dart reuses the exact same in-memory object instance for identical `const` expressions, so `identical(a, const [1, 2])` is `true`. `b` is marked `final`, but initialized with a non-const list literal, creating a distinct heap allocation; comparing it to a newly instantiated non-const `[1, 2]` produces `false` because they occupy distinct memory addresses.

---

### Question 2
**Which Dart keyword allows a class to be used as an interface without creating a subclass hierarchy?**
- A) `extends`
- B) `implements`
- C) `with`
- D) `mixin`

**Correct Answer:** **B**  
**Explanation:** Dart has implicit interfaces: every class implicitly defines an interface containing all of its instance members. Using `implements` forces the concrete class to provide its own implementation for every member declared by the interface. `extends` is for class inheritance, and `with` is for mixin composition.

---

### Question 3
**In Dart 3 null safety, what does the `late` keyword guarantee?**
- A) The variable will always be null.
- B) The variable can never be assigned a value.
- C) The variable's initialization is deferred until it is first accessed, and it enforces non-nullable safety at runtime.
- D) The variable is stored in a background thread.

**Correct Answer:** **C**  
**Explanation:** `late` defers initialization until first read (lazy evaluation) and instructs the static analyzer that a non-nullable variable will be initialized before it is read. If accessed before assignment, a `LateInitializationError` is thrown at runtime.

---

### Question 4
**What is the correct syntax for an initializer list in a Dart constructor?**
- A) `Point(this.x, this.y) { x = 0; }`
- B) `Point(double x, double y) : x = x, y = y;`
- C) `Point(this.x, this.y) : assert(x >= 0), super();`
- D) `Point(this.x, this.y) => super();`

**Correct Answer:** **C**  
**Explanation:** An initializer list begins with a colon `:` after constructor parameters and executes *before* the constructor body and superclass constructor. It can contain field initializations, assertions, and `super()` invocations.

---

### Question 5
**Which mixin constraint enforces that `MixinA` can ONLY be applied to subclasses of `Widget`?**
- A) `mixin MixinA implements Widget`
- B) `mixin MixinA on Widget`
- C) `mixin MixinA with Widget`
- D) `mixin MixinA extends Widget`

**Correct Answer:** **B**  
**Explanation:** The `on` keyword defines a superclass constraint for a mixin, guaranteeing that `MixinA` can only be mixed into classes that extend or implement `Widget`.

---

### Question 6
**What is the result of evaluating `['A', if (false) 'B', for (var i in [1, 2]) '$i']`?**
- A) `['A', 'B', '1', '2']`
- B) `['A', null, '1', '2']`
- C) `['A', '1', '2']`
- D) Throws a syntax error.

**Correct Answer:** **C**  
**Explanation:** Collection-`if` and collection-`for` are built-in Dart collection operators. Because the condition is `false`, `'B'` is omitted entirely from the collection without inserting `null`. The loop evaluates and inserts `'1'` and `'2'`.

---

### Question 7
**What does a Factory Constructor do in Dart?**
- A) It guarantees a new instance is created every time.
- B) It allows returning an existing cached instance from memory or an instance of a subtype.
- C) It compiles Dart to C++ code.
- D) It runs on a background isolate.

**Correct Answer:** **B**  
**Explanation:** Unlike generative constructors, a `factory` constructor does not automatically allocate memory for `this`. It can return an existing instance from a cache (singleton pattern) or return a concrete subtype.

---

## Part 2: Flutter UI Widgets & Layouts (Questions 8–18)

### Question 8
**What are the Three Trees maintained by Flutter's rendering pipeline?**
- A) DOM Tree, CSS Tree, Render Tree
- B) Widget Tree, Element Tree, RenderObject Tree
- C) State Tree, Context Tree, Paint Tree
- D) Layout Tree, Box Tree, GPU Tree

**Correct Answer:** **B**  
**Explanation:** The Widget Tree holds immutable configurations. The Element Tree manages component lifecycle and state persistence. The RenderObject Tree computes layout geometry, box constraints, and GPU painting.

---

### Question 9
**What is the core rule of the Flutter layout protocol?**
- A) "Sizes go down, positions go up, child sets constraints."
- B) "Constraints go down, sizes go up, parent sets position."
- C) "Parent determines size, child determines position."
- D) "Layout is calculated from screen coordinates."

**Correct Answer:** **B**  
**Explanation:** Parents pass `BoxConstraints` down to children. Children choose their own `Size` within those bounds and return it upward. The parent then calculates the child's `Offset(x, y)` position.

---

### Question 10
**What error occurs when a `ListView` is placed directly inside an unconstrained `Column`?**
- A) `NoSuchMethodError`
- B) `RenderFlex overflowed by X pixels`
- C) `Vertical viewport was given unbounded height`
- D) `StateError: Bad state`

**Correct Answer:** **C**  
**Explanation:** `Column` provides infinite maximum height constraints along its vertical main axis. `ListView` attempts to expand infinitely along its scroll axis, causing a fatal unbounded viewport conflict.

---

### Question 11
**How do you fix the unbounded height error when placing a `ListView` inside a `Column`?**
- A) Wrap the `ListView` in `Expanded`.
- B) Set `shrinkWrap: true` on the `ListView`.
- C) Wrap the `ListView` in a `SizedBox` with a fixed height.
- D) All of the above are valid solutions depending on the layout requirement.

**Correct Answer:** **D**  
**Explanation:** Wrapping in `Expanded` fills remaining space. `SizedBox` forces a fixed height. `shrinkWrap: true` bounds height to the list's total content length (though it sacrifices virtualization).

---

### Question 12
**What is the fundamental difference between `Expanded` and `Flexible`?**
- A) `Expanded` forces the child to fill all available space (`FlexFit.tight`), while `Flexible` allows the child to be smaller than the available space (`FlexFit.loose`).
- B) `Expanded` only works in Rows, `Flexible` only works in Columns.
- C) `Flexible` has no `flex` factor property.
- D) There is no difference; `Expanded` is just an alias.

**Correct Answer:** **A**  
**Explanation:** `Expanded` is a subclass of `Flexible` with `fit: FlexFit.tight`, forcing the child to take all allocated flex space. `Flexible` defaults to `FlexFit.loose`, permitting the child to shrink to its intrinsic size.

---

### Question 13
**Which widget should be used to display an overlay card on top of another widget with exact pixel coordinates?**
- A) `Row` with `Spacer`
- B) `Stack` with `Positioned`
- C) `Column` with `Padding`
- D) `Wrap` with `Transform`

**Correct Answer:** **B**  
**Explanation:** `Stack` layers widgets on top of each other along the Z-axis, and `Positioned` establishes absolute offsets (`top`, `left`, `bottom`, `right`) relative to the Stack's bounds.

---

### Question 14
**Why is `ListView.builder` preferred over `ListView(children: [...])` for a 1,000-item list?**
- A) `ListView.builder` automatically sorts items.
- B) `ListView.builder` virtualizes elements, instantiating only visible items on demand.
- C) `ListView.builder` disables scrolling.
- D) `ListView(children: [...])` cannot display images.

**Correct Answer:** **B**  
**Explanation:** `ListView(children: [...])` constructs all 1,000 child widgets in memory upfront. `ListView.builder` virtualizes the list, constructing and laying out only the items currently visible in the viewport plus cache buffer.

---

### Question 15
**What does `CustomScrollView` accept in its `slivers` property?**
- A) Only `Container` and `Card` widgets.
- B) Any standard Box widget directly.
- C) Only widgets that implement the Sliver protocol (e.g. `SliverList`, `SliverAppBar`, or Box widgets wrapped in `SliverToBoxAdapter`).
- D) Only `ListView` widgets.

**Correct Answer:** **C**  
**Explanation:** `CustomScrollView` operates via the Sliver protocol (`RenderSliver`). Box widgets cannot be passed directly; they must be wrapped inside `SliverToBoxAdapter`.

---

### Question 16
**Which `SliverAppBar` property keeps the app bar visible at the top of the screen when scrolled all the way down?**
- A) `floating: true`
- B) `pinned: true`
- C) `snap: true`
- D) `expandedHeight: 200`

**Correct Answer:** **B**  
**Explanation:** `pinned: true` anchors the collapsed AppBar (56px) permanently to the top of the viewport when the user scrolls content down.

---

### Question 17
**What is the purpose of `IndexedStack`?**
- A) It scrolls through images horizontally.
- B) It displays one child at a time from a list of children while preserving the state of all off-screen children.
- C) It sorts database indexes.
- D) It compiles C++ code.

**Correct Answer:** **B**  
**Explanation:** `IndexedStack` sizes itself to the largest child and displays only the child at `index`. Crucially, all non-visible children remain mounted in the Element tree, keeping their scroll positions, text field inputs, and controller states alive.

---

### Question 18
**What does `Wrap` do that `Row` cannot?**
- A) It runs animations.
- B) It automatically breaks children into a new line (run) when horizontal space runs out, preventing overflow errors.
- C) It communicates with native Kotlin code.
- D) It loads images from network.

**Correct Answer:** **B**  
**Explanation:** `Row` lays out children in a single horizontal line and throws a `RenderFlex overflowed` error if children exceed available width. `Wrap` automatically wraps overflowed children to the next line.

---

## Part 3: User Input, Forms, and Navigation (Questions 19–25)

### Question 19
**Which key type is used to validate and save all fields within a `Form` widget?**
- A) `UniqueKey()`
- B) `ValueKey<String>()`
- C) `GlobalKey<FormState>()`
- D) `ObjectKey()`

**Correct Answer:** **C**  
**Explanation:** `GlobalKey<FormState>` provides access to the underlying `FormState` object anywhere in the tree, allowing you to invoke `_formKey.currentState?.validate()` and `save()`.

---

### Question 20
**In Flutter 3.12+, which widget replaces the deprecated `WillPopScope` for handling Android back gestures?**
- A) `BackScope`
- B) `PopScope`
- C) `SystemNavigator`
- D) `WillPopListener`

**Correct Answer:** **B**  
**Explanation:** `WillPopScope` was deprecated because its asynchronous future return broke Android 14's native Predictive Back gesture animation. `PopScope` uses synchronous `canPop` checking and an `onPopInvokedWithResult` callback.

---

### Question 21
**Where should you invoke `_controller.dispose()` for a `TextEditingController`?**
- A) In `initState()`
- B) In `build()`
- C) In `dispose()` of the `StatefulWidget`
- D) It is disposed automatically by garbage collection; manual disposal is unnecessary.

**Correct Answer:** **C**  
**Explanation:** `TextEditingController` attaches listeners to the native OS platform text engine. Failing to call `.dispose()` inside the `State`'s `dispose()` lifecycle method results in severe memory leaks.

---

### Question 22
**What is the purpose of `FocusNode` in Flutter?**
- A) To increase screen brightness.
- B) To identify and control which widget has the active hardware keyboard focus.
- C) To compile Dart to native machine code.
- D) To zoom the camera lens.

**Correct Answer:** **B**  
**Explanation:** A `FocusNode` is an object that can be passed to a focusable widget (like `TextField`) to request focus (`focusNode.requestFocus()`), listen for focus changes, or dismiss the keyboard (`focusNode.unfocus()`).

---

### Question 23
**How do you pass data back to the previous screen when calling `Navigator.pop`?**
- A) `Navigator.pop(context, 'resultData');`
- B) `Navigator.of(context).send('resultData');`
- C) `Navigator.pushNamed(context, '/back', arguments: 'resultData');`
- D) Data cannot be returned via Navigator.

**Correct Answer:** **A**  
**Explanation:** `Navigator.pop(context, [result])` accepts an optional result parameter. The awaiting future at the calling screen (`final result = await Navigator.push(...)`) resolves with this returned value.

---

### Question 24
**What happens if `autovalidateMode: AutovalidateMode.onUserInteraction` is set on a `TextFormField`?**
- A) Validation runs before the screen renders.
- B) Validation errors are displayed only after the user has interacted with the field for the first time.
- C) The user is prevented from typing invalid characters.
- D) Validation runs once every 5 seconds.

**Correct Answer:** **B**  
**Explanation:** It prevents showing red error messages on pristine, untouched forms, displaying validation messages only after the user touches and edits the input.

---

### Question 25
**What is the difference between `Navigator.push` and `Navigator.pushReplacement`?**
- A) `push` animates; `pushReplacement` does not.
- B) `push` stacks a new route on top of the current route; `pushReplacement` disposes the current route and replaces it with the new route, preventing the user from navigating back to it.
- C) `pushReplacement` is only for web.
- D) There is no difference.

**Correct Answer:** **B**  
**Explanation:** `pushReplacement` pops the current route before pushing the new one, ideal for transitioning from a Splash/Login screen to the Main Dashboard where pressing "Back" should exit the app rather than return to login.

---

## Part 4: State Management (Questions 26–31)

### Question 26
**What is the difference between Ephemeral State and App State?**
- A) Ephemeral state is stored on the hard drive; App state is stored in the cloud.
- B) Ephemeral state is local to a single widget and cleanly managed with `setState()`; App state is shared across multiple disconnected screens and managed via Provider, Riverpod, or BLoC.
- C) Ephemeral state only works on iOS.
- D) Ephemeral state requires code generation.

**Correct Answer:** **B**  
**Explanation:** Ephemeral state (e.g. current tab, checkbox toggle, animation progress) lives strictly inside a single widget. App state (e.g. user session, shopping cart, dark mode) is shared globally across the application.

---

### Question 27
**In `Provider`, what is the consequence of calling `context.watch<T>()` inside a button's `onPressed` callback?**
- A) Nothing; it works correctly.
- B) It throws a runtime error because inherited widget subscriptions can only be established during the `build()` phase.
- C) It compiles the app to C++.
- D) It closes the application.

**Correct Answer:** **B**  
**Explanation:** `context.watch()` registers an Element as a listener on an InheritedWidget. Flutter strictly forbids registering new InheritedWidget dependencies outside the `build()` lifecycle method. Use `context.read<T>()` in callbacks.

---

### Question 28
**Why is `Selector` preferred over `Consumer` in Provider for performance optimization?**
- A) `Selector` is easier to type.
- B) `Consumer` cannot read strings.
- C) `Selector` extracts a specific sub-value and rebuilds its child ONLY when that selected sub-value changes, ignoring other property changes on the `ChangeNotifier`.
- D) `Selector` is compile-safe.

**Correct Answer:** **C**  
**Explanation:** `Consumer` rebuilds whenever *any* field triggers `notifyListeners()`. `Selector` filters notifications and triggers rebuilds only if the selected sub-property changes.

---

### Question 29
**In modern Riverpod (2.x / 3.x), what replaces legacy `StateNotifierProvider` for managing asynchronous data?**
- A) `FutureProvider`
- B) `AsyncNotifierProvider`
- C) `StreamNotifier`
- D) `StateProvider`

**Correct Answer:** **B**  
**Explanation:** In modern Riverpod, `AsyncNotifierProvider` (paired with `AsyncNotifier<T>`) is the official replacement for `StateNotifierProvider`, providing automatic `AsyncValue` lifecycle handling and built-in dependency access via `ref`.

---

### Question 30
**What does `ref.watch(myProvider)` do when called inside a `ConsumerWidget.build()` method?**
- A) Reads the value once without listening for changes.
- B) Listens to `myProvider` and triggers a widget rebuild whenever its state updates.
- C) Deletes the provider from memory.
- D) Dispatches an HTTP request.

**Correct Answer:** **B**  
**Explanation:** `ref.watch()` subscribes the widget to the provider, automatically re-running the widget's `build()` method whenever the provider's state updates.

---

### Question 31
**What is the core difference between `Cubit` and `Bloc` in `flutter_bloc`?**
- A) `Cubit` handles network requests; `Bloc` handles UI.
- B) `Cubit` emits state via direct function calls (`cubit.increment()`), while `Bloc` receives explicit event objects (`bloc.add(IncrementEvent())`) and maps them to states.
- C) `Bloc` cannot be used with Flutter.
- D) `Cubit` is deprecated.

**Correct Answer:** **B**  
**Explanation:** `Cubit` is a lightweight, function-driven state manager (`emit()`). `Bloc` is an event-driven state manager requiring formal Event classes, making it ideal for event traceability, analytics, and reactive event transformers.

---

## Part 5: Asynchronous Programming & REST APIs (Questions 32–37)

### Question 32
**Which queue in the Dart Event Loop has higher execution priority?**
- A) Event Queue
- B) Microtask Queue
- C) I/O Queue
- D) Timer Queue

**Correct Answer:** **B**  
**Explanation:** The Dart Event Loop always processes all pending jobs in the Microtask Queue to completion before pulling the next event from the Event Queue.

---

### Question 33
**What is the expected output of this code snippet?**
```dart
void main() {
  print('1');
  Future(() => print('2'));
  scheduleMicrotask(() => print('3'));
  print('4');
}
```
- A) `1, 2, 3, 4`
- B) `1, 4, 2, 3`
- C) `1, 4, 3, 2`
- D) `3, 1, 4, 2`

**Correct Answer:** **C**  
**Explanation:** Synchronous execution runs first (`1`, then `4`). Next, the microtask queue runs (`3`). Finally, the event queue event (the `Future`) runs (`2`).

---

### Question 34
**Why should you avoid creating a new `Future` or stream inside a `build()` method?**
- A) It causes a compile-time error.
- B) Every time the widget rebuilds (e.g. keyboard opens or animation ticks), a brand-new asynchronous network call is spawned, wasting bandwidth and resetting state.
- C) Dart does not allow futures in widgets.
- D) It crashes the GPU.

**Correct Answer:** **B**  
**Explanation:** The `build()` method can execute 60 times a second. Instantiating a `Future` inside `build()` re-executes the network request on every single rebuild. Futures must be initialized in `initState()` or managed inside state controllers.

---

### Question 35
**What is the primary advantage of `Dio` over Dart's native `package:http`?**
- A) `Dio` works without an internet connection.
- B) `Dio` supports Interceptors, global BaseOptions, automatic token refresh, request cancellation (`CancelToken`), and automatic JSON parsing.
- C) `Dio` compiles to Objective-C.
- D) `Dio` is built into the Flutter SDK.

**Correct Answer:** **B**  
**Explanation:** `Dio` is an enterprise HTTP client offering request/response/error interceptors, cancellation tokens, file upload FormData, and connection timeouts out of the box.

---

### Question 36
**What HTTP status code represents an expired or invalid JWT authentication token?**
- A) 200 OK
- B) 400 Bad Request
- C) 401 Unauthorized
- D) 500 Internal Server Error

**Correct Answer:** **C**  
**Explanation:** HTTP 401 Unauthorized signifies that the request lacks valid authentication credentials, signaling interceptors to execute a refresh token routine.

---

### Question 37
**How do you abort an ongoing HTTP request in Dio when a user navigates away?**
- A) `dio.stop()`
- B) Pass a `CancelToken` to the request options and invoke `cancelToken.cancel()`.
- C) Call `Navigator.pop()`.
- D) Set `timeout = Duration.zero`.

**Correct Answer:** **B**  
**Explanation:** Passing a `CancelToken` to `dio.get(..., cancelToken: token)` allows you to call `token.cancel()` at any point, aborting the network socket immediately.

---

## Part 6: Local Data Persistence (Questions 38–41)

### Question 38
**Why should JWT tokens and passwords NEVER be saved in `SharedPreferences`?**
- A) `SharedPreferences` has a 10-character limit.
- B) `SharedPreferences` stores data as unencrypted plaintext XML/plist files on disk, making it easily readable on rooted or jailbroken devices.
- C) `SharedPreferences` deletes all data after 24 hours.
- D) `SharedPreferences` only supports boolean values.

**Correct Answer:** **B**  
**Explanation:** `SharedPreferences` is completely unencrypted. Sensitive credentials must be persisted using hardware-backed cryptographic solutions like `flutter_secure_storage`.

---

### Question 39
**What is a `Box` in the Hive database?**
- A) A 3D widget that renders cubes.
- B) An embedded binary key-value storage container roughly analogous to an SQL table.
- C) An iOS-specific container.
- D) A network socket.

**Correct Answer:** **B**  
**Explanation:** In Hive, all data is stored inside Boxes (`Hive.openBox<T>('name')`). Boxes store data in binary format and keep indexes in RAM for ultra-fast $O(1)$ lookups.

---

### Question 40
**What does `Drift` (formerly Moor) provide over raw `sqflite`?**
- A) Drift writes to Firebase directly.
- B) Drift parses SQL queries at compile time, verifies schema correctness before runtime, and provides reactive auto-updating `Stream` queries (`watch()`).
- C) Drift does not use SQLite.
- D) Drift is an audio player.

**Correct Answer:** **B**  
**Explanation:** Drift is a type-safe reactive persistence library that catches SQL typos at compile time and auto-emits new query results whenever underlying tables mutate.

---

### Question 41
**What is a TypeAdapter in Hive?**
- A) An electronic cable adapter.
- B) A class that serializes and deserializes custom Dart objects into and out of binary bytes.
- C) A Flutter layout widget.
- D) A type converter for REST APIs.

**Correct Answer:** **B**  
**Explanation:** Hive natively only knows how to store primitive types. To persist custom domain models, you write or generate a `TypeAdapter<T>` with a unique `typeId`.

---

## Part 7: Architecture, Testing, and Publishing (Questions 42–45)

### Question 42
**In Clean Architecture, which layer must contain ZERO Flutter imports (`package:flutter`)?**
- A) Presentation Layer
- B) Data Layer
- C) Domain Layer
- D) UI Layer

**Correct Answer:** **C**  
**Explanation:** The Domain layer contains pure business logic, entities, and use cases. It must be 100% independent of UI frameworks, device SDKs, and third-party libraries.

---

### Question 43
**What is the purpose of `tester.pumpAndSettle()` in a widget test?**
- A) It compiles the widget to WebAssembly.
- B) It repeatedly advances the virtual clock until all scheduled animations, timers, and microtasks have completed and no frames remain.
- C) It connects to a real physical phone.
- D) It clicks all buttons on the screen.

**Correct Answer:** **B**  
**Explanation:** `tester.pumpAndSettle()` simulates the passage of time until the UI has completely settled into a stable, non-animating state.

---

### Question 44
**What happens if your iOS app requests camera access without declaring `NSCameraUsageDescription` in `Info.plist`?**
- A) The camera displays a black screen.
- B) The app terminates immediately with a native `SIGABRT` crash upon requesting permission.
- C) iOS automatically displays a default message.
- D) The camera works normally.

**Correct Answer:** **B**  
**Explanation:** iOS strictly enforces privacy descriptions. Missing a purpose string for a protected sensor causes an immediate crash and results in automatic rejection during Apple App Store review.

---

### Question 45
**What file format is mandatory when uploading new applications to the Google Play Store?**
- A) `.apk` (Android Package)
- B) `.aab` (Android App Bundle)
- C) `.ipa` (iOS App Store Package)
- D) `.jar` (Java Archive)

**Correct Answer:** **B**  
**Explanation:** Google Play requires all new apps to be submitted as **Android App Bundles (`.aab`)**, which enables Google Play's Dynamic Delivery system to generate split APKs customized to each user's specific device architecture and display density.
