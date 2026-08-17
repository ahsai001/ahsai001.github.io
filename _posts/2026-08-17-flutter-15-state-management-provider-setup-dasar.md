---
layout: post
title: "Flutter #15: State Management dengan Provider — Setup & Dasar"
date: 2026-08-17 07:00:00 +0700
tags: [flutter, dart, provider, state-management, changenotifier, tutorial, indonesia, pemrograman]
---

Welcome to **Fase Intermediate! 🎉** Dari 14 artikel kemarin, kamu udah bisa bikin app statis: layout, navigasi, form, tema. Tapi ada satu masalah yang makin kerasa tiap project makin besar: **`setState` nggak scalable**.

Bayangin kamu punya data user yang dipakai di 5 layar berbeda. Saat user logout, kamu harus update 5 layar sekaligus. Pakai `setState`? Kamu bakal lempar-lemparan callback antar widget sampe pusing, dan widget tree jadi berantakan. Di sinilah **state management** masuk, dan juaranya untuk pemula adalah **Provider**.

---

## 1. Apa Masalahnya dengan `setState`?

`setState` cuma bisa update state di widget tempat dia dipanggil. Kalau state dipakai widget lain, kamu harus **ngoper data lewat constructor** (prop drilling) atau bikin callback. Semakin dalam tree, semakin ribet.

Provider ngasih solusi elegan: **simpan state di satu tempat (model), terus widget mana pun bisa akses tanpa ngoper-ngoper manual.** Konsepnya mirip "global variable" tapi aman, terstruktur, dan otomatis rebuild widget yang butuh.

---

## 2. Apa itu Provider?

Provider adalah package state management resmi yang direkomendasikan tim Flutter. Intinya cuma 3 komponen:

- **`ChangeNotifier`** — kelas yang nyimpen state dan bisa "ngabarin" saat state berubah.
- **`ChangeNotifierProvider`** — widget yang nyediain model ke widget tree.
- **`Consumer` / `context.watch`** — cara widget baca dan dengerin perubahan state.

Gampangnya: model = otak, provider = kabel listrik, consumer = lampu yang nyala saat ada perubahan.

---

## 3. Setup Provider

Tambahkan dependency di `pubspec.yaml`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  provider: ^6.1.2
```

Terus jalankan `flutter pub get`. Udah, itu aja. Nggak perlu config ribet.

---

## 4. Bikin Model dengan `ChangeNotifier`

Contoh klasik: counter. Bikin file `counter_model.dart`:

```dart
import 'package:flutter/foundation.dart';

class CounterModel extends ChangeNotifier {
  int _count = 0;

  int get count => _count;

  void increment() {
    _count++;
    notifyListeners(); // 🚨 WAJIB: kasih tau semua yang dengerin
  }

  void decrement() {
    _count--;
    notifyListeners();
  }
}
```

Kuncinya ada di **`notifyListeners()`**. Setiap kali state berubah, panggil method ini biar semua widget yang `watch` otomatis rebuild. Lupa manggil ini = widget nggak bakal update, dan itu bug yang paling sering bikin pemula garuk-garuk kepala.

---

## 5. Nyediain Model dengan `ChangeNotifierProvider`

Bungkus `MaterialApp` kamu dengan provider di `main.dart`:

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'counter_model.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => CounterModel(),
      child: MaterialApp(
        debugShowCheckedModeBanner: false,
        home: const HomePage(),
      ),
    );
  }
}
```

Sekarang `CounterModel` bisa diakses di mana pun di bawah `MaterialApp`.

---

## 6. Baca & Ubah State dengan `context.watch` / `context.read`

Bikin halaman yang nampilin counter dan dua tombol:

