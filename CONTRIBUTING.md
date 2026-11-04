# Contributing to the Flutter & Dart Learning Repository

Thank you for your interest in improving and maintaining this Flutter & Dart learning resource! This repository is designed to be a technically rigorous, modern, and practical reference for software engineers preparing for production development and senior technical interviews.

---

## 1. Core Principles & Philosophy

When contributing documentation, code examples, or architectural guides, please adhere to these non-negotiable standards:

1. **Modern Standard Baselines:**
   - All Dart code must be compatible with **Dart 3.x+** (utilizing sound null safety, class modifiers, records, and pattern matching).
   - All Flutter code must follow **Flutter 3.x+** conventions (using `PopScope` instead of deprecated `WillPopScope`, Material 3 design, and Impeller rendering principles).
   - State management examples must use modern recommended syntax (e.g., Riverpod `Notifier` and `AsyncNotifier` rather than legacy `StateNotifierProvider`).
2. **Technical Accuracy Over Brevity:**
   - Never invent APIs or parameters. Verify method names and signatures against [api.flutter.dev](https://api.flutter.dev) and [api.dart.dev](https://api.dart.dev).
   - Clearly differentiate between what runs on the UI thread and what runs on background isolates.
3. **No Code Without Context:**
   - Code snippets should be complete, syntactically correct, and include necessary imports (`package:flutter/material.dart`, etc.).
   - Explicitly document prerequisites when code requires third-party packages or native platform permissions.
4. **Originality & Copyright Compliance:**
   - Do not copy copyrighted text, book chapters, or paid tutorial materials verbatim.
   - Explain concepts in your own words with original code examples and diagrams.

---

## 2. Standard Document Anatomy

Every topic guide in this repository adheres to this standardized markdown structure:

```markdown
# [Topic Title]

> **Module:** [Module Name]  
> **Subsection:** [Subsection Folder]  
> **Prerequisites:** [Links to prerequisite modules]  
> **Official References:** [Links to dart.dev / docs.flutter.dev / pub.dev]

---

## 1. Introduction
- What is this concept?
- Why do we need it?
- Where is it used in production?

## 2. Theoretical Mechanics & Internal Working
- How does the Dart VM or Flutter engine execute this?
- Mermaid architecture, sequence, or lifecycle diagram.

## 3. Syntax and Implementation
- Minimal clear snippet.
- Full real-world production example with error handling and null safety.

## 4. Common Pitfalls & Anti-Patterns
- What typical mistakes do developers make?
- Bad practice vs. Good practice comparison.

## 5. Performance, Memory & Edge Cases
- Big-O complexity or 16.6ms frame budget implications.
- Memory leak and disposal considerations.

## 6. Interview & Assessment Takeaways
- High-frequency interview questions with concise verbal answers.
```

---

## 3. Diagram Guidelines

- Use **Mermaid.js** code blocks (triple backticks with language `mermaid`) for architectures, workflows, and state diagrams.
- Always quote node labels containing special characters: `Node["Label (Details)"]`.
- Prefer `flowchart TD` or `flowchart LR` for architectural diagrams, and `sequenceDiagram` for lifecycle and event flows.

---

## 4. Contribution Workflow

1. **Fork & Branch:** Create a feature branch with a descriptive name:
   ```bash
   git checkout -b fix/riverpod-async-notifier-guide
   ```
2. **Verify Code:**
   - Ensure Dart examples pass static analysis: `dart analyze`.
   - Ensure markdown links are valid and not broken.
3. **Commit Convention:** Follow conventional commits:
   - `docs: update PopScope migration guide in Module 04`
   - `fix: correct typeId allocation in Hive documentation`
   - `feat: add integration test suite to Lab 04`
4. **Submit Pull Request:** Provide a clear description of the technical updates and references to official documentation.
