# Hero Animations and Custom Page Route Transitions

> **Module:** 05-animations-and-motion  
> **Subsection:** 03-hero-and-page-transitions  
> **Prerequisites:** [05-animations-and-motion/02-explicit-animations/explicit-animations-and-controllers.md](../02-explicit-animations/explicit-animations-and-controllers.md)  
> **Official References:** [docs.flutter.dev/ui/widgets/animation#hero-animations](https://docs.flutter.dev/ui/widgets/animation#hero-animations) | [api.flutter.dev/flutter/widgets/Hero-class.html](https://api.flutter.dev/flutter/widgets/Hero-class.html)

---

## 1. Introduction

In modern mobile design, navigating between views should feel continuous rather than jarring. If a user taps an avatar or product thumbnail in a list, that element should smoothly fly across the screen, expanding into the hero banner of the detail view.

Flutter provides this capability out-of-the-box via the **`Hero`** widget and allows deep customization of page transitions using **`PageRouteBuilder`**.

---

## 2. Core Concepts: How `Hero` Works Under the Hood

When a route transition begins, Flutter notices two `Hero` widgets on different routes sharing the exact same `tag`:

```mermaid
sequenceDiagram
    autonumber
    participant Source as Source Screen (Hero tag: 'item-42')
    participant Navigator as Navigator Overlay
    participant Dest as Destination Screen (Hero tag: 'item-42')

    Source->>Navigator: Navigator.push(context, MaterialPageRoute(...))
    Note over Navigator: Framework matches Hero tags across both routes
    Navigator->>Navigator: Creates temporary Hero overlay on top of both screens
    Note over Navigator: Animate position & dimensions along createRectTween curve
    loop 300ms Route Animation
        Navigator->>Navigator: Interpolate bounding rect between Source and Dest
    end
    Navigator->>Dest: Flight completes; overlay removed; Dest Hero rendered in place
```

1. **Tag Matching:** The framework verifies that both widgets share an identical `tag` object.
2. **Overlay Flight:** The framework temporarily moves the widget representation to the application's top-level `Overlay`.
3. **`createRectTween`:** By default, Flutter animates in an arc (`MaterialRectArcTween`), creating natural curved flight paths rather than stiff linear diagonal movement.

---

## 3. Syntax and Implementation

### 1. Hero Shared-Element Transition with `flightShuttleBuilder`

```dart
import 'package:flutter/material.dart';

// --- Screen 1: Master List ---
class ProductListScreen extends StatelessWidget {
  const ProductListScreen({super.key});

  @override
  Widget build(BuildContext context) {
    const String heroTag = 'hero-product-jacket-101';

    return Scaffold(
      appBar: AppBar(title: const Text('Store Catalog')),
      body: Center(
        child: GestureDetector(
          onTap: () {
            Navigator.of(context).push(
              PageRouteBuilder(
                transitionDuration: const Duration(milliseconds: 600),
                pageBuilder: (context, animation, secondaryAnimation) =>
                    const ProductDetailScreen(heroTag: heroTag),
                transitionsBuilder: (context, animation, secondaryAnimation, child) {
                  return FadeTransition(opacity: animation, child: child);
                },
              ),
            );
          },
          child: Hero(
            tag: heroTag,
            // Custom appearance during transit
            flightShuttleBuilder: (flightContext, animation, flightDirection, fromHeroContext, toHeroContext) {
              return Material(
                color: Colors.transparent,
                child: RotationTransition(
                  turns: animation,
                  child: fromHeroContext.widget,
                ),
              );
            },
            child: ClipRRect(
              borderRadius: BorderRadius.circular(16),
              child: Image.network(
                'https://picsum.photos/400/400',
                width: 120,
                height: 120,
                fit: BoxFit.cover,
              ),
            ),
          ),
        ),
      ),
    );
  }
}

// --- Screen 2: Detail Screen ---
class ProductDetailScreen extends StatelessWidget {
  final String heroTag;

  const ProductDetailScreen({super.key, required this.heroTag});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Item Details')),
      body: Column(
        children: [
          Hero(
            tag: heroTag,
            child: Image.network(
              'https://picsum.photos/400/400',
              width: double.infinity,
              height: 340,
              fit: BoxFit.cover,
            ),
          ),
          const Padding(
            padding: EdgeInsets.all(20.0),
            child: Text(
              'All-Weather Technical Parka',
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
          ),
        ],
      ),
    );
  }
}
```

---

### 2. Custom `PageRouteBuilder` Library

Standard platform transitions (slide left on iOS, fade up on Android) can be overridden with custom transitions:

```dart
class AppTransitions {
  /// Smooth Bottom Sheet slide up with ease-out cubic curve
  static Route<T> slideUpRoute<T>(Widget page) {
    return PageRouteBuilder<T>(
      pageBuilder: (context, animation, secondaryAnimation) => page,
      transitionsBuilder: (context, animation, secondaryAnimation, child) {
        const begin = Offset(0.0, 1.0); // Start off-screen at bottom
        const end = Offset.zero;         // Center
        const curve = Curves.easeOutCubic;

        final tween = Tween(begin: begin, end: end).chain(CurveTween(curve: curve));
        final offsetAnimation = animation.drive(tween);

        return SlideTransition(position: offsetAnimation, child: child);
      },
    );
  }

  /// Scale + Fade Zoom Transition
  static Route<T> zoomRoute<T>(Widget page) {
    return PageRouteBuilder<T>(
      pageBuilder: (context, animation, secondaryAnimation) => page,
      transitionsBuilder: (context, animation, secondaryAnimation, child) {
        return ScaleTransition(
          scale: Tween<double>(begin: 0.85, end: 1.0).animate(
            CurvedAnimation(parent: animation, curve: Curves.fastOutSlowIn),
          ),
          child: FadeTransition(opacity: animation, child: child),
        );
      },
    );
  }
}
```

---

## 4. Common Pitfalls & Anti-Patterns

### Pitfall 1: Duplicate Hero Tags on the Same Screen
```dart
// ❌ CRASH: "There are multiple heroes that share the same tag within a subtree."
ListView.builder(
  itemBuilder: (context, index) => Hero(
    tag: 'item-card', // All items share the same static tag!
    child: ...
  ),
)

// ✅ CORRECT: Ensure tags are uniquely identified by entity ID
Hero(
  tag: 'item-card-${item.id}',
  child: ...
)
```

---

## 5. Interview & Assessment Takeaways

| Interview Question | Concise Verbal Answer |
|:---|:---|
| **Where does a `Hero` widget render during transit between two routes?** | It renders inside the application's root `Overlay` widget, above both the exiting route and the entering route, interpolating its bounding rect between the two endpoints. |
| **Why does text inside a `Hero` sometimes appear with a yellow double underline during transition?** | In transit, the Hero is removed from the route's `Scaffold`/`Material` ancestor into the bare `Overlay`. Without a `DefaultTextStyle` or `Material` widget ancestor, Flutter defaults text styling to standard un-themed text (yellow double underline). Fix by wrapping the Hero's child in a `Material(type: MaterialType.transparency, child: ...)`. |
| **What is `flightShuttleBuilder`?** | A callback on `Hero` that allows you to customize the widget displayed *during* the transition flight (e.g. adding rotation or replacing a thumbnail with an animated silhouette). |
