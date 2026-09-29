---
layout: post
title: "Apa itu Flutter & Kenapa Harus Flutter"
date: 2026-09-30 07:00:00 +0700
tags: [flutter, dart, tutorial, indonesia]
---

# Apa itu Flutter & Kenapa Harus Flutter?

Flutter adalah UI toolkit open‑source dari Google untuk membangun aplikasi native yang indah di iOS, Android, Web, dan desktop dari satu basis kode Dart. Dibandingkan dengan framework lain, Flutter menawarkan **hot‑reload** yang super cepat, **widget‑first** architecture, serta performa hampir setara native karena rendering lewat Skia.

## Kenapa Pilih Flutter?

1. **Satu kode, banyak platform** – Tuliskan UI sekali, jalankan di iOS, Android, Web, Windows, macOS, Linux.
2. **Produktivitas tinggi** – Hot‑reload mempercepat iterasi UI, mengurangi waktu debug.
3. **Ekosistem berkembang** – Ribuan paket di `pub.dev` memudahkan integrasi API, state‑management, dan layanan backend.
4. **Kualitas UI** – Setiap widget digambar sendiri, memberi kontrol pixel‑perfect tanpa ketergantungan native widget.
5. **Dukungan Google** – Flutter didukung oleh Google, terus diperbarui, dan dipakai di aplikasi besar seperti Google Ads.

## Struktur Project Flutter

Saat kita `flutter create my_app`, folder utama akan berisi:

```text
my_app/
├─ android/          # Kode Android native
├─ ios/              # Kode iOS native
├─ lib/              # Kode Dart utama
│   └─ main.dart     # Entry point aplikasi
├─ test/             # Unit & widget test
├─ pubspec.yaml      # Dependency & asset deklarasi
└─ README.md
```

File **`main.dart`** berisi fungsi `main()` yang memanggil `runApp(MyApp())`. `MyApp` biasanya memperluas `StatelessWidget` dan mengembalikan `MaterialApp` atau `CupertinoApp`.

## Contoh Minimal Flutter

Berikut contoh aplikasi “Hello World” Flutter yang dapat dijalankan langsung dengan `flutter run`.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      home: Scaffold(
        appBar: AppBar(title: const Text('Flutter Hello')),
        body: const Center(child: Text('Halo, Flutter!')),
      ),
    );
  }
}
```

Aplikasi di atas menampilkan `AppBar` dengan judul dan teks tengah “Halo, Flutter!”. Simpan di `lib/main.dart` lalu jalankan `flutter run` di terminal.

## Membuat Widget Kustom

Widget di Flutter hanyalah kelas Dart yang meng‑override `build`. Berikut contoh widget kartu sederhana yang menerima `title` dan `subtitle`.

```dart
class SimpleCard extends StatelessWidget {
  final String title;
  final String subtitle;

  const SimpleCard({Key? key, required this.title, required this.subtitle}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.all(8),
      child: ListTile(
        title: Text(title),
        subtitle: Text(subtitle),
        leading: const Icon(Icons.info),
      ),
    );
  }
}
```

Gunakan di dalam `ListView` atau `Column` untuk menampilkan daftar data.

## Langkah Selanjutnya

- **Instalasi**: Ikuti artikel nomor 2 untuk meng‑install Flutter SDK & IDE.
- **Widget Dasar**: Pelajari `Text`, `Container`, `Row`, `Column` (artikel 4).
- **State Management**: Kenali perbedaan `StatelessWidget` & `StatefulWidget` (artikel 5).

> **Coba sendiri!** Buat project baru, ganti `main.dart` dengan contoh di atas, jalankan, dan rasakan kecepatan hot‑reload.

---

*Share ke sosial media dan tag @ahsai001*