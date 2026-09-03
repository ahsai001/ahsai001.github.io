---
layout: post
title: "Flutter #26: AlertDialog, BottomSheet & DatePicker — Dialog & Input Interaktif"
date: 2026-09-04 07:00:00 +0700
tags: [flutter, dart, tutorial, dialog, bottomsheet, datepicker, UI]
description: "Pelajari cara menampilkan AlertDialog, BottomSheet, dan DatePicker di Flutter untuk membuat UI interaktif yang menangani input dan konfirmasi user."
---

# Flutter #26: AlertDialog, BottomSheet & DatePicker — Dialog & Input Interaktif

Halo! Di artikel #26 ini kita akan belajar tentang **dialog dan input interaktif** di Flutter. Topik ini penting banget karena hampir semua app butuh konfirmasi (hapus data?), pemilihan tanggal (jadwal appointment?), atau pilihan opsi (BottomSheet). 

Kita akan cover tiga komponen utama: **AlertDialog**, **BottomSheet**, dan **DatePicker**. Let's go!

---

## 1. AlertDialog — Konfirmasi & Info Penting

`AlertDialog` adalah dialog popup yang muncul di tengah layar. Cocok untuk konfirmasi aksi (hapus, logout) atau menampilkan info penting.

### Dasar AlertDialog

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Dialog Demo',
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const DialogDemoPage(),
    );
  }
}

class DialogDemoPage extends StatelessWidget {
  const DialogDemoPage({super.key});

  void _showAlertDialog(BuildContext context) {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        icon: const Icon(Icons.warning_amber_rounded, color: Colors.orange, size: 48),
        title: const Text('Hapus Item?'),
        content: const Text(
          'Apakah kamu yakin ingin menghapus item ini? Tindakan ini tidak dapat dibatalkan.',
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('Batal'),
          ),
          FilledButton(
            onPressed: () {
              Navigator.pop(context);
              ScaffoldMessenger.of(context).showSnackBar(
                const SnackBar(content: Text('Item berhasil dihapus!')),
              );
            },
            child: const Text('Hapus'),
          ),
        ],
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('AlertDialog Demo')),
      body: Center(
        child: FilledButton.icon(
          onPressed: () => _showAlertDialog(context),
          icon: const Icon(Icons.delete_outline),
          label: const Text('Hapus Item'),
        ),
      ),
    );
  }
}
```

**Yang perlu diperhatikan:**

- **`showDialog()`** — fungsi bawaan Flutter untuk menampilkan dialog
- **`actions`** — list tombol di bagian bawah dialog (Batal, Hapus, dll)
- **`Navigator.pop(context)`** — untuk menutup dialog
- **`icon`** — opsional, icon besar di atas judul (Material 3 style)

---

## 2. Custom Dialog — Konten Lebih Kaya

Kadang `AlertDialog` bawaan kurang fleksibel. Kamu bisa bikin dialog sendiri pakai `Dialog` widget:

```dart
void _showCustomDialog(BuildContext context) {
  showDialog(
    context: context,
    builder: (context) => Dialog(
      shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(20)),
      child: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            const CircleAvatar(
              radius: 30,
              backgroundColor: Colors.green,
              child: Icon(Icons.check, color: Colors.white, size: 30),
            ),
            const SizedBox(height: 16),
            const Text(
              'Pembayaran Berhasil!',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            const Text(
              'Pesanan #1234 telah dikonfirmasi. Kam akan menerima email konfirmasi sebentar lagi.',
              textAlign: TextAlign.center,
              style: TextStyle(color: Colors.grey),
            ),
            const SizedBox(height: 20),
            SizedBox(
              width: double.infinity,
              child: FilledButton(
                onPressed: () => Navigator.pop(context),
                child: const Text('OK'),
              ),
            ),
          ],
        ),
      ),
    ),
  );
}
```

**Tip:** Pakai `mainAxisSize: MainAxisSize.min` agar dialog mengencang mengikuti kontennya, tidak melebar penuh layar.

---

## 3. BottomSheet — Opsi dari Bawah

`BottomSheet` muncul dari bawah layar. Cocok untuk action sheet (pilih kamera/galeri), filter, atau opsi yang banyak.

### Persistent vs Modal BottomSheet

- **Modal BottomSheet** — overlay, tap outside to dismiss, ada overlay gelap
- **Persistent BottomSheet** — bagian dari halaman, tidak ada overlay

Kita fokus ke **Modal BottomSheet** yang paling sering dipakai:

```dart
void _showBottomSheet(BuildContext context) {
  showModalBottomSheet(
    context: context,
    shape: const RoundedRectangleBorder(
      borderRadius: BorderRadius.vertical(top: Radius.circular(20)),
    ),
    builder: (context) => SafeArea(
      child: Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          // Handle bar
          Container(
            margin: const EdgeInsets.only(top: 12),
            width: 40,
            height: 4,
            decoration: BoxDecoration(
              color: Colors.grey[300],
              borderRadius: BorderRadius.circular(2),
            ),
          ),
          const SizedBox(height: 16),
          const Text(
            'Pilih Sumber Foto',
            style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
          ),
          const SizedBox(height: 8),
          ListTile(
            leading: const Icon(Icons.camera_alt),
            title: const Text('Kamera'),
            onTap: () {
              Navigator.pop(context);
              ScaffoldMessenger.of(context).showSnackBar(
                const SnackBar(content: Text('Membuka kamera...')),
              );
            },
          ),
          ListTile(
            leading: const Icon(Icons.photo_library),
            title: const Text('Galeri'),
            onTap: () {
              Navigator.pop(context);
              ScaffoldMessenger.of(context).showSnackBar(
                const SnackBar(content: Text('Membuka galeri...')),
              );
            },
          ),
          ListTile(
            leading: const Icon(Icons.folder),
            title: const Text('File Manager'),
            onTap: () => Navigator.pop(context),
          ),
          const SizedBox(height: 8),
        ],
      ),
    ),
  );
}
```

**Kenapa pakai `SafeArea`?** Karena di device dengan notch/home indicator, BottomSheet bisa tertutup. `SafeArea` memastikan konten tidak tertimpa area system.

---

## 4. DatePicker — Pilih Tanggal

`showDatePicker()` adalah cara paling praktis untuk input tanggal di Flutter:

```dart
class DatePickerDemo extends StatefulWidget {
  const DatePickerDemo({super.key});

