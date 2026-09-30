---
layout: post
title: "Flutter #1: Apa itu Flutter & Kenapa Harus Flutter"
date: 2026-10-01 07:00:00 +0700
tags: [flutter, introduction, tutorial, indonesia]
---

## Apa itu Flutter?

Flutter adalah UI toolkit open‑source dari Google untuk membangun aplikasi **native** pada Android, iOS, web, dan desktop dengan satu kode basis **Dart**. Flutter menampilkan *render engine* Skia yang melukis UI secara langsung ke canvas, sehingga tidak tergantung pada komponen native masing‑masing platform. Ini memberi kontrol pixel‑perfect pada tampilan serta performa 60 fps‑60 fps+.

## Kenapa Harus Flutter?

1. **Satu kode, banyak platform** – Tulis satu kali, compile ke Android, iOS, Web, macOS, Windows, Linux.
2. **Hot‑reload** – Perubahan kode langsung terlihat di emulator tanpa rebuild penuh, mempercepat iterasi.
3. **Widget‑first** – Seluruh UI dibangun dari widget, membuat layout deklaratif dan mudah dipahami.
4. **Komunitas besar** – Ribuan paket di `pub.dev`, contoh kode, tutorial, dan dukungan resmi.
5. **Performance native** – Karena menggunakan compiled AOT ke mesin ARM/Intel, aplikasi terasa cepat.

## Struktur Proyek Flutter

Setelah `flutter create my_app`, folder utama akan terlihat seperti ini:

```text
my_app/
├─ android/          # kode Android native
├─ ios/              # kode iOS native
├─ lib/              # kode Dart utama
│   └─ main.dart    # entry point aplikasi
├─ test/             # unit & widget test
├─ pubspec.yaml      # dependencies & assets
└─ ...
```

`lib/main.dart` adalah titik masuk. Berikut contoh *Hello World* sederhana:

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

### Contoh Widget Kustom

Kita buat widget `ColoredBox` yang menerima warna dan teks sebagai properti:

```dart
class ColoredBox extends StatelessWidget {
  final Color color;
  final String label;

  const ColoredBox({required this.color, required this.label, super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      color: color,
      padding: const EdgeInsets.all(16),
      child: Text(label, style: const TextStyle(color: Colors.white)),
    );
  }
}

// Penggunaan di dalam MyApp
class MyApp extends StatelessWidget {
  const MyApp({super.key});
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('Demo Widget')), 
        body: const Center(
          child: ColoredBox(color: Colors.blue, label: 'Flutter is fun!'),
        ),
      ),
    );
  }
}
```

## Langkah Selanjutnya

- Install Flutter SDK dan IDE (VS Code atau Android Studio).
- Jalankan `flutter doctor` untuk memastikan semua dependensi terpenuhi.
- Buat proyek baru dengan `flutter create my_first_app`.
- Eksperimen dengan widget‑widget dasar (Text, Container, Row, Column) dan **hot‑reload**.

> **Coba sendiri!** Buat file `main.dart` di folder `lib/`, jalankan `flutter run`, dan lihat hasilnya.

---

*Selamat belajar Flutter! Share ke sosial media dan tag @ahsai001.*
