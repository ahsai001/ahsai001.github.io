---
layout: post
title: "Flutter #25: Drawer & Navigation Drawer"
date: 2026-09-03 07:00:00 +0700
tags: [flutter, drawer, navigation, widget, tutorial]
---

Halo developer! 👋

Di artikel kali ini kita bakal belajar tentang **Drawer** di Flutter — komponen navigasi samping yang sering banget kamu temuin di app populer seperti Gmail, Instagram, atau Spotify. Drawer itu powerful karena bisa nampung banyak menu navigasi tanpa bikin UI penuh sesak.

Kita bakal cover tiga hal:
1. **Drawer** dasar dengan `Scaffold`
2. **Custom Drawer** dengan header, avatar, dan menu interaktif
3. **End Drawer** (drawer di sisi kanan)

Let's go! 🚀

---

## Apa itu Drawer?

Drawer adalah panel navigasi yang bisa dibuka dengan geser dari tepi layar (biasanya sisi kiri) atau dengan klik ikon hamburger (☰). Di Flutter, kamu tinggal pakai widget `Drawer` di dalam `Scaffold` — gampang banget!

### Kelebihan Drawer vs Bottom Navigation

| Fitur | Drawer | Bottom Navigation |
|-------|--------|-------------------|
| Jumlah menu | Banyak (scrollable) | Maksimal 5 |
| Visibilitas | Tersembunyi | Selalu terlihat |
| Cocok untuk | Pengaturan, profil, About | Aksi utama / fitur utama |
| Discoverability | Kurang (perlu swipe/click) | Tinggi |

---

## 1. Drawer Dasar

Paling basic, tinggal tambahin properti `drawer` di `Scaffold`:

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Drawer Demo',
      theme: ThemeData(
        colorSchemeSeed: Colors.indigo,
        useMaterial3: true,
      ),
      home: const HomeScreen(),
    );
  }
}

class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('My App')),
      drawer: Drawer(
        child: ListView(
          padding: EdgeInsets.zero,
          children: [
            const DrawerHeader(
              decoration: BoxDecoration(color: Colors.indigo),
              child: Text(
                'Menu',
                style: TextStyle(color: Colors.white, fontSize: 24),
              ),
            ),
            ListTile(
              leading: const Icon(Icons.home),
              title: const Text('Beranda'),
              onTap: () => Navigator.pop(context),
            ),
            ListTile(
              leading: const Icon(Icons.person),
              title: const Text('Profil'),
              onTap: () => Navigator.pop(context),
            ),
            ListTile(
              leading: const Icon(Icons.settings),
              title: const Text('Pengaturan'),
              onTap: () => Navigator.pop(context),
            ),
          ],
        ),
      ),
      body: const Center(
        child: Text('Geser dari kiri atau tap ☰ untuk buka menu'),
      ),
    );
  }
}
```

Flutter otomatis nambahin tombol ☰ di `AppBar`. Ketika user tap atau geser dari tepi kiri, drawer muncul. Keren kan?

> **Catatan:** `DrawerHeader` itu widget khusus dari Flutter yang punya padding dan dekorasi bawaan — lebih bagus dari `Container` biasa.

---

## 2. Custom Drawer dengan Header & Avatar

Sekarang kita bikin drawer yang lebih realistis — mirip drawer di app pada umumnya:

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Custom Drawer',
      theme: ThemeData(
        colorSchemeSeed: Colors.teal,
        useMaterial3: true,
      ),
      home: const HomeScreen(),
    );
  }
}

class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Custom Drawer')),
      drawer: _buildDrawer(context),
      body: const Center(child: Text('Menuju halaman yang dipilih')),
    );
  }

  Widget _buildDrawer(BuildContext context) {
    return Drawer(
      child: Column(
        children: [
          // Header dengan info user
          const UserAccountsDrawerHeader(
            accountName: Text('Ahmad Sahai'),
            accountEmail: Text('ahmad@ahsai.my.id'),
            currentAccountPicture: CircleAvatar(
              backgroundColor: Colors.white,
              child: Icon(Icons.person, color: Colors.teal, size: 36),
            ),
            decoration: BoxDecoration(
              gradient: LinearGradient(
                colors: [Colors.teal, Colors.tealAccent],
                begin: Alignment.topLeft,
                end: Alignment.bottomRight,
              ),
            ),
          ),

          // Menu items
          ListTile(
            leading: const Icon(Icons.home_outlined),
            title: const Text('Beranda'),
            onTap: () {
              Navigator.pop(context);
              // Navigate ke home
            },
          ),
          ListTile(
            leading: const Icon(Icons.article_outlined),
            title: const Text('Artikel'),
            onTap: () {
              Navigator.pop(context);
            },
          ),
          ListTile(
            leading: const Icon(Icons.bookmark_outlined),
            title: const Text('Bookmark'),
            onTap: () {
              Navigator.pop(context);
            },
          ),

          const Divider(),

          // Section pengaturan
          ListTile(
            leading: const Icon(Icons.settings_outlined),
            title: const Text('Pengaturan'),
            onTap: () {
              Navigator.pop(context);
            },
          ),
          ListTile(
            leading: const Icon(Icons.info_outline),
            title: const Text('Tentang'),
            onTap: () {
              Navigator.pop(context);
            },
          ),

          const Spacer(),

          // Footer versi
          const Padding(
            padding: EdgeInsets.all(16),
            child: Text(
              'Versi 1.0.0',
              style: TextStyle(color: Colors.grey, fontSize: 12),
            ),
          ),
        ],
      ),
    );
  }
}
```

