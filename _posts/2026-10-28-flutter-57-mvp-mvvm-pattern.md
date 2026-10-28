---
layout: post
title: "Flutter #57: MVP/MVVM Pattern"
date: 2026-10-28 07:00:00 +0700
categories: [flutter]
tags: [flutter, dart, tutorial, indonesia]
---

# Flutter #57: MVP/MVVM Pattern

Pola arsitektur **MVP** (Model‑View‑Presenter) dan **MVVM** (Model‑View‑ViewModel) membantu memisahkan logika bisnis dari UI, membuat kode lebih mudah diuji, dipelihara, dan skalabel. Pada artikel ini kita bahas dasar‑dasarnya, kapan pakai, serta contoh implementasi sederhana di Flutter.

## Kenapa pakai MVP / MVVM?
- **Single Responsibility**: UI hanya menampilkan, Presenter/ViewModel mengatur alur, Model menyimpan data.
- **Testable**: Presenter/ViewModel dapat diuji unit tanpa widget testing.
- **Reusability**: Logika dapat dipakai kembali di beberapa halaman.

## MVP vs MVVM singkat
|                | MVP                               | MVVM                              |
|----------------|-----------------------------------|-----------------------------------|
| **Komponen**   | Model, View, Presenter            | Model, View, ViewModel             |
| **Alur**        | View → Presenter → Model → View    | View ↔ ViewModel ↔ Model          |
| **Binding**    | Manual (setState / callback)       | Reactive (ChangeNotifier / Stream) |
| **Kelebihan**   | Lebih simpel, kontrol penuh        | Lebih deklaratif, data binding    |
| **Kekurangan**  | Boilerplate lebih banyak          | Memerlukan state management lib   |

Kita akan membuat **counter app** yang menampilkan angka dan tombol `+`. Logika penambahan berada di Presenter (MVP) atau ViewModel (MVVM). Kedua contoh dapat dijalankan langsung di `main.dart`.

---

## contoh MVP

### 1. Model (counter_model.dart)
```dart
class CounterModel {
  int value = 0;
  void increment() => value++;
}
```

### 2. Presenter (counter_presenter.dart)
```dart
import 'counter_model.dart';

class CounterPresenter {
  final CounterModel _model;
  final void Function(int) onUpdate;
  CounterPresenter(this._model, this.onUpdate);
  void increment() {
    _model.increment();
    onUpdate(_model.value);
  }
}
```

### 3. View (main.dart)
```dart
import 'package:flutter/material.dart';
import 'counter_model.dart';
import 'counter_presenter.dart';

void main() => runApp(const MyApp());

class MyApp extends StatefulWidget {
  const MyApp({super.key});
  @override
  State<MyApp> createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  late final CounterPresenter _presenter;
  int _counter = 0;

  @override
  void initState() {
    super.initState();
    _presenter = CounterPresenter(CounterModel(), (value) {
      setState(() => _counter = value);
    });
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('MVP Counter')),
        body: Center(child: Text('Count: $_counter', style: const TextStyle(fontSize: 32))),
        floatingActionButton: FloatingActionButton(
          onPressed: _presenter.increment,
          child: const Icon(Icons.add),
        ),
      ),
    );
  }
}
```

Jalankan `flutter run` – setiap kali tombol ditekan, Presenter meng-update Model lalu memanggil callback `onUpdate` yang memicu `setState` di View.

---

## contoh MVVM dengan `ChangeNotifier`

### 1. Model (counter_model.dart) – sama seperti sebelumnya.

### 2. ViewModel (counter_viewmodel.dart)
```dart
import 'package:flutter/foundation.dart';
import 'counter_model.dart';

class CounterViewModel extends ChangeNotifier {
  final CounterModel _model = CounterModel();
  int get value => _model.value;
  void increment() {
    _model.increment();
    notifyListeners();
  }
}
```

### 3. View (main.dart)
```dart
import 'package:flutter/material.dart';
import 'counter_viewmodel.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});
  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => CounterViewModel(),
      child: MaterialApp(
        home: const CounterPage(),
      ),
    );
  }
}

class CounterPage extends StatelessWidget {
  const CounterPage({super.key});
  @override
  Widget build(BuildContext context) {
    final vm = Provider.of<CounterViewModel>(context);
    return Scaffold(
      appBar: AppBar(title: const Text('MVVM Counter')),
      body: Center(child: Text('Count: ${vm.value}', style: const TextStyle(fontSize: 32))),
      floatingActionButton: FloatingActionButton(
        onPressed: vm.increment,
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

> **Catatan**: Tambahkan dependensi `provider: ^6.0.0` pada `pubspec.yaml`.

## Tes unit Presenter / ViewModel
```dart
import 'package:test/test.dart';
import 'counter_model.dart';
import 'counter_presenter.dart';
import 'counter_viewmodel.dart';

void main() {
  test('MVP presenter increments model', () {
    final model = CounterModel();
    var updated = 0;
    final presenter = CounterPresenter(model, (v) => updated = v);
    presenter.increment();
    expect(updated, 1);
    expect(model.value, 1);
  });

  test('MVVM viewmodel notifies listeners', () {
    final vm = CounterViewModel();
    var notified = false;
    vm.addListener(() => notified = true);
    vm.increment();
    expect(vm.value, 1);
    expect(notified, true);
  });
}
```

## Kapan pakai MVP vs MVVM?
- **MVP** cocok untuk projek kecil atau tim yang belum memakai state‑management library.
- **MVVM** lebih cocok bila Anda sudah menggunakan `Provider`, `Riverpod`, atau `GetX` karena binding otomatis mengurangi boilerplate.

## Ringkasan
- Pisahkan **Model**, **View**, dan **Presenter/ViewModel**.
- Presenter meng‑handle callback manual, MVVM memanfaatkan `ChangeNotifier` atau `Stream`.
- Kedua pola meningkatkan testability dan maintainability.

---

**Coba sendiri! Share ke sosial media dan tag @ahsai001**
