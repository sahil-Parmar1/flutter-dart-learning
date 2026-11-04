# Module 05: Animations and Motion

> **Stage:** Stage 2 (Continued) – Layout System, UI Engineering & Motion  
> **Target Audience:** Intermediate to Advanced Flutter Developers  
> **Prerequisites:** [03-flutter-fundamentals/](../03-flutter-fundamentals/) | [04-layout-and-widgets/](../04-layout-and-widgets/)

---

## Overview

Animations in Flutter are first-class architectural citizens. Unlike web environments that rely on CSS keyframes or external Javascript libraries that cross browser threads, Flutter's rendering pipeline computes interpolated values on every frame tick directly within the 60fps / 120fps frame budget (16.6ms or 8.3ms per frame).

This module demystifies Flutter's animation hierarchy, starting from high-level declaratively managed **Implicit Animations**, progressing down to low-level imperative **Explicit Animations** with `AnimationController` and `TickerProvider`, navigating cross-screen **Hero Transitions**, and culminating in custom GPU canvas rendering via **`CustomPainter`**.

---

## Learning Objectives

1. Master the distinction between **Implicit** (declarative, automatic state interpolation) and **Explicit** (imperative, manual lifecycle control) animations.
2. Understand the mechanics of `Ticker`, `AnimationController`, `CurvedAnimation`, and `Tween`.
3. Eliminate unnecessary widget rebuilds during animations using `AnimatedBuilder` and scoped repaint boundaries.
4. Construct complex multi-stage choreographed **Staggered Animations** utilizing `Interval`.
5. Implement smooth shared-element **Hero transitions** across navigation boundaries.
6. Design low-level custom graphics, circular progress rings, and waveforms using `Canvas`, `Paint`, and `Path` with `CustomPainter`.

---

## Directory Roadmap

```
05-animations-and-motion/
├── README.md
├── 01-implicit-animations/
│   └── implicit-animations-deep-dive.md     # AnimatedContainer, AnimatedOpacity, AnimatedSwitcher
├── 02-explicit-animations/
│   └── explicit-animations-and-controllers.md # AnimationController, Ticker, Tween, AnimatedBuilder, Staggered
├── 03-hero-and-page-transitions/
│   └── hero-and-custom-page-transitions.md  # Hero tags, flightShuttleBuilder, PageRouteBuilder
└── 04-custom-painters-and-canvas/
    └── custom-painters-and-canvas.md        # CustomPainter, Canvas, Paint, Path, RepaintBoundary
```