```dart
class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    // watch = rebuild otomatis saat state berubah
    final counter = context.watch<CounterModel>();

    return Scaffold(
      appBar: AppBar(title: const Text('Provider Counter')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text('Nilai counter:'),
            Text(
              '${counter.count}',
              style: const TextStyle(fontSize: 48),
            ),
            const SizedBox(height: 24),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                IconButton(
                  icon: const Icon(Icons.remove),
                  onPressed: () => context.read<CounterModel>().decrement(),
                ),
                IconButton(
                  icon: const Icon(Icons.add),
                  onPressed: () => context.read<CounterModel>().increment(),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

Perhatikan bedanya:

- **`context.watch<T>()`** — baca state **dan** dengerin perubahan. Widget otomatis rebuild. Pakai di `build()`.
- **`context.read<T>()`** — baca state **tanpa** dengerin. Pakai di callback (`onPressed`), di mana kamu cuma mau manggil method, nggak perlu rebuild.

> ⚠️ **Jangan pakai `watch` di dalam `onPressed`** — itu cuma buang-buang resource. Buat event handler, cukup `read`.

---

## 7. `Consumer` — Update Sebagian Widget Aja

`watch` rebuild **seluruh** widget tempat dia dipanggil. Kadang kamu cuma mau bagian kecil yang update (misal cuma angkanya, nggak seluruh halaman). Pakai `Consumer`:

```dart
Consumer<CounterModel>(
  builder: (context, counter, child) {
    return Text('${counter.count}', style: const TextStyle(fontSize: 48));
  },
)
```

`Consumer` cuma rebuild widget yang dibungkusnya, bikin performa lebih hemat di app besar. Ini penting banget buat nanti.

---

## 8. Contoh Nyata: Todo List

Biar nggak cuma counter, ini model Todo sederhana biar kerasa gunanya di app beneran:

```dart
class TodoModel extends ChangeNotifier {
  final List<String> _todos = [];

  List<String> get todos => List.unmodifiable(_todos);

  void add(String todo) {
    if (todo.trim().isEmpty) return;
    _todos.add(todo.trim());
    notifyListeners();
  }

  void removeAt(int index) {
    _todos.removeAt(index);
    notifyListeners();
  }
}
```

Lalu di UI, gabungin dengan `TextField` dan `ListView` (konsep dari artikel #7 dan #9):

```dart
class TodoPage extends StatefulWidget {
  const TodoPage({super.key});
  @override
  State<TodoPage> createState() => _TodoPageState();
}

class _TodoPageState extends State<TodoPage> {
  final _controller = TextEditingController();

  @override
  Widget build(BuildContext context) {
    final todos = context.watch<TodoModel>().todos;

    return Scaffold(
      appBar: AppBar(title: const Text('Todo Provider')),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(16),
            child: Row(
              children: [
                Expanded(
                  child: TextField(
                    controller: _controller,
                    decoration: const InputDecoration(hintText: 'Tambah todo...'),
                  ),
                ),
                const SizedBox(width: 8),
                IconButton(
                  icon: const Icon(Icons.send),
                  onPressed: () {
                    context.read<TodoModel>().add(_controller.text);
                    _controller.clear();
                  },
                ),
              ],
            ),
          ),
          Expanded(
            child: ListView.builder(
              itemCount: todos.length,
              itemBuilder: (context, i) => ListTile(
                title: Text(todos[i]),
                trailing: IconButton(
                  icon: const Icon(Icons.delete),
                  onPressed: () => context.read<TodoModel>().removeAt(i),
                ),
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

Sekarang model Todo dipisah total dari UI. Mau tambah fitur "hapus semua" atau "simpan ke storage" nanti (artikel SharedPreferences & SQLite) tinggal edit modelnya, UI nggak usah diutak-atik. **Itulah kekuatan separation of concerns.**

---

## 9. Best Practice Provider

- **Satu `ChangeNotifier` = satu tanggung jawab.** Jangan bikin satu model raksasa yang ngurus semua hal.
- **`watch` di `build`, `read` di event.** Ingat aturan emas ini.
- **Panggil `notifyListeners()` cuma saat state beneran berubah**, jangan asal manggil biar nggak rebuild sia-sia.
- **`List.unmodifiable`** buat expose list, biar nggak bisa dimodifikasi langsung dari luar model.
- **Banyak provider?** Pakai `MultiProvider` — topik artikel berikutnya!

---

## Ringkasan

- **`setState` nggak scalable** untuk state yang dipakai lintas widget.
- **Provider** = package resmi, gampang dipelajari, jadi fondasi state management.
- **`ChangeNotifier`** = model penyimpan state + `notifyListeners()`.
- **`ChangeNotifierProvider`** = nyediain model ke widget tree.
- **`context.watch`** = baca + dengerin perubahan (di `build`).
- **`context.read`** = baca tanpa dengerin (di event handler).
- **`Consumer`** = rebuild sebagian widget aja.

Provider emang keliatan ribet buat counter 2 tombol, tapi percaya deh — begitu app kamu mulai punya auth, cart, atau data yang dipakai banyak layar, kamu bakal bersyukur pindah dari `setState`. Besok kita lanjut ke **`MultiProvider` & cara ngatur banyak model** biar app makin rapi. Stay tuned! 🚀

**Coba sendiri! Ubah project counter `setState`-mu jadi Provider, terus coba pisahin state jadi 2 model berbeda. Share hasilnya ke sosial media dan tag @ahsai001!**
