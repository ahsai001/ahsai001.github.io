---
layout: post
title: "Flutter #36: Widget Testing"
date: 2026-09-14 07:00:00 +0700
tags: [flutter, dart, widget-testing, testing, tdd]
description: "Pelajari widget testing di Flutter: test UI widget dengan flutter_test, verify tampilan dan interaksi user, hingga best practice."
---

Halo Developer! 👋

Di artikel #35 kemarin kita sudah bahas **Unit Testing** — testing fungsi dan logika bisnis tanpa UI. Sekarang kita naik level: **Widget Testing**. Di sini kita bakal ngetes **tampilan widget** — apakah teksnya muncul, tombolnya bisa diklik, input-nya bisa diketik.

Bayangkan unit testing itu ngetes mesin mobil, sedangkan widget testing itu test drive — kamu nggak perlu jalan di jalan raya (device nyata), tapi kamu sudah bisa rasakan mobilnya dari dalam garasi. 🚗

---

## Apa Itu Widget Testing?

**Widget test** (juga disebut *headless test*) menjalankan widget dalam environment simulasi tanpa emulator atau device fisik. Flutter punya widget tester bawaan dari package `flutter_test` — sudah termasuk di project Flutter baru, nggak perlu install tambahan.

Yang diuji dalam widget testing:

- **Rendering** — widget muncul dengan benar?
- **Interaksi** — tap, swipe, ketik berfungsi?
- **State** — perubahan state mengubah UI sesuai harapan?
- **Routing** — navigasi ke page yang tepat?

Karena tanpa emulator, widget test **jauh lebih cepat** dari integration test tapi **lebih lengkap** dari unit test.

---

## Setup: Package `flutter_test`

Kalau kamu buat project Flutter baru, package ini sudah ada:

```yaml
# pubspec.yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
```

Kalau belum ada, tambahkan lalu jalankan:

```bash
flutter pub get
```

Semua file test harus berada di folder `test/` dan diakhiri `_test.dart`.

---

## Widget Test Pertama

Misalnya kita punya widget sederhana — tombol counter:

```dart
// lib/counter_widget.dart
import 'package:flutter/material.dart';

class CounterWidget extends StatefulWidget {
  const CounterWidget({super.key});

  @override
  State<CounterWidget> createState() => _CounterWidgetState();
}

class _CounterWidgetState extends State<CounterWidget> {
  int _count = 0;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text(
          'Count: $_count',
          key: const Key('counter-text'),
          style: const TextStyle(fontSize: 24),
        ),
        const SizedBox(height: 16),
        ElevatedButton(
          key: const Key('increment-button'),
          onPressed: () {
            setState(() {
              _count++;
            });
          },
          child: const Text('Tambah'),
        ),
        const SizedBox(height: 8),
        OutlinedButton(
          key: const Key('reset-button'),
          onPressed: () {
            setState(() {
              _count = 0;
            });
          },
          child: const Text('Reset'),
        ),
      ],
    );
  }
}
```

Perhatikan kita kasih **Key** pada setiap widget penting — ini sangat membantu saat testing karena bisa langsung target widget tertentu.

Sekarang tulis widget test-nya:

```dart
// test/counter_widget_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:myapp/counter_widget.dart';

void main() {
  testWidgets('counter dimulai dari 0', (WidgetTester tester) async {
    await tester.pumpWidget(
      const MaterialApp(
        home: Scaffold(
          body: CounterWidget(),
        ),
      ),
    );

    // Cek teks awal
    expect(find.text('Count: 0'), findsOneWidget);
    expect(find.text('Count: 1'), findsNothing);
  });

  testWidgets('tekan tombolTambah counter naik', (WidgetTester tester) async {
    await tester.pumpWidget(
      const MaterialApp(
        home: Scaffold(
          body: CounterWidget(),
        ),
      ),
    );

    // Tap tombolTambah
    await tester.tap(find.byKey(const Key('increment-button')));
    await tester.pump(); // rebuild widget

    // Cek teks berubah
    expect(find.text('Count: 0'), findsNothing);
    expect(find.text('Count: 1'), findsOneWidget);
  });

  testWidgets('tekan Reset kembalikan ke 0', (WidgetTester tester) async {
    await tester.pumpWidget(
      const MaterialApp(
        home: Scaffold(
          body: CounterWidget(),
        ),
      ),
    );

    // Tap tambah 3x
    await tester.tap(find.byKey(const Key('increment-button')));
    await tester.pump();
    await tester.tap(find.byKey(const Key('increment-button')));
    await tester.pump();
    await tester.tap(find.byKey(const Key('increment-button')));
    await tester.pump();

    expect(find.text('Count: 3'), findsOneWidget);

    // Tap reset
    await tester.tap(find.byKey(const Key('reset-button')));
    await tester.pump();

    expect(find.text('Count: 0'), findsOneWidget);
    expect(find.text('Count: 3'), findsNothing);
  });
}
```

