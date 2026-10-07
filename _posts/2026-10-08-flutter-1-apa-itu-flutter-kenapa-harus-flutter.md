---
layout: post
title: "Flutter #1: Apa itu Flutter & Kenapa Harus Flutter"
date: 2026-10-08 08:00:00 +0700
tags: [flutter, dart, tutorial, pemula]
---

## Apa itu Flutter?
Flutter adalah UI toolkit open‑source dari Google untuk membangun aplikasi **native** pada mobile, web, dan desktop menggunakan satu basis kode Dart. Dengan **widget**‑based rendering, Flutter tidak bergantung pada komponen native platform; ia menggambar semuanya sendiri lewat Skia, sehingga UI terlihat konsisten di semua OS.

## Kenapa Harus Flutter?
1. **Pengembangan cepat** – Hot‑Reload memungkinkan melihat perubahan UI dalam milidetik.
2. **Satu kode, banyak platform** – Android, iOS, Web, macOS, Windows, Linux.
3. **Performanya hampir native** – Karena rendering lewat Skia dan kompilasi AOT.
4. **Ekosistem kuat** – Ribuan paket di pub.dev, dukungan IDE VS Code & Android Studio.
5. **Komunitas berkembang** – Dokumentasi lengkap, contoh proyek, dan banyak tutorial (seperti seri 90‑hari ini).

## Memulai Project Flutter
Berikut contoh minimal untuk menampilkan *Hello World* di Flutter.

```bash
# Install Flutter SDK (pastikan sudah di PATH)
flutter create hello_world
cd hello_world
flutter run
```

File utama `lib/main.dart`:

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      home: const Scaffold(
        body: Center(
          child: Text('Hello, Flutter!'),
        ),
      ),
    );
  }
}
```

Jalankan `flutter run` pada emulator atau perangkat terhubung, dan lihat teks **Hello, Flutter!** muncul.

## Struktur Project
```
flutter_project/
├─ android/        # kode native Android
├─ ios/            # kode native iOS
├─ lib/            # kode Dart (utama)
│   └─ main.dart   # entry point
├─ test/           # unit & widget test
├─ pubspec.yaml    # dependencies & asset deklarasi
└─ ...
```
- **lib/** berisi semua widget dan logika UI.
- **pubspec.yaml** mengelola paket, font, gambar.

## Contoh Kode Kedua: Widget Stateless Sederhana
Widget Stateless tidak menyimpan state internal. Cocok untuk tampilan statis.

```dart
class Greeting extends StatelessWidget {
  final String name;
  const Greeting({required this.name, super.key});

  @override
  Widget build(BuildContext context) {
    return Text('Halo, $name!', style: const TextStyle(fontSize: 24));
  }
}
```
Gunakan di dalam **Scaffold**:

```dart
home: const Scaffold(
  body: Center(
    child: Greeting(name: 'Ahsai'),
  ),
),
```

## Selanjutnya
Di artikel #2 kita akan **install Flutter SDK**, menyiapkan IDE, dan menyiapkan emulator. Pastikan sudah menginstal JDK 11+ dan Android SDK.

---
Coba sendiri! Share ke sosial media dan tag @ahsai001
