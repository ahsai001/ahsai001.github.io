---
layout: post
title: "Custom Painter & Canvas"
date: 2026-09-23 07:00:00 +0700
tags: [flutter, custom painter, canvas, grafik]
---

# Custom Painter & Canvas

Di Flutter, `CustomPainter` memberi Anda kebebasan melukis apa saja di kanvas. Cocok untuk grafik dinamis, animasi, atau UI unik.

## 1. Struktur dasar

```dart
import 'package:flutter/material.dart';

class MyCanvas extends StatelessWidget {
  const MyCanvas({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return CustomPaint(
      size: const Size(300, 300), // lebar × tinggi
      painter: ShapePainter(),
    );
  }
}

class ShapePainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..color = Colors.blue
      ..strokeWidth = 4
      ..style = PaintingStyle.stroke;
    // gambar lingkaran di tengah
    canvas.drawCircle(Offset(size.width/2, size.height/2), 80, paint);
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => false;
}
```

`CustomPaint` menempelkan `ShapePainter` pada kanvas berukuran 300×300. `paint`‑nya menggambar lingkaran biru.

## 2. Menggambar bentuk kompleks

Anda dapat menggabungkan path, gradien, dan gambar bitmap.

```dart
class ComplexPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    final path = Path()
      ..moveTo(0, size.height)
      ..quadraticBezierTo(size.width/2, 0, size.width, size.height)
      ..close();

    final gradient = LinearGradient(
      colors: [Colors.purple, Colors.orange],
    ).createShader(Rect.fromLTWH(0, 0, size.width, size.height));

    final paint = Paint()
      ..shader = gradient
      ..style = PaintingStyle.fill;

    canvas.drawPath(path, paint);
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => false;
}
```

`ComplexPainter` membuat kurva Bézier berbentuk gelombang dan mengisi dengan gradien.

## 3. Responsif dengan `MediaQuery`

Gunakan `size` yang diberikan oleh `CustomPaint` atau `MediaQuery.of(context).size` untuk menyesuaikan skala pada berbagai layar.

```dart
class ResponsivePainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    final radius = size.shortestSide * 0.3;
    final paint = Paint()..color = Colors.green;
    canvas.drawCircle(size.center(Offset.zero), radius, paint);
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => false;
}
```

## 4. Animasi dengan `AnimationController`

Jika `shouldRepaint` mengembalikan `true` saat nilai animasi berubah, kanvas akan di‑redraw tiap frame.

```dart
class AnimatedCircle extends StatefulWidget {
  const AnimatedCircle({Key? key}) : super(key: key);
  @override
  _AnimatedCircleState createState() => _AnimatedCircleState();
}

class _AnimatedCircleState extends State<AnimatedCircle>
    with SingleTickerProviderStateMixin {
  late final AnimationController _ctrl = AnimationController(
    vsync: this,
    duration: const Duration(seconds: 2),
  )..repeat(reverse: true);

  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: _ctrl,
      builder: (_, __) => CustomPaint(
        size: const Size(200, 200),
        painter: _CirclePainter(_ctrl.value),
      ),
    );
  }

  @override
  void dispose() {
    _ctrl.dispose();
    super.dispose();
  }
}

class _CirclePainter extends CustomPainter {
  final double progress;
  _CirclePainter(this.progress);
  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..color = Colors.red
      ..style = PaintingStyle.fill;
    final radius = 30 + 70 * progress; // 30‑100 piksel
    canvas.drawCircle(size.center(Offset.zero), radius, paint);
  }
  @override
  bool shouldRepaint(covariant _CirclePainter old) => old.progress != progress;
}
```

Animasi mengubah radius lingkaran dari 30 ke 100 piksel secara kontinu.

## 5. Tips & Trik

- **Cache**: Gunakan `RepaintBoundary` bila gambar statis besar agar tidak di‑repaint setiap frame.
- **Performance**: Hindari operasi berat di dalam `paint`; kalkulasi dulu di `initState` atau `shouldRepaint`.
- **Testing**: `CustomPainter` dapat diuji dengan `painter.testPaint` (package:flutter_test).

## Penutup

`CustomPainter` & `Canvas` membuka kemungkinan tak terbatas untuk UI kreatif. Mulailah dengan contoh di atas, eksplorasi path, gradien, serta animasi. 

**Coba sendiri! Share ke sosial media dan tag @ahsai001**