Jalankan test:

```bash
flutter test test/counter_widget_test.dart
```

Output:

```
00:03 +3: counter dimulai dari 0
00:03 +4: tekan tombolTambah counter naik
00:03 +5: tekan Reset kembalikan ke 0

00:03 +5: All tests passed! 🎉
```

---

## Memahami Alur Widget Test

Setiap widget test mengikuti pola 3 langkah:

**1. `pumpWidget()`** — Render widget ke dalam test environment. Kamu harus bungkus widget dengan `MaterialApp` (atau `CupertinoApp`) karena banyak widget butuh context dari Material/Cupertino.

**2. Interaksi** — Simulasikan aksi user: `tap()`, `enterText()`, `drag()`, `longPress()`, dsb.

**3. `pump()` atau `pumpAndSettle()`** — Setelah interaksi, widget perlu rebuild. `pump()` rebuild sekali. `pumpAndSettle()` menunggu sampai semua animasi selesai.

```dart
// pump() — rebuild sekali
await tester.tap(find.byKey(const Key('increment-button')));
await tester.pump();

// pumpAndSettle() — tunggu semua selesai
await tester.tap(find.byKey(const Key('increment-button')));
await tester.pumpAndSettle();
```

---

## Finder: Mencari Widget di Tree

`find` adalah cara kamu lokasi widget tertentu dalam test. Beberapa cara paling umum:

```dart
// Berdasarkan Key
find.byKey(const Key('increment-button'))

// Berdasarkan teks
find.text('Tambah')

// Berdasarkan tipe widget
find.byType(ElevatedButton)

// Berdasarkan icon
find.byIcon(Icons.add)

// Single widget
find.byWidgetPredicate(
  (widget) => widget is Text && widget.data == 'Hello',
)
```

Matcher untuk hasil pencarian:

```dart
findsOneWidget    // tepat 1 widget ditemukan
findsNothing      // tidak ada widget ditemukan
findsWidgets      // minimal 1 widget
findsNWidgets(3)  // tepat 3 widget
```

---

## Test Interaksi Form

Widget test juga bisa simulasi input text dan validasi:

```dart
// test/login_form_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:myapp/login_form.dart';

void main() {
  testWidgets('form validasi — kosongkan field tampilkan error', (WidgetTester tester) async {
    await tester.pumpWidget(
      const MaterialApp(
        home: Scaffold(
          body: LoginForm(),
        ),
      ),
    );

    // Coba submit tanpa isi apapun
    await tester.tap(find.byType(ElevatedButton));
    await tester.pump();

    // Harus muncul error message
    expect(find.text('Email wajib diisi'), findsOneWidget);
    expect(find.text('Password wajib diisi'), findsOneWidget);
  });

  testWidgets('isi form lalu submit — tampilkan sukses', (WidgetTester tester) async {
    await tester.pumpWidget(
      const MaterialApp(
        home: Scaffold(
          body: LoginForm(),
        ),
      ),
    );

    // Ketik email
    await tester.enterText(
      find.byKey(const Key('email-field')),
      'budi@mail.com',
    );

    // Ketik password
    await tester.enterText(
      find.byKey(const Key('password-field')),
      'rahasia123',
    );

    // Submit
    await tester.tap(find.byType(ElevatedButton));
    await tester.pump();

    // Cek berhasil — misal muncul teks selamat datang
    expect(find.text('Selamat datang, Budi!'), findsOneWidget);
    expect(find.text('Email wajib diisi'), findsNothing);
  });
}
```

Perhatikan `enterText()` — langsung set text ke field tanpa perlu simulasi ketik huruf per huruf. Ini testing, bukan usability test. 😄

---

## Test Async Widget (FutureBuilder)

Kalau widget pakai `FutureBuilder` atau `AsyncSnapshot`, kamu perlu handle async di test:

```dart
testWidgets('tampilkan data setelah future selesai', (WidgetTester tester) async {
  await tester.pumpWidget(
    const MaterialApp(
      home: Scaffold(
        body: UserProfile(userId: 1),
      ),
    ),
  );

  // Pertama kali — masih loading
  expect(find.byType(CircularProgressIndicator), findsOneWidget);

  // Tunggu future selesai
  await tester.pumpAndSettle();

  // Data muncul
  expect(find.byType(CircularProgressIndicator), findsNothing);
  expect(find.text('Budi Santoso'), findsOneWidget);
});
```

