---
layout: post
title: "Flutter #22: SharedPreferences — Simpan Data Sederhana di Perangkat"
date: 2026-08-31 07:00:00 +0700
tags: [flutter, dart, sharedpreferences, local-storage, tutorial, indonesia, pemrograman]
---

Welcome to **Flutter #22**! 🎉

Di **Flutter #19** kita fetch data REST API dan tampilkan dalam `ListView`. Di **#20** kita mendalam ke `FutureBuilder` dan `AsyncSnapshot` — widget deklaratif yang menjaga UI bersih saat data asinkron datang dari `Future`. Tapi begitulah dunia nyata: data yang kita dapat sering perlu "diingat" — disimpan di perangkat user agar bisa dibaca kembali kapan saja, bahkan setelah aplikasi ditutup dan dibuka lagi.

Hari ini kita akan bahas **SharedPreferences** — solusi paling sederhana untuk menyimpan data sederhana di perangkat mobile.

---

## 1. Apa itu SharedPreferences dan Kenapa Perlu?

**SharedPreferences** adalah plugin Flutter yang menyediakan abstraksi sederhana untuk menyimpan pasangan kunci-nilai (key-value) di dalam *preferences* yang disediakan oleh platform (SharedPreferences di Android, UserDefaults di iOS). Data-disimpan dalam format *serialized* dan tersimpan di memori internal perangkat.

Seperti kasus berikut: kamu bikin aplikasi pengatur tema (dark mode). User memilih "Gelap" di settingan, dan kamu mau agar pilihan itu "ingat" sampai kali selanjutnya user membuka aplikasi. Tanpa SharedPreferences, setiap kali aplikasi di-restart, theme akan kembali ke default terang.

**Keunggulan SharedPreferences:**
- Sangat ringan — cocok untuk data sederhana
- Akses API yang mudah, tidak butuh model class kompleks
- Data *persisten* hingga dihapus secara eksplisit
- Cross-platform (Android & iOS sama-sama support)

**Kekurangan:**
- Cocok hanya untuk data sederhana (string, bool, int, double)
- Bukan tempat untuk data besar atau struktur kompleks
- Tidak cocok untuk query atau filtering

---

## 2. Perbandingan Cepat: SharedPreferences vs Alternatif

|| SharedPreferences | File I/O | SQLite |
||---|---|---|
|| **Jenis data** | Primitif (string, bool, int, double) | Apa saja | Semua tipe |
|| **Kecerpanan** | *Immediate* | Tergantung `sync` | *Transaction-based* |
|| **Kueri** | Tidak didukung | Manual parsing | `SELECT` query |
|| **Ukuran data** | Kecil (KB) | Batas perangkat | Bisa besar |
|| **Contoh penggunaan** | Tema, setting user | Setting kompleks | Database full-fledged |

---

## 3. Konsep Dasar: SharedPreferences API

SharedPreferences menyediakan dua API utama:

1. **`SharedPreferences.getInstance()`** — mendapatkan instance SharedPreferences
2. **`prefs.*method()`** — operasi baca/tulis

Method umum:
- `setString(key, value)` — simpan string
- `setInt(key, value)` — simpan integer
- `setDouble(key, value)` — simpan double
- `setBool(key, value)` — simpan boolean
- `getString(key, defaultValue)` — ambil string
- `getInt(key, defaultValue)` — ambil integer
- `getBool(key, defaultValue)` — ambil boolean
- `remove(key)` — hapus kunci tertentu
- `clear()` — hapus semua data
- `contains(key)` — apakah kunci ada

Semua method mengembalikan `Future<void>` (kecuali `contains` yang boolean), jadi wajib dipakai `await` atau `.then()`.

---

## 4. Implementasi: Sederhana Menyimpan dan Membaca

```dart
import 'package:shared_preferences/shared_preferences.dart';

class PreferencesService {
  static final PreferencesService _instance = PreferencesService._internal();
  factory PreferencesService() => _instance;
  PreferencesService._internal();

  late final SharedPreferences _prefs;

  Future init() async {
    _prefs = await SharedPreferences.getInstance();
  }

  // --- Menyimpan data ---
  
  Future simpanTheme(bool isDark) async {
    await _prefs.setBool('theme_dark', isDark);
  }

  Future simpanJumlahKunjungan(int count) async {
    await _prefs.setInt('kunjungan', count);
  }

  Future simpanNamaDepan(String nama) async {
    await _prefs.setString('nama_depan', nama);
  }

  // --- Membaca data ---

  bool bacaTheme() {
    return _prefs.getBool('theme_dark') ?? false;
  }

  int bacaJumlahKunjungan() {
    return _prefs.getInt('kunjungan') ?? 0;
  }

  String? bacaNamaDepan() {
    return _prefs.getString('nama_depan');
  }

  // --- Hilangkan data ---

  void hapusTheme() {
    _prefs.remove('theme_dark');
  }

  void clearSemua() {
    _prefs.clear();
  }
}
```

Penting diingat: **wajib dipanggil `init()` sekali** (biasanya di `main()` atau `initState()`) sebelum memanggil method apapun, karena `_prefs` harus diinisialisasi dari `SharedPreferences.getInstance()`.

---

## 5. Contoh Praktis: Aplikasi Setting Sederhana

Mari bangun widget sederhana yang memungkinkan user memilih tema gelap/terang dan nilai disimpan seumur hidup aplikasi.

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

