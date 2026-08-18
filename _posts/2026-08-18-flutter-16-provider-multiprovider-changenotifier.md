---
layout: post
title: "Flutter #16: Provider — MultiProvider & ChangeNotifier, Kelola Banyak State Sekaligus"
date: 2026-08-18 07:00:00 +0700
tags: [flutter, dart, provider, multiprovider, changenotifier, state-management, tutorial, indonesia, pemrograman]
---

Kemarin di **Flutter #15** kita udah berkenalan sama Provider: bikin satu `ChangeNotifier`, nyediain lewat `ChangeNotifierProvider`, terus baca pakai `watch`/`read`. Counter dua tombol beres, todo list jalan. Tapi jujur aja — app beneran nggak cuma punya *satu* state.

App kamu biasanya punya: **auth** (siapa yang login), **cart** (keranjang belanja), **tema** (light/dark), **todo**, **notifikasi**, dan seabrek state lain. Pertanyaannya: gimana nyediain semuanya sekaligus tanpa bikin kode jadi berantakan? Jawabannya: **`MultiProvider`**. Dan kalau ada state yang *bergantung* ke state lain, kita pakai **`ChangeNotifierProxyProvider`**. Yuk bahas.

---

## 1. Masalah: Nested Provider (Piramida Neraka)

Cara "naif" nyediain banyak provider adalah nest satu-satu:

```dart
// ❌ JANGAN gini — makin dalam, makin susah dibaca
ChangeNotifierProvider(
  create: (_) => AuthModel(),
  child: ChangeNotifierProvider(
    create: (_) => CartModel(),
    child: ChangeNotifierProvider(
      create: (_) => SettingsModel(),
      child: MaterialApp(home: HomePage()),
    ),
  ),
)
```

Buat 3 provider masih oke. Coba 6-7 provider — indentasi ke kanan terus, `child` bertumpuk, dan waktu kamu mau nambah/hapus satu provider di tengah, ribetnya minta ampun. **`MultiProvider`** ada buat ngilangin masalah ini.

---

## 2. MultiProvider: Satu Pintu Buat Semua Provider

`MultiProvider` nerima list `providers` dan nyediain semuanya **sejajar**, tanpa nest-nestan:

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'auth_model.dart';
import 'cart_model.dart';
import 'settings_model.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (_) => AuthModel()),
        ChangeNotifierProvider(create: (_) => CartModel()),
        ChangeNotifierProvider(create: (_) => SettingsModel()),
      ],
      child: MaterialApp(
        debugShowCheckedModeBanner: false,
        home: const HomePage(),
      ),
    );
  }
}
```

Nah, jauh lebih bersih kan? Urutan provider **nggak masalah** selama model-modelnya independen. `MultiProvider` cuma syntactic sugar — di belakang layar dia tetap nest provider satu per satu, tapi kamu nggak perlu nulis indentasinya manual.

---

## 3. Dua Model Dasar: AuthModel & CartModel

Biar contohnya nyata, bikin dua model sederhana. Pertama `auth_model.dart`:

```dart
import 'package:flutter/foundation.dart';

class AuthModel extends ChangeNotifier {
  String? _userId;

  bool get isLoggedIn => _userId != null;
  String? get userId => _userId;

  void login(String id) {
    _userId = id;
    notifyListeners();
  }

  void logout() {
    _userId = null;
    notifyListeners();
  }
}
```

Terus `cart_model.dart`:

```dart
import 'package:flutter/foundation.dart';

class CartModel extends ChangeNotifier {
  final Map<String, int> _items = {};

  Map<String, int> get items => Map.unmodifiable(_items);
  int get totalItems => _items.values.fold(0, (sum, qty) => sum + qty);

  void add(String productId, int qty) {
    _items[productId] = (_items[productId] ?? 0) + qty;
    notifyListeners();
  }

  void remove(String productId) {
    _items.remove(productId);
    notifyListeners();
  }
}
```

`SettingsModel` (buat tema) cukup sederhana, kamu bisa bikin sendiri: `bool isDarkMode` + `toggleTheme()` + `notifyListeners()`.

---

## 4. ChangeNotifierProxyProvider: Saat Satu Model Butuh Model Lain

Sekarang kasus yang lebih realistis: **Cart harus tahu siapa user yang login.** Bayangin skenario:

- User **belum login** → nggak boleh masukin barang ke cart.
- User **logout** → cart harus **dikosongkan** otomatis.

Artinya `CartModel` bergantung ke `AuthModel`. Di sinilah **`ChangeNotifierProxyProvider`** masuk — dia "menyambungkan" satu provider ke provider lain, dan otomatis update tiap dependency-nya berubah.

Ubah `CartModel` biar bisa nerima user:

```dart
class CartModel extends ChangeNotifier {
  final Map<String, int> _items = {};
  String? _userId;

