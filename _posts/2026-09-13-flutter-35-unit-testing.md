---
layout: post
title: "Flutter #35: Unit Testing"
date: 2026-09-13 07:00:00 +0700
tags: [flutter, dart, unit-testing, testing, tdd]
description: "Pelajari unit testing di Flutter: install package test, tulis unit test pertama, mocking, dan best practice testing di Dart."
---

Halo Developer! 👋

Di artikel sebelumnya kita sudah belajar **Error Handling** supaya app tidak crash diam-diam. Sekarang kita masuk ke topik yang bikin kode kamu jauh lebih reliable: **Unit Testing**.

Banyak developer meremehkan testing — "toh di emulator jalan normal." Tapi begitu user pakai device yang beda, input yang nggak terduga, atau data yang corrupt, baru deh panik. Unit testing memastikan **setiap fungsi bekerja benar**, bahkan sebelum kamu run app-nya. 🧪

---

## Apa Itu Unit Testing?

**Unit test** adalah pengujian pada **satu unit kode** secara terisolasi — biasanya satu fungsi atau satu method. Tujuannya: pastikan input tertentu menghasilkan output yang benar.

Di Flutter/Dart, package resmi untuk unit testing adalah `test` (bukan `flutter_test` — itu untuk widget testing). Unit testing murni **tidak butuh widget tree atau emulator**, jadi prosesnya super cepat.

