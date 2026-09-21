---
layout: post
title: "Flutter #43: Animasi Implicit vs Explicit"
date: 2026-09-22 07:00:00 +0700
tags: [flutter,animasi,dart,tutorial,indonesia]
---

# Animasi Implicit vs Explicit

Animasi membuat aplikasi terasa hidup. Flutter menyediakan dua pendekatan: **Implicit** (paket dengan widget) dan **Explicit** (controller + `Animation`).

## Implicit Animation

Widget‑widget seperti `AnimatedContainer`, `AnimatedOpacity`, `AnimatedAlign` otomatis meng‑interpolasi nilai ketika properti berubah. Cukup ubah nilai di `setState`, Flutter mengurus transisinya.

```dart
class ImplicitDemo extends StatefulWidget {
  @override
  _ImplicitDemoState createState() => _ImplicitDemoState();
}

class _ImplicitDemoState extends State<ImplicitDemo> {
  bool _large = false;
  @override
  Widget build(BuildContext context) {
    return Center(
      child: GestureDetector(
        onTap: () => setState(() => _large = !_large),
        child: AnimatedContainer(
          width: _large ? 200 : 100,
          height: _large ? 200 : 100,
          color: _large ? Colors.blue : Colors.red,
          alignment: _large ? Alignment.centerRight : Alignment.centerLeft,
          duration: Duration(seconds: 1),
          curve: Curves.easeInOut,
        ),
      ),
    );
  }
}
```

Tombol di atas meng‑toggle ukuran, warna, dan posisi dalam satu detik.

## Explicit Animation

Untuk kontrol granular—misalnya memulai, menghentikan, atau mengulang animasi—pakai `AnimationController` + `Tween`.

```dart
class ExplicitDemo extends StatefulWidget {
  @override
  _ExplicitDemoState createState() => _ExplicitDemoState();
}

class _ExplicitDemoState extends State<ExplicitDemo> with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _scale;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(vsync: this, duration: Duration(seconds: 2))
      ..repeat(reverse: true);
    _scale = Tween(begin: 0.5, end: 1.5).animate(CurvedAnimation(parent: _controller, curve: Curves.elasticOut));
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Center(
      child: ScaleTransition(
        scale: _scale,
        child: Container(width: 100, height: 100, color: Colors.green),
      ),
    );
  }
}
```

`AnimationController` memberi kemampuan **repeat**, **reverse**, **stop**, dan **status listener**.

## Kapan Pakai yang Mana?

| Situasi | Pilihan |
|---|---|
|Animasi sederhana, satu‑dua properti|Implicit (lebih singkat, minim boilerplate) |
|Animasi berulang, sinkronisasi dengan logika|Explicit (kontrol penuh) |
|Integrasi dengan `Hero`, `PageRouteBuilder`|Explicit (butuh `Tween` dan `Curve`) |

## Tips Performansi

- Gunakan `Implicit` bila perubahan terjadi **jarang**; Flutter akan membuat `RenderObject` baru hanya saat nilai berubah.
- Pada `Explicit`, daur hidup `AnimationController` harus **dispose** agar tidak memory‑leak.
- Hindari `setState` di dalam builder `AnimatedBuilder` bila tidak perlu; gunakan widget khusus (`AnimatedOpacity`, dsb.).

## Kesimpulan

Implicit memudahkan prototyping; Explicit memberi kebebasan kreatif untuk animasi kompleks. Kombinasikan keduanya untuk UI yang responsif dan memukau.

**Coba sendiri! Share ke sosial media dan tag @ahsai001**
