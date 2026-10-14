---
layout: post
title: "Flutter #54: Flutter Web - Membuat Aplikasi Web dengan Flutter"
date: 2026-10-15 07:00:00 +0700
categories: [flutter]
tags: [flutter, dart, tutorial, indonesia]
---

Flutter kini tidak hanya untuk mobile, melainkan juga **web**. Pada artikel ini, kita akan membahas cara menyiapkan proyek Flutter untuk target web, menyesuaikan UI agar responsif, serta men-deploy ke GitHub Pages.

## 1. Persiapan Project Flutter

Jika belum ada project, buat dulu:

```bash
flutter create flutter_web_demo
cd flutter_web_demo
```

Pastikan channel `stable` dan versi terbaru:

```bash
flutter upgrade
flutter config --enable-web
```

Perintah di atas menambahkan dukungan web ke SDK dan men-generate folder `web/`.

## 2. Menjalankan di Browser

Jalankan `flutter run -d chrome` untuk melihat aplikasi di Chrome. Flutter akan meng-compile ke JavaScript (Dart → JS) dan menyajikan `index.html` di folder `build/web`.

```bash
flutter run -d chrome
```

Jika ingin melihat hasil build tanpa server, cukup buka `build/web/index.html` di browser.

## 3. Responsif UI dengan MediaQuery & LayoutBuilder

Agar UI menyesuaikan ukuran layar, gunakan `MediaQuery` atau `LayoutBuilder`. Contoh sederhana menampilkan dua kolom pada lebar > 600px, satu kolom di bawahnya.

```dart
import 'package:flutter/material.dart';

class ResponsiveHome extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        if (constraints.maxWidth > 600) {
          // Desktop / tablet layout
          return Row(
            children: [
              Expanded(child: _menu()),
              Expanded(child: _content()),
            ],
          );
        } else {
          // Mobile layout
          return Column(
            children: [_menu(), _content()],
          );
        }
      },
    );
  }

  Widget _menu() => Container(color: Colors.blue, height: 200, child: Center(child: Text('Menu')));
  Widget _content() => Container(color: Colors.green, height: 200, child: Center(child: Text('Konten')));
}
```

Gunakan widget ini sebagai `home` di `MaterialApp`.

## 4. Menyesuaikan Asset untuk Web

Folder `web/` berisi `index.html`, `favicon.png`, dan `manifest.json`. Jika menambah asset (gambar, font), letakkan di `assets/` dan daftarkan di `pubspec.yaml` seperti biasa. Flutter akan menyalin asset ke `build/web/assets`.

```yaml
flutter:
  assets:
    - assets/logo.png
```

## 5. Build untuk Produksi

```bash
flutter build web --release
```

Hasil berada di `build/web`. Semua file statis dapat di‑host di **GitHub Pages**.

## 6. Deploy ke GitHub Pages dengan Jekyll

1. Copy folder `build/web` ke repository root (atau folder `docs/`).
2. Pastikan file `_config.yml` memiliki `baseurl: ""` dan `url: "https://ahsai.my.id"`.
3. Commit & push.

Contoh perintah (jalankan di root repo):

```bash
rm -rf docs && cp -r build/web docs
git add docs
git commit -m "feat: deploy Flutter Web ke GitHub Pages"
git push origin master
```

Setelah beberapa menit, aplikasi dapat diakses di `https://ahsai.my.id/`.

## 7. Tips Tambahan

- **Lazy loading**: gunakan `deferred as` untuk memuat modul besar hanya bila diperlukan.
- **SEO**: tambahkan meta tag di `web/index.html` (title, description, og:image) karena konten masih JavaScript.
- **CORS**: pastikan API yang dipanggil mengizinkan origin `https://ahsai.my.id`.

## 8. Ringkasan

Flutter Web memungkinkan menulis satu basis kode Dart untuk Android, iOS, dan Web. Dengan `MediaQuery`, `LayoutBuilder`, dan build release, Anda dapat menghasilkan aplikasi responsif yang mudah di‑deploy ke GitHub Pages.

---

Coba sendiri! Share ke sosial media dan tag @ahsai001
