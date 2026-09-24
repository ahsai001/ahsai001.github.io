---
layout: post
title: "Flutter #1: Apa itu Flutter & Kenapa Harus Flutter"
date: 2026-09-24 08:00:00 +0700
tags: [flutter, dart, tutorial, indonesia]
---

# Apa itu Flutter?

Flutter adalah framework UI open‑source buatan Google untuk membangun aplikasi **native** di Android, iOS, web, dan desktop dengan satu basis kode **Dart**. Dengan widget‑widget yang dapat dikustomisasi, Flutter menampilkan UI yang konsisten di semua platform tanpa perlu menulis kode native terpisah.

## Kenapa Pilih Flutter?

1. **One codebase, multiple platforms** – satu proyek, hasilkan APK, IPA, web, Windows, macOS, Linux.
2. **Hot‑reload** – lihat perubahan UI dalam hitungan milidetik, mempercepat iterasi.
3. **Performansi native** – engine Skia menggambar langsung ke canvas, hampir setara aplikasi native.
4. **Ekosistem berkembang** – ribuan paket di pub.dev, komunitas aktif, dukungan Google.
5. **UI konsisten** – widget‑widget terisolasi artinya tidak bergantung pada kontrol native platform.

Berikut contoh aplikasi "Hello World" Flutter yang bisa dijalankan di `main.dart`.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      home: const Scaffold(
        appBar: AppBar(title: Text('Hello Flutter')),
        body: Center(child: Text('Halo, dunia Flutter!')),
      ),
    );
  }
}
```

Kode di atas menampilkan layar berwarna biru dengan teks “Halo, dunia Flutter!”. Jalankan dengan:

```bash
flutter create hello_world
cd hello_world
# ganti lib/main.dart dengan kode di atas
flutter run
```

### Stateless vs Stateful Widget

Widget adalah blok penyusun UI. **StatelessWidget** bersifat immutable; UI tidak berubah setelah dibangun. **StatefulWidget** menyimpan state yang dapat di‑update dengan `setState`.

Contoh widget sederhana yang menampilkan counter tanpa state:

```dart
class SimpleMessage extends StatelessWidget {
  final String message;
  const SimpleMessage(this.message, {Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) => Text(message, style: const TextStyle(fontSize: 20));
}
```

Jika ingin menambah nilai counter tiap kali tombol ditekan, gunakan `StatefulWidget`:

```dart
class Counter extends StatefulWidget {
  const Counter({Key? key}) : super(key: key);
  @override
  _CounterState createState() => _CounterState();
}

class _CounterState extends State<Counter> {
  int _count = 0;
  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: [
        Text('Count: $_count', style: const TextStyle(fontSize: 24)),
        ElevatedButton(
          onPressed: () => setState(() => _count++),
          child: const Text('Tambah'),
        ),
      ],
    );
  }
}
```

## Ringkasan

Flutter memberi kekuatan untuk membuat aplikasi cepat, indah, dan lintas platform dengan satu bahasa – Dart. Pada artikel berikutnya kita akan meng‑install SDK, menyiapkan IDE, dan membuat proyek pertama.

**Coba sendiri! Share ke sosial media dan tag @ahsai001**

---

*Artikel ini bagian pertama dari seri “Flutter 90 hari”.*