---
layout: post
title: "Flutter #10: Validasi Form & SnackBar Feedback"
date: 2026-08-12 07:00:00 +0700
tags: [flutter, dart, form, validasi, snackbar, tutorial, indonesia, pemrograman]
---

Kemarin kita udah bikin `Form` + `TextFormField` dan sempat menyinggung `validator`. Tapi validasi yang bener itu bukan cuma "field nggak boleh kosong". User itu kreatif — mereka bakal masukin email tanpa `@`, password cuma 3 huruf, atau konfirmasi password yang beda sama password aslinya. Tugas kamu: **jangan percaya input user sampai lolos validasi**, dan **kasih feedback yang jelas** kalau ada yang salah.

Hari ini kita bongkar tuntas validasi form + cara ngasih feedback lewat **SnackBar** yang user-friendly.

---

## 1. Cara Kerja `validator`

Di `TextFormField`, ada parameter `validator` yang berbentuk function:

```dart
validator: (value) {
  if (value == null || value.trim().isEmpty) {
    return 'Nggak boleh kosong!';
  }
  return null; // ⬅️ null = VALID, nggak ada error
}
```

**Aturan emas validator:**
- Return `null` → field dianggap **valid** (nggak ada pesan error).
- Return `String` → field dianggap **invalid**, string itu ditampilkan merah di bawah field.
- `value` bisa `null` kalau field dikosongkan — selalu cek `null` dulu!

---

## 2. Kumpulan Validator yang Wajib Kamu Punya

Daripada nulis validasi berulang-ulang, bikin kumpulan validator reusable:

```dart
class Validators {
  // ✅ Wajib diisi
  static String? required(String? value, [String? label]) {
    if (value == null || value.trim().isEmpty) {
      return '${label ?? 'Field ini'} nggak boleh kosong!';
    }
    return null;
  }

  // ✅ Email pakai regex
  static String? email(String? value) {
    final v = value?.trim() ?? '';
    if (v.isEmpty) return 'Email nggak boleh kosong!';
    final regex = RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$');
    if (!regex.hasMatch(v)) return 'Format email nggak valid!';
    return null;
  }

  // ✅ Minimal panjang
  static String? minLength(String? value, int min, [String? label]) {
    final v = value?.trim() ?? '';
    if (v.isEmpty) return '${label ?? 'Field ini'} nggak boleh kosong!';
    if (v.length < min) return 'Minimal $min karakter!';
    return null;
  }

  // ✅ Konfirmasi password (butuh nilai field lain)
  static String? match(String? value, String other, [String? label]) {
    if (value != other) return '${label ?? 'Konfirmasi'} nggak cocok!';
    return null;
  }
}
```

Sekarang form kamu bersih banget:

```dart
TextFormField(
  decoration: const InputDecoration(labelText: 'Email'),
  validator: Validators.email,
),
```

---

## 3. `autovalidateMode` — Validasi Saat Mengetik

Default-nya, validator cuma jalan saat `.validate()` dipanggil (biasanya saat tombol submit ditekan). Kadang user pengen tau error-nya langsung tanpa nunggu tekan tombol. Di situlah `autovalidateMode` berperan:

```dart
TextFormField(
  autovalidateMode: AutovalidateMode.onUserInteraction,
  validator: Validators.email,
),
```

**Pilihan mode:**
- `AutovalidateMode.disabled` — default, cuma validasi saat submit
- `AutovalidateMode.onUserInteraction` — validasi tiap kali user berinteraksi (nulis/hapus) — **paling recommended**
- `AutovalidateMode.always` — validasi terus-menerus, bahkan sejak pertama build

> **Pro tip:** Jangan pakai `always` kecuali memang perlu. Validasi yang nyala sebelum user sempat ngetik bikin form berasa "marah-marah" ke user.

---

## 4. SnackBar — Feedback yang Elegan

Setelah validasi lolos/gagal, kasih tau user hasilnya. `SnackBar` adalah notifikasi kecil yang muncul di bawah layar — nggak ngeblok interaksi kayak `AlertDialog`.

```dart
// Sukses ✅
ScaffoldMessenger.of(context).showSnackBar(
  SnackBar(
    content: Text('✅ Registrasi berhasil! Selamat datang!'),
    backgroundColor: Colors.green,
    behavior: SnackBarBehavior.floating,
    shape: RoundedRectangleBorder(
      borderRadius: BorderRadius.circular(10),
    ),
    duration: const Duration(seconds: 3),
  ),
);

// Gagal ❌
ScaffoldMessenger.of(context).showSnackBar(
  const SnackBar(
    content: Text('❌ Ada field yang masih salah, cek lagi ya!'),
    backgroundColor: Colors.red,
  ),
);
```

**Kenapa pakai `ScaffoldMessenger.of(context)`?** Karena kalau pakai `Scaffold.of(context)` dan halaman sudah pindah/`Scaffold` udah ke-dispose, SnackBar bakal error. `ScaffoldMessenger` lebih aman — dia dikelola di level `MaterialApp`, jadi nggak keilangan referensi.

---

