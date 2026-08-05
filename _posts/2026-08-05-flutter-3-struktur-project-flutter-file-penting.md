---
layout: post
title: "Flutter #3: Struktur Project Flutter & File Penting yang Wajib Kamu Kenali"
date: 2026-08-05 07:00:00 +0700
tags: [flutter, dart, mobile, tutorial]
---

## Pengantar

Di artikel sebelumnya kita sudah berhasil install Flutter SDK dan setup IDE. Waktu kamu jalankan `flutter create` pertama kali, Flutter langsung menghasilkan puluhan file dan folder. Jangan panik! Hari ini kita akan membedah satu per satu — mana yang penting, mana yang bisa kamu abaikan, dan apa fungsi masing-masing.

---

## Membuat Project Baru

Buka terminal dan jalankan:

```bash
flutter create belajar_flutter
cd belajar_flutter
```

Setelah perintah selesai, buka folder `belajar_flutter` di VS Code atau Android Studio. Kamu akan melihat struktur seperti ini:

```
belajar_flutter/
├── .dart_tool/
├── .idea/
├── android/
├── ios/
├── lib/
│   └── main.dart
├── linux/
├── macos/
├── test/
│   └── widget_test.dart
├── web/
├── windows/
├── .gitignore
├── .metadata
├── analysis_options.yaml
├── pubspec.yaml
└── README.md
```

---

## Folder & File Penting (Wajib Tahu)

### 1. `lib/main.dart` — **Jantung Aplikasi**

Ini adalah **entry point** aplikasi Flutter-mu. Setiap kali Flutter dijalankan, ia akan mencari fungsi `main()` di file ini. Buka `main.dart`, kamu akan melihat kode default:

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const MyHomePage(title: 'Flutter Demo Home Page'),
    );
  }
}
```

> **Ingat:** `main()` adalah fungsi pertama yang dipanggil. Tanpa `main()`, aplikasi gak akan jalan.

### 2. `pubspec.yaml` — **Manager Dependencies & Aset**

File YAML ini adalah pusat konfigurasi project. Di sini kamu mendaftarkan:
- **Dependencies (package)**: library dari pub.dev
- **Aset (gambar, font, JSON)**: file statis yang dipakai aplikasi
- **Metadata project**: nama, deskripsi, versi

Contoh `pubspec.yaml`:

```yaml
name: belajar_flutter
description: Aplikasi Flutter pertamaku
version: 1.0.0+1

environment:
  sdk: ">=3.0.0 <4.0.0"

dependencies:
  flutter:
    sdk: flutter
  # Tambahkan package dari pub.dev di sini
  http: ^1.1.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^3.0.0

flutter:
  # Daftarkan aset di sini
  assets:
    - assets/images/
    - assets/data/sample.json
  # Font custom
  fonts:
    - family: Poppins
      fonts:
        - asset: assets/fonts/Poppins-Regular.ttf
```

> **Tips:** Setiap kali kamu mengubah `pubspec.yaml`, jalankan `flutter pub get` di terminal untuk mengunduh dependency baru.

### 3. `android/` & `ios/` — **Kode Native Platform**

Folder ini berisi project Android dan iOS native. Kamu **gak perlu sering-sering menyentuhnya** kecuali:
- Konfigurasi permission (kamera, lokasi, notifikasi)
- Setup Firebase (menambahkan `google-services.json` / `GoogleService-Info.plist`)
- Menambahkan kemampuan platform-specific (misalnya widget native)

### 4. `test/widget_test.dart` — **Testing**

Flutter otomatis menyediakan satu file test sederhana. Ini folder tempat kamu menulis **unit test** dan **widget test**. Sangat penting saat aplikasi mulai besar — kita bahas lebih detail di artikel lanjutan.

### 5. `analysis_options.yaml` — **Aturan Kode**

File ini mengatur static analysis — semacam "linter" yang membantu kamu menulis kode Dart yang rapi dan konsisten:

```yaml
include: package:flutter_lints/flutter.yaml

linter:
  rules:
    - prefer_const_constructors
    - prefer_const_literals_to_create_immutables
    - avoid_print
```

---

## Folder yang Bisa Diabaikan (Untuk Sekarang)

| Folder | Fungsi | Jangan Dihapus? |
|---|---|---|
| `.dart_tool/` | Cache internal Dart | Jangan dihapus manual, gitignore sudah menanganinya |
| `.idea/` | Konfigurasi IntelliJ/Android Studio | Bisa dihapus kalau pakai VS Code |
| `linux/`, `macos/`, `windows/` | Platform desktop | Biarkan saja, Flutter tangani otomatis |
| `web/` | Konfigurasi Flutter Web | Nanti berguna kalau deploy ke web |
| `.metadata` | Versi Flutter yang dipakai | Jangan diubah manual |

---

## `.gitignore` — File Wajib Saat Push ke Git

File ini sudah disediakan Flutter secara otomatis. Isinya mendaftarkan file/folder yang **tidak** akan di-push ke GitHub:

```
# Contoh isi .gitignore Flutter
.dart_tool/
.packages
build/
*.iml
.idea/
.metadata
```

Dengan `.gitignore`, project kamu tetap bersih dan tidak membawa file sampah saat di-commit.

---

## Tips Praktis

1. **Jangan hapus `pubspec.lock`** — file ini mencatat versi spesifik dari setiap dependency agar project tetap konsisten antar developer.
2. **Gunakan folder `lib/` untuk semua kode Dart** — Flutter mencari kode di sini. Jangan taruh kode aplikasi di root folder.
3. **Strukturkan `lib/` sejak awal** — walaupun masih kecil, biasakan bagi kode ke subfolder seperti `screens/`, `widgets/`, `models/`, `services/`. Nanti kita bahas arsitektur di artikel lanjutan.
4. **Selalu jalankan `flutter doctor`** setelah install package baru atau mengubah konfigurasi platform — untuk memastikan semuanya OK.

---

## Penutup

Sekarang kamu sudah paham isi setiap folder dan file di project Flutter. Dengan pemahaman ini, kamu gak akan bingung lagi saat membaca tutorial atau debugging error yang menyangkut path file.

**Selanjutnya:** Kita akan mulai ngoding beneran — mengenal **Widget Dasar: Text, Container, Row, dan Column** sebagai fondasi UI Flutter!

---

Sampai jumpa di artikel berikutnya. Coba sendiri struktur project yang udah kita bahas, lalu share ke sosial media dan tag **@ahsai001** — aku penasaran lihat progress belajarmu! 🚀
