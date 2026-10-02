---
title: "Flutter #1: Apa itu Flutter & Kenapa Harus Flutter"
date: 2026-10-02 08:00:00 +0700
categories: [flutter, fundamentals]
tags: [flutter, dart, tutorial, indonesia]
---

## Apa Itu Flutter?

Flutter adalah UI toolkit open‑source yang dikembangkan Google untuk membangun aplikasi native pada **mobile**, **web**, dan **desktop** dengan satu basis kode Dart. Dibandingkan dengan framework lain, Flutter menampilkan **render engine Skia**‑nya sendiri, artinya UI tidak bergantung pada komponen native masing‑masing platform. Hasilnya, tampilan konsisten di Android, iOS, Chrome, dan Windows.

### Kenapa Harus Flutter?

1. **Single Codebase** – Tulis satu kali, deploy ke banyak platform.
2. **Hot Reload** – Perubahan kode langsung terlihat tanpa harus rebuild seluruh aplikasi.
3. **Performance** – Karena menggunakan native ARM code dan tidak lewat bridge JavaScript, Flutter seringkali lebih cepat daripada hybrid frameworks.
4. **Rich Widget Library** – Ribuan widget siap pakai, mudah dikustomisasi, dan mendukung material design serta cupertino.
5. **Komunitas Besar** – Paket Pub.dev menyediakan ribuan plugin, mulai dari akses kamera hingga Firebase.

## Contoh Kode Flutter Sederhana

Berikut contoh aplikasi *Hello World* yang menampilkan teks di tengah layar.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

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

Aplikasi di atas hanya membutuhkan **main.dart** dan file `pubspec.yaml` standar. Jalankan dengan `flutter run` pada emulator atau perangkat.

## Stateless vs Stateful Widget

Widget di Flutter terbagi menjadi dua tipe utama:

- **StatelessWidget** – Tidak memiliki state yang berubah setelah dibangun. Cocok untuk UI statis.
- **StatefulWidget** – Memiliki **State** yang dapat di‑update menggunakan `setState`.

Contoh perbandingan:

```dart
// Stateless contoh
class Greeting extends StatelessWidget {
  final String name;
  const Greeting({required this.name, super.key});

  @override
  Widget build(BuildContext context) => Text('Hai, $name!');
}

// Stateful contoh – counter
class Counter extends StatefulWidget {
  const Counter({super.key});
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
        Text('Tap count: $_count'),
        ElevatedButton(
          onPressed: () => setState(() => _count++),
          child: const Text('Tambah'),
        ),
      ],
    );
  }
}
```

Stateless tidak memanggil `setState`, sehingga tidak ada overhead untuk rebuild. Stateful berguna ketika UI harus merespon perubahan data, misalnya input pengguna atau hasil network request.

## Ringkasan

Flutter memberi Anda **kecepatan pengembangan**, **konsistensi UI**, dan **performansi tinggi** dengan satu bahasa (Dart). Pada tutorial selanjutnya kita akan meng‑install SDK, menyiapkan IDE, dan menelusuri struktur project Flutter.

---

**Coba sendiri! Share ke sosial media dan tag @ahsai001**