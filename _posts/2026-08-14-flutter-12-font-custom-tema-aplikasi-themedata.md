---
layout: post
title: "Flutter #12: Font Custom & Tema Aplikasi (ThemeData)"
date: 2026-08-14 07:00:00 +0700
tags: [flutter, dart, font, tema, themedata, tutorial, indonesia, pemrograman]
---

Kemarin kita udah bikin app dengan gambar yang rapi. Tapi ada satu hal yang bikin app kamu masih keliatan "default banget": **warna biru bawaan dan font Roboto yang itu-itu aja**. App yang profesional punya **identitas visual** — warna khas, font khas, gaya tombol yang konsisten. Dan di Flutter, semua itu diatur lewat **`ThemeData`**.

Bayangin kamu bikin 20 halaman, tiap tombol kamu tulis `color: Colors.teal` manual. Terus client minta ganti warna jadi oranye. Nyeri kan? Dengan `ThemeData`, kamu ganti **satu tempat**, semua tombol ikut berubah. Itulah kekuatan tema.

---

## 1. Kenapa Harus Pakai `ThemeData`

Tanpa tema, tiap widget harus dikasih style manual satu per satu. Akibatnya:

- **Nggak konsisten** — halaman A tombolnya teal, halaman B lupa jadi biru.
- **Susah diubah** — ganti warna = cari satu-satu di seluruh codebase.
- **Duplikasi** — warna yang sama ditulis berulang-ulang.

Solusinya: definisikan semua style **sekali** di `MaterialApp`, terus pakai di mana-mana.

```dart
MaterialApp(
  theme: ThemeData(
    colorScheme: ColorScheme.fromSeed(seedColor: Colors.teal),
    useMaterial3: true,
  ),
  home: const HomeScreen(),
)
```

Cuma segini, seluruh app kamu otomatis pakai warna teal. Tombol, switch, input, checkbox — semua ikut. Ini karena `ColorScheme.fromSeed` nge-generate palet warna lengkap (primer, sekunder, error, dll) dari satu "benih" warna.

---

## 2. Kustomisasi Lebih Dalam

`ThemeData` punya banyak komponen yang bisa diatur. Ini yang paling sering dipakai:

```dart
ThemeData(
  colorScheme: ColorScheme.fromSeed(seedColor: Colors.teal),
  useMaterial3: true,

  // Font default seluruh app
  fontFamily: 'Poppins',

  // Style AppBar di semua halaman
  appBarTheme: const AppBarTheme(
    centerTitle: true,
    elevation: 0,
  ),

  // Style tombol elevated di semua tempat
  elevatedButtonTheme: ElevatedButtonThemeData(
    style: ElevatedButton.styleFrom(
      padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 14),
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(12),
      ),
      textStyle: const TextStyle(fontWeight: FontWeight.w600),
    ),
  ),

  // Style input field di semua form
  inputDecorationTheme: InputDecorationTheme(
    border: OutlineInputBorder(
      borderRadius: BorderRadius.circular(12),
    ),
    filled: true,
    fillColor: Colors.grey.shade100,
  ),

  // Style card
  cardTheme: CardTheme(
    elevation: 2,
    shape: RoundedRectangleBorder(
      borderRadius: BorderRadius.circular(16),
    ),
  ),
)
```

Sekarang tiap `AppBar`, `ElevatedButton`, `TextFormField`, dan `Card` yang kamu buat **otomatis** ngikut style di atas. Nggak perlu set manual lagi.

---

## 3. Ambil Nilai Tema di Widget — `Theme.of(context)`

Kadang kamu butuh akses warna/teks dari tema di dalam widget. Gunakan `Theme.of(context)`:

```dart
final theme = Theme.of(context);

// Warna primer
final primary = theme.colorScheme.primary;

// Style heading
theme.textTheme.titleLarge

// Warna error
theme.colorScheme.error
```

Contoh nyata — bikin teks yang warnanya ngikut tema:

```dart
Text(
  'Selamat datang!',
  style: Theme.of(context).textTheme.headlineSmall?.copyWith(
        color: Theme.of(context).colorScheme.primary,
        fontWeight: FontWeight.bold,
      ),
)
```

> ⚠️ **Jangan pakai `Colors.teal` langsung** di dalam widget kalau warnanya udah didefinisikan di tema. Selalu ambil dari `Theme.of(context)` biar konsisten dan ikut berubah saat tema berganti.

---

## 4. Font Custom

Font adalah bagian terbesar dari identitas visual. Ada dua cara populer:

### 4.1 Pakai `google_fonts` (paling gampang)

Tambah dependency di `pubspec.yaml`:

```yaml
dependencies:
  google_fonts: ^6.2.0
```

Terus pakai:

```dart
import 'package:google_fonts/google_fonts.dart';

Text(
  'Halo Flutter!',
  style: GoogleFonts.poppins(
    fontSize: 24,
    fontWeight: FontWeight.bold,
  ),
)
```

`google_fonts` otomatis nge-fetch font saat runtime, atau bisa dibundel offline. Cocok buat prototyping dan project yang nggak butuh kontrol penuh atas file font.

### 4.2 Manual dengan File `.ttf` (lebih kontrol)

Download file font (misal `Poppins-Regular.ttf`), taruh di folder `assets/fonts/`, terus daftarkan di `pubspec.yaml`:

```yaml
flutter:
  fonts:
    - family: Poppins
      fonts:
        - asset: assets/fonts/Poppins-Regular.ttf
        - asset: assets/fonts/Poppins-Bold.ttf
          weight: 700
```

