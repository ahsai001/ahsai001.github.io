---
layout: post
title: "Flutter #28: Custom Widget & Reusable Component — Bikin Widget Sendiri yang Dipakai Ulang"
date: 2026-09-06 07:00:00 +0700
tags: [flutter, dart, tutorial, widget, custom-widget, reusable, component, architecture]
description: "Pelajari cara membuat custom widget dan reusable component di Flutter agar kode lebih rapi, DRY, dan mudah dimaintenance."
---

# Flutter #28: Custom Widget & Reusable Component — Bikin Widget Sendiri yang Dipakai Ulang

Halo! Di artikel #27 kita sudah main-main sama Hero Animation dan Page Transition. Sekarang saatnya upgrade arsitektur kode kita — dari "tulis ulang terus" jadi **"bikin sekali, pakai di mana-mana."**

Custom widget adalah kunci bikin app yang scalable. Bayangkan kamu punya card profile, button style, atau badge yang muncul di 10 halaman berbeda. Kalau copy-paste di setiap halaman, pas mau ubah satu hal harus ubah 10 tempat. Makan hati. 😅

Hari ini kita belajar bikin widget sendiri yang reusable, clean, dan enak dipakai. Let's go!

---

## 1. Apa Itu Custom Widget?

Di Flutter, **semua yang kamu lihat di layar adalah widget**. `Text`, `Container`, `Row` — itu semua widget. Custom widget cuma artinya kamu bikin widget sendiri yang menggabungkan beberapa widget jadi satu komponen yang bisa dipakai ulang.

Dua cara bikin custom widget:

| Pendekatan | Kapan Pakai | Contoh |
|------------|-------------|--------|
| **StatelessWidget** | Widget tidak punya state/internal data | Badge, Avatar, InfoCard |
| **StatefulWidget** | Widget punya state yang bisa berubah | AnimatedButton, ToggleChip |

**Aturan praktis:** Mulai dari `StatelessWidget` dulu. Kalau butuh internal state baru upgrade ke `StatefulWidget`.

---

## 2. Contoh 1: Custom Info Card

Kita mulai dari sesuatu yang sering muncul di banyak halaman — info card dengan icon, title, dan subtitle.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Custom Widget Demo',
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const DashboardPage(),
    );
  }
}

// ========== CUSTOM WIDGET: InfoCard ==========
class InfoCard extends StatelessWidget {
  final IconData icon;
  final String title;
  final String subtitle;
  final Color color;

  const InfoCard({
    super.key,
    required this.icon,
    required this.title,
    required this.subtitle,
    this.color = Colors.blue,
  });

