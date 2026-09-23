---
layout: post
title: "Flutter #1: Apa itu Flutter & Kenapa Harus Flutter"
date: 2026-09-23 08:00:00 +0700
tags: [flutter, dart, tutorial, indonesia, pemrograman]
---

# Flutter #1 – Apa itu Flutter & Kenapa Harus Flutter?

Flutter adalah UI toolkit open‑source yang dibuat oleh Google untuk membangun aplikasi native yang indah di **Android**, **iOS**, **web**, **desktop** (Windows, macOS, Linux) dengan satu basis kode Dart. Dengan **single codebase**, kamu dapat menulis UI sekali lalu dijalankan di semua platform tanpa menulis kode platform‑spesifik.

## Kenapa Pilih Flutter?

1. **Kecepatan Pengembangan** – Hot‑Reload memungkinkan melihat perubahan UI dalam milidetik tanpa rebuild full.
2. **Kinerja Native** – Flutter meng‑compile ke ARM/native code, tidak bergantung pada WebView.
3. **UI Konsisten** – Semua widget digambar oleh Flutter Engine, jadi tampilan seragam di semua OS.
4. **Ekosistem Kaya** – Ribuan paket di pub.dev, termasuk integrasi Firebase, state‑management, testing, dll.
5. **Gratis & Open‑Source** – Tanpa lisensi, didukung komunitas global.

## Struktur Project Flutter

Setelah `flutter create my_app`, struktur dasar akan tampak seperti ini:

```text
my_app/
├─ android/          # kode Android native
├─ ios/              # kode iOS native
├─ lib/               # kode Dart utama
│   └─ main.dart     # entry point aplikasi
├─ test/              # unit & widget test
├─ pubspec.yaml       # dependensi, asset, font
└─ README.md
```

Folder **lib/** berisi semua widget dan logika UI. `main.dart` berisi fungsi `main()` yang memanggil `runApp(MyApp())`. `MyApp` biasanya meng‑extend `StatelessWidget` dan meng‑return `MaterialApp` (atau `CupertinoApp`).

## Contoh Kode Sederhana

Berikut contoh aplikasi “Hello Flutter” yang menampilkan teks di tengah layar:

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
          child: Text('Halo, Flutter!'),
        ),
      ),
    );
  }
}
```

Aplikasi di atas menggunakan tiga widget built‑in:
- **MaterialApp** – wrapper aplikasi yang menyediakan tema & navigasi.
- **Scaffold** – struktur layar standar (appBar, body, floatingActionButton, dll).
- **Center** & **Text** – menampilkan teks di tengah.

### Interaktif dengan StatefulWidget

Jika kamu butuh UI yang berubah‑ubah, pakai `StatefulWidget`. Contoh counter sederhana:

```dart
import 'package:flutter/material.dart';

void main() => runApp(const CounterApp());

class CounterApp extends StatelessWidget {
  const CounterApp({super.key});
  @override
  Widget build(BuildContext context) {
    return const MaterialApp(home: CounterPage());
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
      body: Center(child: Text('Nilai: $_count', style: const TextStyle(fontSize: 24))),
      floatingActionButton: FloatingActionButton(onPressed: _increment, child: const Icon(Icons.add)),
    );
  }
}
```

Setiap kali tombol FAB ditekan, `_increment()` memanggil `setState`, memberitahu framework untuk rebuild widget dengan nilai baru.

## Mulai Flutter Sekarang

1. **Pasang SDK** – Unduh dari https://flutter.dev/docs/get-started/install.
2. **Setup IDE** – Instal plugin Flutter & Dart di VS Code atau Android Studio.
3. **Buat Project** – `flutter create flutter_tutorial && cd flutter_tutorial`.
4. **Jalankan** – `flutter run` pada emulator atau perangkat fisik.

Jika emulator belum terpasang, gunakan Android Studio AVD Manager atau `flutter emulators --launch chrome` untuk web preview.

## Ringkasan

Flutter memberi kamu satu basis kode Dart untuk menghasilkan aplikasi native cepat, responsif, dan tampak modern di semua platform. Dengan hot‑reload, widget berbasis deklaratif, dan ekosistem paket yang terus berkembang, Flutter menjadi pilihan utama untuk pengembangan mobile‑first dan multi‑platform.

---

*Coba sendiri! Share ke sosial media dan tag @ahsai001*