  Map<String, int> get items => Map.unmodifiable(_items);
  int get totalItems => _items.values.fold(0, (sum, qty) => sum + qty);

  // Dipanggil ProxyProvider tiap AuthModel berubah
  void updateUser(String? userId) {
    _userId = userId;
    if (userId == null) _items.clear(); // logout → cart dibersihkan
    notifyListeners();
  }

  void add(String productId, int qty) {
    if (_userId == null) return; // harus login dulu
    _items[productId] = (_items[productId] ?? 0) + qty;
    notifyListeners();
  }
}
```

Lalu di `main.dart`, ganti `ChangeNotifierProvider` untuk cart jadi `ChangeNotifierProxyProvider`:

```dart
MultiProvider(
  providers: [
    ChangeNotifierProvider(create: (_) => AuthModel()),
    // CartModel "mendengarkan" AuthModel
    ChangeNotifierProxyProvider<AuthModel, CartModel>(
      create: (_) => CartModel(),
      update: (_, auth, cart) => cart!..updateUser(auth.userId),
    ),
    ChangeNotifierProvider(create: (_) => SettingsModel()),
  ],
  child: MaterialApp(
    debugShowCheckedModeBanner: false,
    home: const HomePage(),
  ),
)
```

Yang terjadi: tiap `AuthModel` manggil `notifyListeners()` (misal login/logout), `update` dipanggil ulang dan `CartModel` nerima `userId` terbaru. Pas logout, `updateUser(null)` bakal ngosongin cart otomatis. **Clean dan nggak perlu kode sinkronisasi manual.** 🎯

> 💡 Ada juga `ProxyProvider` (tanpa ChangeNotifier) buat kasus kamu cuma butuh *transform* data, nggak butuh notify. Tapi buat state yang berubah, `ChangeNotifierProxyProvider` yang kamu pakai.

---

## 5. Pakai di UI: Badge Cart di AppBar

Contoh kecil biar dua model tadi langsung kerasa gunanya. Badge keranjang yang update otomatis:

```dart
class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Toko Provider'),
        actions: [
          IconButton(
            icon: Badge(
              label: Text('${context.watch<CartModel>().totalItems}'),
              child: const Icon(Icons.shopping_cart),
            ),
            onPressed: () => Navigator.pushNamed(context, '/cart'),
          ),
          IconButton(
            icon: Icon(
              context.watch<AuthModel>().isLoggedIn
                  ? Icons.logout
                  : Icons.login,
            ),
            onPressed: () {
              final auth = context.read<AuthModel>();
              auth.isLoggedIn ? auth.logout() : auth.login('user-123');
            },
          ),
        ],
      ),
      body: const Center(child: Text('Hello!')),
    );
  }
}
```

Coba tekan tombol login → cart bisa diisi. Tekan logout → badge balik ke 0 dan cart bersih. Semua karena `ProxyProvider` nyambungin dua model.

---

## 6. Tips & Best Practice

- **`context.select<T, R>()`** — buat widget yang cuma butuh *sebagian* state. Misal `context.select<CartModel, int>((c) => c.totalItems)` cuma rebuild saat `totalItems` berubah, bukan tiap ada perubahan apa pun di cart. Lebih hemat dibanding `watch` penuh.
- **Provider otomatis `dispose`** — `ChangeNotifierProvider` bakal manggil `dispose()` pada model saat provider-nya dihapus dari tree. Kamu nggak perlu dispose manual.
- **Pisahkan tanggung jawab** — satu model satu urusan. Jangan bikin `GodModel` yang ngurus auth + cart + tema sekaligus. Nanti artikel **repository pattern** (#32) bakal makin kerasa kenapa ini penting.
- **`Provider.value`** — kalau kamu udah punya instance model (misal dari luar widget tree), pakai `.value` biar nggak bikin instance baru.

---

## Ringkasan

- **`MultiProvider`** = nyediain banyak provider sekaligus tanpa nest-nestan.
- **`ChangeNotifierProxyProvider`** = nyambungin provider yang bergantung ke provider lain, update otomatis.
- **`update`** dipanggil tiap dependency berubah — tempat aman buat sinkronisasi antar-model.
- **`context.select`** = rebuild cuma saat bagian state tertentu berubah.

Dengan `MultiProvider` + `ChangeNotifierProxyProvider`, kamu udah bisa bangun app dengan banyak state yang saling terhubung — tanpa balik ke jaman `setState` yang berantakan. Besok kita mulai nyentuh data sungguhan: **HTTP Request dengan package `http`** (#17). Siap-siap bikin app yang ngomong sama server! 🚀

**Coba sendiri! Ubah project Provider #15-mu jadi pakai `MultiProvider`, terus bikin satu model yang bergantung ke model lain pakai `ChangeNotifierProxyProvider`. Share hasilnya ke sosial media dan tag @ahsai001!**