  @override
  Widget build(BuildContext context) {
    return Card(
      elevation: 2,
      shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),
      child: Padding(
        padding: const EdgeInsets.all(20),
        child: Row(
          children: [
            Container(
              width: 56,
              height: 56,
              decoration: BoxDecoration(
                color: color.withValues(alpha: 0.15),
                borderRadius: BorderRadius.circular(14),
              ),
              child: Icon(icon, color: color, size: 28),
            ),
            const SizedBox(width: 16),
            Expanded(
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    title,
                    style: const TextStyle(
                      fontSize: 16,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  const SizedBox(height: 4),
                  Text(
                    subtitle,
                    style: TextStyle(
                      fontSize: 14,
                      color: Colors.grey[600],
                    ),
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}

// ========== PAGE YANG PAKAI InfoCard ==========
class DashboardPage extends StatelessWidget {
  const DashboardPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Dashboard')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Tinggal panggil InfoCard — satu baris!
            const InfoCard(
              icon: Icons.people,
              title: 'Total Users',
              subtitle: '12,345 aktif bulan ini',
              color: Colors.blue,
            ),
            const SizedBox(height: 12),
            const InfoCard(
              icon: Icons.attach_money,
              title: 'Revenue',
              subtitle: 'Rp 45.000.000',
              color: Colors.green,
            ),
            const SizedBox(height: 12),
            const InfoCard(
              icon: Icons.star,
              title: 'Rating',
              subtitle: '4.8 dari 2,100 review',
              color: Colors.orange,
            ),
          ],
        ),
      ),
    );
  }
}
```

**Yang perlu diperhatikan:**

- **Constructor yang jelas** — Parameter wajib pakai `required`, optional pakai default value. Siapa pun yang pakai widget ini langsung tahu harus isi apa.
- **Properti `key`** — Selalu terima `super.key` di constructor. Ini standar Flutter untuk identifikasi widget.
- **Pisahkan custom widget dari halaman** — `InfoCard` didefinisikan terpisah dari `DashboardPage`. Inilah prinsip reusable: widget punya dunianya sendiri.

---

## 3. Contoh 2: Custom Button dengan Banyak Variasi

Kadang kamu butuh tombol dengan style berbeda-beda, tapi strukturnya sama. Solusinya: bikin satu widget dengan variasi parameter.

```dart
// ========== CUSTOM WIDGET: CustomButton ==========
enum ButtonStyle { primary, secondary, danger }

class CustomButton extends StatelessWidget {
  final String label;
  final VoidCallback onPressed;
  final ButtonStyle style;
  final IconData? icon;
  final bool isLoading;

  const CustomButton({
    super.key,
    required this.label,
    required this.onPressed,
    this.style = ButtonStyle.primary,
    this.icon,
    this.isLoading = false,
  });

  // Getter untuk warna berdasarkan style
  Color get _backgroundColor {
    switch (style) {
      case ButtonStyle.primary:
        return Colors.blue;
      case ButtonStyle.secondary:
        return Colors.grey;
      case ButtonStyle.danger:
        return Colors.red;
    }
  }

  Color get _textColor {
    switch (style) {
      case ButtonStyle.primary:
      case ButtonStyle.danger:
        return Colors.white;
      case ButtonStyle.secondary:
        return Colors.white;
    }
  }

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: double.infinity,
      child: ElevatedButton.icon(
        onPressed: isLoading ? null : onPressed,
        icon: isLoading
            ? const SizedBox(
                width: 20,
                height: 20,
                child: CircularProgressIndicator(
                  strokeWidth: 2,
                  color: Colors.white,
                ),
              )
            : Icon(icon ?? Icons.arrow_forward, color: _textColor),
        label: Text(
          label,
          style: TextStyle(
            color: _textColor,
            fontWeight: FontWeight.w600,
            fontSize: 16,
          ),
        ),
        style: ElevatedButton.styleFrom(
          backgroundColor: _backgroundColor,
          padding: const EdgeInsets.symmetric(vertical: 16, horizontal: 24),
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(12),
          ),
        ),
      ),
    );
  }
}

