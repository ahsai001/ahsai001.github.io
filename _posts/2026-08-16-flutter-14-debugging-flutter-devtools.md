---
layout: post
title: "Flutter #14: Debugging & Flutter DevTools — Nemuin Bug Kayak Detektif"
date: 2026-08-16 07:00:00 +0700
tags: [flutter, dart, debugging, devtools, debugprint, assert, tutorial, indonesia, pemrograman]
---

Selamat! Hari ini kita resmi **nutup Fase Fundamental**. Dari nol sampai bisa bikin app dengan tema, layout responsif, navigasi, dan form — kamu udah pegang semua fondasinya. Tapi ada satu skill yang nggak pernah diajarin di tutorial "bikin app pertama", padahal ini yang paling sering kamu pakai tiap hari: **debugging**.

Bug itu bukan tanda kamu payah. Bug itu **rutinitas**. Developer profesional habiskan sebagian besar waktunya bukan nulis kode baru, tapi **nemuin kenapa kode yang lama nggak jalan**. Makanya, mari belajar jadi detektif dengan dua senjata utama Flutter: **`debugPrint` + `assert`** dan **Flutter DevTools**.

---

## 1. `debugPrint` vs `print` — Kenapa Nggak Boleh Pakai `print`?

`print()` di Flutter punya satu masalah: dia bisa **dipotong**. Kalau output-nya kepanjangan (misal JSON besar), sistem operasi cuma nampilin sebagian dan sisanya hilang. Makanya Flutter nyediain `debugPrint` yang lebih aman dan bisa di-throttle.

```dart
import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  int hitungLuas(int panjang, int lebar) {
    // assert: cuma aktif di mode DEBUG, hilang di release
    assert(panjang > 0, 'Panjang harus lebih dari 0!');
    assert(lebar > 0, 'Lebar harus lebih dari 0!');
    return panjang * lebar;
  }

  @override
  Widget build(BuildContext context) {
    final luas = hitungLuas(10, 5);
    debugPrint('🔍 Luas persegi panjang: $luas');

    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: Center(child: Text('Luas: $luas')),
      ),
    );
  }
}
```

Coba ubah `hitungLuas(10, 5)` jadi `hitungLuas(-3, 5)`. Di mode debug, app bakal langsung **crash dengan pesan jelas** dari `assert` — bukan crash misterius. Di mode release, `assert` diabaikan total, jadi nggak ngeberatin performa.

