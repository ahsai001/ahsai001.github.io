---
layout: post
title: "Flutter #23: SQLite & Drift Database — Local Storage yang Powerful"
date: 2026-09-01 07:00:00 +0700
tags: [flutter, dart, sqlite, drift, database, local-storage, tutorial]
description: "Pelajari cara menyimpan data lokal di Flutter menggunakan SQLite dan Drift. Mulai dari setup database, CRUD operation, sampai best practice untuk production."
---

# Flutter #23: SQLite & Drift Database — Local Storage yang Powerful

Halo! Di artikel sebelumnya kita sudah belajar **SharedPreferences** untuk menyimpan data sederhana. Tapi kalau kamu butuh menyimpan data yang lebih kompleks — seperti daftar tugas, catatan, atau data pengguna — SharedPreferences sudah tidak cukup.

Masalahnya, SharedPreferences cuma cocok untuk key-value sederhana. Kalau kamu butuh **relasi antar data**, **query yang kompleks**, atau **menyimpan ratusan record** dengan efisien, kamu butuh database asli.

Di sinilah **SQLite** masuk. Dan di Flutter, cara paling powerful untuk pakai SQLite adalah lewat **Drift** (sebelumnya bernama Moor).

## Kenapa Drift, Bukan Langsung sqflite?

SQLite di Flutter sebenarnya bisa diakses langsung lewat package `sqflite`. Tapi ada beberapa masalah:

- Query ditulis sebagai **string mentah** — rentan typo
- Tidak ada **type safety** — error baru ketahuan saat runtime
- Tidak ada **codegen** — CRUD manual untuk setiap tabel

**Drift** menyelesaikan semua masalah ini:

- **Type-safe queries** — error ketahuan saat compile
- **Code generation** — tabel dan query otomatis dibuatkan
- **Reactive streams** — data berubah, UI otomatis update
- **Migrasi database** — lebih mudah dikelola

Yuk langsung praktek!

## Setup Project

Tambahkan dependency ke `pubspec.yaml`:

```yaml
dependencies:
  drift: ^2.14.0
  sqlite3_flutter_libs: ^0.5.0
  path_provider: ^2.1.0
  path: ^1.8.0

dev_dependencies:
  drift_dev: ^2.14.0
  build_runner: ^2.4.0
```

Lalu jalankan:

```bash
flutter pub get
dart run build_runner build --delete-conflicting-outputs
```

## Definisi Database & Tabel

Buat file `lib/database/app_database.dart`:

```dart
import 'dart:io';
import 'package:drift/drift.dart';
import 'package:drift/native.dart';
import 'package:path_provider/path_provider.dart';
import 'package:path/path.dart' as p;

// Import generated part file
part 'app_database.g.dart';

// Definisi tabel
class Tasks extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text().withLength(min: 1, max: 100)();
  TextColumn get description => text().nullable()();
  BoolColumn get isDone => boolean().withDefault(const Constant(false))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
}

// Class database utama
@DriftDatabase(tables: [Tasks])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());

  // Versi schema — naikkan saat ada perubahan tabel
  @override
  int get schemaVersion => 1;

  // CRUD Methods
  Future<List<Task>> getAllTasks() => select(tasks).get();

  Stream<List<Task>> watchAllTasks() => select(tasks).watch();

  Future<int> insertTask(TasksCompanion task) => into(tasks).insert(task);

  Future<bool> updateTask(Task task) => update(tasks).replace(task);

  Future<int> deleteTask(int id) =>
      (delete(tasks)..where((t) => t.id.equals(id))).go();

  // Query: ambil task yang belum selesai
  Stream<List<Task>> watchPendingTasks() =>
      (select(tasks)..where((t) => t.isDone.equals(false))).watch();
}

LazyDatabase _openConnection() {
  return LazyDatabase(() async {
    final dbFolder = await getApplicationDocumentsDirectory();
    final file = File(p.join(dbFolder.path, 'tasks.sqlite'));
    return NativeDatabase.createInBackground(file);
  });
}
```

Jalankan code generator untuk membuat file `.g.dart`:

```bash
dart run build_runner build --delete-conflicting-outputs
```

## Menggunakan Database di Widget

Sekarang kita pakai database di halaman utama:

