---
layout: post
title: "Flutter #62: Theming Dinamis"
date: 2026-11-02 07:00:00 +0700
categories: [flutter]
tags: [flutter, dart, tutorial, indonesia]
---

## Pendahuluan
Theming dinamis memungkinkan aplikasi menyesuaikan tampilan (warna, font, ukuran) secara real‑time tanpa harus rebuild. Pada artikel ini kita akan membahas dua pendekatan populer di Flutter:

1. **ThemeMode + Provider** – mengganti tema global dengan state management.
2. **MediaQuery + AnimatedTheme** – memanfaatkan sistem bawaan Flutter untuk transisi halus.

Kita akan implementasikan contoh aplikasi sederhana yang memiliki tombol toggle antara *Light* dan *Dark* theme.

---

## 1. Persiapan Proyek
Buat proyek baru atau gunakan yang sudah ada:
```bash
flutter create theming_demo
cd theming_demo
```
Tambahkan dependency **provider** di `pubspec.yaml`:
```yaml
dependencies:
  flutter:
    sdk: flutter
  provider: ^6.0.5
```
Jalankan `flutter pub get`.
---

## 2. Membuat Model Tema dengan Provider
Buat file `lib/theme_provider.dart`:
```dart
import 'package:flutter/material.dart';

class ThemeProvider extends ChangeNotifier {
  ThemeMode _mode = ThemeMode.light;
  ThemeMode get mode => _mode;

  void toggleTheme() {
    _mode = _mode == ThemeMode.light ? ThemeMode.dark : ThemeMode.light;
    notifyListeners();
  }
}
```
Model ini menyimpan `ThemeMode` dan men-trigger UI tiap kali mode berubah.
---

## 3. Menghubungkan Provider di `main.dart`
```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'theme_provider.dart';

void main() {
  runApp(
    ChangeNotifierProvider(
      create: (_) => ThemeProvider(),
      child: const MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    final themeProvider = Provider.of<ThemeProvider>(context);
    return MaterialApp(
      title: 'Theming Dinamis',
      theme: ThemeData.light(),
      darkTheme: ThemeData.dark(),
      themeMode: themeProvider.mode,
      home: const HomePage(),
    );
  }
}

class HomePage extends StatelessWidget {
  const HomePage({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    final themeProvider = Provider.of<ThemeProvider>(context, listen: false);
    return Scaffold(
      appBar: AppBar(title: const Text('Theming Dinamis')),
      body: const Center(child: Text('Tekan tombol untuk ubah tema')),
      floatingActionButton: FloatingActionButton(
        onPressed: themeProvider.toggleTheme,
        child: const Icon(Icons.brightness_6),
      ),
    );
  }
}
```
Tekan tombol FAB, tema akan berubah secara otomatis dan transisi menggunakan animasi default Material.
---

## 4. Alternatif: `AnimatedTheme` + `MediaQuery`
Jika tidak ingin menambah dependency, gunakan `AnimatedTheme` bersama `MediaQuery` untuk mendeteksi preferensi sistem:
```dart
class MyApp extends StatefulWidget {
  const MyApp({Key? key}) : super(key: key);
  @override
  State<MyApp> createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  bool _isDark = false;
  @override
  Widget build(BuildContext context) {
    return AnimatedTheme(
      data: _isDark ? ThemeData.dark() : ThemeData.light(),
      duration: const Duration(milliseconds: 500),
      child: MaterialApp(
        home: Scaffold(
          appBar: AppBar(title: const Text('Theming Dinamis')),
          body: Center(
            child: SwitchListTile(
              title: const Text('Mode Gelap'),
              value: _isDark,
              onChanged: (val) => setState(() => _isDark = val),
            ),
          ),
        ),
      ),
    );
  }
}
```
`AnimatedTheme` memberikan animasi halus tiap pergantian tema.
---

## 5. Penutup
Dengan dua pendekatan di atas, Anda dapat menambahkan theming dinamis ke aplikasi Flutter apa pun. Pilih **Provider** bila tema harus di‑share lintas widget, atau **AnimatedTheme** bila hanya butuh toggle sederhana.

> **Coba sendiri!** Tambahkan tombol atau switch di aplikasi Anda, ubah warna utama, dan lihat hasilnya. Share ke sosial media dan tag @ahsai001.