  @override
  State<DatePickerDemo> createState() => _DatePickerDemoState();
}

class _DatePickerDemoState extends State<DatePickerDemo> {
  DateTime? _selectedDate;

  Future<void> _pickDate(BuildContext context) async {
    final picked = await showDatePicker(
      context: context,
      initialDate: _selectedDate ?? DateTime.now(),
      firstDate: DateTime(2020),
      lastDate: DateTime(2030),
      helpText: 'PILIH TANGGAL LAHIR',
      cancelText: 'BATAL',
      confirmText: 'PILIH',
      locale: const Locale('id', 'ID'), // Butuh localizationsDelegate
    );

    if (picked != null && picked != _selectedDate) {
      setState(() => _selectedDate = picked);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('DatePicker Demo')),
      body: Padding(
        padding: const EdgeInsets.all(20),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: [
            OutlinedButton.icon(
              onPressed: () => _pickDate(context),
              icon: const Icon(Icons.calendar_today),
              label: Text(
                _selectedDate == null
                    ? 'Pilih Tanggal Lahir'
                    : 'Tanggal: ${_selectedDate!.day}/${_selectedDate!.month}/${_selectedDate!.year}',
              ),
            ),
            const SizedBox(height: 20),
            if (_selectedDate != null)
              Card(
                child: Padding(
                  padding: const EdgeInsets.all(16),
                  child: Column(
                    children: [
                      const Icon(Icons.cake, size: 48, color: Colors.pink),
                      const SizedBox(height: 8),
                      Text(
                        'Tanggal Lahir: ${_selectedDate!.day}/${_selectedDate!.month}/${_selectedDate!.year}',
                        style: const TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
                      ),
                    ],
                  ),
                ),
              ),
          ],
        ),
      ),
    );
  }
}
```

**Parameter penting DatePicker:**

| Parameter | Fungsi |
|-----------|--------|
| `initialDate` | Tanggal yang muncul pertama kali |
| `firstDate` | Batas minimum tanggal yang bisa dipilih |
| `lastDate` | Batas maksimum tanggal |
| `helpText` | Judul di bagian atas date picker |
| `locale` | Lokalisasi (bahasa Indonesia) |

---

## 5. Kombinasi: Konfirmasi + Tanggal

Berikut contoh nyata — form booking dengan DatePicker + konfirmasi AlertDialog:

```dart
class BookingPage extends StatefulWidget {
  const BookingPage({super.key});

