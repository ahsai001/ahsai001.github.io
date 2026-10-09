---
title: "Flutter #1: Apa itu Flutter & Kenapa Harus Flutter"
date: 2026-10-10 07:00:00 +0700
layout: post
tags: [flutter, dart, tutorial, pemula]
---

# Flutter #1 – Apa itu Flutter & Kenapa Harus Flutter?

Flutter adalah UI toolkit open‑source dari Google yang memungkinkan kamu menulis satu kode Dart lalu menghasilkan aplikasi native untuk **Android**, **iOS**, **Web**, **Desktop** (Windows, macOS, Linux), bahkan **embed** pada perangkat lain. Semua UI dirender oleh **Skia**, mesin grafis 2D yang cepat, sehingga tampilan konsisten di semua platform tanpa bergantung pada komponen native masing‑masing.

## Kenapa Pilih Flutter?

1. **Satu basis kode** – satu project, satu `pubspec.yaml`, satu set widget. Mengurangi duplikasi logika dan mempercepat iterasi.
2. **Hot Reload** – perubahan kode muncul dalam milidetik tanpa rebuild penuh, ideal untuk eksperimen UI.
3. **Performa near‑native** – Flutter meng‑compile ke ARM (Android/iOS) atau ke JavaScript (Web) sehingga performa hampir setara native.
4. **Ekosistem widget kaya** – Material dan Cupertino siap pakai, plus ribuan paket di `pub.dev`.
5. **Dukungan lintas platform** – satu tim developer, satu skill set, satu alur CI/CD.

## Struktur Project Flutter

```
my_app/
├─ android/        # kode native Android (gradle)
├─ ios/            # kode native iOS (Xcode)
├─ lib/            # kode Dart utama
│   └─ main.dart   # entry point
├─ test/           # unit & widget test
├─ pubspec.yaml    # dependensi & assets
└─ README.md
```

Folder `lib/` adalah inti. Semua UI dibangun dengan **widget**, yaitu class immutable yang menghasilkan **element** pada runtime. Dua tipe utama:

- **StatelessWidget** – tidak memiliki state internal, render hanya berdasarkan `build(context)`.
- **StatefulWidget** – memiliki objek `State` yang dapat dipanggil `setState()` untuk memperbarui UI.

## Contoh kode sederhana

### 1. Hello World dengan `StatelessWidget`
```dart
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      home: Scaffold(
        body: Center(
          child: Text('Hello, Flutter!'),
        ),
      ),
    );
  }
}
```

### 2. Counter dengan `StatefulWidget`
```dart
import 'package:flutter/material.dart';

void main() => runApp(const CounterApp());

class CounterApp extends StatelessWidget {
  const CounterApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(home: CounterPage());
  }
}

class CounterPage extends StatefulWidget {
  const CounterPage({Key? key}) : super(key: key);

  @override
  _CounterPageState createState() => _CounterPageState();
}

class _CounterPageState extends State<CounterPage> {
  int _count = 0;

  void _increment() => setState(() => _count++);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Demo Counter')),
      body: Center(child: Text('Count: $_count', style: const TextStyle(fontSize: 24))),
      floatingActionButton: FloatingActionButton(
        onPressed: _increment,
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

Kedua contoh di atas memperlihatkan pola dasar: `main()` memanggil `runApp()`, widget root berupa `MaterialApp`, dan seterusnya.

## Langkah selanjutnya

- **Instalasi SDK** – lihat artikel #2 untuk setup Flutter di Windows/macOS/Linux.
- **Mempelajari widget** – eksplorasi `Container`, `Row`, `Column`, `Stack` di artikel #4.
- **Buat project pertama** – `flutter create my_first_app` lalu jalankan `flutter run`.

Flutter memberi kamu kemampuan membangun aplikasi modern dengan cepat, sekaligus menyiapkan fondasi kuat untuk proyek skala besar. Jadi, jangan ragu memulai!

---

*Coba sendiri! Share ke sosial media dan tag @ahsai001*
