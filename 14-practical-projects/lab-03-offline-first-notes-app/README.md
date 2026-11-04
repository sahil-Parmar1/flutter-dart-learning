# Lab 03: Offline-First Notes and Journal App (Riverpod + Drift)

> **Module:** 14-practical-projects  
> **Lab Number:** 03  
> **Prerequisites:** [06-state-management/03-riverpod-modern/](../../06-state-management/03-riverpod-modern/) | [08-local-storage-and-databases/03-relational-sqlite-and-drift/](../../08-local-storage-and-databases/03-relational-sqlite-and-drift/) | [10-architecture-and-clean-code/](../../10-architecture-and-clean-code/)  
> **Estimated Completion Time:** 4 Hours

---

## 1. Project Requirements

Build a production-grade, offline-first notes and journal application adhering to Clean Architecture principles.

### Functional & Technical Requirements:
1. **Clean Architecture Separation:** Domain (Entities & Use Cases), Data (Drift SQLite DAO & Repository), Presentation (Riverpod AsyncNotifier & UI).
2. **Reactive Local Database:** Use **Drift** SQLite database so that modifications automatically stream to the UI via `Stream<List<Note>>`.
3. **Optimistic UI Updates:** Deletions and additions update the screen instantly without waiting for disk I/O confirmation.
4. **Riverpod 2.x/3.x State Management:** Leverage `AsyncNotifierProvider` and handle `AsyncValue` data, loading, and error states exhaustively.

---

## 2. Architecture & Folder Structure

```
lab-03-offline-first-notes-app/
├── lib/
│   ├── main.dart
│   ├── core/
│   │   └── database/app_database.dart       # Drift SQLite schema & DAO
│   └── features/
│       └── notes/
│           ├── domain/
│           │   ├── entities/note_entity.dart
│           │   └── repositories/notes_repository.dart
│           ├── data/
│           │   └── repositories/notes_repository_impl.dart
│           └── presentation/
│               ├── controllers/notes_controller.dart
│               └── views/notes_list_screen.dart
└── test/
    └── notes_controller_test.dart            # Unit test with mock repository
```

---

## 3. Implementation Steps & Code

### Step 1: Database Schema with Drift (`lib/core/database/app_database.dart`)

```dart
import 'dart:io';
import 'package:drift/drift.dart';
import 'package:drift/native.dart';
import 'package:path_provider/path_provider.dart';
import 'package:path/path.dart' as p;

part 'app_database.g.dart';

class NotesTable extends Table {
  TextColumn get id => text()();
  TextColumn get title => text().withLength(min: 1, max: 120)();
  TextColumn get content => text()();
  DateTimeColumn get updatedAt => dateTime()();

  @override
  Set<Column> get primaryKey => {id};
}

@DriftDatabase(tables: [NotesTable])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());

  @override
  int get schemaVersion => 1;

  Stream<List<NotesTableData>> watchAllNotes() {
    return (select(notesTable)..orderBy([(t) => OrderingTerm.desc(t.updatedAt)])).watch();
  }

  Future<int> insertOrUpdateNote(NotesTableCompanion entry) {
    return into(notesTable).insertOnConflictUpdate(entry);
  }

  Future<int> deleteNoteById(String id) {
    return (delete(notesTable)..where((t) => t.id.equals(id))).go();
  }
}

LazyDatabase _openConnection() {
  return LazyDatabase(() async {
    final dbFolder = await getApplicationDocumentsDirectory();
    final file = File(p.join(dbFolder.path, 'notes.sqlite'));
    return NativeDatabase.createInBackground(file);
  });
}
```

---

### Step 2: Domain Layer (`lib/features/notes/domain/`)

```dart
// 1. Entity
class NoteEntity {
  final String id;
  final String title;
  final String content;
  final DateTime updatedAt;

  const NoteEntity({
    required this.id,
    required this.title,
    required this.content,
    required this.updatedAt,
  });
}

// 2. Repository Contract
abstract class NotesRepository {
  Stream<List<NoteEntity>> watchNotes();
  Future<void> saveNote(NoteEntity note);
  Future<void> deleteNote(String id);
}
```

---

### Step 3: Data Layer (`lib/features/notes/data/`)

```dart
import '../../../../core/database/app_database.dart';
import '../../domain/entities/note_entity.dart';
import '../../domain/repositories/notes_repository.dart';
import 'package:drift/drift.dart';

class NotesRepositoryImpl implements NotesRepository {
  final AppDatabase _db;

  NotesRepositoryImpl(this._db);

  @override
  Stream<List<NoteEntity>> watchNotes() {
    return _db.watchAllNotes().map((rows) => rows
        .map((row) => NoteEntity(
              id: row.id,
              title: row.title,
              content: row.content,
              updatedAt: row.updatedAt,
            ))
        .toList());
  }

  @override
  Future<void> saveNote(NoteEntity note) async {
    await _db.insertOrUpdateNote(
      NotesTableCompanion(
        id: Value(note.id),
        title: Value(note.title),
        content: Value(note.content),
        updatedAt: Value(note.updatedAt),
      ),
    );
  }

  @override
  Future<void> deleteNote(String id) async {
    await _db.deleteNoteById(id);
  }
}
```

