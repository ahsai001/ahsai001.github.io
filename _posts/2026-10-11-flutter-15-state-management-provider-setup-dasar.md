---
layout: post
title: "Flutter #15: State Management dengan Provider – Setup & Dasar"
date: 2026-10-11 07:00:00 +0700
tags: [flutter, provider, state-management, tutorial]
---

## Pendahuluan
Provider adalah paket resmi Flutter untuk manajemen state yang simpel namun powerful. Ia menggunakan *InheritedWidget* di balik layar, memberi cara bersih untuk mengakses dan memperbarui data di seluruh widget tree.

## Instalasi
Buka `pubspec.yaml` dan tambahkan:
```yaml
dependencies:
  provider: ^6.1.0
```
Jalankan `flutter pub get`.

## Contoh 1 – Counter sederhana
Berikut contoh minimal memakai `ChangeNotifier`.
```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() => runApp(const MyApp());

class Counter extends ChangeNotifier {
  int _value = 0;
  int get value => _value;
  void increment() {
    _value++;
    notifyListeners();
  }
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);
  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => Counter(),
      child: MaterialApp(
        home: const CounterPage(),
      ),
    );
  }
}

class CounterPage extends StatelessWidget {
  const CounterPage({Key? key}) : super(key: key);
  @override
  Widget build(BuildContext context) {
    final counter = Provider.of<Counter>(context);
    return Scaffold(
      appBar: AppBar(title: const Text('Provider Counter')),
      body: Center(child: Text('Nilai: ${counter.value}', style: const TextStyle(fontSize: 24))),
      floatingActionButton: FloatingActionButton(
        onPressed: counter.increment,
        child: const Icon(Icons.add),
      ),
    );
  }
}
```
Jalankan `flutter run` – tiap kali tombol ditekan nilai bertambah, UI otomatis refresh.

## Contoh 2 – Menggunakan `Consumer`
`Consumer` memberi rebuild lebih terkontrol.
```dart
class CounterPage2 extends StatelessWidget {
  const CounterPage2({Key? key}) : super(key: key);
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Consumer Demo')),
      body: Center(
        child: Consumer<Counter>(
          builder: (_, counter, __) => Text('Nilai: ${counter.value}', style: const TextStyle(fontSize: 24)),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => Provider.of<Counter>(context, listen: false).increment(),
        child: const Icon(Icons.add),
      ),
    );
  }
}
```
`listen: false` pada `Provider.of` mencegah rebuild pada FAB.

## Struktur Project yang Disarankan
```
lib/
 ├─ main.dart          ← entry point
 ├─ models/
 │   └─ counter.dart   ← ChangeNotifier
 └─ screens/
     └─ counter_page.dart
```
Pisahkan logika (`Counter`) dari UI (`CounterPage`).

## Tips & Trik
- **Gunakan `MultiProvider`** bila ada lebih dari satu provider.
- **Jangan panggil `notifyListeners` terlalu sering** – batch perubahan bila memungkinkan.
- **Debug dengan `ProviderScope`** (package `riverpod`) jika butuh inspeksi state.

## Kesimpulan
Provider memberikan API yang mudah dipahami, cocok untuk aplikasi kecil‑menengah. Dengan `ChangeNotifier`, `Consumer`, dan `MultiProvider`, Anda dapat mengelola state secara terpusat tanpa boilerplate berlebih.

---
*Coba sendiri! Share ke sosial media dan tag @ahsai001*