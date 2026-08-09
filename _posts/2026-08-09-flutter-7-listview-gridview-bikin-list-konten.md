---
layout: post
title: "Flutter #7: ListView & GridView — Bikin List Konten"
date: 2026-08-09 07:00:00 +0700
tags: [flutter, dart, listview, gridview, scroll, tutorial, indonesia, pemrograman]
---

Aplikasi yang cuma satu layar statis itu ibarat warung kosong — sepi, gak ada yang bisa di-scroll. Nah, dua widget paling penting untuk bikin konten yang bisa digulir adalah **ListView** dan **GridView**. Mau bikin feed berita, daftar kontak, galeri foto, sampai katalog produk? Dua inilah senjata utamamu.

Hari ini kita akan bahas dari yang paling sederhana sampai yang performanya juara. Let's go!

---

## 1. ListView — Raja Daftar Vertikal

`ListView` adalah widget untuk menampilkan daftar item yang bisa di-scroll secara vertikal (atau horizontal). Cara paling simpel:

### ListView Sederhana

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MaterialApp(home: ListBasicScreen()));

class ListBasicScreen extends StatelessWidget {
  const ListBasicScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('ListView Dasar')),
      body: ListView(
        children: const [
          ListTile(
            leading: Icon(Icons.person),
            title: Text('Budi Santoso'),
            subtitle: Text('Flutter Developer'),
          ),
          ListTile(
            leading: Icon(Icons.person),
            title: Text('Siti Aminah'),
            subtitle: Text('UI/UX Designer'),
          ),
          ListTile(
            leading: Icon(Icons.person),
            title: Text('Joko Widodo'),
            subtitle: Text('Project Manager'),
          ),
        ],
      ),
    );
  }
}
```

Cocok buat list yang **pendek dan jumlahnya fix**. Tapi kalau datanya ratusan? Jangan! Kenapa? Karena `ListView(children: [...])` langsung me-render *semua* child widget sekaligus — memory boros, performa jeblok.

---

## 2. ListView.builder — Pahlawan Performa

Untuk data banyak (dari API, database, atau list panjang), pakai `ListView.builder`. Dia cuma me-render item yang **terlihat di layar** — mirip RecyclerView di Android.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MaterialApp(home: ListBuilderScreen()));

class ListBuilderScreen extends StatelessWidget {
  const ListBuilderScreen({super.key});

  // Simulasi data dari API
  final List<Map<String, String>> _contacts = const [
    {'name': 'Ahmad Fathan', 'role': 'Senior Flutter Dev'},
    {'name': 'Rina Marlina', 'role': 'Backend Engineer'},
    {'name': 'Doni Prasetyo', 'role': 'DevOps'},
    {'name': 'Putri Ayu', 'role': 'Product Manager'},
    {'name': 'Eko Nugroho', 'role': 'Mobile Developer'},
    {'name': 'Fitri Handayani', 'role': 'QA Engineer'},
    {'name': 'Bayu Saputra', 'role': 'Data Scientist'},
    {'name': 'Citra Lestari', 'role': 'Tech Lead'},
    {'name': 'Galih Prakoso', 'role': 'Frontend Dev'},
    {'name': 'Indah Permatasari', 'role': 'Scrum Master'},
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('ListView.builder'),
        centerTitle: true,
      ),
      body: ListView.builder(
        itemCount: _contacts.length,
        itemBuilder: (context, index) {
          final contact = _contacts[index];
          return Card(
            margin: const EdgeInsets.symmetric(horizontal: 12, vertical: 4),
            child: ListTile(
              leading: CircleAvatar(
                backgroundColor: Colors.indigo.shade100,
                child: Text(
                  contact['name']![0],
                  style: const TextStyle(fontWeight: FontWeight.bold),
                ),
              ),
              title: Text(contact['name']!),
              subtitle: Text(contact['role']!),
              trailing: const Icon(Icons.chevron_right),
              onTap: () {
                ScaffoldMessenger.of(context).showSnackBar(
                  SnackBar(content: Text('Klik: ${contact['name']}')),
                );
              },
            ),
          );
        },
      ),
    );
  }
}
```

### Kunci ListView.builder

| Parameter | Wajib? | Fungsi |
|-----------|--------|--------|
| `itemCount` | Ya (untuk fix-length) | Jumlah total item |
| `itemBuilder` | Ya | Callback yang dipanggil per item, nerima `(context, index)` |
| `itemExtent` | Tidak | Kalau semua item tingginya sama, set ini biar makin kencang |
| `scrollDirection` | Tidak | Default `Axis.vertical`, bisa juga `Axis.horizontal` |

> **Pro tip:** Kalau tinggi semua item seragam (misal 80px), tambahin `itemExtent: 80`. Flutter gak perlu ngukur tinggi satu per satu → scrolling lebih mulus.

---

## 3. ListView.separated — Tambahin Pembatas