```dart
import 'package:flutter/material.dart';
import 'package:drift/drift.dart' as drift;
import '../database/app_database.dart';

class TaskListPage extends StatefulWidget {
  const TaskListPage({super.key});

  @override
  State<TaskListPage> createState() => _TaskListPageState();
}

class _TaskListPageState extends State<TaskListPage> {
  late AppDatabase _database;

  @override
  void initState() {
    super.initState();
    _database = AppDatabase();
  }

  @override
  void dispose() {
    _database.close();
    super.dispose();
  }

  void _addTask() {
    final titleController = TextEditingController();

    showDialog(
      context: context,
      builder: (ctx) => AlertDialog(
        title: const Text('Tugas Baru'),
        content: TextField(
          controller: titleController,
          decoration: const InputDecoration(
            hintText: 'Judul tugas...',
            border: OutlineInputBorder(),
          ),
          autofocus: true,
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(ctx),
            child: const Text('Batal'),
          ),
          FilledButton(
            onPressed: () {
              if (titleController.text.isNotEmpty) {
                _database.insertTask(
                  TasksCompanion.insert(title: titleController.text),
                );
              }
              Navigator.pop(ctx);
            },
            child: const Text('Simpan'),
          ),
        ],
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Daftar Tugas')),
      floatingActionButton: FloatingActionButton(
        onPressed: _addTask,
        child: const Icon(Icons.add),
      ),
      // StreamBuilder = reactive! UI update otomatis
      body: StreamBuilder<List<Task>>(
        stream: _database.watchAllTasks(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) {
            return const Center(child: CircularProgressIndicator());
          }

          final tasks = snapshot.data!;

          if (tasks.isEmpty) {
            return const Center(
              child: Text(
                'Belum ada tugas.\nTap + untuk menambah!',
                textAlign: TextAlign.center,
                style: TextStyle(fontSize: 16, color: Colors.grey),
              ),
            );
          }

          return ListView.builder(
            itemCount: tasks.length,
            itemBuilder: (context, index) {
              final task = tasks[index];
              return CheckboxListTile(
                value: task.isDone,
                onChanged: (value) {
                  // Update status done
                  _database.updateTask(
                    task.copyWith(isDone: value ?? false),
                  );
                },
                title: Text(
                  task.title,
                  style: TextStyle(
                    decoration: task.isDone
                        ? TextDecoration.lineThrough
                        : null,
                    color: task.isDone ? Colors.grey : null,
                  ),
                ),
                subtitle: task.description != null
                    ? Text(task.description!)
                    : null,
                secondary: IconButton(
                  icon: const Icon(Icons.delete, color: Colors.red),
                  onPressed: () => _database.deleteTask(task.id),
                ),
              );
            },
          );
        },
      ),
    );
  }
}
```

## Penjelasan Kunci

### Type-Safe Queries

Perhatikan baris ini:

```dart
// Tidak perlu tulis string SQL manual!
Stream<List<Task>> watchPendingTasks() =>
    (select(tasks)..where((t) => t.isDone.equals(false))).watch();
```

Drift menggunakan **table dan column objects** untuk membangun query. Kalau kamu salah nama kolom, error muncul **saat compile** — bukan saat user pakai aplikasi.

### Reactive Streams dengan watch()

`watchAllTasks()` mengembalikan `Stream`. Setiap kali data di database berubah (insert, update, delete), stream **otomatis emit** data baru. **StreamBuilder** di UI akan otomatis rebuild.

Kamu tidak perlu refresh manual — semua reactif!

### Migrasi Database

Ketika kamu menambah kolom atau tabel baru di masa depan, naikkan `schemaVersion` dan tambahkan migrasi:

```dart
@override
int get schemaVersion => 2; // Naikkan dari 1 ke 2

@override
MigrationStrategy get migration => MigrationStrategy(
  onCreate: (m) => m.createAll(),
  onUpgrade: (m, from, to) async {
    // Tambah kolom 'priority' di tabel tasks
    if (from < 2) {
      await m.addColumn(tasks, tasks.priority);
    }
  },
);
```

## Tips & Best Practice

1. **Inisialisasi sekali** — buat instance `AppDatabase()` di root app (biasanya di `main.dart` atau Provider) dan bagikan ke semua widget
2. **Pakai StreamBuilder** untuk data yang sering berubah — reactive, otomatis update
3. **Pakai FutureBuilder** untuk data yang jarang berubah (misal load data awal sekali)
4. **Pisahkan file database** — jangan campur UI dan logic di satu file
5. **Gunakan Drift's generated code** — jangan menulis SQL mentah kecuali benar-benar perlu

## Perbandingan: SharedPreferences vs Drift

| Aspek | SharedPreferences | Drift/SQLite |
|-------|------------------|--------------|
| Data sederhana | ✅ Ideal | ⚠️ Overkill |
| Data kompleks | ❌ Tidak cocok | ✅ Ideal |
| Relasi antar data | ❌ Tidak ada | ✅ Full support |
| Query/filter | ❌ Manual | ✅ Powerful |
| Performance (banyak data) | ⚠️ Lambat | ✅ Sangat cepat |
| Type safety | ❌ Tipe campuran | ✅ Strongly typed |

## Kapan Pakai Apa?

- **SharedPreferences**: token auth, settings ringan, theme preference
- **Drift/SQLite**: daftar kontak, catatan, riwayat, data offline, cache API kompleks

---

## Ringkasan Hari Ini

- **SQLite** adalah database lokal yang powerful dan sudah built-in di Flutter
- **Drift** adalah package wrapper yang memberikan type safety, code generation, dan reactive streams
- **`watch()`** mengembalikan stream — UI otomatis update saat data berubah
- **Migration strategy** memudahkan upgrade schema tanpa kehilangan data
- Drift cocok untuk data kompleks, SharedPreferences untuk data sederhana

Di artikel berikutnya kita akan belajar **Bottom Navigation & TabBar** — bikin navigasi yang profesional di aplikasi Flutter!

---

*Coba sendiri! Buat aplikasi todo list sederhana dengan Drift, lalu tambahkan fitur filter (selesai/belum). Share ke sosial media dan tag **@ahsai001**!* 🚀
