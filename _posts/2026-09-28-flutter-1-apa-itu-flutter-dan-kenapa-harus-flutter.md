---
title: "Flutter #1: Apa itu Flutter & Kenapa Harus Flutter"
date: 2026-09-28 08:00:00 +0700
layout: post
tags: [flutter, dart, tutorial, indonesia, pemrograman]
---

## Apa itu Flutter?

Flutter adalah framework UI open‑source buatan Google untuk membuat aplikasi **native** di iOS, Android, web, dan desktop dengan satu basis kode Dart. Karena menggunakan **render engine Skia**, semua widget digambar sendiri, sehingga tampilan konsisten di semua platform.

## Kenapa Harus Flutter?

1. **Produktivitas Tinggi** – Hot‑reload memungkinkan melihat perubahan UI dalam milidetik.
2. **Satu Kode, Banyak Platform** – Tidak perlu menulis kode terpisah untuk Android (Kotlin/Java) dan iOS (Swift/ObjC).
3. **Performa Near‑Native** – Dart dikompilasi ke ARM atau JavaScript, menghasilkan UI 60fps.
4. **Ekosistem Besar** – Ribuan paket pub.dev, dukungan IDE (VS Code, Android Studio).
5. **Komunitas Global** – Banyak tutorial, contoh, dan plugin.

## Contoh Kode Flutter Sederhana

Berikut contoh aplikasi “Hello World” yang menampilkan teks di tengah layar.

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
        body: Center(
          child: Text('Hello, Flutter!'),
        ),
      ),
    );
  }
}
```

Aplikasi di atas hanya membutuhkan satu file **main.dart**. Jalankan dengan:

```bash
flutter run
```

## Struktur Project Flutter

Setelah `flutter create my_app`, folder utama akan terlihat seperti ini:

```
my_app/
├─ android/          # kode native Android
├─ ios/              # kode native iOS
├─ lib/              # kode Dart (utama)
│   └─ main.dart    # entry point
├─ test/             # unit & widget test
├─ pubspec.yaml      # dependencies & assets
└─ ...
```

Folder **lib/** tempat Anda menulis semua widget. **pubspec.yaml** mirip `package.json` di JavaScript: mendeklarasikan paket eksternal, asset gambar, dan konfigurasi lainnya.

## Membuat Widget Pertama

Widget adalah blok bangunan UI di Flutter. Semua elemen UI (tombol, teks, gambar) adalah widget. Berikut contoh **StatelessWidget** yang menampilkan kartu dengan judul dan deskripsi.

```dart
class SimpleCard extends StatelessWidget {
  final String title;
  final String description;

  const SimpleCard({Key? key, required this.title, required this.description})
      : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(title, style: Theme.of(context).textTheme.headline6),
            const SizedBox(height: 8),
            Text(description),
          ],
        ),
      ),
    );
  }
}
```

Gunakan widget ini di dalam `Scaffold`:

```dart
home: Scaffold(
  appBar: AppBar(title: const Text('Demo Card')),
  body: const SimpleCard(
    title: 'Flutter itu Keren',
    description: 'Dengan satu basis kode, Anda dapat membuat aplikasi lintas platform.',
  ),
),
```

## Ringkasan

*Flutter* memberikan **kecepatan**, **konsistensi**, dan **fleksibilitas** untuk membangun aplikasi modern. Di artikel berikutnya, kita bahas cara **install Flutter SDK** dan menyiapkan IDE VS Code.

---

**Coba sendiri! Share ke sosial media dan tag @ahsai001**
