---
layout: post
title: "Flutter #27: Hero Animation & Page Transition — Transisi Antar Halaman yang Smooth"
date: 2026-09-05 07:00:00 +0700
tags: [flutter, dart, tutorial, animation, hero, page-transition, UI]
description: "Pelajari cara membuat Hero Animation dan custom Page Transition di Flutter agar transisi antar halaman terasa smooth, natural, dan eye-catching."
---

# Flutter #27: Hero Animation & Page Transition — Transisi Antar Halaman yang Smooth

Halo! Di artikel #27 ini kita akan belajar tentang **Hero Animation** dan **Page Transition** di Flutter. Dua fitur ini bikin user experience app kamu naik level — dari "bisa dipakai" jadi "enak dipandang dan dipakai."

Kalau selama ini halaman kamu cuma ngeslide dari samping, hari ini kita akan upgrade transisinya jadi jauh lebih smooth dan profesional. Let's go!

---

## 1. Hero Animation — Terbang dari Halaman ke Halaman

`Hero Animation` adalah animasi yang bikin sebuah widget "terbang" dari posisi di halaman A ke posisi di halaman B. Contoh paling umum: gambar thumbnail di list → membesar jadi gambar full di detail page.

### Cara Kerja Hero

1. Di halaman asal, bungkus widget dengan `Hero(tag: 'unik-id')`
2. Di halaman tujuan, bungkus widget serupa dengan `Hero(tag: 'sama')`
3. Flutter otomatis animasikan transisi posisi + ukuran

### Contoh Dasar Hero Animation

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Hero Demo',
      theme: ThemeData(colorSchemeSeed: Colors.teal, useMaterial3: true),
      home: const GalleryPage(),
    );
  }
}

// Halaman Galeri — daftar gambar
class GalleryPage extends StatelessWidget {
  const GalleryPage({super.key});

  // Data dummy gambar
  final List<Map<String, String>> photos = const [
    {'tag': 'photo1', 'color': '#FF6B6B', 'title': 'Sunset'},
    {'tag': 'photo2', 'color': '#4ECDC4', 'title': 'Ocean'},
    {'tag': 'photo3', 'color': '#45B7D1', 'title': 'Sky'},
    {'tag': 'photo4', 'color': '#96CEB4', 'title': 'Forest'},
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Galeri Foto')),
      body: GridView.builder(
        padding: const EdgeInsets.all(12),
        gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 2,
          crossAxisSpacing: 12,
          mainAxisSpacing: 12,
        ),
        itemCount: photos.length,
        itemBuilder: (context, index) {
          final photo = photos[index];
          return GestureDetector(
            onTap: () => Navigator.push(
              context,
              MaterialPageRoute(
                builder: (_) => DetailPage(
                  tag: photo['tag']!,
                  color: photo['color']!,
                  title: photo['title']!,
                ),
              ),
            ),
            child: Hero(
              tag: photo['tag']!,
              child: Container(
                decoration: BoxDecoration(
                  color: Color(int.parse(photo['color']!.replaceFirst('#', '0xFF'))),
                  borderRadius: BorderRadius.circular(16),
                ),
                alignment: Alignment.center,
                child: Text(
                  photo['title']!,
                  style: const TextStyle(
                    color: Colors.white,
                    fontSize: 18,
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ),
            ),
          );
        },
      ),
    );
  }
}

// Halaman Detail — tampilan penuh
class DetailPage extends StatelessWidget {
  final String tag;
  final String color;
  final String title;

