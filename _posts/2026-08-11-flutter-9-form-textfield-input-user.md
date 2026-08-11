---
layout: post
title: "Flutter #9: Form & TextField — Input dari User"
date: 2026-08-11 07:00:00 +0700
tags: [flutter, dart, form, textfield, input, tutorial, indonesia, pemrograman]
---

Aplikasi yang cuma bisa tampil doang itu kayak TV tanpa remote — cuma bisa dilihat, nggak bisa diajak interaksi. Realitasnya, hampir semua aplikasi butuh **input dari user**: login, register, search, chat, notes, form order, dll. Di Flutter, senjata utamanya ada dua: **TextField** (input satuan) dan **Form** (kumpulan input dengan validasi).

Hari ini kita bahas cara menerima input dari user dengan benar. Mulai dari TextField sederhana sampai Form multi-field yang siap divalidasi (validasi lengkapnya besok, ya!).

---

## 1. TextField — Input Satuan

`TextField` adalah widget paling dasar untuk menerima teks dari user. Mirip `<input>` di HTML. Contoh paling sederhana:

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MaterialApp(home: InputSederhana()));

class InputSederhana extends StatefulWidget {
  const InputSederhana({super.key});

  @override
  State<InputSederhana> createState() => _InputSederhanaState();
}

class _InputSederhanaState extends State<InputSederhana> {
  // 🔑 Controller = jembatan antara TextField dan logic kita
  final TextEditingController _controller = TextEditingController();

  @override
  void dispose() {
    _controller.dispose(); // WAJIB: bersihin controller biar nggak bocor memori
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('📝 Input Sederhana')),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          children: [
            TextField(
              controller: _controller,
              decoration: const InputDecoration(
                labelText: 'Nama kamu',
                hintText: 'Ketik di sini...',
                border: OutlineInputBorder(),
              ),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () {
                // 📍 Ambil teks dari controller
                final nama = _controller.text;
                if (nama.isNotEmpty) {
                  ScaffoldMessenger.of(context).showSnackBar(
                    SnackBar(content: Text('Halo, $nama! 👋')),
                  );
                }
              },
              child: const Text('Sapa Aku!'),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Yang wajib kamu ingat:**
- `TextEditingController` — jembatan baca/tulis ke TextField
- `dispose()` — selalu panggil di `dispose()` state, kalau nggak → memory leak
- `decoration` — styling placeholder, label, border, icon, dll

### TextField Properties yang Sering Dipakai

```dart
TextField(
  controller: _controller,
  obscureText: true,          // password mode (••••••)
  maxLines: 5,                // textarea (default 1 = single line)
  keyboardType: TextInputType.emailAddress,  // keyboard @ muncul
  textInputAction: TextInputAction.done,     // tombol "Done" di keyboard
  onChanged: (value) {        // real-time listener, setiap ketikan
    print('User mengetik: $value');
  },
  onSubmitted: (value) {     // user tekan Enter/Done
    print('User submit: $value');
  },
  decoration: const InputDecoration(
    icon: Icon(Icons.person),    // icon di kiri
    labelText: 'Email',
    hintText: 'contoh@email.com',
    helperText: 'Masukkan email aktif',
    prefixText: '@',             // teks sebelum input
    suffixIcon: Icon(Icons.check),
  ),
)
```

---

## 2. Form Widget — Multi-Field Terstruktur

Kalau cuma satu input, `TextField` cukup. Tapi begitu kamu punya 3-4 field (misal: nama, email, password, alamat), lebih rapi pakai widget `Form`.

`Form` adalah **container** yang membungkus beberapa `TextFormField` (adiknya TextField). `Form` punya `GlobalKey<FormState>` yang bisa kamu pakai untuk **validasi, reset, atau save** semua field sekaligus.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MaterialApp(home: FormRegisterScreen()));

class FormRegisterScreen extends StatefulWidget {
  const FormRegisterScreen({super.key});

  @override
  State<FormRegisterScreen> createState() => _FormRegisterScreenState();
}

class _FormRegisterScreenState extends State<FormRegisterScreen> {
  // 🔑 Kunci form — wajib untuk akses FormState
  final _formKey = GlobalKey<FormState>();

  // 🗃️ Controller buat ambil nilai user
  final _namaCtrl = TextEditingController();
  final _emailCtrl = TextEditingController();
  final _passCtrl = TextEditingController();

  @override
  void dispose() {
    _namaCtrl.dispose();
    _emailCtrl.dispose();
    _passCtrl.dispose();
    super.dispose();
  }

  void _submitForm() {
    // Cek validasi semua field dulu
    if (_formKey.currentState!.validate()) {
      // Ambil data
      final nama = _namaCtrl.text;
      final email = _emailCtrl.text;

      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text('✅ Akun $nama ($email) berhasil dibuat!'),
          backgroundColor: Colors.green,
        ),
      );

      // Reset form setelah submit
      _formKey.currentState!.reset();
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('📋 Form Registrasi')),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Form(
          key: _formKey,
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              // === FIELD 1: NAMA ===
              TextFormField(
                controller: _namaCtrl,
                decoration: const InputDecoration(
                  labelText: 'Nama Lengkap',
                  icon: Icon(Icons.person),
                  border: OutlineInputBorder(),
                ),
                validator: (value) {
                  if (value == null || value.trim().isEmpty) {
                    return '❌ Nama nggak boleh kosong!';
                  }
                  return null;
                },
              ),

              const SizedBox(height: 16),

              // === FIELD 2: EMAIL ===
              TextFormField(
                controller: _emailCtrl,
                keyboardType: TextInputType.emailAddress,
                decoration: const InputDecoration(
                  labelText: 'Email',
                  icon: Icon(Icons.email),
                  border: OutlineInputBorder(),
                ),
                validator: (value) {
                  if (value == null || value.trim().isEmpty) {
                    return '❌ Email nggak boleh kosong!';
                  }
                  if (!value.contains('@') || !value.contains('.')) {
                    return '❌ Format email nggak valid!';
                  }
                  return null;
                },
              ),

              const SizedBox(height: 16),

              // === FIELD 3: PASSWORD ===
              TextFormField(
                controller: _passCtrl,
                obscureText: true,
                decoration: const InputDecoration(
                  labelText: 'Password',
                  icon: Icon(Icons.lock),
                  border: OutlineInputBorder(),
                ),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return '❌ Password nggak boleh kosong!';
                  }
                  if (value.length < 6) {
                    return '❌ Password minimal 6 karakter!';
                  }
                  return null;
                },
              ),

