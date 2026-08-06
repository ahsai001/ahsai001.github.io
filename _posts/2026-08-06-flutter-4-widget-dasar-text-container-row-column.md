---
layout: post
title: "Flutter #4: Widget Dasar — Text, Container, Row, Column"
date: 2026-08-06 07:00:00 +0700
tags: [flutter, dart, tutorial, indonesia, pemrograman, widget]
---

Halo, teman-teman! Setelah kita kenalan sama Flutter (#1), install SDK-nya (#2), dan paham struktur project (#3), sekarang saatnya masuk ke bagian paling seru: **ngoding UI**! Di Flutter, semuanya adalah widget. Mau bikin teks? Widget. Mau bikin kotak? Widget. Mau atur tata letak? Widget lagi. Jadi hari ini kita bahas empat widget paling fundamental yang bakal kamu pakai di setiap halaman: `Text`, `Container`, `Row`, dan `Column`.

---

## Widget = Lego Blocks

Bayangin Flutter kayak main Lego. Satu brick kecil itu **widget**. Kamu gabungin brick-brick itu buat bikin layar aplikasi. Semua yang kamu lihat di Flutter — tombol, teks, gambar, spasi, scroll — semuanya widget.

Flutter punya dua pendekatan UI:
1. **Imperatif** — kayak Android XML: "bikin TextView, kasih warna, taruh di kiri atas."
2. **Deklaratif** — Flutter style: "gambarin UI kayak gini *sekarang*, kalau state berubah ya gambar ulang."

Nah, kita langsung gaspol ke empat widget paling dasar.

---

## 1. Text — Bikin Tulisan

Widget paling sederhana: `Text()`. Cukup kasih string, langsung muncul di layar.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(MaterialApp(
    home: Scaffold(
      body: Center(
        child: Text('Halo, Flutter!'),
      ),
    ),
  ));
}
```

Segampang itu. Tapi `Text` punya banyak styling:

```dart
Text(
  'Belajar Flutter itu menyenangkan!',
  style: TextStyle(
    fontSize: 24,
    fontWeight: FontWeight.bold,
    color: Colors.blue.shade700,
    fontStyle: FontStyle.italic,
    letterSpacing: 1.2,
    height: 1.5, // line-height
  ),
  textAlign: TextAlign.center,
  maxLines: 2,
  overflow: TextOverflow.ellipsis,
)
```

**Yang perlu kamu ingat:**
- `style` pakai `TextStyle` — ini tempat semua properti font
- `textAlign` buat rata kiri/kanan/tengah
- `maxLines` + `overflow` berguna biar teks gak kepanjangan

### Text Direction & Softwrap

Flutter otomatis `softwrap` teks yang panjang. Tapi kalau container-nya sempit dan gak ada scroll, teks bisa meluap (muncul garis kuning-hitam di debug mode). Solusinya: bungkus `Text` dengan `Expanded` atau `Flexible` kalau dia ada di dalam `Row`/`Column`.

---

## 2. Container — Kotak Serbaguna

Kalau `Text` itu isi, `Container` itu **bungkusnya**. Dia kotak yang bisa kamu kasih warna, padding, margin, border, shadow — kayak `<div>`-nya Flutter.

```dart
Container(
  width: 200,
  height: 150,
  padding: EdgeInsets.all(16),
  margin: EdgeInsets.symmetric(horizontal: 20, vertical: 10),
  decoration: BoxDecoration(
    color: Colors.teal.shade100,
    borderRadius: BorderRadius.circular(12),
    border: Border.all(color: Colors.teal, width: 2),
    boxShadow: [
      BoxShadow(
        color: Colors.teal.withOpacity(0.3),
        blurRadius: 8,
        offset: Offset(0, 4),
      ),
    ],
  ),
  child: Text('Ini dalam Container', style: TextStyle(fontSize: 18)),
)
```

**Container properties wajib tahu:**

| Property | Fungsi |
|----------|--------|
| `width` / `height` | Ukuran kotak |
| `padding` | Jarak dari border ke child |
| `margin` | Jarak dari luar ke border |
| `decoration` | Warna, border, shadow (pakai `BoxDecoration`) |
| `child` | Widget di dalamnya |

**⚠️ PENTING:** Jangan pakai `color` bareng `decoration`! Kalau kamu pakai `decoration: BoxDecoration(...)`, properti `color` harus dimasukin ke `BoxDecoration`, bukan langsung ke `Container`. Kalau kamu tulis dua-duanya, Flutter error.

```dart
// ❌ SALAH — bakal error
Container(color: Colors.red, decoration: BoxDecoration(color: Colors.blue))