Setelah itu, tinggal set `fontFamily: 'Poppins'` di `ThemeData` (lihat bagian 2). Semua teks di app otomatis pakai Poppins. Cara ini lebih "bersih" karena font dibundel langsung ke app, tanpa fetch runtime.

---

## 5. Dark Mode — `darkTheme` & `themeMode`

Theme nggak cuma satu. App modern punya **light & dark mode**. Caranya:

```dart
MaterialApp(
  theme: ThemeData.light(),            // tema terang
  darkTheme: ThemeData.dark(),         // tema gelap
  themeMode: ThemeMode.system,         // ikut settingan HP user
  home: const HomeScreen(),
)
```

- `ThemeMode.system` — ikut mode HP user (paling recommended)
- `ThemeMode.light` — selalu terang
- `ThemeMode.dark` — selalu gelap

Biasanya kamu definisikan dua `ThemeData` custom (light & dark) biar warnanya tetap konsisten di kedua mode.

---

## 6. Contoh Lengkap: App dengan Tema + Dark Mode Toggle

Gabungin semuanya — tema teal, font Poppins, dan tombol ganti mode:

```dart
import 'package:flutter/material.dart';
import 'package:google_fonts/google_fonts.dart';

void main() => runApp(const MyApp());

class MyApp extends StatefulWidget {
  const MyApp({super.key});
  @override
  State<MyApp> createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  bool _darkMode = false;

  ThemeData _buildTheme(Brightness brightness) {
    final scheme = ColorScheme.fromSeed(
      seedColor: Colors.teal,
      brightness: brightness,
    );
    return ThemeData(
      colorScheme: scheme,
      useMaterial3: true,
      textTheme: GoogleFonts.poppinsTextTheme(
        ThemeData(brightness: brightness).textTheme,
      ),
      elevatedButtonTheme: ElevatedButtonThemeData(
        style: ElevatedButton.styleFrom(
          padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 14),
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(12),
          ),
        ),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Tema App',
      debugShowCheckedModeBanner: false,
      theme: _buildTheme(Brightness.light),
      darkTheme: _buildTheme(Brightness.dark),
      themeMode: _darkMode ? ThemeMode.dark : ThemeMode.light,
      home: HomeScreen(
        darkMode: _darkMode,
        onToggle: () => setState(() => _darkMode = !_darkMode),
      ),
    );
  }
}

class HomeScreen extends StatelessWidget {
  final bool darkMode;
  final VoidCallback onToggle;
  const HomeScreen({
    super.key,
    required this.darkMode,
    required this.onToggle,
  });

  @override
  Widget build(BuildContext context) {
    final theme = Theme.of(context);
    return Scaffold(
      appBar: AppBar(title: const Text('🎨 Tema & Font')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: [
            Text(
              'Identitas Visual',
              style: theme.textTheme.headlineMedium?.copyWith(
                color: theme.colorScheme.primary,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 8),
            Text(
              'Font, warna, dan gaya tombol diatur dari satu tempat.',
              style: theme.textTheme.bodyLarge,
            ),
            const SizedBox(height: 24),
            ElevatedButton.icon(
              onPressed: () {},
              icon: const Icon(Icons.favorite),
              label: const Text('Tombol Bertema'),
            ),
            const SizedBox(height: 16),
            OutlinedButton(
              onPressed: () {},
              child: const Text('Tombol Sekunder'),
            ),
            const SizedBox(height: 24),
            Card(
              child: SwitchListTile(
                title: const Text('Dark Mode'),
                subtitle: const Text('Ganti tema terang/gelap'),
                value: darkMode,
                onChanged: (_) => onToggle(),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Jalankan dan coba:** semua tombol, AppBar, dan teks otomatis mengikuti tema. Geser switch dark mode → seluruh app berubah warna secara instan, tanpa ubah satu baris pun di widget-nya. Itulah kekuatan `ThemeData`.

---

## 7. Best Practice

- **Selalu pakai `ColorScheme.fromSeed`** — lebih modern dan menghasilkan palet harmonis daripada set warna manual satu-satu.
- **Ambil warna dari `Theme.of(context)`**, jangan hardcode `Colors.x` di widget.
- **Pisahkan `_buildTheme()`** jadi function terpisah biar kode `main.dart` bersih.
- **Set `useMaterial3: true`** — Material 3 udah jadi default di Flutter terbaru dan tampilannya jauh lebih fresh.
- **Untuk font, bundle manual (`.ttf`)** kalau app butuh offline & konsisten lintas platform. Pakai `google_fonts` buat prototyping cepat.
- **Dark mode wajib** — user zaman sekarang ekspektasinya app punya mode gelap.

---

## Ringkasan

- **`ThemeData`** = definisi warna, font, dan gaya widget di satu tempat
- **`ColorScheme.fromSeed`** = generate palet warna dari satu warna benih
- **`Theme.of(context)`** = ambil nilai tema di widget manapun
- **Font custom** = via `google_fonts` (cepat) atau file `.ttf` (kontrol penuh)
- **`darkTheme` + `themeMode`** = dukung light & dark mode
- **Satu perubahan di tema = seluruh app ikut berubah**

Dengan tema yang rapi, app kamu nggak cuma enak dilihat, tapi juga **mudah di-maintain**. Ini fondasi penting sebelum kita masuk ke state management nanti.

Besok kita bahas **MediaQuery & Orientation Builder** — bikin layout yang responsif di berbagai ukuran layar. Stay tuned! 📱

**Coba sendiri! Bikin app dengan tema khas kamu (warna + font favorit) dan tombol toggle dark mode. Share hasilnya ke sosial media dan tag @ahsai001!**
