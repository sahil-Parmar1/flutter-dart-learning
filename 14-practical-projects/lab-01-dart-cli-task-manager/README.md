# Lab 01: Asynchronous CLI Task Manager in Pure Dart 3

> **Module:** 14-practical-projects  
> **Lab Number:** 01  
> **Prerequisites:** [01-dart-fundamentals/](../../01-dart-fundamentals/) | [02-advanced-dart/](../../02-advanced-dart/)  
> **Estimated Completion Time:** 2 Hours

---

## 1. Project Requirements

Build a robust, standalone Dart command-line application that manages tasks without any Flutter UI dependencies.

### Functional Requirements:
1. **Interactive Command Loop:** Read standard user input (`stdin`) continuously until the user types `exit`.
2. **Task Schema:** Store task ID, title, priority (`low`, `medium`, `urgent`), completed boolean, and ISO-8601 creation timestamp.
3. **Dart 3 Records & Pattern Matching:** Return multiple values from methods using Dart 3 records `(bool success, String message)` and use switch expressions for priority formatting.
4. **Local Disk Persistence:** Automatically persist and load tasks from a local `tasks.json` file using `dart:io`.
5. **Background Isolate Workload:** Offload task search and analytics calculation onto a background isolate using `Isolate.run()` to prevent blocking the CLI event loop.

---

## 2. Architecture & Folder Structure

```
lab-01-dart-cli-task-manager/
├── bin/
│   └── main.dart                  # CLI entrypoint & interactive REPL loop
├── lib/
│   ├── models/
│   │   ├── priority.dart          # Enum with pattern matching extensions
│   │   └── task.dart              # Immutable Task class with copyWith
│   ├── repositories/
│   │   └── task_repository.dart   # File I/O operations with dart:io
│   └── services/
│       └── task_analytics.dart    # Background Isolate computation
├── test/
│   └── task_test.dart             # Pure Dart unit test suite
└── pubspec.yaml
```

---

## 3. Implementation Steps & Code

### Step 1: Model & Enum Definition (`lib/models/priority.dart` & `task.dart`)

```dart
enum Priority {
  low,
  medium,
  urgent;

  // Dart 3 pattern matching switch expression
  String get badge => switch (this) {
        Priority.low => '[🟢 LOW]',
        Priority.medium => '[🟡 MEDIUM]',
        Priority.urgent => '[🔴 URGENT]',
      };

  static Priority fromString(String val) => switch (val.toLowerCase().trim()) {
        'urgent' || '3' => Priority.urgent,
        'medium' || '2' => Priority.medium,
        _ => Priority.low,
      };
}

class Task {
  final String id;
  final String title;
  final Priority priority;
  final bool isCompleted;
  final DateTime createdAt;

  const Task({
    required this.id,
    required this.title,
    required this.priority,
    this.isCompleted = false,
    required this.createdAt,
  });

  Task copyWith({bool? isCompleted, Priority? priority}) {
    return Task(
      id: id,
      title: title,
      priority: priority ?? this.priority,
      isCompleted: isCompleted ?? this.isCompleted,
      createdAt: createdAt,
    );
  }

  Map<String, dynamic> toJson() => {
        'id': id,
        'title': title,
        'priority': priority.name,
        'isCompleted': isCompleted,
        'createdAt': createdAt.toIso8601String(),
      };

  factory Task.fromJson(Map<String, dynamic> json) => Task(
        id: json['id'] as String,
        title: json['title'] as String,
        priority: Priority.fromString(json['priority'] as String),
        isCompleted: json['isCompleted'] as bool? ?? false,
        createdAt: DateTime.parse(json['createdAt'] as String),
      );
}
```

---

### Step 2: Persistence Layer (`lib/repositories/task_repository.dart`)

```dart
import 'dart:convert';
import 'dart:io';
import '../models/task.dart';

class TaskRepository {
  final File _file;

  TaskRepository([String filePath = 'tasks.json']) : _file = File(filePath);

  Future<List<Task>> loadTasks() async {
    if (!await _file.exists()) return [];
    try {
      final rawContent = await _file.readAsString();
      if (rawContent.trim().isEmpty) return [];
      final List<dynamic> jsonList = jsonDecode(rawContent) as List<dynamic>;
      return jsonList.map((e) => Task.fromJson(e as Map<String, dynamic>)).toList();
    } catch (e) {
      stderr.writeln('Warning: Failed to parse tasks.json ($e)');
      return [];
    }
  }

  Future<void> saveTasks(List<Task> tasks) async {
    final jsonString = jsonEncode(tasks.map((t) => t.toJson()).toList());
    await _file.writeAsString(jsonString);
  }
}
```

---

### Step 3: Background Isolate Analytics (`lib/services/task_analytics.dart`)

