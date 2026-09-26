---
layout: post
title: "Flutter #45: Sliver & CustomScrollView"
date: 2026-09-27 08:00:00 +0700
tags: [flutter, sliver, customscrollview, layout, tutorial]
---

# Menguasai Sliver & CustomScrollView

Jika kamu sudah nyaman dengan `ListView` atau `GridView`, saatnya naik level. `Sliver` dan `CustomScrollView` memberi kontrol penuh atas perilaku scrolling, memungkinkan efek **parallax**, **sticky header**, dan layout kompleks yang tetap efisien.

## Apa itu Sliver?

`Sliver` adalah potongan scrollable yang dapat digabungkan dalam satu `CustomScrollView`. Setiap Sliver menghasilkan **sliver geometry** yang memberi tahu engine berapa ruang yang dibutuhkan, apakah bisa di‑shrink, dsb. Flutter menyediakan beberapa built‑in:

- `SliverAppBar` – app bar yang dapat memperluas atau menempel saat scroll.
- `SliverList` – list berbasis lazy‑loading.
- `SliverGrid` – grid yang sama seperti `GridView`.
- `SliverToBoxAdapter` – membungkus widget biasa menjadi sliver.

## Struktur Dasar

```dart
class MySliverPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomScrollView(
        slivers: [
          // SliverAppBar, SliverList, dll
        ],
      ),
    );
  }
}
```

`CustomScrollView` menerima list `slivers`. Urutan penting: `SliverAppBar` biasanya paling atas, diikuti konten.

## Contoh 1 – Sticky Header dengan SliverAppBar

```dart
CustomScrollView(
  slivers: [
    SliverAppBar(
      pinned: true, // tetap menempel di atas saat scroll
      expandedHeight: 200.0,
      flexibleSpace: FlexibleSpaceBar(
        title: Text('Flutter Sliver Demo'),
        background: Image.asset('assets/header.jpg', fit: BoxFit.cover),
      ),
    ),
    SliverList(
      delegate: SliverChildBuilderDelegate(
        (context, index) => ListTile(title: Text('Item #$index')),
        childCount: 30,
      ),
    ),
  ],
);
```

Hasil: app bar memperluas dengan gambar, lalu **menempel** setelah mencapai puncak.

## Contoh 2 – Parallax & Grid dengan SliverGrid

```dart
CustomScrollView(
  slivers: [
    SliverAppBar(
      expandedHeight: 250,
      flexibleSpace: FlexibleSpaceBar(
        title: Text('Parallax Grid'),
        background: Image.network(
          'https://picsum.photos/800/600',
          fit: BoxFit.cover,
        ),
        stretchModes: [StretchMode.zoomBackground],
      ),
      pinned: true,
    ),
    SliverPadding(
      padding: EdgeInsets.all(8.0),
      sliver: SliverGrid(
        gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 2,
          mainAxisSpacing: 8,
          crossAxisSpacing: 8,
          childAspectRatio: 1,
        ),
        delegate: SliverChildBuilderDelegate(
          (context, index) => Container(
            color: Colors.primaries[index % Colors.primaries.length],
            child: Center(child: Text('Box $index')),
          ),
          childCount: 12,
        ),
      ),
    ),
    SliverToBoxAdapter(
      child: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Text('Akhir konten. Scroll kembali ke atas untuk melihat efek sticky.'),
      ),
    ),
  ],
);
```

`SliverGrid` menampilkan kotak 2 kolom, sementara `SliverAppBar` memberikan efek **parallax** ketika digulir.

## Tips Praktis

1. **Gunakan `SliverFillRemaining`** bila ingin mengisi ruang kosong setelah konten utama.
2. **Jangan campur `ListView` di dalam `CustomScrollView`** – menyebabkan konflik scroll. Jika butuh list biasa, ubah menjadi `SliverList`.
3. **Performance:** semua sliver bersifat lazy; widget hanya dibuat ketika masuk viewport.
4. **Kombinasi:** `SliverToBoxAdapter` memungkinkan menaruh widget biasa (misal banner) di tengah sliver list.

## Debugging Layout

Jika tampilan tidak seperti yang diharapkan, periksa:
- `pinned` vs `floating` pada `SliverAppBar`.
- `shrinkWrap` tidak diperlukan karena sliver sudah mengatur ukuran.
- Pastikan `CustomScrollView` tidak berada dalam widget scrollable lain.

## CTA

Coba sendiri! Buat halaman baru dengan `CustomScrollView` dan eksperimen dengan `SliverAppBar`, `SliverGrid`, serta `SliverToBoxAdapter`. Share ke sosial media dan tag @ahsai001.

---