  const DetailPage({
    super.key,
    required this.tag,
    required this.color,
    required this.title,
  });

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomScrollView(
        slivers: [
          SliverAppBar(
            expandedHeight: 300,
            pinned: true,
            flexibleSpace: FlexibleSpaceBar(
              title: Text(title),
              background: Hero(
                tag: tag,
                child: Container(
                  color: Color(int.parse(color.replaceFirst('#', '0xFF'))),
                  alignment: Alignment.center,
                  child: Icon(
                    Icons.photo,
                    size: 80,
                    color: Colors.white.withValues(alpha: 0.5),
                  ),
                ),
              ),
            ),
          ),
          SliverToBoxAdapter(
            child: Padding(
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    'Tentang $title',
                    style: Theme.of(context).textTheme.headlineSmall,
                  ),
                  const SizedBox(height: 12),
                  const Text(
                    'Ini adalah halaman detail foto. Hero animation membuat gambar '
                    'terbang dari thumbnail di galeri ke posisi full di sini. '
                    'Animasi otomatis oleh Flutter — cukup pakai Hero(tag: ...) '
                    'di kedua halaman!',
                    style: TextStyle(fontSize: 16, height: 1.5),
                  ),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

**Yang perlu diperhatikan:**

- **`tag` harus unik per hero** — Setiap Hero punya tag yang sama di halaman A dan B. Jangan pakai tag yang sama untuk dua hero berbeda!
- **Widget tidak harus identik** — Yang penting tag-nya sama. Bisa Thumbnail di halaman A → gambar besar di halaman B.
- **Tanpa Navigator.push perlu** — Hero animasi otomatis jalan setiap kali kamu navigasi ke halaman yang punya widget Hero dengan tag yang sama.

---

## 2. Custom Page Transition — Upgrade dari Default Slide

Secara default, `MaterialPageRoute` bikin halaman geser dari kanan. Tapi kamu bisa bikin transisi yang lebih keren: fade, scale, rotate, atau kombinasi.

### Contoh: Custom Fade + Slide Transition

```dart
// Custom page route dengan fade + slide
class FadeSlidePageRoute<T> extends PageRouteBuilder<T> {
  final Widget page;

  FadeSlidePageRoute({required this.page})
      : super(
          transitionDuration: const Duration(milliseconds: 400),
          pageBuilder: (context, animation, secondaryAnimation) => page,
          transitionsBuilder: (context, animation, secondaryAnimation, child) {
            // Tween untuk slide dari bawah
            final slideTween = Tween<Offset>(
              begin: const Offset(0, 0.1),
              end: Offset.zero,
            ).chain(CurvedAnimation(
              parent: animation,
              curve: Curves.easeOutCubic,
            ));

            // Tween untuk fade
            final fadeTween = Tween<double>(begin: 0.0, end: 1.0);

            return SlideTransition(
              position: slideTween,
              child: FadeTransition(
                opacity: fadeTween.animate(animation),
                child: child,
              ),
            );
          },
        );
}

// Cara pakai:
// Navigator.push(context, FadeSlidePageRoute(page: TargetPage()));
```

### Contoh: Scale Transition (Zoom In)

```dart
class ScalePageRoute<T> extends PageRouteBuilder<T> {
  final Widget page;

  ScalePageRoute({required this.page})
      : super(
          transitionDuration: const Duration(milliseconds: 350),
          pageBuilder: (context, animation, secondaryAnimation) => page,
          transitionsBuilder: (context, animation, secondaryAnimation, child) {
            final scaleTween = Tween<double>(begin: 0.8, end: 1.0)
                .chain(CurveTween(curve: Curves.easeOutBack));

            final fadeTween = Tween<double>(begin: 0.0, end: 1.0);

            return ScaleTransition(
              scale: scaleTween.animate(animation),
              child: FadeTransition(
                opacity: fadeTween.animate(animation),
                child: child,
              ),
            );
          },
        );
}

// Pakai untuk detail card:
// Navigator.push(context, ScalePageRoute(page: DetailPage()));
```

---

## 3. Contoh Lengkap: App dengan Hero + Custom Transition

Berikut app sederhana yang menggabungkan Hero Animation dengan Custom Page Transition:

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Fancy Transitions',
      theme: ThemeData(colorSchemeSeed: Colors.deepPurple, useMaterial3: true),
      home: const HomePage(),
    );
  }
}

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  final List<Map<String, dynamic>> items = const [
    {'id': 1, 'icon': Icons.pets, 'label': 'Animals', 'color': 0xFFFF6B6B},
    {'id': 2, 'icon': Icons.restaurant, 'label': 'Food', 'color': 0xFF4ECDC4},
    {'id': 3, 'icon': Icons.flight, 'label': 'Travel', 'color': 0xFF45B7D1},
    {'id': 4, 'icon': Icons.music_note, 'label': 'Music', 'color': 0xFFF7DC6F},
    {'id': 5, 'icon': Icons.brush, 'label': 'Art', 'color': 0xFFBB8FCE},
    {'id': 6, 'icon': Icons.code, 'label': 'Code', 'color': 0xFF82E0AA},
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Fancy Transitions')),
      body: ListView.builder(
        padding: const EdgeInsets.all(16),
        itemCount: items.length,
        itemBuilder: (context, index) {
          final item = items[index];
          return Card(
            margin: const EdgeInsets.only(bottom: 12),
            child: ListTile(
              onTap: () {
                Navigator.push(
                  context,
                  FadeSlidePageRoute(
                    page: ItemDetailPage(item: item),
                  ),
                );
              },
              leading: Hero(
                tag: 'item-${item['id']}',
                child: CircleAvatar(
                  backgroundColor: Color(item['color']),
                  radius: 28,
                  child: Icon(item['icon'] as IconData, color: Colors.white),
                ),
              ),
              title: Text(item['label'] as String, style: const TextStyle(fontWeight: FontWeight.w600)),
              subtitle: const Text('Tap untuk lihat detail'),
              trailing: const Icon(Icons.chevron_right),
            ),
          );
        },
      ),
    );
  }
}