```dart
import 'dart:isolate';
import '../models/task.dart';

class TaskAnalytics {
  /// Offloads heavy statistics calculation onto a background isolate
  static Future<(int total, int completed, int urgentPending)> computeStats(
      List<Task> tasks) async {
    return await Isolate.run(() {
      int completed = 0;
      int urgentPending = 0;

      for (final task in tasks) {
        if (task.isCompleted) {
          completed++;
        } else if (task.priority.name == 'urgent') {
          urgentPending++;
        }
      }

      // Return Dart 3 Record
      return (tasks.length, completed, urgentPending);
    });
  }
}
```

---

### Step 4: Interactive CLI REPL Loop (`bin/main.dart`)

```dart
import 'dart:io';
import 'package:cli_task_manager/models/priority.dart';
import 'package:cli_task_manager/models/task.dart';
import 'package:cli_task_manager/repositories/task_repository.dart';
import 'package:cli_task_manager/services/task_analytics.dart';

void main() async {
  final repo = TaskRepository();
  final List<Task> tasks = await repo.loadTasks();

  stdout.writeln('====================================');
  stdout.writeln('  ⚡ Dart 3 Enterprise Task Manager');
  stdout.writeln('  Loaded ${tasks.length} tasks from disk.');
  stdout.writeln('====================================');
  stdout.writeln('Commands: list | add <title> | done <id> | stats | exit\n');

  while (true) {
    stdout.write('> ');
    final input = stdin.readLineSync()?.trim();
    if (input == null || input == 'exit') {
      stdout.writeln('Saving tasks and exiting. Goodbye!');
      await repo.saveTasks(tasks);
      break;
    }

    final parts = input.split(' ');
    final command = parts.first.toLowerCase();

    switch (command) {
      case 'list':
        if (tasks.isEmpty) {
          stdout.writeln('No tasks recorded yet.');
          break;
        }
        for (var i = 0; i < tasks.length; i++) {
          final t = tasks[i];
          final status = t.isCompleted ? '[✓]' : '[ ]';
          stdout.writeln('$status #${i + 1} ${t.priority.badge} ${t.title}');
        }
        break;

      case 'add':
        if (parts.length < 2) {
          stdout.writeln('Error: Title required. Usage: add <task title>');
          break;
        }
        final title = parts.sublist(1).join(' ');
        stdout.write('Select Priority (1: Low, 2: Med, 3: Urgent): ');
        final prioInput = stdin.readLineSync() ?? '1';
        final priority = Priority.fromString(prioInput);

        final newTask = Task(
          id: DateTime.now().millisecondsSinceEpoch.toString(),
          title: title,
          priority: priority,
          createdAt: DateTime.now(),
        );

        tasks.add(newTask);
        await repo.saveTasks(tasks);
        stdout.writeln('✅ Task added successfully!');
        break;

      case 'done':
        if (parts.length < 2) {
          stdout.writeln('Usage: done <task number>');
          break;
        }
        final index = int.tryParse(parts[1]);
        if (index == null || index < 1 || index > tasks.length) {
          stdout.writeln('Invalid task number.');
          break;
        }
        tasks[index - 1] = tasks[index - 1].copyWith(isCompleted: true);
        await repo.saveTasks(tasks);
        stdout.writeln('🎯 Task #$index marked as completed!');
        break;

      case 'stats':
        final (total, completed, urgent) = await TaskAnalytics.computeStats(tasks);
        stdout.writeln('--- Analytics Overview ---');
        stdout.writeln('Total Tasks: $total');
        stdout.writeln('Completed:   $completed');
        stdout.writeln('Urgent Open: $urgent');
        break;

      default:
        stdout.writeln('Unknown command. Available: list, add, done, stats, exit');
    }
  }
}
```

---

## 4. Automated Testing

File: `test/task_test.dart`

```dart
import 'package:test/test.dart';
import 'package:cli_task_manager/models/priority.dart';
import 'package:cli_task_manager/models/task.dart';

void main() {
  group('Task Model & Pattern Matching Tests', () {
    test('Priority.fromString maps numbers and names correctly', () {
      expect(Priority.fromString('urgent'), Priority.urgent);
      expect(Priority.fromString('3'), Priority.urgent);
      expect(Priority.fromString('medium'), Priority.medium);
      expect(Priority.fromString('unknown'), Priority.low);
    });

    test('Task copyWith preserves immutability while updating status', () {
      final task = Task(
        id: '101',
        title: 'Review PR',
        priority: Priority.urgent,
        createdAt: DateTime.now(),
      );

      final updated = task.copyWith(isCompleted: true);

      expect(task.isCompleted, isFalse);
      expect(updated.isCompleted, isTrue);
      expect(updated.title, equals('Review PR'));
    });
  });
}
```

---

## 5. Possible Future Improvements

1. **Args Parsing:** Use `package:args` for standard POSIX flag parsing (`task_cli add --priority=urgent "Refactor DB"`).
2. **Terminal Styling:** Integrate `package:tint` or `package:chalkdart` for colored terminal output.
3. **SQLite Backing:** Replace JSON file persistence with `drift` for indexed relational searching.
