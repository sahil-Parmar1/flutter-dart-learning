# StatelessWidget vs. StatefulWidget and Async UI Builders

> **Module:** 03-flutter-fundamentals  
> **Subsection:** 03-stateless-and-stateful-widgets  
> **Prerequisites:** [02-widget-tree-element-tree-renderobject/the-three-trees.md](../02-widget-tree-element-tree-renderobject/the-three-trees.md)  
> **Official References:** [docs.flutter.dev/ui/widgets/basics](https://docs.flutter.dev/ui/widgets/basics) | [api.flutter.dev/flutter/widgets/FutureBuilder-class.html](https://api.flutter.dev/flutter/widgets/FutureBuilder-class.html)

---

## 1. Introduction

In Flutter, the user interface is declared as a tree of widgets. A widget describes what the view should look like given its current configuration and state.

Every component in Flutter falls into one of two primary categories:
- **`StatelessWidget`**: Depends exclusively on its constructor parameters and ambient `InheritedWidget`s. Does not hold mutable state over time.
- **`StatefulWidget`**: Retains mutable state that can change over time in response to user gestures, network responses, or timers.

---

## 2. Core Concepts

### Why is `StatefulWidget` Split into Two Classes?

```mermaid
flowchart TD
    subgraph Ephemeral["Widget Tree (Destroyed & Recreated on Rebuild)"]
        W1["MyStatefulWidget<br/>(Immutable Configuration Blueprint)"]
    end

    subgraph Persistent["Element Tree & Heap (Survives Rebuilds)"]
        E1["StatefulElement"]
        S1["State<MyStatefulWidget><br/>(Retains mutable variables in memory)"]
        E1 --- S1
    end

    W1 -.->|createState() called once| E1
    W1 -.->|Replaced with new instance on rebuild| E1
```

A common question asked by junior developers is: *"Why isn't state stored directly inside the widget?"*

The answer lies in Flutter's performance architecture:
1. **Widgets are immutable and disposable:** Flutter destroys and reallocates widget instances constantly.
2. If state were stored directly inside the widget, every parent rebuild would erase your mutable state (e.g. form inputs or animation positions).
3. By splitting the component into an immutable **`StatefulWidget`** (the blueprint) and a persistent **`State<T>`** object (managed by the `StatefulElement`), the widget instance can be discarded while the state object remains safely stored in memory!

---

### The `setState()` Contract
`setState(VoidCallback fn)` notifies the framework that the internal state of this object has changed.
- The callback `fn` executes immediately and synchronously.
- The framework marks the `StatefulElement` as **dirty**.
- The framework schedules a `build()` pass for the next VSync frame.

**The Golden Rules of `setState()`:**
1. **Never call `setState()` inside `build()`:** Doing so causes an infinite rebuild loop (`Build scheduled during frame`).
2. **Never execute asynchronous work inside `setState()`:**
   ```dart
   // WRONG (Illegal):
   setState(() async { await fetchData(); });

   // CORRECT:
   final data = await fetchData();
   if (!mounted) return; // Guard against disposed widget!
   setState(() { _data = data; });
   ```
3. **Always check `mounted` before calling `setState()`** after an `await` point.

---

## 3. Syntax and Implementation

```dart
// lib/widget_types_demo.dart

import 'package:flutter/material.dart';

void main() => runApp(const MaterialApp(home: WidgetTypesDemoScreen()));

// ==========================================
// 1. STATELESS WIDGET (Immutable Presentation)
// ==========================================
class UserCardStateless extends StatelessWidget {
  final String username;
  final String role;
  final VoidCallback onTap;

  // All fields MUST be final in a StatelessWidget!
  const UserCardStateless({
    super.key,
    required this.username,
    required this.role,
    required this.onTap,
  });

  @override
  Widget build(BuildContext context) {
    return Card(
      child: ListTile(
        leading: CircleAvatar(child: Text(username[0])),
        title: Text(username),
        subtitle: Text(role),
        trailing: const Icon(Icons.chevron_right),
        onTap: onTap,
      ),
    );
  }
}

// ==========================================
// 2. STATEFUL WIDGET (Interactive State)
// ==========================================
class LikeButtonStateful extends StatefulWidget {
  final int initialLikes;
  const LikeButtonStateful({super.key, this.initialLikes = 0});

  @override
  State<LikeButtonStateful> createState() => _LikeButtonStatefulState();
}

class _LikeButtonStatefulState extends State<LikeButtonStateful> {
  late int _likeCount;
  bool _isLiked = false;

  @override
  void initState() {
    super.initState();
    // Initialize mutable state from widget configuration
    _likeCount = widget.initialLikes;
  }

  void _toggleLike() {
    setState(() {
      _isLiked = !_isLiked;
      _likeCount += _isLiked ? 1 : -1;
    });
  }

  @override
  Widget build(BuildContext context) {
    return IconButton(
      icon: Icon(_isLiked ? Icons.favorite : Icons.favorite_border),
      color: _isLiked ? Colors.red : Colors.grey,
      onPressed: _toggleLike,
      tooltip: '$_likeCount Likes',
    );
  }
}

// ==========================================
// 3. ASYNC BUILDER (FutureBuilder Pattern)
// ==========================================
class AsyncDataScreen extends StatefulWidget {
  const AsyncDataScreen({super.key});

  @override
  State<AsyncDataScreen> createState() => _AsyncDataScreenState();
}

class _AsyncDataScreenState extends State<AsyncDataScreen> {
  late Future<List<String>> _profileFuture;

  @override
  void initState() {
    super.initState();
    // CRITICAL: Call the future in initState, NOT inside build()!
    _profileFuture = _loadMockData();
  }

  Future<List<String>> _loadMockData() async {
    await Future.delayed(const Duration(milliseconds: 1200));
    return ['Profile Overview', 'Security Credentials', 'Cloud Storage'];
  }

  @override
  Widget build(BuildContext context) {
    return FutureBuilder<List<String>>(
      future: _profileFuture,
      builder: (context, snapshot) {
        // State 1: Loading
        if (snapshot.connectionState == ConnectionState.waiting) {
          return const Center(child: CircularProgressIndicator());
        }

        // State 2: Error
        if (snapshot.hasError) {
          return Center(
            child: Text('Error loading data: ${snapshot.error}', style: const TextStyle(color: Colors.red)),
          );
        }

        // State 3: Empty Data
        final items = snapshot.data ?? [];
        if (items.isEmpty) {
          return const Center(child: Text('No records found.'));
        }

        // State 4: Success
        return ListView.builder(
          itemCount: items.length,
          itemBuilder: (context, index) => ListTile(
            leading: const Icon(Icons.check_circle_outline),
            title: Text(items[index]),
          ),
        );
      },
    );
  }
}

class WidgetTypesDemoScreen extends StatelessWidget {
  const WidgetTypesDemoScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Widget Types & Async Builders')),
      body: Column(
        children: [
          UserCardStateless(
            username: 'Senior Architect',
            role: 'Lead Flutter Developer',
            onTap: () {},
          ),
          const LikeButtonStateful(initialLikes: 42),
          const Expanded(child: AsyncDataScreen()),
        ],
      ),
    );
  }
}
```

---

## 4. Comparison: `StatelessWidget` vs `StatefulWidget`

| Dimension | `StatelessWidget` | `StatefulWidget` |
|:---|:---|:---|
| **Mutable State** | **None** | Yes (`State<T>` object) |
| **Element Type** | `StatelessElement` | `StatefulElement` |
| **Rebuild Triggers** | Parent rebuild, InheritedWidget change | Parent rebuild, InheritedWidget, **`setState()`** |
| **Memory Cost** | Extremely low | Low to moderate (Retains state object) |
| **Primary Use Cases** | Text, Icons, Badges, Presentation UI | Forms, animations, timers, tabs |

---

## 5. Interview Preparation

### Frequently Asked Questions

#### Q1: Why should you never instantiate a `Future` directly inside a `FutureBuilder`'s `future:` parameter?
- **Detailed Answer:** If you write:
  ```dart
  FutureBuilder(
    future: apiService.fetchData(), // BAD PRACTICE!
    builder: (context, snapshot) => ...
  )
  ```
  Every time the parent widget rebuilds (due to a screen rotation, keyboard open/close, or parent `setState()`), `build()` runs again, instantiating a **brand new Future** and restarting the HTTP network request from scratch! This causes unnecessary network spam and UI flashing. Always instantiate the future once in `initState()` and assign it to a member variable.

#### Q2: What is the purpose of `mounted` in a `State` object?
- **Answer:** `mounted` is a boolean getter on `State`. It returns `true` after `createState()` associates the state with a `StatefulElement`, and turns `false` once `dispose()` is called and the element is unmounted. Calling `setState()` when `mounted == false` throws an unhandled `FlutterError: setState() called after dispose()`. Always check `if (mounted) setState(...)` after an `await` gap.

---

## 6. References
- [Flutter Official Documentation: Stateful vs. Stateless Widgets](https://docs.flutter.dev/ui/widgets/basics)
- [Flutter API Reference: FutureBuilder](https://api.flutter.dev/flutter/widgets/FutureBuilder-class.html)