// ========== CONTOH PAKAI ==========
class FormPage extends StatelessWidget {
  const FormPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Form Demo')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Tombol primary dengan icon
            CustomButton(
              label: 'Simpan Data',
              icon: Icons.save,
              onPressed: () {
                ScaffoldMessenger.of(context).showSnackBar(
                  const SnackBar(content: Text('Tersimpan!')),
                );
              },
            ),
            const SizedBox(height: 12),
            // Tombol secondary
            CustomButton(
              label: 'Batal',
              style: ButtonStyle.secondary,
              icon: Icons.close,
              onPressed: () => Navigator.pop(context),
            ),
            const SizedBox(height: 12),
            // Tombol danger
            CustomButton(
              label: 'Hapus Akun',
              style: ButtonStyle.danger,
              icon: Icons.delete,
              onPressed: () {
                ScaffoldMessenger.of(context).showSnackBar(
                  const SnackBar(content: Text('Akun dihapus')),
                );
              },
            ),
            const SizedBox(height: 12),
            // Tombol dalam loading state
            const CustomButton(
              label: 'Loading...',
              isLoading: true,
              onPressed: null,
            ),
          ],
        ),
      ),
    );
  }
}
```

**Fitur keren dari custom widget ini:**

- **Enum untuk variasi** — `ButtonStyle.primary`, `.secondary`, `.danger` lebih aman dan readable daripada pakai string atau warna langsung.
- **Parameter optional dengan default** — `icon`, `isLoading`, `style` punya default value. Widget tetap bisa dipanggil dengan minimal parameter.
- **`VoidCallback onPressed`** — Tipe standar Flutter untuk callback tombol. Bisa `null` kalau disabled.
- **Loading state built-in** — Cukup set `isLoading: true`, tombol otomatis nonaktif dan tampilkan spinner.

---

## 4. Pola Organisasi File yang Bersih

Begitu custom widget mulai banyak, kamu butuh struktur folder yang rapi:

```
lib/
├── main.dart
├── models/
│   └── item_model.dart
├── pages/
│   ├── home_page.dart
│   └── detail_page.dart
└── widgets/              ← Custom widgets di sini
    ├── info_card.dart
    ├── custom_button.dart
    ├── user_avatar.dart
    └── badge_chip.dart
```

**Tips naming:**
- Folder `widgets/` khusus untuk komponen reusable
- Nama file sama dengan nama class: `info_card.dart` → class `InfoCard`
- Kalau widget spesifik untuk satu halaman, taruh di foldernya halaman itu
- Gunakan `export` untuk mempermudah import:

```dart
// lib/widgets/widgets.dart (barrel file)
export 'info_card.dart';
export 'custom_button.dart';
export 'user_avatar.dart';
export 'badge_chip.dart';
```

Lalu cukup satu import di halaman:
```dart
import '../widgets/widgets.dart';
```

---

## 5. Best Practice Custom Widget

| Prinsip | Penjelasan |
|---------|------------|
| **Satu tanggung jawab** | Widget hanya handle satu hal. Jangan bikin "god widget" yang bisa segalanya |
| **Constructor yang jelas** | Pakai `required` untuk yang wajib, default value untuk optional |
| **Terima `key`** | Selalu pakai `super.key` di constructor |
| **Jangan hardcode style** | Pakai `Theme.of(context)` atau parameter untuk warna/ukuran |
| **Naming konsisten** | Widget card → `*Card`, button → `*Button`, tile → `*Tile` |
| **Naming konsisten** | Widget card → `*Card`, button → `*Button`, tile → `*Tile` |
| **Export pattern** | Satu barrel file untuk semua widgets di folder yang sama |

---

## Recap Hari Ini

| Konsep | Fungsi | Kapan Pakai |
|--------|--------|-------------|
| **Custom StatelessWidget** | Widget tanpa state yang bisa dipakai ulang | Card, Badge, Avatar, Button |
| **Custom StatefulWidget** | Widget dengan state internal | Toggle, Form field, Animation |
| **Enum untuk variasi** | Tipe aman untuk variasi widget | Button styles, Card types |
| **Barrel export** | Satu import untuk semua widgets | Ketika widget sudah banyak |
| **Folder widgets/** | Organisasi file yang rapi | Project yang mulai besar |

**Yang harus diingat:**
- Mulai dari `StatelessWidget`, upgrade ke `StatefulWidget` kalau butuh state
- Properti wajib pakai `required`, optional pakai default value
- Pisahkan widget reusable ke folder `widgets/`
- Nama file = nama class = kegunaan widget
- Custom widget yang baik bikin kode pemanggil lebih pendek dan lebih jelas

---

Di artikel #29 kita akan masuk **FASE 3: ADVANCED** dengan **BLoC: Cubit & BlocProvider** — state management profesional yang dipakai di app production! 🏗️

---

Coba sendiri! Bikin library 3 custom widget sendiri (card, button, badge) dan pakai di satu halaman. Share hasilnya ke sosial media dan tag **@ahsai001**! 🚀
