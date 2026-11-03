---
layout: post
title: "Flutter #64: Dart Extension & Generic — Tulis Kode Lebih Ekspresif & Type-Safe"
date: 2026-11-04 07:00:00 +0700
categories: [flutter]
tags: [flutter, dart, tutorial, indonesia]
---

Hari ini kita bahas dua fitur Dart yang bikin kode jadi lebih bersih, ekspresif, dan type-safe: **Extension Methods** dan **Generics**. Keduanya adalah "secret sauce" di balik library populer seperti `flutter_riverpod`, `freezed`, dan `dio`. Kalau kamu sudah nyaman dengan Dart dasar, ini langkah selanjutnya biar kode kamu terasa lebih "Dart-idiomatic".

---

## Extension Methods: Tambah Method ke Class Existing

Bayangkan kamu mau nambah method `formatRupiah()` ke `int` atau `double`. Tanpa extension, kamu harus bikin helper function terpisah atau subclass (yang gak bisa di `int` soalnya final). Extension methods solve ini elegan.

### Syntax Dasar

```dart
extension CurrencyFormatter on num {
  String formatRupiah({String symbol = 'Rp', int decimalDigits = 0}) {
    final formatter = NumberFormat.currency(
      locale: 'id_ID',
      symbol: symbol,
      decimalDigits: decimalDigits,
    );
    return formatter.format(this);
  }
  
  String get toKFormat {
    if (this >= 1000000) {
      return '${(this / 1000000).toStringAsFixed(1)}M';
    } else if (this >= 1000) {
      return '${(this / 1000).toStringAsFixed(1)}K';
    }
    return toString();
  }
}
```

### Pakai di Kode

```dart
void main() {
  final harga = 1500000;
  print(harga.formatRupiah());        // Rp1.500.000
  print(harga.formatRupiah(symbol: '$')); // $1,500,000.00
  print(2500000.toKFormat);           // 2.5M
  print(800.toKFormat);               // 800
}
```

> **Catatan:** Import `package:intl/intl.dart` buat `NumberFormat`. Extension methods dipanggil seperti method biasa, tapi sebenarnya static — tidak ada overhead runtime.

---

## Extension pada Generic Types

Ini dia bagian powerful-nya. Kamu bisa bikin extension yang **hanya berlaku untuk tipe tertentu** menggunakan `where`.

```dart
extension ListExtensions<T> on List<T> {
  // Hanya berlaku kalau T extends Comparable (bisa dibandingin)
  T? get max where T : Comparable<T> {
    if (isEmpty) return null;
    T maxVal = first;
    for (final item in this) {
      if (item.compareTo(maxVal) > 0) maxVal = item;
    }
    return maxVal;
  }
  
  // Hanya untuk List<String>
  String joinWith(String separator) where T == String {
    return join(separator);
  }
  
  // Hanya untuk List<num>
  num get sum where T : num {
    return fold<num>(0, (a, b) => a + b);
  }
}
```

```dart
void main() {
  print([3, 1, 4, 1, 5].max);           // 5
  print(['a', 'z', 'm'].max);           // z
  print([1, 2, 3].sum);                 // 6
  print(['Flutter', 'Dart'].joinWith(' + ')); // Flutter + Dart
  
  // Ini error compile-time:
  // [Object()].max;  // Error: Object tidak implements Comparable
}
```

---

## Generic Classes & Functions: Reusable & Type-Safe

Generics bikin kamu nulis **satu implementasi** yang kerja untuk banyak tipe, tapi tetap **type-safe** di compile time.

### Generic Class: Wrapper Hasil API

```dart
class ApiResponse<T> {
  final bool success;
  final T? data;
  final String? errorMessage;
  final int? statusCode;
  
  const ApiResponse._({
    required this.success,
    this.data,
    this.errorMessage,
    this.statusCode,
  });
  
  factory ApiResponse.success(T data, {int statusCode = 200}) {
    return ApiResponse._(success: true, data: data, statusCode: statusCode);
  }
  
  factory ApiResponse.error(String message, {int statusCode = 500}) {
    return ApiResponse._(
      success: false, 
      errorMessage: message, 
      statusCode: statusCode,
    );
  }
  
  // Pattern matching (Dart 3+)
  void when({
    required void Function(T data) onSuccess,
    required void Function(String error) onError,
  }) {
    if (success && data != null) {
      onSuccess(data!);
    } else {
      onError(errorMessage ?? 'Unknown error');
    }
  }
}
```

### Pakai Generic Class