Mau ada divider di antara item? `ListView.separated` jawabannya:

```dart
ListView.separated(
  itemCount: _contacts.length,
  separatorBuilder: (context, index) => const Divider(
    height: 1,
    indent: 72,  // biar rata dengan text, bukan avatar
  ),
  itemBuilder: (context, index) {
    final contact = _contacts[index];
    return ListTile(
      leading: CircleAvatar(
        backgroundColor: Colors.indigo.shade100,
        child: Text(contact['name']![0]),
      ),
      title: Text(contact['name']!),
      subtitle: Text(contact['role']!),
    );
  },
)
```

`separatorBuilder` cuma dipanggil di **antara** item — bukan di awal atau akhir. Simpel tapi powerful.

---

## 4. GridView — Saat Konten Perlu Tampil "Kotak-Kotak"

Kalau kontenmu lebih cocok tampil dalam grid (galeri foto, katalog produk, menu), pakai `GridView`. Mirip `ListView`, ada beberapa varian:

### GridView.count — Grid dengan Jumlah Kolom Fix

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MaterialApp(home: GridBasicScreen()));

class GridBasicScreen extends StatelessWidget {
  const GridBasicScreen({super.key});

  final List<Map<String, dynamic>> _products = const [
    {'icon': Icons.phone_android, 'name': 'Smartphone', 'color': Colors.blue},
    {'icon': Icons.laptop, 'name': 'Laptop', 'color': Colors.teal},
    {'icon': Icons.watch, 'name': 'Smartwatch', 'color': Colors.orange},
    {'icon': Icons.headphones, 'name': 'Headset', 'color': Colors.purple},
    {'icon': Icons.camera_alt, 'name': 'Kamera', 'color': Colors.red},
    {'icon': Icons.tablet, 'name': 'Tablet', 'color': Colors.green},
    {'icon': Icons.speaker, 'name': 'Speaker', 'color': Colors.brown},
    {'icon': Icons.keyboard, 'name': 'Keyboard', 'color': Colors.indigo},
    {'icon': Icons.mouse, 'name': 'Mouse', 'color': Colors.pink},
    {'icon': Icons.monitor, 'name': 'Monitor', 'color': Colors.cyan},
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('GridView Katalog Produk'),
        centerTitle: true,
      ),
      body: GridView.count(
        crossAxisCount: 2,           // 2 kolom
        crossAxisSpacing: 12,        // jarak horizontal antar item
        mainAxisSpacing: 12,         // jarak vertikal antar item
        padding: const EdgeInsets.all(16),
        children: _products.map((product) {
          return Card(
            elevation: 2,
            shape: RoundedRectangleBorder(
              borderRadius: BorderRadius.circular(16),
            ),
            child: InkWell(
              borderRadius: BorderRadius.circular(16),
              onTap: () {
                ScaffoldMessenger.of(context).showSnackBar(
                  SnackBar(content: Text('Kamu pilih: ${product['name']}')),
                );
              },
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Icon(
                    product['icon'] as IconData,
                    size: 48,
                    color: product['color'] as Color,
                  ),
                  const SizedBox(height: 12),
                  Text(
                    product['name'] as String,
                    style: const TextStyle(
                      fontWeight: FontWeight.w600,
                      fontSize: 16,
                    ),
                  ),
                ],
              ),
            ),
          );
        }).toList(),
      ),
    );
  }
}
```

### GridView.builder — Buat Data Banyak

Sama seperti `ListView.builder`, ada `GridView.builder` untuk data yang banyak:

```dart
GridView.builder(
  gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
    crossAxisCount: 3,       // 3 kolom
    crossAxisSpacing: 8,
    mainAxisSpacing: 8,
    childAspectRatio: 0.85,  // ratio width:height (0.85 = lebih tinggi dari lebar)
  ),
  itemCount: _products.length,
  itemBuilder: (context, index) {
    final product = _products[index];
    return Card(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Icon(product['icon'] as IconData, size: 36, color: product['color'] as Color),
          const SizedBox(height: 8),
          Text(product['name'] as String, textAlign: TextAlign.center),
        ],
      ),
    );
  },
)
```

### GridDelegate — Kunci Flexibilitas Grid

Ada dua delegate penting:

| Delegate | Fungsi |
|----------|--------|
| `SliverGridDelegateWithFixedCrossAxisCount` | Jumlah kolom fix (2, 3, 4 kolom) |
| `SliverGridDelegateWithMaxCrossAxisExtent` | Lebar maksimal per item fix, jumlah kolom menyesuaikan |

Contoh `MaxCrossAxisExtent` (responsif!):

```dart
SliverGridDelegateWithMaxCrossAxisExtent(
  maxCrossAxisExtent: 180,   // maks 180 pixel per item → di layar lebar bisa 4 kolom
  crossAxisSpacing: 10,
  mainAxisSpacing: 10,
  childAspectRatio: 1.0,     // kotak sempurna
)
```

Ini berguna banget buat layout responsif: di HP muncul 2 kolom, di tablet 4 kolom — otomatis tanpa media query.

---

## 5. Horizontal Scroll — Bikin Carousel ala Netflix

ListView dan GridView juga bisa horizontal! Tinggal tambah `scrollDirection: Axis.horizontal`:

```dart
SizedBox(
  height: 180,
  child: ListView.builder(
    scrollDirection: Axis.horizontal,
    itemCount: _products.length,
    padding: const EdgeInsets.symmetric(horizontal: 12),
    itemBuilder: (context, index) {
      final product = _products[index];
      return Container(
        width: 140,
        margin: const EdgeInsets.only(right: 12),
        decoration: BoxDecoration(
          color: (product['color'] as Color).withOpacity(0.1),
          borderRadius: BorderRadius.circular(16),
          border: Border.all(color: product['color'] as Color, width: 2),
        ),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(product['icon'] as IconData, size: 48, color: product['color'] as Color),
            const SizedBox(height: 8),
            Text(product['name'] as String, textAlign: TextAlign.center),
          ],
        ),
      );
    },
  ),
)
```

Bungkus dengan `SizedBox(height: ...)` karena horizontal ListView butuh constraint tinggi dari parent.

---

## 6. Pull-to-Refresh — Sentuhan Akhir

Tambahkan `RefreshIndicator` supaya user bisa swipe-down untuk refresh:

```dart
RefreshIndicator(
  onRefresh: () async {
    // panggil API / reload data
    await Future.delayed(const Duration(seconds: 1));
  },
  child: ListView.builder(
    itemCount: _contacts.length,
    itemBuilder: (context, index) { ... },
  ),
)
```

---

## 7. Perbandingan Lengkap: Kapan Pakai yang Mana?

| Kasus | Widget | Alasan |
|-------|--------|--------|
| List pendek, fix (< 20 item) | `ListView(children: [...])` | Simpel, gak perlu builder |
| List panjang, banyak data | `ListView.builder` | Lazy loading, performa optimal |
| List dengan divider | `ListView.separated` | `separatorBuilder` built-in |
| Grid jumlah kolom fix | `GridView.count` / `.builder` + `FixedCrossAxisCount` | Kolom statis |
| Grid responsif | `.builder` + `MaxCrossAxisExtent` | Kolom menyesuaikan lebar layar |
| Carousel / horizontal scroll | `ListView(horizontal)` / `.builder` | Bungkus dengan `SizedBox(height)` |
| Galeri foto / grid padat | `GridView.builder` | Hanya render yang terlihat |

---

## 8. Common Mistakes (Jangan Diulang!)

### ❌ ListView di dalam Column tanpa Expanded

```dart
// ERROR: ListView gak tahu tinggi maksimalnya
Column(
  children: [
    Text('Judul'),
    ListView.builder(...),  // ❌ unbounded height error!
  ],
)