class ItemDetailPage extends StatelessWidget {
  final Map<String, dynamic> item;

  const ItemDetailPage({super.key, required this.item});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(item['label'] as String)),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Hero(
              tag: 'item-${item['id']}',
              child: CircleAvatar(
                backgroundColor: Color(item['color']),
                radius: 60,
                child: Icon(item['icon'] as IconData, color: Colors.white, size: 48),
              ),
            ),
            const SizedBox(height: 24),
            Text(
              item['label'] as String,
              style: Theme.of(context).textTheme.headlineMedium?.copyWith(
                    fontWeight: FontWeight.bold,
                  ),
            ),
            const SizedBox(height: 12),
            const Text(
              'Hero animation membuat icon terbang dari list ke sini.\n'
              'Halaman ini dibuka dengan FadeSlide transition.',
              textAlign: TextAlign.center,
              style: TextStyle(fontSize: 16, color: Colors.grey),
            ),
          ],
        ),
      ),
    );
  }
}

// FadeSlidePageRoute (definisi lengkap di atas)
class FadeSlidePageRoute<T> extends PageRouteBuilder<T> {
  final Widget page;

  FadeSlidePageRoute({required this.page})
      : super(
          transitionDuration: const Duration(milliseconds: 400),
          pageBuilder: (context, animation, secondaryAnimation) => page,
          transitionsBuilder: (context, animation, secondaryAnimation, child) {
            final slideTween = Tween<Offset>(
              begin: const Offset(0, 0.1),
              end: Offset.zero,
            ).chain(CurvedAnimation(
              parent: animation,
              curve: Curves.easeOutCubic,
            ));
            final fadeTween = Tween<double>(begin: 0.0, end: 1.0);
            return SlideTransition(
              position: slideTween,
              child: FadeTransition(
                opacity: fadeTween.animate(animation),
                child: child,
              ),
            );
          },
        );
}
```

---

## 4. Page Transition Bawaan Flutter

Selain custom route, Flutter juga punya beberapa transisi bawaan yang bisa kamu pakai tanpa bikin custom `PageRouteBuilder`:

```dart
// Fade transition
Navigator.push(context, PageRouteBuilder(
  pageBuilder: (_, __, ___) => const TargetPage(),
  transitionsBuilder: (_, animation, __, child) =>
    FadeTransition(opacity: animation, child: child),
));

// Scale transition
Navigator.push(context, PageRouteBuilder(
  pageBuilder: (_, __, ___) => const TargetPage(),
  transitionsBuilder: (_, animation, __, child) =>
    ScaleTransition(scale: animation, child: child),
));

// Rotation transition
Navigator.push(context, PageRouteBuilder(
  pageBuilder: (_, __, ___) => const TargetPage(),
  transitionsBuilder: (_, animation, __, child) =>
    RotationTransition(turns: animation, child: child),
));
```

**Tip:** Untuk transisi sederhana, cara inline di atas sudah cukup. Kalau transisi-nya dipakai di banyak tempat, buat class sendiri seperti `FadeSlidePageRoute` agar tidak copy-paste.

---

## Recap Hari Ini

| Teknik | Fungsi | Kapan Pakai |
|--------|--------|-------------|
| **Hero Animation** | Widget terbang antar halaman | Thumbnail → Detail, avatar → Profile |
| **PageRouteBuilder** | Transisi custom per navigasi | Fade, scale, rotate, atau kombinasi |
| **SlideTransition** | Geser dari posisi tertentu | Bottom-to-top, left-to-right |
| **FadeTransition** | Opacity dari 0 → 1 | Transisi subtle dan elegan |
| **ScaleTransition** | Zoom in/out | Popup feel, card expand |

**Yang harus diingat:**
- Hero `tag` harus unik dan sama di kedua halaman
- Widget Hero tidak harus identis — yang penting tag-nya match
- Custom `PageRouteBuilder` beri `transitionDuration` agar timing pas
- Kombinasikan `SlideTransition` + `FadeTransition` untuk hasil yang lebih polished
- `Curves.easeOutCubic` dan `Curves.easeOutBack` bikin animasi terasa natural

---

Di artikel #28 kita akan belajar tentang **Custom Widget & Reusable Component** — bikin widget sendiri yang bisa dipakai ulang di seluruh app! 🧩

---

Coba sendiri! Bikin galeri foto sederhana dengan Hero Animation. Share hasilnya ke sosial media dan tag **@ahsai001**! 🚀
