---
layout: post
title: "Flutter #6: Layouting — Padding, Margin, SizedBox, Expanded & Flexible"
date: 2026-08-08 07:00:00 +0700
tags: [flutter, dart, layouting, padding, margin, sizedbox, expanded, flexible, tutorial]
---

Kalau widget adalah bata, layouting adalah cara kita menyusun bata itu jadi rumah yang rapi. Di Flutter, urusan tata letak itu hampir semuanya pakai widget juga — dan justru di situlah kekuatannya. Nggak ada XML layout terpisah, nggak ada constraint-based layout yang bikin pusing. Semua adalah widget, semua composable.

Hari ini kita bahas lima "alat" layouting paling fundamental: **Padding & Margin**, **SizedBox**, **Expanded**, dan **Flexible**. Kuasai kelimanya, dan kamu bisa bikin layout apapun.

---

## 1. Padding & Margin — Bedanya Apa?

Di Flutter, konsep "margin" dan "padding" disatukan dalam satu widget: **`Padding`** dan properti **`margin`** pada `Container`. Bedanya sederhana:

- **Padding** = ruang *di dalam* widget, antara border dan child
- **Margin** = ruang *di luar* widget, antara widget dan parent/saudaranya

### Padding dengan Widget `Padding`

```dart
Padding(
  padding: const EdgeInsets.all(16.0),
  child: Container(
    color: Colors.blue,
    child: const Text('Ada jarak 16 di semua sisi'),
  ),
)
```

`EdgeInsets` punya beberapa constructor yang berguna:

```dart
EdgeInsets.all(16)           // semua sisi 16
EdgeInsets.symmetric(        // horizontal 24, vertical 12
  horizontal: 24,
  vertical: 12,
)
EdgeInsets.only(             // hanya kiri & atas
  left: 32,
  top: 8,
)
EdgeInsets.fromLTRB(8, 16, 8, 16) // left, top, right, bottom
```

### Margin dengan `Container`

```dart
Container(
  margin: const EdgeInsets.symmetric(horizontal: 20, vertical: 10),
  padding: const EdgeInsets.all(12),
  color: Colors.grey[200],
  child: const Text('Margin luar 20 horizontal, padding dalam 12'),
)
```

> **Tips:** Kalau cuma butuh padding tanpa dekorasi lain, pakai widget `Padding` langsung. `Container` lebih "berat" karena punya banyak properti. Untuk kasus simpel, `Padding` lebih ringan secara rendering.

### Visualisasi Box Model

```
┌────────── MARGIN ──────────┐
│  ┌────── BORDER ──────┐   │
│  │  ┌── PADDING ──┐   │   │
│  │  │              │   │   │
│  │  │   CONTENT    │   │   │
│  │  │              │   │   │
│  │  └──────────────┘   │   │
│  └─────────────────────┘   │
└─────────────────────────────┘
```

---

## 2. SizedBox — Bikin Jarak & Ukuran Fix

`SizedBox` adalah widget paling sederhana untuk membuat *ruang kosong* dengan ukuran tertentu. Dua kegunaan utama:

### Sebagai Spacer (Jarak Antar Widget)

```dart
Column(
  children: [
    ElevatedButton(onPressed: () {}, child: const Text('Tombol 1')),
    const SizedBox(height: 16),  // jarak vertikal 16 pixel
    ElevatedButton(onPressed: () {}, child: const Text('Tombol 2')),
  ],
)
```

### Sebagai Fixed-Size Container

```dart
SizedBox(
  width: 200,
  height: 100,
  child: ElevatedButton(
    onPressed: () {},
    child: const Text('Lebar fix 200, tinggi 100'),
  ),
)
```

### SizedBox vs Container

| Aspek | SizedBox | Container |
|-------|----------|-----------|
| **Fungsi utama** | Ukuran fix atau spacer | Multi-purpose box |
| **Performance** | Ringan | Lebih berat |
| **Dekorasi** | ❌ Tidak bisa | ✅ `decoration`, `borderRadius` |
| **Transform** | ❌ Tidak bisa | ✅ `transform` |
| **Gunakan saat** | Cuma perlu ukuran | Perlu styling (warna, border, shadow, dll) |

---

## 3. Expanded — Ambil Semua Sisa Ruang

`Expanded` adalah widget yang **mengambil semua sisa ruang** dari parent `Row` atau `Column`. Ini adalah senjata utama untuk layout responsif.

```dart
Row(
  children: [
    Container(width: 80, height: 50, color: Colors.red),
    const SizedBox(width: 8),
    Expanded(
      child: Container(height: 50, color: Colors.blue),
    ),  // ← mengambil sisa lebar setelah 80+8 pixel
  ],
)
```

### Properti `flex` — Pembagian Proporsional

Kalau ada beberapa `Expanded`, properti `flex` menentukan rasio pembagiannya:

```dart
Row(
  children: [
    Expanded(
      flex: 2,
      child: Container(height: 50, color: Colors.green),
    ),  // dapat 2/5 lebar
    SizedBox(width: 8),
    Expanded(
      flex: 3,
      child: Container(height: 50, color: Colors.orange),
    ),  // dapat 3/5 lebar
  ],
)
```

Total flex = 2 + 3 = 5. Hijau dapat 2/5, orange dapat 3/5.

---

## 4. Flexible — Mirip Expanded, Tapi Bisa "Ngeliat" Dulu