              const SizedBox(height: 24),

              // === TOMBOL SUBMIT ===
              ElevatedButton.icon(
                onPressed: _submitForm,
                icon: const Icon(Icons.check_circle),
                label: const Text('Daftar'),
                style: ElevatedButton.styleFrom(
                  padding: const EdgeInsets.symmetric(vertical: 16),
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

**Apa yang terjadi di kode di atas:**

1. `GlobalKey<FormState>` — kunci ajaib yang ngasih kamu akses `.validate()`, `.reset()`, `.save()` ke seluruh form
2. `TextFormField` — punya parameter `validator` yang nggak dimiliki `TextField` biasa
3. `validator` — return `null` kalau valid, return `String` error kalau invalid
4. `.validate()` — memanggil semua validator, return `true` hanya jika SEMUA field valid

---

## 3. TextField vs TextFormField — Kapan Pakai?

| Widget | Best For |
|--------|----------|
| `TextField` | Input tunggal, search bar, chat input, field yang nggak perlu validasi |
| `TextFormField` | Di dalam `Form`, butuh `validator`, butuh validasi kolektif |

**Pro tip:** Kalau kamu cuma punya satu field (misal: search bar), pakai `TextField` — lebih ringan. Kalau lebih dari satu field dan ada logic validasi (register, login, checkout), pakai `Form` + `TextFormField` — lebih rapi dan terstruktur.

---

## 4. Pola Nyata: Form Auto-Save ke State

Kadang kamu nggak langsung submit ke server. Kamu cuma simpan dulu ke state lokal:

```dart
// Di dalam State class
String _nama = '';
String _email = '';
String _password = '';

// Simpan data sebelum validasi
_formKey.currentState!.save();  // ⬅️ trigger semua onSaved callback

// Setelah save, semua variabel di atas terisi
print('Data tersimpan: $_nama, $_email');
```

Caranya: tiap `TextFormField` kasih callback `onSaved`:

```dart
TextFormField(
  decoration: const InputDecoration(labelText: 'Nama Lengkap'),
  validator: (v) => v!.isEmpty ? 'Wajib diisi' : null,
  onSaved: (v) => _nama = v ?? '',   // ← di sini!
)
```

Pola `save()` + `validate()` ini lazim di production app.

---

Hari ini kamu udah bisa menerima input user dengan dua cara: **TextField** buat simpel, **Form + TextFormField** buat yang butuh validasi terstruktur. Ini adalah fondasi penting karena **hampir semua aplikasi nyata butuh form**.

Besok kita akan lanjut ke **Validasi Form & SnackBar Feedback** — bagaimana kasih pesan error yang jelas dan user-friendly. Stay tuned! 🚀

**Coba sendiri! Bikin form login 2 field (email + password) dengan validasi. Share hasilnya ke sosial media dan tag @ahsai001!**
