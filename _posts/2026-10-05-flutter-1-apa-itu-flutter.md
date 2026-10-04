---
layout: post
title: "Flutter #1: Apa Itu Flutter & Kenapa Harus Flutter?"
date: 2026-10-05 07:00:00 +0700
tags: [flutter, dart, tutorial, indonesia]
---

# Flutter itu apa?

Flutter adalah UI toolkit open‑source yang dikembangkan Google untuk membangun aplikasi native — baik mobile (iOS, Android), web, maupun desktop — dengan satu basis kode Dart. Karena Flutter meng‑compile ke kode mesin (ARM, x86) dan menggunakan rendering engine Skia, UI yang dihasilkan konsisten di semua platform tanpa bergantung pada komponen native masing‑masing.

## Kenapa harus Flutter?

1. **Produktivitas tinggi** – Hot‑reload memungkinkan Anda melihat perubahan kode dalam hitungan detik. Tidak perlu rebuild penuh.
2. **Satu basis kode** – Satu project untuk Android, iOS, web, Windows, macOS, dan Linux. Mengurangi duplikasi logika.
3. **Performance native** – Karena Flutter tidak menggunakan bridge JavaScript, animasi berjalan mulus pada 60 fps.
4. **Komunitas besar** – Ribuan paket di `pub.dev` untuk hampir semua kebutuhan, mulai dari state‑management hingga integrasi Firebase.
5. **Dukungan Google** – Update reguler, dokumentasi lengkap, dan integrasi dengan Firebase, Google Maps, dll.

## Contoh kode sederhana

Berikut contoh aplikasi “Hello World” Flutter yang dapat dijalankan langsung di `main.dart`:

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: Text('Flutter Demo')),
        body: Center(child: Text('Halo, dunia!')),
      ),
    );
  }
}
```

Aplikasi di atas menampilkan layar berwarna biru dengan teks *Halo, dunia!* di tengah. Cukup jalankan `flutter run` pada emulator atau perangkat.

### Mengubah warna latar belakang dengan `Container`

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Container(
          color: Colors.teal,
          alignment: Alignment.center,
          child: const Text(
            'Flutter itu menyenangkan',
            style: TextStyle(fontSize: 24, color: Colors.white),
          ),
        ),
      ),
    );
  }
}
```

Kode ini menunjukkan bagaimana widget `Container` dapat dipakai untuk mengatur warna latar, alignment, dan menampilkan teks.

## Ringkasan

Flutter memberi Anda satu bahasa (Dart), satu toolchain, dan satu UI engine untuk menciptakan aplikasi yang cepat, indah, dan konsisten di semua platform. Dengan komunitas yang terus berkembang, belajar Flutter kini menjadi investasi yang menguntungkan bagi pengembang yang ingin membangun produk multiplatform.

---

Coba sendiri! Share ke sosial media dan tag @ahsai001