// ✅ BENER: bungkus dengan Expanded
Column(
  children: [
    Text('Judul'),
    Expanded(
      child: ListView.builder(...),  // ✅ dapat constraint tinggi
    ),
  ],
)
```

### ❌ Pakai `ListView(children: [])` untuk ratusan item

Kalau datanya dari API, **selalu** pakai `.builder`. Memory dan performa kamu akan berterima kasih.

### ❌ Lupa `itemCount` di `ListView.builder`

Tanpa `itemCount`, Flutter anggap list-nya infinite. Scroll bar bisa aneh, dan kalau index out of bounds → crash.

---

## 9. Latihan Mandiri

Coba implementasikan tiga latihan ini sendiri:

1. **Daftar kontak dari JSON**: Bikin model class `Contact`, parse JSON lokal, tampilkan di `ListView.separated` dengan avatar, nama, dan nomor telepon.
2. **Katalog produk e-commerce**: `GridView.builder` 2 kolom dengan gambar, nama produk, harga, dan rating bintang.
3. **Horizontal story carousel**: `ListView` horizontal menampilkan circle avatar dengan nama di bawahnya, seperti Instagram Stories.

---

## Penutup

Hari ini kita udah menguasai dua widget paling fundamental untuk konten scrollable: **ListView** dan **GridView**. Dari yang simpel (`children: []`) sampai yang performa juara (`.builder`), dari scroll vertikal ke horizontal, dari list ke grid responsif — semuanya udah di tangan kamu.

Materi ini bakal terus kepakai di sepanjang perjalanan Flutter kamu. Feed berita? Pakai ListView. Katalog produk? Pakai GridView. Stories? Horizontal ListView. Inbox chat? ListView lagi.

Di artikel berikutnya kita akan bahas **Navigasi**: pindah-pindah halaman dengan `Navigator.push`, named routes, dan passing data antar screen. Stay tuned!

Coba sendiri! Implementasikan tiga latihan di atas, share hasilnya ke sosial media, dan tag **@ahsai001** 🚀
