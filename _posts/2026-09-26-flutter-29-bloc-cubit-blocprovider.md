---
layout: post
title: "Flutter #29: BLoC – Cubit & BlocProvider"
date: 2026-09-26 07:00:00 +0700
tags: [flutter, bloc, state-management, tutorial, indonesia]
---

# Pendahuluan
BLoC (Business Logic Component) adalah pola arsitektur yang memisahkan UI dari logika bisnis. Pada artikel ini kita fokus pada **Cubit** – varian ringan BLoC – dan cara menghubungkannya ke widget dengan **BlocProvider** serta **BlocBuilder**.

# Mengapa Cubit?
- Lebih sedikit boilerplate dibandingkan `Bloc<Event, State>`.
- Cocok untuk state sederhana atau ketika event tidak diperlukan.
- Tetap kompatibel dengan ekosistem `flutter_bloc`.

# Instalasi
```bash
flutter pub add flutter_bloc
```
> `flutter_bloc` sudah meng‑include `bloc` package.

# Contoh 1: CounterCubit
Berikut contoh paling dasar: counter yang dapat di‑increment dan reset.

```dart
import 'package:bloc/bloc.dart';

class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);

  void increment() => emit(state + 1);
  void reset() => emit(0);
}
```
> `Cubit<int>` menyimpan nilai integer sebagai state.

# Menggunakan BlocProvider & BlocBuilder
Kita bungkus UI dengan `BlocProvider` agar `CounterCubit` tersedia di subtree.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'counter_cubit.dart';

class CounterPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (_) => CounterCubit(),
      child: Scaffold(
        appBar: AppBar(title: const Text('Cubit Counter')),
        body: Center(
          child: BlocBuilder<CounterCubit, int>(
            builder: (context, count) {
              return Text('Count: $count', style: Theme.of(context).textTheme.headline4);
            },
          ),
        ),
        floatingActionButton: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            FloatingActionButton(
              heroTag: 'inc',
              child: const Icon(Icons.add),
              onPressed: () => context.read<CounterCubit>().increment(),
            ),
            const SizedBox(height: 8),
            FloatingActionButton(
              heroTag: 'reset',
              child: const Icon(Icons.refresh),
              onPressed: () => context.read<CounterCubit>().reset(),
            ),
          ],
        ),
      ),
    );
  }
}
```
Dengan `BlocBuilder` UI otomatis rebuild setiap kali `emit` dipanggil.

# Contoh 2: TodoCubit dengan List State
Untuk data yang lebih kompleks gunakan tipe custom.

```dart
class Todo {
  final String id;
  final String title;
  final bool done;
  Todo({required this.id, required this.title, this.done = false});
}

class TodoCubit extends Cubit<List<Todo>> {
  TodoCubit() : super([]);

  void add(String title) {
    final todo = Todo(id: DateTime.now().toIso8601String(), title: title);
    emit([...state, todo]);
  }

  void toggle(String id) {
    emit(state.map((t) => t.id == id ? Todo(id: t.id, title: t.title, done: !t.done) : t).toList());
  }

  void remove(String id) => emit(state.where((t) => t.id != id).toList());
}
```
Dan di UI:

```dart
BlocBuilder<TodoCubit, List<Todo>>(
  builder: (context, todos) {
    return ListView.builder(
      itemCount: todos.length,
      itemBuilder: (_, i) {
        final t = todos[i];
        return ListTile(
          title: Text(t.title, style: TextStyle(decoration: t.done ? TextDecoration.lineThrough : null)),
          leading: Checkbox(value: t.done, onChanged: (_) => context.read<TodoCubit>().toggle(t.id)),
          trailing: IconButton(icon: Icon(Icons.delete), onPressed: () => context.read<TodoCubit>().remove(t.id)),
        );
      },
    );
  },
)
```

# Tips & Best Practices
- **Gunakan `const`** bila memungkinkan untuk mengurangi rebuild.
- **Pisahkan cubit per feature** untuk memudahkan testing.
- **Gunakan `equatable`** bila state menjadi kelas kompleks.
- **Jangan melakukan I/O di Cubit**, delegasikan ke repository.

# Kesimpulan
Cubit memberi cara cepat menambahkan state management yang terstruktur. Dengan `BlocProvider` dan `BlocBuilder` integrasinya mulus, sehingga UI tetap reaktif tanpa boilerplate berlebih.

*Coba sendiri! Share ke sosial media dan tag @ahsai001* → skipped: additional tests, add when scaling to multi‑feature app.
