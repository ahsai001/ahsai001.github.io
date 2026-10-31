---
layout: post
title: "Flutter #61: Aksesibilitas untuk Semua Pengguna"
date: 2026-11-01 07:00:00 +0700
categories: [flutter]
tags: [flutter, dart, tutorial, indonesia, aksesibilitas]
---

# Flutter #61 – Aksesibilitas untuk Semua Pengguna

Aksesibilitas (accessibility) memastikan aplikasi dapat dipakai oleh semua orang, termasuk pengguna dengan keterbatasan penglihatan, pendengaran, atau motorik. Flutter sudah menyediakan widget‑widget dan properti yang memudahkan kita menambahkan label, ukuran tap target, dan navigasi keyboard. Pada artikel ini kita bahas tiga hal utama:

1. **Semantics** – menambahkan deskripsi bagi screen reader.
2. **Tap target yang cukup besar** – memakai `MaterialTapTargetSize` dan `SizedBox`.
3. **Navigasi Keyboard** – mengaktifkan fokus dan aksi via `FocusNode`.

## 1. Semantics

`Semantics` widget membungkus widget lain dan memberi informasi ekstra ke pembaca layar (TalkBack, VoiceOver). Contoh sederhana menambahkan label pada ikon tombol:

```dart
import 'package:flutter/material.dart';

class IconButtonWithSemantics extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Semantics(
      label: 'Tambah item ke keranjang',
      button: true,
      child: IconButton(
        icon: Icon(Icons.add_shopping_cart),
        onPressed: () {},
      ),
    );
  }
}
```

Jika `label` tidak diberikan, screen reader akan membaca nama ikon (`add_shopping_cart`) yang tidak selalu jelas. Dengan `Semantics` kita dapat menuliskan kalimat yang menggambarkan aksi.

### Menggunakan `excludeSemantics`

Kadang widget sudah mengelola semantics secara otomatis, misalnya `TextButton`. Jika ingin menimpa, gunakan `excludeSemantics: true` di dalam `Semantics`.

## 2. Tap Target yang Memadai

Google merekomendasikan ukuran target minimal **48 dp**. Kita dapat mengatur ukuran dengan membungkus widget dalam `SizedBox` atau `Container` dengan `constraints`. Contoh:

```dart
class LargeTapButton extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: 56,
      height: 56,
      child: ElevatedButton(
        style: ElevatedButton.styleFrom(
          shape: CircleBorder(),
          padding: EdgeInsets.zero,
        ),
        onPressed: () {},
        child: Icon(Icons.edit),
      ),
    );
  }
}
```

Ukuran 56 dp memberi ruang ekstra di sekitar ikon sehingga jari tidak mudah melewatkan tombol.

## 3. Navigasi Keyboard & Fokus

Untuk aplikasi desktop atau web, pengguna dapat menavigasi UI lewat tab. Setiap widget yang dapat difokuskan harus memiliki `FocusNode`. Contoh menambahkan fokus pada `TextField` dan menangani `Enter`:

```dart
class FocusDemo extends StatefulWidget {
  @override
  _FocusDemoState createState() => _FocusDemoState();
}

class _FocusDemoState extends State<FocusDemo> {
  final FocusNode _focusNode = FocusNode();
  final TextEditingController _controller = TextEditingController();

  @override
  void initState() {
    super.initState();
    // otomatis fokus saat halaman dibuka
    WidgetsBinding.instance.addPostFrameCallback((_) {
      _focusNode.requestFocus();
    });
  }

  @override
  void dispose() {
    _focusNode.dispose();
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return TextField(
      focusNode: _focusNode,
      controller: _controller,
      onSubmitted: (value) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('Kamu mengetik: $value')),
        );
      },
      decoration: InputDecoration(labelText: 'Masukkan teks'),
    );
  }
}
```

`onSubmitted` dipicu ketika pengguna menekan **Enter** atau tombol “Done” pada keyboard virtual.

## Tips Praktis

- **Gunakan `debugPaintSizeEnabled = true`** di `main()` untuk memvisualisasikan batas ukuran tap target.
- **Uji dengan TalkBack / VoiceOver** pada perangkat Android atau iOS.
- **Periksa kontras warna** lewat tool seperti `flutter_contrast_checker`.
- **Berikan alternatif teks** pada gambar menggunakan `Image.asset(..., semanticLabel: 'Deskripsi gambar')`.

## Kesimpulan

Aksesibilitas bukan fitur tambahan, melainkan keharusan. Dengan menambahkan `Semantics`, memastikan ukuran tap target, dan mendukung navigasi keyboard, aplikasi Flutter Anda menjadi inklusif untuk semua pengguna.

---

Coba sendiri! Share ke sosial media dan tag @ahsai001
