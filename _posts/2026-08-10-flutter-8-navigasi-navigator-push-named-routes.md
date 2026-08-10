---
layout: post
title: "Flutter #8: Navigasi — Navigator.push & Named Routes"
date: 2026-08-10 07:00:00 +0700
tags: [flutter, dart, navigasi, navigator, routes, tutorial, indonesia, pemrograman]
---

Aplikasi yang cuma satu layar itu namanya bukan aplikasi — itu cuma gambar. Aplikasi beneran pasti punya beberapa halaman: dari halaman login ke dashboard, dari list produk ke detail, dari keranjang ke checkout. Nah, di Flutter, perpindahan antar halaman ini disebut **Navigasi**, dan senjatanya adalah class `Navigator`.

Hari ini kita bahas dua metode navigasi yang paling sering dipakai: **Navigator.push** (imperatif) dan **Named Routes** (deklaratif). Dua-duanya wajib kamu kuasai!

---

## 1. Konsep Dasar: Stack Navigation

Flutter menggunakan konsep **stack** — ingat tumpukan piring. Setiap kamu "push" halaman baru, dia numpuk di atas halaman sebelumnya. Setiap "pop", halaman paling atas dibuang dan kamu kembali ke halaman di bawahnya.

```
┌─────────────┐
│  Detail     │ ← push (tambah)
├─────────────┤
│  List Menu  │ ← halaman sebelumnya
├─────────────┤
│  Home       │ ← halaman awal
└─────────────┘
```

Sederhana, tapi powerful. Tanpa perlu bikin logic ribet, Flutter udah handle animasi transisi dan back-button Android secara otomatis.

---

## 2. Navigator.push — Cara Paling Simpel

`Navigator.push` menerima dua parameter: `context` dan `MaterialPageRoute`. Ini metode paling cocok buat pemula karena eksplisit.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MaterialApp(home: HomeScreen()));

// ========== HALAMAN PERTAMA ==========
class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('🏠 Halaman Utama')),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            // 🌟 NAVIGASI KE HALAMAN KEDUA
            Navigator.push(
              context,
              MaterialPageRoute(builder: (context) => const DetailScreen()),
            );
          },
          child: const Text('Lihat Detail ➡️'),
        ),
      ),
    );
  }
}

// ========== HALAMAN KEDUA ==========
class DetailScreen extends StatelessWidget {
  const DetailScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('📄 Halaman Detail')),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            // ⬅️ KEMBALI KE HALAMAN SEBELUMNYA
            Navigator.pop(context);
          },
          child: const Text('⬅️ Kembali'),
        ),
      ),
    );
  }
}
```

**Penjelasan:**
- `Navigator.push(...)` → dorong halaman baru ke atas stack
- `MaterialPageRoute` → animasi transisi sesuai platform (slide dari kanan di Android, fade di iOS)
- `Navigator.pop(context)` → buang halaman saat ini dari stack, kembali ke sebelumnya

### Pass Data Antar Halaman

Kirim data dari Home ke Detail? Gampang. Karena `DetailScreen` adalah constructor biasa:

```dart
// Kirim dari HomeScreen
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (context) => DetailScreen(nama: 'Ahmad', umur: 25),
  ),
);

// DetailScreen terima lewat constructor
class DetailScreen extends StatelessWidget {
  final String nama;
  final int umur;

  const DetailScreen({super.key, required this.nama, required this.umur});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Halo, $nama!')),
      body: Center(child: Text('Umur: $umur tahun')),
    );
  }
}
```

Dan kalau mau **balikin data** dari DetailScreen ke HomeScreen:

```dart
// Di DetailScreen, pop sambil bawa data
Navigator.pop(context, 'Data dari Detail');