// ✅ BENAR
Container(decoration: BoxDecoration(color: Colors.blue))
```

---

## 3. Row & Column — Atur Tata Letak

Inilah dua widget paling dipakai buat layouting. **Row** menyusun child horizontal (kiri ke kanan), **Column** menyusun vertikal (atas ke bawah). Mirip `flexbox` di CSS.

### Row

```dart
Row(
  mainAxisAlignment: MainAxisAlignment.spaceEvenly,
  crossAxisAlignment: CrossAxisAlignment.center,
  children: [
    Icon(Icons.star, color: Colors.amber, size: 32),
    Icon(Icons.favorite, color: Colors.red, size: 32),
    Icon(Icons.thumb_up, color: Colors.blue, size: 32),
    Icon(Icons.share, color: Colors.green, size: 32),
  ],
)
```

### Column

```dart
Column(
  mainAxisAlignment: MainAxisAlignment.center,
  crossAxisAlignment: CrossAxisAlignment.start,
  children: [
    Text('Nama: Ahsai', style: TextStyle(fontSize: 20)),
    SizedBox(height: 8),
    Text('Role: Flutter Developer', style: TextStyle(fontSize: 16)),
    SizedBox(height: 8),
    Text('Hobi: Ngoding & Kopi', style: TextStyle(fontSize: 16)),
  ],
)
```

### MainAxisAlignment vs CrossAxisAlignment

Ini konsep penting banget. Gampangnya:

- **Main axis** = arah susunan utama. Untuk `Column` = vertikal, untuk `Row` = horizontal.
- **Cross axis** = arah yang tegak lurus. Untuk `Column` = horizontal, untuk `Row` = vertikal.

| MainAxisAlignment | Efek |
|-------------------|------|
| `start` | Rapat atas (Column) / kiri (Row) |
| `end` | Rapat bawah (Column) / kanan (Row) |
| `center` | Tengah |
| `spaceBetween` | Renggang, space di antara item |
| `spaceAround` | Space merata, setengah di ujung |
| `spaceEvenly` | Space benar-benar merata |

### Contoh Praktek: Card Profile

Gabungin semuanya jadi satu widget keren:

```dart
Card(
  elevation: 4,
  shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),
  child: Padding(
    padding: EdgeInsets.all(20),
    child: Row(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        // Avatar
        Container(
          width: 60,
          height: 60,
          decoration: BoxDecoration(
            color: Colors.deepPurple,
            borderRadius: BorderRadius.circular(30),
          ),
          child: Icon(Icons.person, color: Colors.white, size: 32),
        ),
        SizedBox(width: 16),
        // Info
        Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              'Ahsai Dev',
              style: TextStyle(
                fontSize: 20,
                fontWeight: FontWeight.bold,
              ),
            ),
            SizedBox(height: 4),
            Text('Flutter Developer • 3 tahun'),
            SizedBox(height: 8),
            Row(
              children: [
                Icon(Icons.location_on, size: 16, color: Colors.grey),
                SizedBox(width: 4),
                Text('Jakarta, Indonesia'),
              ],
            ),
          ],
        ),
      ],
    ),
  ),
)
```

Outputnya: sebuah card profile dengan avatar bulat di kiri, teks dan icon di kanan — rapi, clean, dan cuma pakai `Row`, `Column`, `Container`, dan `Text`.

---

## Tips Anti-Pusing Layouting

1. **Mulai dari luar ke dalam** — bungkus pakai `Container` dulu, baru isi child.
2. **Row/Column bisa nested** — jangan takut bikin `Column` di dalam `Row` atau sebaliknya.
3. **`SizedBox` is your best friend** — buat kasih jarak antar widget tanpa ribet ngatur margin.
4. **Gunakan `debugPaintSizeEnabled = true`** — aktifkan di `main()` buat liat garis batas setiap widget. Super membantu debugging layout!

---

## Kesimpulan

Empat widget ini adalah fondasi dari semua UI Flutter:

| Widget | Fungsi |
|--------|--------|
| `Text` | Menampilkan tulisan |
| `Container` | Kotak pembungkus dengan styling |
| `Row` | Susun child horizontal |
| `Column` | Susun child vertikal |

Kuasai empat ini dulu, nanti widget lain seperti `ListView`, `Stack`, `GridView` bakal jauh lebih gampang dipahami. Di artikel selanjutnya, kita masuk ke **State**: bedanya `StatelessWidget` dan `StatefulWidget` serta magic-nya `setState`.

---

Sampai jumpa di artikel #5! Kalau ada pertanyaan, pantau terus blog ini — tiap hari ada tutorial baru. Jangan lupa share ke sosial media dan tag [@ahsai001](https://ahsai.my.id). Selamat ngoding! 🚀