`pumpAndSettle()` otomatis menunggu sampai semua futures resolve dan UI stabil.

---

## Test dengan Mock (Menggabungkan Unit + Widget Test)

Seringkali widget bergantung ke service/repository. Kita bisa **inject mock** untuk memisahkan dependency:

```dart
// test/product_list_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';
import 'package:myapp/models/product.dart';
import 'package:myapp/repositories/product_repository.dart';
import 'package:myapp/screens/product_list_screen.dart';

class MockProductRepository extends Mock implements ProductRepository {}

void main() {
  late MockProductRepository mockRepo;

  setUp(() {
    mockRepo = MockProductRepository();
  });

  testWidgets('tampilkan daftar produk dari repository', (WidgetTester tester) async {
    // Atur mock kembalikan 2 produk
    when(() => mockRepo.getProducts()).thenAnswer(
      (_) async => [
        Product(id: 1, name: 'Laptop', price: 10000000),
        Product(id: 2, name: 'Mouse', price: 150000),
      ],
    );

    await tester.pumpWidget(
      MaterialApp(
        home: ProductListScreen(repository: mockRepo),
      ),
    );

    // Loading dulu
    expect(find.byType(CircularProgressIndicator), findsOneWidget);

    // Tunggu data load
    await tester.pumpAndSettle();

    // Produk muncul
    expect(find.text('Laptop'), findsOneWidget);
    expect(find.text('Mouse'), findsOneWidget);
    expect(find.text('Rp 10.000.000'), findsOneWidget);

    // Verifikasi repository dipanggil tepat 1 kali
    verify(() => mockRepo.getProducts()).called(1);
  });
}
```

Kombinasi mock + widget testing memberikan **kendali penuh** atas data yang masuk ke widget tanpa perlu koneksi internet atau database sungguhan.

---

## Perbedaan Widget Test vs Integration Test

| Aspek | Widget Test | Integration Test |
|---|---|---|
| Environment | Headless (tanpa device) | Device/emulator nyata |
| Kecepatan | ⚡ Sangat cepat (~detik) | 🐢 Lambat (~menit) |
| Scope | Satu widget/screen | Seluruh flow app |
| Package | `flutter_test` | `integration_test` |
| Mocking | Perlu mock external deps | Data real dari API/DB |
| Cocok untuk | Testing UI & interaksi | End-to-end user flow |

---

## Best Practice Widget Testing

**1. Bungkus widget dengan `MaterialApp`** — Hampir semua widget butuh context Material. Tanpa ini, test sering error aneh.

**2. Pakai Key untuk target penting** — `find.byKey()` lebih stabil daripada `find.byType()` kalau ada beberapa widget sejenis di satu screen.

**3. Jangan test implementation** — Test *apa yang tampil*, bukan *bagaimana widget dibangun internalnya*. Contoh buruk: `expect(widget.toString(), contains('StatefulWidget'))`.

**4. Pisahkan test per behavior** — Satu test = satu asumsi. Jangan gabungkan test render + test interaksi + test navigasi dalam satu `testWidgets()`.

**5. Gunakan `pumpAndSettle()` untuk animasi** — Kalau widget punya animasi atau async, `pump()` biasa bisa gagal karena widget belum stabil.

---

## Ringkasan

| Konsep | Penjelasan |
|---|---|
| Package | `flutter_test` (bawaan Flutter) |
| Fungsi test | `testWidgets('deskripsi', (tester) { ... })` |
| Render | `tester.pumpWidget(MaterialApp(...))` |
| Interaksi | `tester.tap()`, `tester.enterText()`, `tester.drag()` |
| Rebuild | `tester.pump()` atau `tester.pumpAndSettle()` |
| Cari widget | `find.text()`, `find.byKey()`, `find.byType()` |
| Jalankan | `flutter test` |

Widget testing adalah **jembatan sempurna** antara unit testing (logika murni) dan integration testing (device nyata). Dengan widget test, kamu bisa yakin UI berfungsi dengan benar **sebelum** deploy ke device. 💪

---

Coba sendiri! Buat widget test untuk minimal 3 widget di projekmu — pastikan text muncul, tombol bisa diklik, dan state berubah sesuai harapan. Share ke sosial media dan tag @ahsai001 — tunjukkan bahwa testing itu penting dan seru! 🚀