## 5. Contoh Lengkap: Form Registrasi dengan Validasi Kuat

Gabungin semua jadi satu form register yang "production-ready":

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MaterialApp(home: RegisterScreen()));

class RegisterScreen extends StatefulWidget {
  const RegisterScreen({super.key});
  @override
  State<RegisterScreen> createState() => _RegisterScreenState();
}

class _RegisterScreenState extends State<RegisterScreen> {
  final _formKey = GlobalKey<FormState>();
  final _emailCtrl = TextEditingController();
  final _passCtrl = TextEditingController();
  final _confirmCtrl = TextEditingController();

  @override
  void dispose() {
    _emailCtrl.dispose();
    _passCtrl.dispose();
    _confirmCtrl.dispose();
    super.dispose();
  }

  void _submit() {
    // 1️⃣ Validasi semua field
    if (!_formKey.currentState!.validate()) {
      // Feedback gagal
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(
          content: Text('❌ Cek lagi field yang merah di atas!'),
          backgroundColor: Colors.red,
        ),
      );
      return;
    }

    // 2️⃣ Kalau lolos, save + proses
    _formKey.currentState!.save();
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text('✅ Akun ${_emailCtrl.text} berhasil dibuat!'),
        backgroundColor: Colors.green,
        behavior: SnackBarBehavior.floating,
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('🔐 Register')),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Form(
          key: _formKey,
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              TextFormField(
                controller: _emailCtrl,
                keyboardType: TextInputType.emailAddress,
                autovalidateMode: AutovalidateMode.onUserInteraction,
                decoration: const InputDecoration(
                  labelText: 'Email',
                  icon: Icon(Icons.email),
                  border: OutlineInputBorder(),
                ),
                validator: Validators.email,
              ),
              const SizedBox(height: 16),
              TextFormField(
                controller: _passCtrl,
                obscureText: true,
                decoration: const InputDecoration(
                  labelText: 'Password',
                  icon: Icon(Icons.lock),
                  border: OutlineInputBorder(),
                ),
                validator: (v) => Validators.minLength(v, 6, 'Password'),
              ),
              const SizedBox(height: 16),
              TextFormField(
                controller: _confirmCtrl,
                obscureText: true,
                autovalidateMode: AutovalidateMode.onUserInteraction,
                decoration: const InputDecoration(
                  labelText: 'Konfirmasi Password',
                  icon: Icon(Icons.lock_outline),
                  border: OutlineInputBorder(),
                ),
                validator: (v) =>
                    Validators.match(v, _passCtrl.text, 'Konfirmasi password'),
              ),
              const SizedBox(height: 24),
              ElevatedButton.icon(
                onPressed: _submit,
                icon: const Icon(Icons.check_circle),
                label: const Padding(
                  padding: EdgeInsets.symmetric(vertical: 12),
                  child: Text('Daftar'),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Jalankan dan coba:**
1. Tekan "Daftar" tanpa isi apa-apa → semua field merah + SnackBar merah muncul.
2. Isi email `abc` → langsung muncul "Format email nggak valid!" saat ngetik (karena `onUserInteraction`).
3. Password `123`, konfirmasi `456` → error "Konfirmasi password nggak cocok!".
4. Isi semua dengan benar → SnackBar hijau "berhasil dibuat!".

---

## 6. Styling Pesan Error

Pesan error default Flutter udah bagus, tapi kamu bisa ubah:

```dart
TextFormField(
  decoration: InputDecoration(
    labelText: 'Email',
    errorStyle: const TextStyle(color: Colors.orange, fontSize: 12),
    errorBorder: OutlineInputBorder(
      borderSide: const BorderSide(color: Colors.orange, width: 2),
      borderRadius: BorderRadius.circular(8),
    ),
    focusedErrorBorder: OutlineInputBorder(
      borderSide: const BorderSide(color: Colors.red, width: 2),
      borderRadius: BorderRadius.circular(8),
    ),
  ),
  validator: Validators.email,
)
```

- `errorStyle` — styling teks error
- `errorBorder` — border field saat error (nggak fokus)
- `focusedErrorBorder` — border field saat error DAN sedang fokus

---

## Ringkasan

- **`validator`** return `null` (valid) atau `String` (pesan error)
- **Bikin validator reusable** — jangan nulis ulang tiap form
- **`autovalidateMode.onUserInteraction`** — validasi real-time tanpa bikin user kesel
- **`SnackBar`** — feedback elegan, pakai `ScaffoldMessenger.of(context)` biar aman
- **Pola baku**: `validate()` → gagal (SnackBar merah) / lolos (SnackBar hijau + proses data)

Validasi + feedback yang bener bikin aplikasimu berasa **profesional dan peduli user**. Ini skill yang bakal kepakai di semua project nyata — dari login sampai form checkout.

Besok kita lanjut ke **Asset & Image — Gambar Lokal & Network**. Siap-siap bikin tampilan makin hidup! 🎨

**Coba sendiri! Bikin form login (email + password) dengan validasi lengkap dan SnackBar. Share hasilnya ke sosial media dan tag @ahsai001!**
