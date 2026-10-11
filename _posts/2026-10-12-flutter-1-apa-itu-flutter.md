---
layout: post
title: "Apa itu Flutter & Kenapa Harus Flutter"
date: 2026-10-12 07:00:00 +0800
tags: [flutter, introduction, tutorial, indonesia]
---

## Apa Itu Flutter?

Flutter adalah UI toolkit open‑source dari Google untuk membangun aplikasi native lintas platform — Android, iOS, Web, bahkan desktop — dengan satu basis kode Dart. Semua elemen UI dirender oleh engine Skia, sehingga tampilan konsisten di semua perangkat.

## Kenapa Harus Flutter?

1. **Pengembangan cepat** – Hot Reload mempercepat iterasi UI.
2. **Satu kode, banyak platform** – Hemat waktu & biaya.
3. **Performa native** – Compile ke ARM & x86, tidak ada bridge JavaScript.
4. **Ekosistem kaya** – Ribuan paket di pub.dev, support IDE kuat.

## Contoh Kode Pertama: Hello World

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

Jalankan dengan `flutter run` pada emulator atau perangkat fisik.

## Contoh Widget Stateful: Counter

```dart
import 'package:flutter/material.dart';

void main() => runApp(const CounterApp());

class CounterApp extends StatelessWidget {
  const CounterApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      home: CounterPage(),
    );
  }
}

class CounterPage extends StatefulWidget {
  const CounterPage({super.key});
  @override
  State<CounterPage> createState() => _CounterPageState();
}

class _CounterPageState extends State<CounterPage> {
  int _count = 0;
  void _increment() => setState(() => _count++);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Counter')),
      body: Center(child: Text('Count: $_count', style: const TextStyle(fontSize: 24))),
      floatingActionButton: FloatingActionButton(onPressed: _increment, child: const Icon(Icons.add)),
    );
  }
}
```

Tekan tombol **+** untuk menambah angka. StatefulWidget memungkinkan UI memperbarui ketika state berubah.

## CTA

Coba sendiri! Build aplikasi pertama dengan Flutter, share ke sosial media dan tag @ahsai001.