Fitur keren dari kode di atas:

- **`UserAccountsDrawerHeader`** — widget bawaan Flutter yang langsung kasih avatar, nama, email, dan gradient background. Tinggal isi datanya!
- **`Divider`** — pemisah visual antara section menu
- **`Spacer()`** — push versi ke bawah biar keliatan rapi

### Navigasi ke Halaman Lain

Kalau mau navigate ke halaman baru dari drawer, tinggal pakai `Navigator.push`:

```dart
onTap: () {
  Navigator.pop(context); // Tutup drawer dulu
  Navigator.push(
    context,
    MaterialPageRoute(builder: (_) => const PengaturanScreen()),
  );
},
```

Atau kalau pakai Named Routes:

```dart
onTap: () {
  Navigator.pop(context);
  Navigator.pushNamed(context, '/settings');
},
```

---

## 3. End Drawer (Sisi Kanan)

Mau taruh drawer di sisi kanan? Gampang, tinggal pakai `endDrawer`:

```dart
Scaffold(
  appBar: AppBar(title: const Text('End Drawer')),
  endDrawer: Drawer(
    child: ListView(
      padding: EdgeInsets.zero,
      children: [
        const DrawerHeader(
          decoration: BoxDecoration(color: Colors.deepOrange),
          child: Text(
            'Notifikasi',
            style: TextStyle(color: Colors.white, fontSize: 24),
          ),
        ),
        const ListTile(
          leading: Icon(Icons.notifications),
          title: Text('Ada 3 notifikasi baru'),
        ),
        const ListTile(
          leading: Icon(Icons.mail),
          title: Text('Pesan dari Budi'),
        ),
        const ListTile(
          leading: Icon(Icons.system_update),
          title: Text('Update tersedia'),
        ),
      ],
    ),
  ),
  body: const Center(
    child: Text('Geser dari kanan untuk lihat notifikasi'),
  ),
);
```

Kamu juga bisa pakai **keduanya sekaligus** — `drawer` di kiri dan `endDrawer` di kanan. Flutter akan handle swipe gesture untuk masing-masing sisi secara otomatis.

---

## Tips & Best Practices

1. **Tutup drawer sebelum navigate** — Selalu pakai `Navigator.pop(context)` dulu sebelum `push`/`pushNamed`. Kalau enggak, drawer masih terbuka di belakang halaman baru.

2. **Gunakan `UserAccountsDrawerHeader`** untuk drawer dengan info user — jangan bikin manual dari Container, `UserAccountsDrawerHeader` udah handle padding dan layout dengan baik.

3. **Batas menu 6-8 item** — Kalau menu terlalu banyak, user bakal kesulitan cari. Pertimbangkan pakai submenu atau section.

4. **Highlight menu aktif** — Tambahin properti `selected: true` di `ListTile` untuk menu yang sedang aktif biar user tau dia di mana.

5. **Akses drawer dari AppBar** — Flutter otomatis nambahin tombol ☰ (hamburger). Kalau mau custom, set manual pakai `actions` di `AppBar`.

---

## Ringkasan

| Widget | Fungsi |
|--------|--------|
| `Drawer` | Container utama drawer |
| `DrawerHeader` | Header sederhana dengan padding bawaan |
| `UserAccountsDrawerHeader` | Header dengan avatar, nama, email |
| `ListTile` | Item menu di dalam drawer |
| `endDrawer` | Drawer di sisi kanan layar |

---

## Yang Akan Kita Pelajari Selanjutnya

Di artikel berikutnya, kita bakal belajar tentang **AlertDialog, BottomSheet, dan DatePicker** — komponen dialog dan input interaktif yang sering banget dipakai di aplikasi production. Stay tuned! 🎯

---

Coba sendiri! Bikin drawer versimu sendiri, tambahin icon, warna, dan navigasi. Share ke sosial media dan tag **@ahsai001** — aku senang banget lihat hasil karya kalian! 💪

*#Flutter #Dart #Tutorial #Drawer #Navigation #Indonesia*