`Flexible` adalah kakak yang lebih santai dari `Expanded`. Bedanya:

| Fitur | Expanded | Flexible |
|-------|----------|----------|
| **Default fit** | `FlexFit.tight` | `FlexFit.loose` |
| **Memaksa isi** | Widget child mengisi SEMUA ruang | Child bebas lebih kecil dari ruang |
| **Cocok untuk** | Widget yang memang harus full-width | Widget yang punya ukuran natural |

```dart
Row(
  children: [
    Flexible(
      flex: 1,
      child: Container(
        height: 50,
        color: Colors.purple,
        child: const Text('Aku Flexible — tidak harus full'),
      ),
    ),
    const SizedBox(width: 8),
    Expanded(
      flex: 1,
      child: Container(
        height: 50,
        color: Colors.teal,
        child: const Text('Aku Expanded — selalu full'),
      ),
    ),
  ],
)
```

### Kapan Pakai Flexible vs Expanded?

- **Expanded**: Card, button, container yang harus memenuhi sisa ruang
- **Flexible**: Text panjang yang boleh wrap, chip/tag yang ukurannya natural
- Konversi mudah: `Expanded()` sebenarnya sama dengan `Flexible(fit: FlexFit.tight)`

---

## 5. Studi Kasus: Card Profile Responsif

Mari gabungkan semuanya dalam satu contoh nyata. Buat card profile yang:
- Ada avatar bulat di kiri
- Nama dan bio di kanan
- Responsif di berbagai ukuran layar

```dart
import 'package:flutter/material.dart';

class ProfileCard extends StatelessWidget {
  const ProfileCard({super.key});

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.all(16),          // margin luar
      child: Padding(
        padding: const EdgeInsets.all(16),        // padding dalam
        child: Row(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Avatar — ukuran fix 60x60
            const SizedBox(
              width: 60,
              height: 60,
              child: CircleAvatar(
                radius: 30,
                backgroundImage: NetworkImage('https://i.pravatar.cc/150'),
              ),
            ),
            const SizedBox(width: 16),             // spacer
            // Info profile — ambil sisa lebar
            Expanded(
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  const Text(
                    'Ahmad Fathan',
                    style: TextStyle(
                      fontSize: 18,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  const SizedBox(height: 4),
                  Text(
                    'Senior Flutter Developer • Iya, bikin app dari nol sampai Play Store cuma modal ngopi.',
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
```

Jalankan:

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MaterialApp(home: Scaffold(body: ProfileCard())));
```

Hasilnya: avatar 60×60 bulat di kiri, teks fleksibel di kanan. Kalau layar kecil, teks otomatis wrap — berkat `Expanded` yang membatasi lebar maksimalnya.

---

## 6. Cheat Sheet Cepat

```
Pengen jarak antar widget?      → SizedBox(width/height: X)
Pengen padding di dalam?        → Padding(edgeInsets: ...)
Pengen margin di luar?          → Container(margin: ...)
Pengen isi sisa ruang penuh?    → Expanded(flex: ...)
Pengen isi sisa ruang fleksibel? → Flexible(flex: ...)
Pengen ukuran fix?              → SizedBox(width: X, height: Y)
Pengen ukuran fix + styling?    → Container(width: X, height: Y, decoration: ...)
```

---

## 7. Common Mistakes (Jangan Dilakukan!)

### ❌ `Expanded` di luar `Row`/`Column`/`Flex`

```dart
// ERROR: Expanded harus child langsung dari Flex widget
Scaffold(
  body: Expanded(child: ...),  // ❌ Tidak bisa!
)
```

### ❌ Lupa `crossAxisAlignment` saat pakai `Expanded`

```dart
Row(
  children: [
    SizedBox(width: 60, height: 60, child: ...),
    Expanded(
      child: Column(...),   // kalau Row, cross axis = vertical
    ),
  ],
)
// Tambahkan crossAxisAlignment: CrossAxisAlignment.start pada Row
// supaya avatar tidak stretch!
```

### ❌ `Container` banyak properti untuk hal simpel

```dart
// ❌ Berat
Container(
  margin: const EdgeInsets.only(top: 12),
  child: const Text('Halo'),
)

// ✅ Ringan
Padding(
  padding: const EdgeInsets.only(top: 12),
  child: const Text('Halo'),
)
```

---

## 8. Latihan Mandiri

Coba bikin layout berikut sendiri:

1. **Card produk**: Gambar produk di atas (height fix 200), judul + harga + rating di bawah (pakai Padding + SizedBox + Expanded)
2. **Bar navigasi bawah buatan**: Empat icon dalam Row, masing-masing pakai Expanded(flex: 1), biar lebar otomatis rata
3. **Chat bubble**: Pesan kiri (flexible, align kiri) dan pesan kanan (flexible, align kanan) dalam satu Column

---

## Penutup

Lima widget ini — `Padding`, `SizedBox`, `Expanded`, `Flexible`, dan `Container(margin)` — adalah 90% dari pekerjaan layouting kamu sehari-hari. Begitu paham kombinasi ketiganya, bikin UI kompleks pun rasanya kayak main lego: tinggal susun, atur jarak, atur fleksibilitas, beres.

Di artikel berikutnya kita akan naik level: **ListView & GridView** — bikin list konten yang bisa di-scroll, mulai dari list kontak, galeri foto, sampai feed berita.

Coba sendiri! Share hasil latihan kamu ke sosial media dan tag **@ahsai001** 🚀
