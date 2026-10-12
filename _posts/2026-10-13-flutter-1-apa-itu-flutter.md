---
layout: post
title: "Flutter #1: Apa itu Flutter & Kenapa Harus Flutter"
date: 2026-10-13 07:00:00 +0700
tags: [flutter, dart, tutorial, indonesia, pemrograman]
---

Flutter adalah framework UI open-source dari Google untuk membangun aplikasi **mobile, web, dan desktop** dari **satu codebase**. Berbeda dengan React Native atau Ionic yang menggunakan bridge ke native, Flutter merender UI sendiri dengan engine **Skia** (sekarang **Impeller**) — hasilnya: performa mendekati native, UI konsisten di semua platform, dan *hot reload* beneran cepat.

---

## Kenapa Flutter?

| Alasan | Penjelasan |
|--------|------------|
| **Single codebase** | Tulis sekali, jalanin di iOS, Android, Web, Windows, macOS, Linux |
| **Hot Reload** | Perubahan kode terefleksi < 1 detik, tanpa restart app |
| **Widget-based** | Semua itu widget — dari tombol sampai layout, customizable total |
| **Dart** | Bahasa modern, sound null safety, AOT + JIT compilation |
| **Ecosystem** | 40k+ packages di pub.dev, Firebase first-class support |
| **Comunity & Jobs** | Dipakai Gojek, Tokopedia, Traveloka, BMW, eBay, dll. |

---

## Flutter vs Alternatif

```dart
// React Native: bridge ke native components
<View style={styles.container}>
  <Text>Hello</Text>
</View>

// Flutter: render sendiri, zero bridge
Container(
  child: Text('Hello'),
)
```

| Fitur | Flutter | React Native | Native (Kotlin/Swift) |
|-------|---------|--------------|----------------------|
| Codebase | 1 | 1 (tapi butuh native module) | 2 (iOS + Android) |
| Performa | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| UI Konsisten | Ya | Butuh effort | Platform-specific |
| Learning Curve | Sedang | Rendah (JS) | Tinggi (2 bahasa) |

---

## Arsitektur Singkat

```
┌─────────────────────────────────────┐
│           Your Dart Code            │
├─────────────────────────────────────┤
│         Flutter Framework           │  ← Widgets, Rendering, Gestures
├─────────────────────────────────────┤
│            Engine (C++)             │  ← Skia/Impeller, Dart VM
├─────────────────────────────────────┤
│           Embedder (Platform)       │  ← Android/iOS/Web/Desktop shell
└─────────────────────────────────────┘
```

- **Framework**: Pure Dart, semua widget di sini
- **Engine**: C++, handle rendering, text shaping, plugin channel
- **Embedder**: Platform-specific entry point (Activity, UIViewController, dll.)

---

## Cocok Buat Siapa?

- **Pemula** — Dart mirip Java/JS, dokumentasi lengkap, error message jelas
- **Startup/Solo dev** — Cepat MVP, tim kecil handle multi-platform
- **Enterprise** — Google support, stable release 3+ tahun, LTS tersedia
- **Desainer** — Pixel-perfect control, Material 3 + Cupertino built-in

---

## Mulai Hari Ini

Artikel selanjutnya: **Install Flutter SDK & Setup IDE (VS Code/Android Studio)**. Kita bahas instalasi lengkap, `flutter doctor`, dan setup VS Code biar nyaman coding.

---

Coba sendiri! Share ke sosial media dan tag @ahsai001