// Di HomeScreen, tangkap hasilnya
final hasil = await Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => const DetailScreen()),
);
print('Hasil: $hasil'); // Output: Hasil: Data dari Detail
```

---

## 3. Named Routes — Navigasi Deklaratif

Kalau aplikasi udah punya banyak halaman, `MaterialPageRoute` satu-satu mulai ribet. Solusinya **named routes** — daftarin semua route di awal, panggil tinggal sebut nama.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Navigation Demo',
      initialRoute: '/',                        // halaman default
      routes: {
        '/': (context) => const HomeScreen(),   // ← wajib ada initialRoute
        '/detail': (context) => const DetailScreen(),
        '/profile': (context) => const ProfileScreen(),
      },
    );
  }
}

class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('🏠 Home')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () {
                Navigator.pushNamed(context, '/detail');  // ← by name!
              },
              child: const Text('Ke Detail ➡️'),
            ),
            const SizedBox(height: 12),
            ElevatedButton(
              onPressed: () {
                Navigator.pushNamed(context, '/profile');
              },
              child: const Text('Ke Profil 👤'),
            ),
          ],
        ),
      ),
    );
  }
}

class DetailScreen extends StatelessWidget {
  const DetailScreen({super.key});
  @override
  Widget build(BuildContext context) => Scaffold(
    appBar: AppBar(title: const Text('📄 Detail')),
    body: Center(child: Text('Halaman Detail')),
  );
}

class ProfileScreen extends StatelessWidget {
  const ProfileScreen({super.key});
  @override
  Widget build(BuildContext context) => Scaffold(
    appBar: AppBar(title: const Text('👤 Profil')),
    body: Center(child: Text('Halaman Profil')),
  );
}
```

**Kelebihan Named Routes:**
- Semua route kelihatan di satu tempat — gampang maintenancenya
- Cocok untuk aplikasi menengah-besar
- Bisa pakai `onGenerateRoute` untuk dynamic routing (ada parameter, guard auth, dsb)

### Pass Arguments dengan Named Routes

```dart
// Kirim argument
Navigator.pushNamed(
  context,
  '/detail',
  arguments: {'nama': 'Siti', 'role': 'admin'},
);

// Terima di halaman tujuan
class DetailScreen extends StatelessWidget {
  const DetailScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final args = ModalRoute.of(context)!.settings.arguments as Map<String, String>;
    return Scaffold(
      appBar: AppBar(title: Text('Halo, ${args['nama']}!')),
      body: Center(child: Text('Role: ${args['role']}')),
    );
  }
}
```

---

## 4. Catatan Penting Navigasi

| Metode | Kapan Dipakai |
|--------|---------------|
| `Navigator.push` + `MaterialPageRoute` | Aplikasi kecil (1-5 halaman), butuh pass data sederhana |
| Named Routes (`routes: {...}`) | Aplikasi menengah, banyak halaman static |
| `onGenerateRoute` | Dynamic routing, butuh auth guard, parsing URL parameter |

**Pro tip:** Jangan kirim objek yang kompleks lewat arguments — mending kirim ID, lalu fetch datanya di halaman tujuan. Lebih bersih dan lebih aman.

---

## 5. Push vs PushNamed: Beda Tipis Tapi Penting

```dart
// IMPERATIF — kamu yang atur route-nya manual
Navigator.push(context, MaterialPageRoute(builder: (_) => TargetScreen()));

// DEKLARATIF — tinggal sebut nama, Flutter yang handle
Navigator.pushNamed(context, '/target');
```

Keduanya valid. Pilih yang bikin kode kamu lebih rapi dan mudah dibaca. Di proyek real, tim biasanya pakai named routes + `onGenerateRoute` biar scalable.

---

Hari ini kamu udah belajar cara berpindah halaman — skill fundamental yang **ada di 100% aplikasi Flutter**. Dari dua layar sederhana sampai puluhan halaman dengan auth guard, semuanya berawal dari `Navigator.push` dan `pushNamed`.

Besok kita lanjut ke **Form & TextField** — bagaimana menerima input dari user dengan benar. Stay tuned! 🚀

**Coba sendiri! Bikin aplikasi 3 halaman dengan named routes. Share hasilnya ke sosial media dan tag @ahsai001!**
