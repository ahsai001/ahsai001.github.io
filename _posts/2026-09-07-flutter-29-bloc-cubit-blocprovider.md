---
layout: post
title: "Flutter #29: BLoC — Cubit & BlocProvider"
date: 2026-09-07 07:00:00 +0700
tags: [flutter, dart, tutorial, bloc, cubit, state-management, provider, clean-architecture]
description: "Mulai journey BLoC di Flutter! Pelajari Cubit sebagai versi ringkas BLoC dan BlocProvider untuk menanam state ke widget tree."
---

# Flutter #29: BLoC — Cubit & BlocProvider

Halo! Di artikel #28 kita sudah bikin custom widget yang reusable. Sekarang waktunya upgrade state management kita — dari `Provider` ke **BLoC (Business Logic Component)**.

Kenapa BLoC? Karena Provider cukup untuk app kecil, tapi begitu app-nya tumbuh (banyak screen, banyak state, banyak API call), kamu butuh sesuatu yang lebih **terstruktur, predictable, dan testable**. BLoC hadir untuk itu.

Hari ini kita mulai dari **Cubit** — versi ringkas dari BLoC yang lebih simpel tapi sudah sangat powerful. Next article (#30) kita lanjut ke BLoC penuh dengan Event & State. Let's go!

---

## 1. Kenapa BLoC?

Sebelum kita code, pahami dulu kenapa BLoC itu penting:

| Masalah di Provider | Solusi BLoC |
|---------------------|-------------|
| Logic campur di UI | Logic terpisah di Cubit/BLoC |
| Hard to test | Cubit testable tanpa Flutter |
| Race condition async | State berubah predictably |
| State tumpuk susah debug | State history bisa di-track |

BLoC memisahkan **UI** dari **Business Logic**. UI cuma render state, Cubit/BLoC cuma keluarkan state baru. Clean!

---

## 2. Instalasi

Tambahkan ini ke `pubspec.yaml`:

```yaml
dependencies:
  flutter_bloc: ^8.1.0
  equatable: ^2.0.5
```

Lalu jalankan:

```bash
flutter pub get
```

- **`flutter_bloc`** — package utama untuk BLoC + Cubit
- **`equatable`** — biar perbandingan state gampang (tanpa harus override `==` manual)

---

## 3. Apa Itu Cubit?

**Cubit** adalah class yang meng-extend `Cubit<State>`. Tugasnya cuma satu: **emit state baru**.

Bedanya sama BLoC:
- **Cubit** → fungsi biasa yang emit state langsung
- **BLoC** → pakai Event → proses → emit state (lebih terstruktur)

Kalau baru mulai, **mulai dari Cubit dulu**. Nanti kalau logic-nya makin kompleks, tinggal migrasi ke BLoC.

---

## 4. Contoh Pertama: Counter Cubit

Bikin file `counter_cubit.dart`:

```dart
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:equatable/equatable.dart';

// === State ===
// Equatable bikin perbandingan state jadi mudah
class CounterState extends Equatable {
  final int count;
  final String message;

  const CounterState({required this.count, this.message = ''});

  // Method untuk copy state baru (immutable pattern)
  CounterState copyWith({int? count, String? message}) {
    return CounterState(
      count: count ?? this.count,
      message: message ?? this.message,
    );
  }

  @override
  List<Object?> get props => [count, message];
}

// === Cubit ===
class CounterCubit extends Cubit<CounterState> {
  // Mulai dengan state awal
  CounterCubit() : super(const CounterState(count: 0));

  void increment() {
    final newCount = state.count + 1;
    emit(state.copyWith(
      count: newCount,
      message: 'Count sekarang: $newCount',
    ));
  }

  void decrement() {
    final newCount = state.count > 0 ? state.count - 1 : 0;
    emit(state.copyWith(
      count: newCount,
      message: newCount == 0 ? 'Minimal 0 ya!' : 'Count: $newCount',
    ));
  }

  void reset() {
    emit(const CounterState(count: 0, message: 'Di-reset!'));
  }
}
```

**Penjelasan:**

1. **`CounterState`** — merepresentasikan semua data yang dibutuhkan UI. Pakai `Equatable` biar kalau state-nya sama, Flutter gak perlu rebuild.
2. **`CounterCubit`** — punya fungsi `increment()`, `decrement()`, `reset()` yang masing-masing **emit state baru**.
3. **`emit()`** — fungsi dari Cubit untuk "kirim" state baru ke UI yang consume.

---

## 5. BlocProvider — Menanam Cubit ke Widget Tree

`BlocProvider` fungsinya sama seperti `Provider` di package Provider — yaitu **menanam Cubit/BLoC ke widget tree** supaya bisa diakses dari mana aja.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'counter_cubit.dart'; // import Cubit kita

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    // BlocProvider menanam CounterCubit ke tree
    return BlocProvider(
      create: (_) => CounterCubit(),
      child: const MaterialApp(
        home: CounterPage(),
      ),
    );
  }
}
```

**Key point:** `BlocProvider` itu parent-nya `MaterialApp`, jadi Cubit bisa diakses dari **semua screen** di dalam app.

---

## 6. BlocConsumer — Listen & Build

`BlocConsumer` adalah widget yang paling sering dipakai. Dia punya dua callback:

- **`listener`** — buat side-effect (show SnackBar, navigate, dll)
- **`builder`** — buat rebuild UI berdasarkan state

```dart
class CounterPage extends StatelessWidget {
  const CounterPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Counter Cubit')),
      body: Center(
        child: BlocConsumer<CounterCubit, CounterState>(
          // Listener: side-effect (sekali aksi)
          listenWhen: (previous, current) {
            // Cuma listen kalau message berubah
            return previous.message != current.message;
          },
          listener: (context, state) {
            // Munculkan SnackBar kalau message ada
            if (state.message.isNotEmpty) {
              ScaffoldMessenger.of(context).showSnackBar(
                SnackBar(
                  content: Text(state.message),
                  duration: const Duration(seconds: 1),
                ),
              );
            }
          },
          // Builder: rebuild UI
          buildWhen: (previous, current) {
            // Cuma rebuild kalau count berubah
            return previous.count != current.count;
          },
          builder: (context, state) {
            return Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Text(
                  '${state.count}',
                  style: const TextStyle(
                    fontSize: 64,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                const SizedBox(height: 20),
                Row(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    ElevatedButton.icon(
                      onPressed: () {
                        context.read<CounterCubit>().decrement();
                      },
                      icon: const Icon(Icons.remove),
                      label: const Text('-'),
                    ),
                    const SizedBox(width: 16),
                    ElevatedButton.icon(
                      onPressed: () {
                        context.read<CounterCubit>().increment();
                      },
                      icon: const Icon(Icons.add),
                      label: const Text('+'),
                    ),
                    const SizedBox(width: 16),
                    OutlinedButton(
                      onPressed: () {
                        context.read<CounterCubit>().reset();
                      },
                      child: const Text('Reset'),
                    ),
                  ],
                ),
              ],
            );
          },
        ),
      ),
    );
  }
}
```

**Penjelasan penting:**

- **`context.read<CounterCubit>()`** — ambil Cubit yang sudah ditanam oleh `BlocProvider`
- **`listenWhen` & `buildWhen`** — filter supaya rebuild gak terus-terusan (performa!)
- **`listener`** dipanggil setiap state berubah, tapi **bukan untuk rebuild UI**
- **`builder`** yang rebuild UI berdasarkan state terbaru

---

## 7. BlocBuilder vs BlocListener vs BlocConsumer

Pusing? Ini cheat sheet-nya:

| Widget | Fungsi | Kapan Pakai |
|--------|--------|-------------|
| **BlocBuilder** | Rebuild UI dari state | Kebanyakan kasus UI update |
| **BlocListener** | Eksekusi side-effect | Show SnackBar, Navigate |
| **BlocConsumer** | Builder + Listener | Butuh keduanya sekaligus |

---

## 8. Contoh Nyata: Todo Cubit

Counter kan terlalu simpel. Yuk bikin sesuatu yang lebih real — **Todo Cubit**!

```dart
class Todo {
  final String id;
  final String title;
  final bool isDone;