---

### Step 4: Presentation Layer with Riverpod (`lib/features/notes/presentation/`)

#### Controller:
```dart
import 'dart:async';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../../domain/entities/note_entity.dart';
import '../../domain/repositories/notes_repository.dart';

// DI Providers
final databaseProvider = Provider<AppDatabase>((ref) => AppDatabase());
final notesRepositoryProvider = Provider<NotesRepository>((ref) {
  return NotesRepositoryImpl(ref.watch(databaseProvider));
});

// Stream Provider listening to database changes
final notesStreamProvider = StreamProvider<List<NoteEntity>>((ref) {
  return ref.watch(notesRepositoryProvider).watchNotes();
});

// Action Notifier
class NotesController extends Notifier<void> {
  @override
  void build() {}

  Future<void> addNote(String title, String content) async {
    final newNote = NoteEntity(
      id: DateTime.now().millisecondsSinceEpoch.toString(),
      title: title,
      content: content,
      updatedAt: DateTime.now(),
    );
    await ref.read(notesRepositoryProvider).saveNote(newNote);
  }

  Future<void> deleteNote(String id) async {
    await ref.read(notesRepositoryProvider).deleteNote(id);
  }
}

final notesControllerProvider = NotifierProvider<NotesController, void>(() {
  return NotesController();
});
```

#### View:
```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../controllers/notes_controller.dart';

class NotesListScreen extends ConsumerWidget {
  const NotesListScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final notesAsync = ref.watch(notesStreamProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('Offline-First Notes')),
      body: notesAsync.when(
        loading: () => const Center(child: CircularProgressIndicator()),
        error: (err, _) => Center(child: Text('Error: $err')),
        data: (notes) {
          if (notes.isEmpty) {
            return const Center(child: Text('No notes recorded yet.'));
          }
          return ListView.builder(
            itemCount: notes.length,
            itemBuilder: (context, index) {
              final note = notes[index];
              return Dismissible(
                key: Key(note.id),
                direction: DismissDirection.endToStart,
                background: Container(
                  color: Colors.red,
                  alignment: Alignment.centerRight,
                  padding: const EdgeInsets.only(right: 20),
                  child: const Icon(Icons.delete, color: Colors.white),
                ),
                onDismissed: (_) {
                  ref.read(notesControllerProvider.notifier).deleteNote(note.id);
                },
                child: ListTile(
                  title: Text(note.title, style: const TextStyle(fontWeight: FontWeight.bold)),
                  subtitle: Text(note.content, maxLines: 2, overflow: TextOverflow.ellipsis),
                  trailing: Text(
                    '${note.updatedAt.hour}:${note.updatedAt.minute.toString().padLeft(2, '0')}',
                    style: const TextStyle(color: Colors.grey, fontSize: 12),
                  ),
                ),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _showAddNoteDialog(context, ref),
        child: const Icon(Icons.add),
      ),
    );
  }

  void _showAddNoteDialog(BuildContext context, WidgetRef ref) {
    final titleCtrl = TextEditingController();
    final contentCtrl = TextEditingController();

    showDialog(
      context: context,
      builder: (ctx) => AlertDialog(
        title: const Text('New Note'),
        content: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            TextField(controller: titleCtrl, decoration: const InputDecoration(labelText: 'Title')),
            TextField(controller: contentCtrl, decoration: const InputDecoration(labelText: 'Content')),
          ],
        ),
        actions: [
          TextButton(onPressed: () => Navigator.pop(ctx), child: const Text('Cancel')),
          FilledButton(
            onPressed: () {
              if (titleCtrl.text.isNotEmpty) {
                ref.read(notesControllerProvider.notifier).addNote(
                      titleCtrl.text,
                      contentCtrl.text,
                    );
                Navigator.pop(ctx);
              }
            },
            child: const Text('Save Note'),
          ),
        ],
      ),
    );
  }
}
```

---

## 4. Automated Testing

File: `test/notes_controller_test.dart`

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:offline_notes_app/features/notes/domain/entities/note_entity.dart';
import 'package:offline_notes_app/features/notes/domain/repositories/notes_repository.dart';
import 'package:offline_notes_app/features/notes/presentation/controllers/notes_controller.dart';

class MockNotesRepository extends Mock implements NotesRepository {}

void main() {
  test('addNote delegates to NotesRepository.saveNote with valid entity', () async {
    final mockRepo = MockNotesRepository();
    when(() => mockRepo.saveNote(any())).thenAnswer((_) async {});

    final container = ProviderContainer(
      overrides: [
        notesRepositoryProvider.overrideWithValue(mockRepo),
      ],
    );

    final controller = container.read(notesControllerProvider.notifier);
    await controller.addNote('Meeting Agenda', 'Discuss quarterly roadmap.');

    verify(() => mockRepo.saveNote(any(that: isA<NoteEntity>()))).called(1);
  });
}
```

---

## 5. Possible Future Improvements

1. **Cloud Sync Engine:** Add a background worker that replicates SQLite changes to Google Cloud Firestore or Supabase using timestamped revision vectors.
2. **Full-Text Search:** Enable SQLite FTS5 (Full-Text Search) in Drift for instant search highlighting across millions of words.