Perbedaan penting:
- **Unit test** → test fungsi/logika bisnis, tanpa UI
- **Widget test** → test satu widget atau screen (bahas di artikel #36)
- **Integration test** → test seluruh flow di device nyata (artikel #37)

---

## Setup: Tambah Package `test`

Buka `pubspec.yaml` di bagian `dev_dependencies`:

```yaml
dev_dependencies:
  test: ^1.25.0
```

Jalankan:

```bash
flutter pub get
```

Selesai! Sekarang buat folder `test/` di root project (biasanya sudah ada). Setiap file test harus diakhiri `_test.dart`.

---

## Unit Test Pertama

Bayangkan kamu punya fungsi sederhana untuk menghitung diskon:

```dart
// lib/models/discount_calculator.dart
class DiscountCalculator {
  double calculate(double price, int discountPercent) {
    if (price < 0) throw ArgumentError('Harga tidak boleh negatif');
    if (discountPercent < 0 || discountPercent > 100) {
      throw ArgumentError('Diskon harus antara 0-100');
    }
    return price - (price * discountPercent / 100);
  }
}
```

Sekarang tulis unit test-nya:

```dart
// test/models/discount_calculator_test.dart
import 'package:test/test.dart';
import 'package:myapp/models/discount_calculator.dart';

void main() {
  late DiscountCalculator calculator;

  setUp(() {
    calculator = DiscountCalculator();
  });

  test('diskon 10% dari harga 100.000 menghasilkan 90.000', () {
    final result = calculator.calculate(100000, 10);
    expect(result, equals(90000.0));
  });

  test('diskon 0% tidak mengubah harga', () {
    final result = calculator.calculate(50000, 0);
    expect(result, equals(50000.0));
  });

  test('diskon 100% menghasilkan harga 0', () {
    final result = calculator.calculate(200000, 100);
    expect(result, equals(0.0));
  });

  test('harga negatif melempar ArgumentError', () {
    expect(
      () => calculator.calculate(-10000, 10),
      throwsA(isA<ArgumentError>()),
    );
  });
}
```

Jalankan semua test:

```bash
flutter test
```

Output yang diharapkan:

```
00:04 +4: All tests passed! 🎉
```

Setiap `test()` adalah satu kasus uji. `expect()` adalah assertion — kalau hasilnya nggak sesuai, test gagal (merah). Kalau semua hijau, kode kamu aman! ✅

---

## Memahami Matcher Penting

`expect()` menggunakan **matcher** untuk membandingkan. Beberapa yang sering dipakai:

```dart
// Equality
expect(value, equals(42));
expect(value, 42); // shorthand — sama dengan di atas

// Boolean
expect(isActive, isTrue);
expect(isActive, isFalse);

// Null
expect(name, isNull);
expect(name, isNotNull);

// Tipe
expect(result, isA<String>());

// String
expect(message, contains('berhasil'));
expect(message, startsWith('OK'));

// Collection
expect(list, hasLength(3));
expect(list, contains('Flutter'));

// Exception
expect(() => parse('abc'), throwsFormatException);
```

---

## Grouping Test dengan `group()`

Kalau kamu punya banyak test untuk fungsi yang sama, pakai `group()` supaya output lebih rapi:

```dart
void main() {
  late DiscountCalculator calculator;

  setUp(() {
    calculator = DiscountCalculator();
  });

  group('DiscountCalculator.calculate()', () {
    test('diskon normal', () {
      expect(calculator.calculate(100000, 10), 90000.0);
    });

    test('diskon 0%', () {
      expect(calculator.calculate(50000, 0), 50000.0);
    });

    test('diskon maksimal', () {
      expect(calculator.calculate(200000, 100), 0.0);
    });
  });

  group('DiscountCalculator — validasi input', () {
    test('harga negatif', () {
      expect(
        () => calculator.calculate(-1, 10),
        throwsA(isA<ArgumentError>()),
      );
    });

    test('diskon melebihi 100%', () {
      expect(
        () => calculator.calculate(100, 150),
        throwsA(isA<ArgumentError>()),
      );
    });
  });
}
```

Output sekarang jadi lebih terstruktur:

```
DiscountCalculator.calculate()
  ✓ diskon normal
  ✓ diskon 0%
  ✓ diskon maksimal
DiscountCalculator — validasi input
  ✓ harga negatif
  ✓ diskon melebihi 100%

00:00 +5: All tests passed!
```

---

## Mocking: Test Fungsi yang Bergantung ke External

Fungsi yang manggil API atau database nggak bisa ditest langsung. Solusinya: **mocking** — buat palsu yang meniru perilaku asli.

Tambah package `mocktail`:

```yaml
dev_dependencies:
  test: ^1.25.0
  mocktail: ^1.0.4
```

Contoh: kamu punya repository yang memanggil API:

```dart
// lib/repositories/user_repository.dart
abstract class UserRepository {
  Future<Map<String, dynamic>> getUser(int id);
}

class ApiUserRepository implements UserRepository {
  @override
  Future<Map<String, dynamic>> getUser(int id) async {
    // ... HTTP request ke server
    return {'id': id, 'name': 'Budi'};
  }
}
```

Sekarang test fungsi yang pakai repository **tanpa hit API asli**:

```dart
// test/repositories/user_repository_test.dart
import 'package:mocktail/mocktail.dart';
import 'package:test/test.dart';
import 'package:myapp/repositories/user_repository.dart';

class MockUserRepository extends Mock implements UserRepository {}

void main() {
  late MockUserRepository mockRepo;

  setUp(() {
    mockRepo = MockUserRepository();
  });

  test('getUser mengembalikan data user yang benar', () async {
    // Atur behavior mock
    when(() => mockRepo.getUser(1)).thenAnswer(
      (_) async => {'id': 1, 'name': 'Budi'},
    );

    // Panggil
    final result = await mockRepo.getUser(1);

    // Verifikasi
    expect(result['name'], equals('Budi'));
    verify(() => mockRepo.getUser(1)).called(1);
  });
}
```

`mocktail` membuat class palsu yang bisa kamu atur responsnya. Tidak ada HTTP request, tidak ada internet — test tetap jalan di mana saja, termasuk di CI/CD.

---

## Menjalankan Test

```bash
# Jalankan semua test
flutter test

# Jalankan satu file test
flutter test test/models/discount_calculator_test.dart

# Jalankan test yang match nama tertentu
flutter test --name "diskon normal"

# Lihat coverage (berapa % kode yang di-test)
flutter test --coverage
```

Coverage report menghasilkan file di `coverage/lcov.info`. Kamu bisa pakai `lcov` atau `genhtml` untuk lihat HTML report:

```bash
genhtml coverage/lcov.info -o coverage/html
open coverage/html/index.html
```

---

## Best Practice Unit Testing

**1. naming yang jelas** — Nama test harus menjelaskan *apa* yang diuji dan *kondisi* apa:
```dart
test('getUser melempar exception saat id negatif', () { ... });
```

**2. Arrange-Act-Assert** — Struktur test selalu 3 bagian:
```dart
test('contoh', () {
  // Arrange — siapkan data
  final calculator = DiscountCalculator();
  
  // Act — jalankan aksi
  final result = calculator.calculate(100000, 10);
  
  // Assert — cek hasil
  expect(result, 90000.0);
});
```

**3. Satu test = satu asumsi** — Jangan cek 5 hal sekaligus dalam satu test. Kalau gagal, kamu nggak tahu yang mana.

**4. Test edge case** — Bukan cuma input normal. Test juga: null, kosong, batas maksimum, negatif, spesial character.

**5. Jangan test implementation detail** — Test *output*, bukan *cara kerja internal*. Kalau kamu refactor kode dan test gagal meski outputnya benar, berarti test-nya terlalu tightly coupled.

---

## Ringkasan

| Konsep | Penjelasan |
|---|---|
| Package | `test` untuk unit test |
| Fungsi test | `test('nama', () { ... })` |
| Assertion | `expect(actual, matcher)` |
| Grouping | `group('nama', () { ... })` |
| Mocking | `mocktail` untuk palsukan dependency |
| Jalankan | `flutter test` |

Unit testing memang terasa lambat di awal — tapi begitu kode mulai banyak, test yang sudah ada jadi **safety net** yang menyeluruh dari regression bug. Refactor bebas, push percaya diri. 🛡️

---

Coba sendiri! Buat minimal 3 unit test untuk fungsi yang ada di projekmu. Share ke sosial media dan tag @ahsai001 — biar makin banyak yang sadar pentingnya testing! 🚀
