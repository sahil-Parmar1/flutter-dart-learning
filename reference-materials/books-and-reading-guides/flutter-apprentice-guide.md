# Book Study Guide: Flutter Apprentice (Kodeco / Ray Wenderlich)

> **Book:** *Flutter Apprentice: Beginning Flutter with Dart*  
> **Authors:** Michael Katz, Kevin David Moore, Vincent Ngo  
> **Publisher:** Kodeco (formerly Ray Wenderlich)  
> **Role in Curriculum:** Practical, project-centric tutorials covering UI building, networking, and local caching.

---

## 1. Book Summary & Pedagogical Value

*Flutter Apprentice* adopts a project-driven tutorial approach. Rather than dwelling purely on theoretical language mechanics, it guides readers through building functional consumer applications (recipes, groceries, food delivery apps), incrementally introducing navigation, API integration, and database persistence.

---

## 2. Chapter Breakdown & Curriculum Mapping

| Book Section | Practical Projects | Corresponding Repository Module | Modernization Notes for Flutter 3.x |
|:---|:---|:---|:---|
| **Section I: Hello, Flutter!** | Recipe Calculator App | [Module 03: Flutter Fundamentals](../../03-flutter-fundamentals/) | Uses basic `StatelessWidget` and `StatefulWidget` composition. |
| **Section II: Everything's a Widget** | Fooderlich Interactive UI, Scrollables | [Module 04: Layout & Widgets](../../04-layout-and-widgets/) | Emphasizes `ListView`, `GridView`, and custom tab bars. |
| **Section III: Navigating & State** | Declarative Navigation, InheritedWidget, Provider | [Module 06: State Management](../../06-state-management/) | Book uses Navigator 2.0 RouterDelegate; modern apps prefer `go_router` or declarative Riverpod routing. |
| **Section IV: Networking & Storage** | REST API recipes, SQLite, Moor/Drift, SharedPreferences | [Module 07: Networking](../../07-networking-and-backend/) & [Module 08: Storage](../../08-local-storage-and-databases/) | Book refers to "Moor", which has been renamed to **`Drift`**. Use our Module 08 for modern Drift syntax. |
| **Section V: Deployment** | Android APKs, iOS IPAs | [Module 12: Deployment & DevOps](../../12-deployment-and-devops/) | Book predates modern Android Gradle Kotlin DSL (`build.gradle.kts`) and `--dart-define-from-file`. |

---

## 3. Recommended Study Strategy

- Use *Flutter Apprentice* when you want to build rapid UI prototypes and see how widgets assemble visually.
- Refer back to our repository's [Module 10 (Clean Architecture)](../../10-architecture-and-clean-code/) to refactor the book's tightly coupled controller code into enterprise-grade separated layers.