  @override
  State<BookingPage> createState() => _BookingPageState();
}

class _BookingPageState extends State<BookingPage> {
  DateTime? _bookingDate;

  Future<void> _pickDate() async {
    final picked = await showDatePicker(
      context: context,
      initialDate: DateTime.now(),
      firstDate: DateTime.now(),
      lastDate: DateTime.now().add(const Duration(days: 365)),
    );
    if (picked != null) setState(() => _bookingDate = picked);
  }

  void _confirmBooking() {
    if (_bookingDate == null) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('Pilih tanggal dulu!')),
      );
      return;
    }

    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('Konfirmasi Booking'),
        content: Text(
          'Booking untuk tanggal ${_bookingDate!.day}/${_bookingDate!.month}/${_bookingDate!.year}?\n\nPastikan tanggal sudah benar.',
        ),
        actions: [
          TextButton(onPressed: () => Navigator.pop(context), child: const Text('Ubah')),
          FilledButton(
            onPressed: () {
              Navigator.pop(context);
              ScaffoldMessenger.of(context).showSnackBar(
                SnackBar(content: Text('Booking dikonfirmasi! Tanggal: ${_bookingDate!.day}/${_bookingDate!.month}/${_bookingDate!.year}')),
              );
            },
            child: const Text('Konfirmasi'),
          ),
        ],
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Booking')),
      body: Padding(
        padding: const EdgeInsets.all(20),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: [
            OutlinedButton.icon(
              onPressed: _pickDate,
              icon: const Icon(Icons.calendar_today),
              label: Text(_bookingDate == null
                  ? 'Pilih Tanggal Booking'
                  : '${_bookingDate!.day}/${_bookingDate!.month}/${_bookingDate!.year}'),
            ),
            const SizedBox(height: 20),
            FilledButton(onPressed: _confirmBooking, child: const Text('Booking Sekarang')),
          ],
        ),
      ),
    );
  }
}
```

---

## Recap Hari Ini

| Komponen | Fungsi | Fungsi Utama |
|----------|--------|--------------|
| **AlertDialog** | Dialog popup tengah layar | Konfirmasi, info penting |
| **Dialog** | Dialog custom bebas layout | Konten dialog lebih kaya |
| **BottomSheet** | Panel dari bawah layar | Action sheet, pilihan opsi |
| **DatePicker** | Pemilih tanggal | Input tanggal dari user |

**Yang harus diingat:**
- Selalu tutup dialog/sheet pakai `Navigator.pop(context)`
- Pakai `SafeArea` di BottomSheet untuk device modern
- DatePicker wajib set `firstDate` dan `lastDate`
- Kombinasikan beberapa dialog untuk flow yang lebih natural

---

Di artikel #27 kita akan belajar tentang **Hero Animation & Page Transition** — bikin transisi antar halaman yang smooth dan eye-catching! 🎬

---

Coba sendiri! Bikin app booking sederhana yang pakai DatePicker + AlertDialog konfirmasi. Share hasilnya ke sosial media dan tag **@ahsai001**! 🚀