```dart
class User {
  final String name;
  final String email;
  User(this.name, this.email);
  @override String toString() => 'User($name, $email)';
}

void handleResponse(ApiResponse<User> response) {
  response.when(
    onSuccess: (user) => print('Login: $user'),
    onError: (err) => print('Gagal: $err'),
  );
}

void main() {
  handleResponse(ApiResponse.success(User('Budi', 'budi@mail.com')));
  handleResponse(ApiResponse.error('Token expired', statusCode: 401));
}
```

---

## Generic Function dengan Type Constraints

```dart
// Hanya accept tipe yang implements Comparable
T findMax<T extends Comparable<T>>(List<T> items) {
  if (items.isEmpty) throw ArgumentError('List kosong');
  T max = items.first;
  for (final item in items.skip(1)) {
    if (item.compareTo(max) > 0) max = item;
  }
  return max;
}

// Multiple type parameters dengan constraint
Map<K, V> zipLists<K, V>(List<K> keys, List<V> values) {
  if (keys.length != values.length) {
    throw ArgumentError('Panjang list harus sama');
  }
  return Map.fromIterables(keys, values);
}

void main() {
  print(findMax([10, 5, 20, 8]));        // 20
  print(findMax(['apple', 'banana', 'cherry'])); // cherry
  
  final map = zipLists(['id', 'name'], [1, 'Budi']);
  print(map); // {id: 1, name: Budi}
}
```

---

## Kombinasi Extension + Generic: Fluent API

Ini pattern yang sering dipakai di library modern (Riverpod, Dio interceptors, dsb):

```dart
extension AsyncValueExtensions<T> on Future<T> {
  Future<ApiResponse<T>> toApiResponse() async {
    try {
      final data = await this;
      return ApiResponse.success(data);
    } catch (e) {
      return ApiResponse.error(e.toString());
    }
  }
  
  Future<T> withTimeout(Duration timeout, {T? fallback}) {
    return this.timeout(timeout, onTimeout: () {
      if (fallback != null) return fallback;
      throw TimeoutException('Timeout after $timeout');
    });
  }
}

extension IterableExtensions<T> on Iterable<T> {
  List<T> whereNotNull() => where((e) => e != null).cast<T>().toList();
  
  Map<K, List<T>> groupBy<K>(K Function(T) keySelector) {
    final map = <K, List<T>>{};
    for (final item in this) {
      final key = keySelector(item);
      map.putIfAbsent(key, () => []).add(item);
    }
    return map;
  }
}
```

### Contoh Pemakaian Nyata

```dart
class Product {
  final String category;
  final String name;
  final int price;
  Product(this.category, this.name, this.price);
}

Future<void> demo() async {
  // Simulasi fetch API
  final productsFuture = Future.delayed(Duration(millis: 100), () => [
    Product('Snack', 'Chips', 5000),
    Product('Minuman', 'Teh', 3000),
    Product('Snack', 'Kacang', 7000),
  ]);
  
  // Fluent chain: fetch -> timeout -> handle error -> group by category
  final grouped = await productsFuture
      .withTimeout(Duration(seconds: 2))
      .then((list) => list.groupBy((p) => p.category));
  
  grouped.forEach((cat, items) {
    print('$cat: ${items.map((e) => e.name).join(", ")}');
  });
  // Snack: Chips, Kacang
  // Minuman: Teh
}
```

---

## Best Practices & Tips

| Tips | Penjelasan |
|------|------------|
| **Nama extension descriptive** | `extension CurrencyFormat on num` lebih jelas dari `extension Ext on num` |
| **Gunakan `where` constraint** | Bikin extension lebih spesifik & aman (misal `where T : Comparable`) |
| **Jangan over-engineer** | Extension untuk utility kecil. Logic bisnis kompleks tetap di class/service |
| **Generic = compile-time safety** | Hindari `dynamic` atau `Object` kalau bisa pakai generic |
| **Dart 3: Pattern matching** | Pakai `switch` / `when` di generic sealed class buat exhaustiveness checking |

---

## Kesimpulan

- **Extension methods** = tambah behavior ke class existing tanpa inheritance
- **Generics** = tulis sekali, pakai banyak tipe, tetap type-safe
- **Kombinasi keduanya** = fluent API yang readable & powerful (lihat `AsyncValueExtensions` di atas)

Fitur ini adalah fondasi dari banyak package modern Flutter. Pahami konsepnya, dan kamu bakal lebih gampang baca source code library, bikin utility sendiri, dan kontribusi ke open source.

---

Coba sendiri! Buat extension `DateTime` buat format `dd MMM yyyy` atau `timeAgo` (misal "5 menit lalu"). Share ke sosial media dan tag **@ahsai001** 🚀