# Flutter & Dart Comprehensive Production Learning Repository

[![Flutter](https://img.shields.io/badge/Flutter-3.x+-02569B?logo=flutter&logoColor=white)](https://flutter.dev)
[![Dart](https://img.shields.io/badge/Dart-3.x+-0175C2?logo=dart&logoColor=white)](https://dart.dev)
[![Engine](https://img.shields.io/badge/Rendering-Impeller%20%2F%20Skia-FF6F00)](https://flutter.dev)
[![Certification](https://img.shields.io/badge/Certification-Android%20ATC%20AFD--200-success)](https://androidatc.com)
[![Status](https://img.shields.io/badge/Status-100%25%20Complete-brightgreen)]()
[![Content Files](https://img.shields.io/badge/Guides-120%20Files-blue)]()

A structured, industry-aligned, production-accurate learning repository and reference handbook for mastering **Dart 3** and **Flutter 3.x+**.

Engineered by transforming a personal archive of 183 learning documents, reference books, and cheat sheets into an authoritative, GitHub-ready engineering knowledge base.

---

## 📌 Repository Core Navigation

| Document | Purpose | Description |
|:---|:---|:---|
| 🗺️ **[ROADMAP.md](./ROADMAP.md)** | **5-Stage Mastery Roadmap** | Progressive learning trajectory with milestones, timelines, and verification projects. |
| 📊 **[PROGRESS.md](./PROGRESS.md)** | **Documentation Progress Tracker** | Live tracking matrix of all 14 modules, practical labs, and reference guides. |
| 🏗️ **[DOCUMENTATION_STRUCTURE.md](./DOCUMENTATION_STRUCTURE.md)** | **Repository Architecture** | Complete file tree, module layout, and file-level design specifications. |
| 🔍 **[AUDIT.md](./AUDIT.md)** | **Source Material Audit** | Exhaustive technical audit of all 183 source files from `G:\My Drive\flutter_learn`. |
| 🤝 **[CONTRIBUTING.md](./CONTRIBUTING.md)** | **Contribution & Style Guide** | Standardized markdown template, code style rules, and contribution workflow. |

---

## 📚 Curriculum & Module Directory

The repository is organized into 14 progressive technical modules, followed by hands-on labs and a textbook reference library:

### Stage 1: Modern Dart 3 Mastery
- **[Module 01: Dart Fundamentals](./01-dart-fundamentals/README.md)** — Variables, sound null safety, immutability (`final` vs `const`), collections, functions, closures, OOP, and Dart 3 class modifiers (`sealed`, `base`, `interface`, `final`).
- **[Module 02: Advanced Dart](./02-advanced-dart/README.md)** — Event loop mechanics, microtasks, Futures, Streams, Generators (`yield`, `yield*`), Isolates (`compute`, `Isolate.run`), Dart 3 Patterns, Records, and Wildcard Variables (`_`).

### Stage 2: Flutter Core & UI Engineering
- **[Module 03: Flutter Fundamentals](./03-flutter-fundamentals/README.md)** — Framework layers, compilation pipeline (JIT vs AOT), Three Trees (Widget, Element, RenderObject), `State` lifecycle chronology, `BuildContext`, and Keys.
- **[Module 04: Layout and Widgets](./04-layout-and-widgets/README.md)** — Box constraints rules (*"Constraints go down, sizes go up, parent sets position"*), Flex (`Row`, `Column`, `Expanded`, `Flexible`), `Stack`, `CustomScrollView`, Slivers, and modern back navigation (`PopScope`).
- **[Module 05: Animations and Motion](./05-animations-and-motion/README.md)** — Implicit animations (`AnimatedContainer`, `AnimatedSwitcher`), explicit controllers (`AnimationController`, `Tween`), Hero animations, and `CustomPainter` canvas drawing.

### Stage 3: Enterprise State & Data Integration
- **[Module 06: State Management](./06-state-management/README.md)** — In-depth architectural state management: Provider (`ChangeNotifier`, `Consumer`, `Selector`), Modern Riverpod 2/3 (`@riverpod` code generation, `Notifier`, `AsyncNotifier`), BLoC / Cubit, and reactive Signals.
- **[Module 07: Networking and Backend](./07-networking-and-backend/README.md)** — REST APIs (`package:http`, `dio`), interceptors, 401 token refresh queue, error mapping, `json_serializable`, `@freezed` immutable union models, WebSockets, and Cloud Firestore.
- **[Module 08: Local Storage and Databases](./08-local-storage-and-databases/README.md)** — `shared_preferences`, `flutter_secure_storage`, NoSQL databases (Hive, Hive-CE, Isar), and type-safe relational SQL (`drift`, `sqflite`).

### Stage 4: Native Systems, Performance & Clean Architecture
- **[Module 09: Platform Integration](./09-platform-integration/README.md)** — Platform Channels (`MethodChannel`, `EventChannel`), type-safe interop via Pigeon, Android Gradle Kotlin DSL / SHA fingerprints, and iOS Xcode setups.
- **[Module 10: Architecture and Clean Code](./10-architecture-and-clean-code/README.md)** — Clean Architecture (Domain, Data, Presentation), Repository Pattern, Dependency Injection (`get_it`, Riverpod DI), and SOLID principles in Flutter.
- **[Module 11: Testing and Performance](./11-testing-and-performance/README.md)** — Unit testing (`test`), Widget testing (`WidgetTester`), Integration testing, Flutter DevTools, Impeller vs Skia graphics rendering, and memory leak prevention.

### Stage 5: Release Engineering, Certification & Senior Interviews
- **[Module 12: Deployment and DevOps](./12-deployment-and-devops/README.md)** — Multi-environment flavoring (`dev`, `staging`, `prod`), Android Keystore & App Bundles (AAB), ProGuard/R8 obfuscation, and iOS TestFlight / App Store deployment.
- **[Module 13: Interview and Certification](./13-interview-and-certification/README.md)** — Complete guide to Android ATC AFD-200 certification with step-by-step solutions to 45 official exam questions, junior-to-senior technical Q&A, and mobile system design scenarios.
- **[Module 14: Practical Projects (Labs)](./14-practical-projects/README.md)** — 4 progressive hands-on projects:
  - **[Lab 01: Asynchronous CLI Task Manager](./14-practical-projects/lab-01-dart-cli-task-manager/README.md)** (Pure Dart 3, Records, Async Streams, Isolates, JSON I/O)
  - **[Lab 02: Responsive Analytics Dashboard](./14-practical-projects/lab-02-responsive-dashboard/README.md)** (Adaptive Shell, CustomScrollView, Slivers, CustomPainter Area Charts)
  - **[Lab 03: Offline-First Notes App](./14-practical-projects/lab-03-offline-first-notes-app/README.md)** (Riverpod 2/3 Notifiers, Drift SQLite, Optimistic UI, Clean Architecture)
  - **[Lab 04: Full-Stack E-Commerce Storefront](./14-practical-projects/lab-04-fullstack-ecommerce-app/README.md)** (Dio Interceptors, Freezed Union DTOs, Firebase Auth/Firestore, Automated Tests)
- **[Reference Materials](./reference-materials/README.md)** — Systematic reading guides and cross-references for major Flutter textbooks (*Flutter Apprentice*, *Flutter In Action*, *Flutter Complete Reference*).

---

## 🚀 How to Use This Repository

### 🎯 Track A: The Structured Career Learner
1. Start at **[ROADMAP.md](./ROADMAP.md)**.
2. Progress through Stages 1 through 5 sequentially.
3. Complete each verification lab before proceeding to the next stage.
4. Prepare for certification using Module 13.

### ⚡ Track B: Production Engineer & Quick Lookup
1. Jump directly to the module of interest (e.g. `04-layout-and-widgets` for custom slivers, or `06-state-management` for Riverpod code-gen).
2. Each documentation file follows a standardized format with runnable code examples, Mermaid diagrams, and "Common Pitfalls (Watch Out!)" callouts.

### 💼 Track C: Interview Candidate & Certification Candidate
1. Focus on **[Module 13: Interview & Certification](./13-interview-and-certification/README.md)**.
2. Solve the 45 official AFD-200 certification sample questions.
3. Review mobile system design case studies (offline-first sync, large list caching).

---

## 🛠️ Recommended Development Environment

- **Dart SDK:** `3.3.0` or higher (Tested on `3.12.2`)
- **Flutter SDK:** `3.19.0` or higher (Tested on `3.44.9` with Impeller)
- **Primary IDE:** Visual Studio Code (with official Flutter & Dart extensions) or Android Studio
- **Operating Systems:** Windows / macOS / Linux
- **Mobile Toolchains:**
  - Android: Android Studio SDK, JDK 17, Android Build Tools
  - iOS (macOS only): Xcode 15+, CocoaPods 1.14+

---

## 🛡️ Source Material & Modernization Notice

This repository synthesizes materials originally stored in `G:\My Drive\flutter_learn`. During the technical audit (detailed in **[AUDIT.md](./AUDIT.md)**), several outdated or deprecated APIs were identified:
- **`WillPopScope`** $\to$ Replaced with modern Flutter 3.12+ **`PopScope`**.
- **Skia-only graphics engine** $\to$ Modernized with Google's next-generation **Impeller** rendering engine.
- **Legacy Riverpod (`StateNotifierProvider`)** $\to$ Modernized with Riverpod 2/3 **`@riverpod` code generation and `Notifier` / `AsyncNotifier`**.
- **Verbose constructors** $\to$ Modernized with Dart 3 **super-parameters (`super.key`)**.

All legacy patterns are retained in designated "Legacy & Migration" sections for backward compatibility and real-world legacy codebase maintenance.

---

## 📈 Project Status

- [x] **Phase 1: Research, Audit & Repository Architecture** (Completed)
- [x] **Phase 2: Progressive Documentation Authoring (Modules 01 & 02)** (Completed)
- [x] **Phase 3: Advanced Flutter, State Management, Labs & Certification (Modules 03–14)** (Completed)
- [x] **Phase 4: Technical Audit, Link Verification & GitHub Readiness** (Completed)