  const Todo({
    required this.id,
    required this.title,
    this.isDone = false,
  });

  Todo copyWith({String? title, bool? isDone}) {
    return Todo(
      id: id,
      title: title ?? this.title,
      isDone: isDone ?? this.isDone,
    );
  }
}

class TodoState extends Equatable {
  final List<Todo> todos;
  final String filter; // 'all', 'done', 'undone'

  const TodoState({
    this.todos = const [],
    this.filter = 'all',
  });

  List<Todo> get filteredTodos {
    switch (filter) {
      case 'done':
        return todos.where((t) => t.isDone).toList();
      case 'undone':
        return todos.where((t) => !t.isDone).toList();
      default:
        return todos;
    }
  }

  @override
  List<Object?> get props => [todos, filter];
}

class TodoCubit extends Cubit<TodoState> {
  TodoCubit() : super(const TodoState());

  void addTodo(String title) {
    final newTodo = Todo(
      id: DateTime.now().millisecondsSinceEpoch.toString(),
      title: title,
    );
    emit(state.copyWith(
      todos: [...state.todos, newTodo],
    ));
  }

  void toggleTodo(String id) {
    final updated = state.todos.map((todo) {
      if (todo.id == id) {
        return todo.copyWith(isDone: !todo.isDone);
      }
      return todo;
    }).toList();
    emit(state.copyWith(todos: updated));
  }

