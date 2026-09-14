---
layout: post
title: "Flutter #37: Integration Testing — Uji Aplikasi dari Layar ke Layar"
date: 2026-09-15 07:00:00 +0700
tags: [flutter, dart, integration-testing, testing, tdd, tutorial, indonesia, pemrograman]
description: "Pelajari integration testing di Flutter: uji alur lengkap aplikasi di emulator/device nyata dengan package integration_test, dari setup sampai running test."
---

Halo Developer! 👋

Dua artikel terakhir kita sudah bahas **Unit Testing** (#35) dan **Widget Testing** (#36). Sekarang kita tutup seri testing dengan level tertinggi: **Integration Testing** — uji alur lengkap aplikasi di emulator atau device nyata, dari layar satu ke layar lainnya. Ini perbedaan terbesarnya: unit test ngetes fungsi, widget test ngetes satu layar, integration test ngetes **seluruh perjalanan user**.

Bayangkan kamu sudah test drive mobil di garasi (#36), sekarang saatnya **nyetir di jalan raya sungguhan** — lengkap dengan lampu merah, polisi tidur, dan pengguna jalan lain. 🚗💨

---

## Apa Itu Integration Testing?

**Integration test** menjalankan aplikasi Flutter secara utuh di device/emulator, lalu mensimulasikan interaksi user dari awal sampai akhir. Bedanya dengan widget test:

| Aspek | Widget Test | Integration Test |
|---|---|---|
| Environment | Headless (tanpa device) | Device/emulator nyata |
| Kecepatan | ⚡ Detik | 🐢 Beberapa menit |
| Scope | Satu widget/screen | Flow antar screen |
| Data | Wajib mock | API/database sungguhan |
| Cocok untuk | Verifikasi UI cepat | End-to-end user journey |

Integration test paling berguna untuk **alur kritis** aplikasi: login → masuk dashboard, checkout → pembayaran, register → verifikasi. Alur yang kalau rusak = user kabur. 😅

---

## Setup Package `integration_test`

Package ini bawaan Flutter SDK, tapi harus didaftarkan di `pubspec.yaml`:

```yaml
# pubspec.yaml
dev_dependencies:
  integration_test:
    sdk: flutter
  flutter_test:
    sdk: flutter
```

Jalankan `flutter pub get`, lalu buat folder `integration_test/`. Bedanya dengan folder `test/`: integration test **butuh device**, jadi nggak jalan dengan `flutter test` biasa.

---

## Contoh 1: Test Alur Login Lengkap

Kita mulai dari yang paling umum — uji alur login dari layar awal sampai masuk ke halaman utama:

```dart
// integration_test/login_flow_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';
import 'package:myapp/main.dart' as app;

void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  testWidgets('alur login lengkap: login -> dashboard', (WidgetTester tester) async {
    // Jalankan aplikasi sungguhan (bukan widget terisolasi)
    app.main();
    await tester.pumpAndSettle();

    // 1. Ketik email & password di halaman login
    await tester.enterText(
      find.byKey(const Key('email-field')),
      'budi@mail.com',
    );
    await tester.enterText(
      find.byKey(const Key('password-field')),
      'rahasia123',
    );

    // 2. Tap tombol login
    await tester.tap(find.byKey(const Key('login-button')));
    await tester.pumpAndSettle();

    // 3. Verifikasi: kita pindah ke dashboard
    expect(find.text('Selamat datang, Budi!'), findsOneWidget);
    expect(find.byType(NavigationBar), findsOneWidget);

    // 4. Lanjut: tap menu profil
    await tester.tap(find.byKey(const Key('nav-profile')));
    await tester.pumpAndSettle();
    expect(find.text('Halaman Profil'), findsOneWidget);
  });
}
```

Perhatikan perbedaannya dengan widget test (#36): di sini kita panggil **`app.main()`** — aplikasi berjalan lengkap dengan routing, splash screen, dan koneksi API aslinya. Kalau API-nya butuh data test, siapkan di environment staging.

---

## Contoh 2: Ambil Screenshot + Test Performa

Salah satu keunggulan integration test: bisa ambil **screenshot otomatis** dan **ukur performa** (frame rate, waktu render). Ini sangat berguna untuk regression test visual dan verifikasi animasi nggak lag:

```dart
// integration_test/product_detail_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';
import 'package:myapp/main.dart' as app;

void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  testWidgets('buka detail produk dan catat performa', (WidgetTester tester) async {
    app.main();
    await tester.pumpAndSettle();

    // Tap produk pertama di list
    await tester.tap(find.byKey(const Key('product-0')));
    await tester.pumpAndSettle();

    // Verifikasi halaman detail tampil
    expect(find.byKey(const Key('product-title')), findsOneWidget);
    expect(find.text('Laptop Pro 14"'), findsOneWidget);

    // Simpan screenshot sebagai bukti visual
    await binding.takeScreenshot('product-detail');

    // Ukur performa selama scroll
    final watch = Stopwatch()..start();
    for (var i = 0; i < 5; i++) {
      await tester.drag(find.byType(ListView), const Offset(0, -300));
      await tester.pump();
    }
    watch.stop();

    // Verifikasi rata-rata frame masih mulus
    final summary = await binding.traceAction(() async {
      await tester.drag(find.byType(ListView), const Offset(0, -300));
      await tester.pump();
    });
    debugPrint('Waktu scroll 5x: ${watch.elapsedMilliseconds} ms');
    expect(summary['average_frame_build_time_millis'], lessThan(10));
  });
}
```

`traceAction()` mengumpulkan data frame — kalau `average_frame_build_time_millis` melewati batas, kamu langsung tahu ada jank sebelum user yang mengeluh. 🔥

---

## Menjalankan Integration Test

Karena butuh device, jalankan dengan device aktif (emulator, atau HP via USB):

```bash
# Android emulator / device
flutter test integration_test/login_flow_test.dart -d emulator-5554

# Semua test di folder integration_test
flutter test integration_test -d emulator-5554

# iOS simulator
flutter test integration_test -d iphone
```

Output sukses:

```
00:42 +1: alur login lengkap: login -> dashboard
00:42 +2: buka detail produk dan catat performa

00:42 +2: All tests passed! 🎉
```

> **Tips:** Untuk CI/CD (nanti kita bahas di #41), bisa pakai Firebase Test Lab atau device farm supaya nggak perlu nyimpen emulator di server.

---

## Best Practice Integration Testing

**1. Fokus ke alur kritis saja** — Integration test itu lambat (menit), jadi jangan dipakai untuk semua hal. Pilih 3-5 alur utama: login, checkout, register, pencarian.

**2. Test di environment terpisah** — Jangan pernah jalankan integration test ke production API. Pakai staging server atau mock server dengan data palsu yang bisa di-reset.

**3. Siapkan data awal (seed)** — Integration test butuh kondisi yang bisa diprediksi. Buat helper yang mengosongkan database dan menyuntik data awal sebelum test jalan.

**4. Jangan test hal yang sudah di-cover** — Kalau widget test sudah verifikasi form login validasi error, integration test cukup fokus ke *alurnya*: koneksi API, navigasi, state global.

**5. Tambahkan ke CI, bukan cuma lokal** — Integration test yang hanya jalan di laptopmu = nggak ada gunanya buat tim. Jalankan otomatis tiap push (kita bahas GitHub Actions di #41).

---

## Piramida Testing Flutter

```text
        /\          Integration Test  — sedikit, lambat, end-to-end
       /  \         (alur kritis: login, checkout)
      /    \
     /──────\       Widget Test       — sedang, cepat, verifikasi UI
    /────────\      (tiap screen & interaksi)
   /──────────\
  /────────────\    Unit Test         — banyak, super cepat, logika
 /──────────────\   (fungsi, model, repository)
```

Semakin tinggi piramida, semakin sedikit jumlah test dan semakin mahal (lambat) menjalankannya. Semakin rendah, semakin banyak dan murah. **Porsi terbesar harus unit test** — murah dan cepat mendeteksi bug logika.

---

## Ringkasan

| Konsep | Penjelasan |
|---|---|
| Package | `integration_test` (bawaan Flutter SDK) |
| Environment | Emulator / device nyata |
| Fungsi | `testWidgets()` dengan `app.main()` |
| Binding | `IntegrationTestWidgetsFlutterBinding.ensureInitialized()` |
| Fitur unik | Screenshot otomatis, `traceAction()` untuk performa |
| Jalankan | `flutter test integration_test -d <device>` |

Dengan unit test (#35), widget test (#36), dan integration test (#37), aplikasimu sekarang punya **jaring pengaman tiga lapis** — mulai dari logika paling dalam sampai perjalanan user paling panjang. Ini modal besar sebelum kita masuk ke Firebase (#38-40) dan CI/CD (#41). 💪

---

Coba sendiri! Tulis integration test untuk alur paling penting di aplikasimu — login, checkout, atau register — dan jalankan di emulator. Share ke sosial media dan tag @ahsai001 — buktikan aplikasimu layak disebut production-grade! 🚀