void main() async {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    // Inisialisasi shared preferences secara global
    final prefs = SharedPreferences.getInstance();
    final isDark = prefs.getBool('theme_dark') ?? false;

    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData(brightness: Brightness.light),
      darkTheme: ThemeData(brightness: Brightness.dark),
      themeMode: isDark ? ThemeMode.dark : ThemeMode.light,
      home: const SettingScreen(),
    );
  }
}

class SettingScreen extends StatefulWidget {
  const SettingScreen({super.key});
  @override
  State<SettingScreen> createState() => _SettingScreenState();
}

class _SettingScreenState extends State<SettingScreen> {
  bool _isDark = false;
  int _counter = 0;
  String _name = '';

  @override
  void initState() {
    super.initState();
    _loadPreferences();
  }

  Future _loadPreferences() async {
    final sp = await SharedPreferences.getInstance();
    setState(() {
      _isDark = sp.getBool('theme_dark') ?? false;
      _counter = sp.getInt('counter') ?? 0;
      _name = sp.getString('name') ?? '';
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Setting SharedPreferences')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            SwitchListTile(
              title: const Text('Tema Gelap'),
              value: _isDark,
              onChanged: (bool value) async {
                setState(() => _isDark = value);
                final sp = await SharedPreferences.getInstance();
                await sp.setBool('theme_dark', value);
              },
            ),
            const SizedBox(height: 16),
            Text('Counter: $_counter'),
            const SizedBox(height: 16),
            TextField(
              onSubmitted: (val) async {
                final sp = await SharedPreferences.getInstance();
                await sp.setString('name', val);
                setState(() => _name = val);
              },
              decoration: const InputDecoration(
                labelText: 'Nama',
                hintText: 'Masukkan nama Anda',
              ),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () async {
                final sp = await SharedPreferences.getInstance();
                await sp.setInt('counter', _counter + 1);
                setState(() => _counter++);
              },
              child: const Text('Increment & Simpan'),
            ),
          ],
        ),
      ),
    );
  }
}
```

Setiap kali tombol di-press, nilai disimpan ke SharedPreferences dan akan tersedia kembali keesokan harinya ketika aplikasi dibuka.

---

## 6. Contoh Lanjut: Menyimpan Multiple Setting bersamaan

Kadang kita perlu menyimpan banyak setting sekaligus. Cara yang baik adalah mengelompokkannya dalam satu `init()` call dan menyimpan secara bergrup.

```dart
Future simpanSemuaSetting({
  required bool themeDark,
  required int counter,
  required String name,
}) async {
  final sp = await SharedPreferences.getInstance();
  await Future.wait([
    sp.setBool('theme_dark', themeDark),
    sp.setInt('counter', counter),
    sp.setString('name', name),
  ]);
  // Semua nilai otomatis tersimpan ke storage native setelah Future.wait selesai.
}
```

---

## 7. Best Practice: Jangan Lupa Dispose & Handle Null

Ini paling sering dilupakan pemula:

1. **Selalu beri nilai default** — `getBool('key') ?? false`, `getInt('key') ?? 0`
2. **Inisialisasi di `initState()` atau `main()`** — sekali saja, jangan di `build()`
3. **Gunakan `prefs.getKeys()`** untuk melihat semua kunci yang tersimpan saat debug
4. **Jangan simpan terlalu banyak** — SharedPreferences tidak dirancang untuk banyak kunci yang ekstrem
5. **Hapus data yang tidak digunakan** — gunakan `remove()` bukan menumpuk `clear()`

---

## 8. Common Pitfalls

### ❌ `setState` sebelum `initSharedPreferences`

```dart
// JANGAN LAKUKAN INI
@override
void initState() {
  setState(() => _value = prefs.getString('key') ?? ''); // Error: prefs belum init
  super.initState();
}
```

**Solusi:** Panggil `await prefs.init()` atau `SharedPreferences.getInstance()` sebelum `setState`.

### ❌ Simpan tipe data yang salah

```dart
// JANGAN: menyimpan objek kompleks langsung
await prefs.setString('user', userModel.toJson()); // Bisa, tapi mengabaikan struktur
```

**Solusi:** SharedPreferences hanya untuk primitif. Untuk model class, gunakan JSON serialization manual atau package seperti `json_serializable`.

### ❌ Lupa handle `null` return

```dart
// TIDAK AMAN: bisa menyebabkan tipe null error di widget tree
final value = prefs.getInt('tidak_ada'); // Returns null if key missing
```

**Solusi:** Selalu beri default: `final value = prefs.getInt('tidak_ada') ?? 0`.

---

## Ringkasan

- **`SharedPreferences`** — plugin untuk menyimpan pasangan key-value primitif
- **Tipe data:** string, bool, int, double — hanya primitif saja
- **Method kunci:** `setX(key, value)` & `getX(key, defaultValue)`
- **Wajib:** `SharedPreferences.getInstance()` sebelum membaca/menggulis
- **Gunakan untuk:** tema aplikasi, setting user, flag singkat, counter local
- **Jangan gunakan untuk:** data besar, struktur kompleks, kebutuhan query

Sekarang kamu bisa dengan mudah menyimpan preferensi pengguna di aplikasi Fluttermu! Data akan tetap ada meskipun user menutup dan membuka aplikasi kembali.

---

**Coba sendiri!** Bikin aplikasi sederhana yang menyimpan pilihan tema gelap dan counter menggunakan SharedPreferences. Coba juga menambah fitur "bersihkan semua data" dengan tombol clear. Share hasilnya ke sosial media dan tag @ahsai001! 🚀