  void deleteTodo(String id) {
    final updated = state.todos.where((t) => t.id != id).toList();
    emit(state.copyWith(todos: updated));
  }

  void setFilter(String filter) {
    emit(state.copyWith(filter: filter));
  }
}
```

Ini pattern yang umum di production app:
1. **State punya computed property** (`filteredTodos`) — logic filtering di state, bukan di UI
2. **Cubit punya action methods** — `addTodo()`, `toggleTodo()`, `deleteTodo()`
3. **Immutable updates** — setiap perubahan bikin list baru, gak mutasi langsung

---

## 9. BlocSelector — Selective Rebuild

Mau rebuild cuma bagian tertentu dari UI? Pakai `BlocSelector`:

```dart
// Cuma rebuild Text angka, bukan tombol
BlocSelector<CounterCubit, CounterState, int>(
  selector: (state) => state.count,
  builder: (context, count) {
    return Text(
      '$count',
      style: const TextStyle(fontSize: 48),
    );
  },
),
```

`BlocSelector` cuma rebuild kalau **value yang dipilih** berubah. Super hemat performa!

---

## 10. Pattern yang Disarankan

Kalau project sudah mulai besar, ikuti pattern ini:

```
lib/
├── cubit/
│   ├── counter_cubit.dart
│   └── todo_cubit.dart
├── state/
│   ├── counter_state.dart
│   └── todo_state.dart
├── page/
│   ├── counter_page.dart
│   └── todo_page.dart
└── main.dart
```

**Pisahkan state, cubit, dan page** — jangan campur jadi satu file. Ini bikin codebase rapi dan mudah di-maintain.

---

## Kesimpulan

Hari ini kita belajar:

1. **Cubit** — state management simpel dari BLoC ecosystem
2. **`emit()`** — cara update state
3. **`BlocProvider`** — tanam Cubit ke widget tree
4. **`BlocConsumer`** — listener + builder dalam satu widget
5. **`BlocSelector`** — selective rebuild untuk performa
6. **Pattern** — pisahkan state, cubit, dan UI

Di artikel #30 kita lanjut ke **BLoC penuh** dengan `Event` dan `State` class yang terpisah — lebih terstruktur untuk app yang kompleks.

---

**Coba sendiri!** Bikin Cubit untuk theme switcher (dark/light mode), lalu pakai `BlocProvider` di level `MaterialApp`. Share hasilnya ke sosial media dan tag **@ahsai001** 🚀
