---
layout: post
title: "Flutter #11: Asset & Image — Gambar Lokal & Network"
date: 2026-08-13 07:00:00 +0700
tags: [flutter, dart, image, asset, tutorial, indonesia, pemrograman]
---

Kemarin kita udah bikin form yang valid & responsif. Sekarang waktunya bikin tampilan makin hidup dengan **gambar**. Aplikasi tanpa gambar itu kayak nasi tanpa lauk — hambar. Entah foto profil, banner promosi, icon produk, atau ilustrasi, hampir semua app butuh gambar.

Flutter ngasih beberapa cara buat nampilin gambar:

1. `Image.asset` — gambar lokal yang dibundel bareng app
2. `Image.network` — gambar dari internet
3. `Image.file` / `Image.memory` — gambar dari file device / bytes (kasus khusus)

Hari ini kita bedah tuntas cara pakai **asset lokal** & **gambar network**, plus best practice-nya.

---

## 1. Gambar Lokal dengan `Image.asset`

### 1.1 Daftarkan Asset di `pubspec.yaml`

Gambar lokal **harus didaftarkan dulu** di `pubspec.yaml`, bagian `flutter:`:

```yaml
flutter:
  uses-material-design: true
  assets:
    - assets/images/
```

Kalau kamu tulis nama folder (`assets/images/`), semua file di dalamnya otomatis terbundel. Kalau cuma mau satu file, tulis path lengkapnya (`assets/images/logo.png`).

**Struktur folder:**

```
project/
├── pubspec.yaml
└── assets/
    └── images/
        ├── logo.png
        └── banner.jpg
```

> ⚠️ **Indentasi penting!** `assets:` harus sejajar dengan `uses-material-design:` (2 spasi dari `flutter:`), dan path-nya menjorok 4 spasi di bawah `assets:`. Salah indentasi = asset nggak kebaca, dan error-nya kadang nggak jelas.

### 1.2 Tampilkan dengan `Image.asset`

```dart
Image.asset(
  'assets/images/logo.png',
  width: 120,
  height: 120,
  fit: BoxFit.cover,
)
```

Semudah itu! Tapi jangan lupa kasih `width`/`height` kalau gambarnya besar, biar nggak meledak memenuhi layar.

---

## 2. Gambar dari Internet dengan `Image.network`

Nggak perlu daftar di `pubspec.yaml`. Cukup kasih URL:

```dart
Image.network('https://picsum.photos/400/300')
```

Tapi dunia nyata nggak semulus itu. Jaringan bisa lambat, URL bisa 404, gambar bisa gagal dimuat. Makanya `Image.network` punya dua parameter yang **wajib** kamu pakai:

- `loadingBuilder` — tampilkan progress/placeholder saat gambar masih loading
- `errorBuilder` — tampilkan fallback kalau gambar gagal dimuat

```dart
Image.network(
  'https://picsum.photos/400/300',
  width: double.infinity,
  height: 200,
  fit: BoxFit.cover,
  loadingBuilder: (context, child, progress) {
    if (progress == null) return child; // selesai dimuat
    return Container(
      height: 200,
      alignment: Alignment.center,
      child: CircularProgressIndicator(
        value: progress.expectedTotalBytes != null
            ? progress.cumulativeBytesLoaded / progress.expectedTotalBytes!
            : null,
      ),
    );
  },
  errorBuilder: (context, error, stackTrace) {
    return Container(
      height: 200,
      color: Colors.grey.shade200,
      alignment: Alignment.center,
      child: const Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          Icon(Icons.broken_image, size: 48, color: Colors.grey),
          SizedBox(height: 8),
          Text('Gagal memuat gambar'),
        ],
      ),
    );
  },
)
```

---

## 3. `BoxFit` — Atur Cara Gambar Mengisi Kotak

`BoxFit` menentukan bagaimana gambar "masuk" ke ukuran widget-nya. Ini yang paling sering bikin pemula bingung:

| BoxFit | Efek |
|---|---|
| `BoxFit.cover` | Mengisi penuh, **dipotong** kalau rasio beda — paling sering dipakai |
| `BoxFit.contain` | Gambar terlihat utuh, mungkin ada ruang kosong |
| `BoxFit.fill` | Dipaksa mengisi penuh, **rasio bisa rusak** (melebar/memanjang) |
| `BoxFit.fitWidth` | Lebar pas, tinggi menyesuaikan |
| `BoxFit.fitHeight` | Tinggi pas, lebar menyesuaikan |

**Aturan praktis:** foto profil & banner → `BoxFit.cover`, logo → `BoxFit.contain`.

---

## 4. Gambar Bulat dengan `ClipRRect` & `CircleAvatar`

Foto profil bulat itu standar banget. Dua cara:

```dart
// Cara 1: ClipRRect
ClipRRect(
  borderRadius: BorderRadius.circular(60), // 60 = setengah dari 120
  child: Image.asset(
    'assets/images/avatar.png',
    width: 120,
    height: 120,
    fit: BoxFit.cover,
  ),
)

// Cara 2: CircleAvatar (otomatis bulat, khusus profil)
CircleAvatar(
  radius: 60,
  backgroundImage: NetworkImage('https://i.pravatar.cc/300'),
)
```

---

## 5. Contoh Lengkap: Card Profil

Gabungin semuanya jadi satu card profil yang siap pakai:

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MaterialApp(home: ProfileScreen()));

class ProfileScreen extends StatelessWidget {
  const ProfileScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('👤 Profil')),
      body: Center(
        child: Card(
          elevation: 4,
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(16),
          ),
          child: Padding(
            padding: const EdgeInsets.all(20),
            child: Column(
              mainAxisSize: MainAxisSize.min,
              children: [
                // Foto profil bulat dari network
                const CircleAvatar(
                  radius: 50,
                  backgroundImage: NetworkImage('https://i.pravatar.cc/200'),
                ),
                const SizedBox(height: 16),
                const Text(
                  'Ahmad Saifullah',
                  style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
                ),
                const SizedBox(height: 4),
                Text(
                  'Software Developer & Urban Farmer',
                  style: TextStyle(color: Colors.grey.shade600),
                ),
                const SizedBox(height: 16),
                // Banner dari asset lokal
                ClipRRect(
                  borderRadius: BorderRadius.circular(12),
                  child: Image.asset(
                    'assets/images/banner.jpg',
                    width: 260,
                    height: 120,
                    fit: BoxFit.cover,
                  ),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

---

## 6. Best Practice

- **Kompres gambar** sebelum masukin ke asset. PNG 2MB bakal bikin APK gemuk. WebP lebih hemat dengan kualitas hampir sama.
- **Kasih `width`/`height`** — gambar tanpa ukuran bisa bikin layout error/overflow.
- **Selalu pasang `errorBuilder`** di `Image.network` — user nggak boleh lihat error merah di layar.
- **Pakai `cacheWidth`** kalau nampilin gambar besar tapi cuma butuh versi kecil (misal thumbnail). Flutter bakal menyimpan versi yang sudah dikecilkan:

```dart
Image.network(
  url,
  cacheWidth: 400, // simpan versi lebar 400px, hemat memori
)
```

- **Asset lokal** untuk gambar yang sering dipakai (logo, icon, ilustrasi). **Network** untuk gambar dinamis (foto user, produk).

---

## Ringkasan

- **`Image.asset`** — gambar lokal, daftarkan dulu di `pubspec.yaml`
- **`Image.network`** — gambar internet, wajib pasang `loadingBuilder` + `errorBuilder`
- **`BoxFit.cover`** — default terbaik buat foto/banner
- **`ClipRRect` / `CircleAvatar`** — bikin gambar bulat
- **Kompres & kasih ukuran** — biar app ringan dan nggak overflow

Gambar adalah separuh dari UX. App yang gambarnya rapi, loading-nya mulus, dan punya fallback yang baik langsung terasa profesional.

Besok kita bahas **Font Custom & Tema Aplikasi (ThemeData)** — saatnya bikin app yang punya "identitas" visual sendiri. Stay tuned! 🎨

**Coba sendiri! Bikin card profil dengan foto bulat dari network + banner dari asset lokal. Share hasilnya ke sosial media dan tag @ahsai001!**