> ⚠️ **`assert` itu buat nangkep kesalahan logic programmer, bukan buat validasi input user.** Validasi input user pakai form validator (udah kita bahas di artikel #10).

---

## 2. Baca Error Message dengan Benar

Pemula sering panik liat layar merah. Padahal layar merah itu **justru teman baikmu** — Flutter ngejelasin persis apa yang salah. Error paling umum:

**`RenderFlex overflowed`** — konten lebih besar dari tempatnya:

```
A RenderFlex overflowed by 45 pixels on the bottom.
```

Artinya: `Column` kamu kelebihan konten 45px. Solusinya: bungkus child yang panjang dengan `Expanded`, atau kasih `SingleChildScrollView`, atau kecilin ukuran.

**`Null check operator used on a null value`** — kamu pakai `!` di variabel yang ternyata `null`.

**`setState() called after dispose()`** — kamu manggil `setState` setelah widget dihapus dari layar (biasanya karena async call selesai telat). Solusinya: cek `if (!mounted) return;` sebelum `setState`.

Kuncinya: **jangan skip baca pesan error**. Scroll ke baris paling atas, baca pesan pertama — itu akar masalahnya.

---

## 3. Flutter DevTools — Markas Besar Detektif

DevTools adalah **suite alat debugging visual** yang nempel di browser. Cara bukanya ada beberapa:

- **VS Code**: jalankan debug (F5), terus klik ikon DevTools di toolbar / `Cmd+Shift+P` → "Dart: Open DevTools".
- **Android Studio**: tab **Flutter Inspector** & **Flutter Performance** udah built-in.
- **CLI**: `flutter run` terus tekan `v` di terminal (buka DevTools), `w` (dump widget tree), `t` (dump render tree).

Panel yang paling sering kepakai:

| Panel | Fungsi |
|-------|--------|
| **Inspector** | Liat widget tree, klik widget di layar, temuin overflow |
| **Performance** | Rekam frame render, deteksi *jank* (lag) |
| **Debugger** | Breakpoint, step-through kode baris per baris |
| **Logging** | Semua output `debugPrint` terpusat rapi |
| **Network** | Pantau HTTP request/response (buat nanti di fase REST API) |
| **Memory** | Deteksi memory leak |

**Widget Inspector** adalah yang paling sering kamu pakai. Misal ada widget yang nggak keliatan atau ke-cut, klik "Select Widget Mode", terus klik widget di layar — DevTools nunjukin posisinya di tree, ukurannya, dan propertinya.

---

## 4. Breakpoint — Stop di Baris Tertentu

Daripada `debugPrint` di mana-mana, kadang lebih enak **nge-freeze** eksekusi di baris tertentu. Klik di samping nomor baris di VS Code/Android Studio (muncul titik merah), terus jalankan debug. Saat kode nyampe baris itu, app **pause** dan kamu bisa inspeksi nilai semua variabel, terus *step over* (`F10`) atau *step into* (`F11`) baris per baris.

```dart
void prosesData(List<int> angka) {
  final total = angka.fold(0, (a, b) => a + b); // pasang breakpoint di sini
  final rataRata = total / angka.length;
  debugPrint('Rata-rata: $rataRata');
}
```

Breakpoint + **Variables panel** di IDE = kamu bisa liat nilai `total` sebelum lanjut, tanpa spam `debugPrint`.

---

## 5. `kDebugMode` & `kReleaseMode` — Kode Khusus Mode

Kadang kamu mau kode yang **cuma jalan di development**. Misal: tombol reset data, atau logging verbose. Pakai flag dari `package:flutter/foundation.dart`:

```dart
import 'package:flutter/foundation.dart';

void main() {
  if (kDebugMode) {
    debugPrint('🚧 Aplikasi berjalan di mode DEBUG');
  } else {
    debugPrint('🚀 Aplikasi berjalan di mode RELEASE');
  }
  runApp(const MyApp());
}
```

Ini berguna banget buat nanti — misal nunjukin tombol "Dev Panel" cuma saat develop, tapi otomatis hilang di app yang di-upload ke Play Store.

---

## 6. Best Practice Debugging

- **Pakai `debugPrint`, bukan `print`** — aman dari output terpotong.
- **Pakai `assert`** buat validasi asumsi logic di mode development.
- **Baca error dari baris paling atas** — pesan pertama adalah akar masalah.
- **Hot Reload (`r`) buat coba perubahan cepat, Hot Restart (`R`) buat reset state.** Kalau `r` nggak nampilin perubahan state, coba `R`.
- **`debugShowCheckedModeBanner: false`** buat ilangin banner "DEBUG" merah di pojok (ingat, ini cuma kosmetik, bukan berarti jadi release).
- **Jangan biarin `debugPrint` numpuk di production** — pisahkan pakai `kDebugMode`.
- **Familiar sama Widget Inspector** — 90% masalah layout bisa ketauan dari sana dalam hitungan detik.

---

## Ringkasan

- **`debugPrint`** = pengganti `print` yang lebih aman
- **`assert`** = cek asumsi logic, cuma aktif di debug
- **Error message Flutter** = teman, baca dari baris paling atas
- **Flutter DevTools** = suite alat debugging visual (Inspector, Performance, Debugger, dll)
- **Breakpoint** = pause eksekusi & inspeksi variabel
- **`kDebugMode` / `kReleaseMode`** = kode khusus mode build

Skill debugging inilah yang bedain "tutorial warrior" sama **developer beneran**. Semakin cepat kamu bisa nemuin dan benerin bug, semakin cepat kamu nyelesaiin project.

Dan dengan ini, **Fase Fundamental SELESAI! 🎉** Kamu udah nglewati 14 artikel fondasi. Besok kita masuk **Fase Intermediate** dengan topik yang paling ditunggu-tunggu: **State Management dengan Provider**. Di sanalah app kamu mulai "hidup" dan scalable. Stay tuned! 🚀

**Coba sendiri! Bikin app sederhana, sengaja bikin 2 bug (misal overflow & assert gagal), terus temuin & benerin pakai DevTools. Share pengalaman debugging-mu ke sosial media dan tag @ahsai001!**
