---
layout: post
title: "Flutter #1: Apa itu Flutter & Kenapa Harus Flutter"
date: 2026-08-04 07:00:00 +0700
tags: [flutter, dart, mobile, tutorial]
---

## Pengantar

Selamat datang di seri **"Flutter dari Nol sampai Expert"**! Ini adalah artikel pertama dari 90 artikel yang akan menemanimu dari benar-benar nol — bahkan belum pernah install Flutter — sampai bisa bikin aplikasi full-stack dan deploy ke production. Kenapa kita mulai dari "apa itu"? Karena sebelum nyemplung, penting buat paham *big picture*-nya dulu.

---

## Apa Itu Flutter?

Flutter adalah **UI toolkit open-source** buatan Google untuk membangun aplikasi yang dikompilasi secara native. Satu kode, banyak platform: **Android, iOS, Web, Windows, macOS, dan Linux**. Flutter pertama kali dirilis tahun 2017 (versi alpha) dan sekarang sudah versi 3.x yang sangat matang.

Bahasa pemrograman yang dipakai: **Dart** — juga buatan Google, didesain khusus untuk UI development.

Coba lihat perbandingan sederhana ini:

```dart
// React Native (JavaScript)
<View style={{flexDirection: 'row'}}>
  <Text>Halo</Text>
  <Text>Dunia</Text>
</View>

// Flutter (Dart)
Row(
  children: [
    Text('Halo'),
    Text('Dunia'),
  ],
)
```

Semua di Flutter adalah **widget** — dari `Text`, `Container`, `Row`, sampai `Scaffold`. Gak ada markup terpisah seperti XML atau HTML. Semuanya ditulis dalam Dart. Ini bikin kode lebih konsisten dan gampang dibaca.

---

## Kenapa Harus Flutter?

### 1. **Hot Reload — Fitur Paling Bikin Candu**

Hot Reload memungkinkan kamu lihat perubahan kode dalam **kurang dari 1 detik** tanpa harus rebuild ulang aplikasi. Ubah warna tombol? Simpan. Langsung muncul. Ini **super produktif** untuk iterasi desain UI.

```dart
// Ubah warna dari biru ke ungu, Ctrl+S, boom — langsung berubah!
ElevatedButton(
  style: ElevatedButton.styleFrom(
    backgroundColor: Colors.deepPurple, // ganti ini, hot reload!
  ),
  onPressed: () {},
  child: Text('Klik Aku'),
),
```

### 2. **Single Codebase, Multi-Platform**

Satu project Flutter bisa kamu build jadi:
- APK/AAB untuk Android
- IPA untuk iOS
- Web app (HTML/JS/CSS)
- Desktop app (Windows, macOS, Linux)

Bukan cuma "jalan", tapi **benar-benar native performance**. Flutter gak pakai bridge seperti React Native; dia render langsung ke canvas menggunakan engine Skia.

### 3. **Widget System yang Konsisten**

Material Design (Android-style) dan Cupertino (iOS-style) tersedia out-of-the-box. Mau tampilan yang persis native di kedua platform? Flutter bisa.

```dart
// Android-style switch
Switch(value: true, onChanged: (v) {})

// iOS-style switch
CupertinoSwitch(value: true, onChanged: (v) {})
```

### 4. **Ekosistem & Komunitas Besar**

- **pub.dev** punya 40.000+ package siap pakai
- Dokumentasi resmi Google sangat lengkap
- Komunitas global aktif — StackOverflow, Discord, Reddit, GitHub
- 500.000+ apps di Google Play dibuat dengan Flutter

### 5. **Performa Native**

Flutter compile ke ARM/x86 native code (bukan interpreted). Tidak ada JavaScript bridge. Jadi performanya mendekati aplikasi native murni — 60 FPS smooth, bahkan untuk animasi kompleks.

---

## Arsitektur Flutter (Gambaran Singkat)

```
┌─────────────────────────┐
│     Framework (Dart)     │  ← Kamu coding di sini
│  Material / Cupertino    │
│  Widgets / Rendering     │
├─────────────────────────┤
│     Engine (C++)         │  ← Skia, Dart VM, Text
├─────────────────────────┤
│    Embedder (Platform)   │  ← Android/iOS/Web/Desktop
└─────────────────────────┘
```

> **Gak perlu hafal sekarang** — nanti di artikel-artikel selanjutnya kamu bakal paham layer demi layer.

---

## Mitos & Fakta Flutter

| Mitos | Fakta |
|-------|-------|
| "Flutter cuma buat Android" | Flutter support **6 platform** |
| "Flutter lambat" | Flutter compile ke native code, 60 FPS |
| "Dart susah dipelajari" | Mirip Java/JavaScript/C#, 1-2 minggu udah nyaman |
| "Flutter gak cocok buat app besar" | Google Ads, Alibaba, BMW, Grab pakai Flutter |

---

## Siapa yang Pakai Flutter di Dunia Nyata?

- **Google Ads** — dashboard iklan
- **Alibaba** (Xianyu app) — 50+ juta pengguna
- **BMW** — My BMW App
- **Grab** — GrabMerchant, GrabFood
- **Tokopedia, Gojek** — beberapa micro-app internal
- **Reflectly** — journal app pemenang design award

---

## Latihan Hari Ini

Karena ini masih artikel perkenalan, latihanmu ringan aja:

1. **Google**: Cari 3 aplikasi di Play Store yang dibuat pakai Flutter selain yang disebutkan di atas
2. **Refleksi**: Tulis di notes kenapa *kamu* mau belajar Flutter — apa yang pengen kamu bikin?
3. **Install**: Kalau udah gak sabaran, install Flutter SDK (step-by-step di artikel besok!)

---

## Apa Selanjutnya?

Besok kita bakal **install Flutter SDK & setup IDE** — dari nol sampai project pertama berhasil jalan di emulator. Pastikan laptopmu siap: RAM minimal 8GB, storage kosong ~20GB.

> 🚀 Coba sendiri, kalau stuck tanya di kolom komentar! Jangan malu — semua developer hebat dulunya juga bingung pas awal.

---

*Seri "Flutter dari Nol sampai Expert" — artikel ke-1*
