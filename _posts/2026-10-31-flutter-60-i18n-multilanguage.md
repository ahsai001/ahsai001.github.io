---
layout: post
title: "Flutter #60: Internationalization (i18n) & Multi‑language"
date: 2026-10-31 07:00:00 +0700
categories: [flutter]
tags: [flutter, dart, tutorial, indonesia]
---

## Pendahuluan
Flutter memungkinkan aplikasi berjalan di berbagai platform dengan satu codebase. Tapi kalau aplikasi hanya berbahasa Inggris, pengguna di Indonesia, Malaysia, atau Jepang akan kesulitan. Internationalization (i18n) adalah teknik mengisolasi teks, tanggal, dan format angka sehingga aplikasi dapat menerjemahkannya ke bahasa lain secara dinamis.

## 1. Menyiapkan Paket `flutter_localizations`
Flutter sudah menyertakan paket `flutter_localizations`. Kita hanya perlu menambahkannya di `pubspec.yaml` dan mengaktifkan `MaterialApp`‑nya.

```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_localizations:
    sdk: flutter # <‑ penting
```

Setelah `flutter pub get`, ubah `MaterialApp` menjadi:

```dart
import 'package:flutter_localizations/flutter_localizations.dart';
import 'generated/l10n.dart'; // akan dibuat di langkah berikut

MaterialApp(
  localizationsDelegates: const [
    S.delegate, // generated class
    GlobalMaterialLocalizations.delegate,
    GlobalWidgetsLocalizations.delegate,
    GlobalCupertinoLocalizations.delegate,
  ],
  supportedLocales: S.delegate.supportedLocales,
  locale: const Locale('id'), // default bahasa Indonesia
  home: const MyHomePage(),
);
```

## 2. Membuat File Arb (Application Resource Bundle)
Arb adalah format JSON yang mudah dibaca. Kita buat folder `lib/l10n/` dan file `app_en.arb` serta `app_id.arb`.

`lib/l10n/app_en.arb`
```json
{
  "@@locale": "en",
  "title": "Welcome to Flutter",
  "greeting": "Hello {name}!",
  "@greeting": {
    "description": "Greeting with user name",
    "placeholders": {"name": {"type": "String"}}
  }
}
```

`lib/l10n/app_id.arb`
```json
{
  "@@locale": "id",
  "title": "Selamat Datang di Flutter",
  "greeting": "Halo {name}!",
  "@greeting": {
    "description": "Sapaan dengan nama pengguna",
    "placeholders": {"name": {"type": "String"}}
  }
}
```

## 3. Meng‑generate Kode dengan `flutter gen-l10n`
Flutter menyediakan perintah otomatis yang meng‑generate kelas `S` dari file arb.

```bash
flutter gen-l10n
```

Perintah ini akan membuat `lib/generated/l10n.dart`. Kelas `S` berisi getter untuk tiap string.

## 4. Menggunakan Teks Terjemahan di UI
Setelah kode di‑generate, pakai `S.of(context)` di widget.

```dart
class MyHomePage extends StatelessWidget {
  const MyHomePage({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    final s = S.of(context);
    return Scaffold(
      appBar: AppBar(title: Text(s.title)),
      body: Center(
        child: Text(s.greeting('Ahsai')),
      ),
    );
  }
}
```

## 5. Mengganti Bahasa Secara Dinamis
Kita dapat menambahkan dropdown di AppBar untuk pilih bahasa.

```dart
class LanguageSwitcher extends StatelessWidget {
  const LanguageSwitcher({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return DropdownButton<Locale>(
      value: Localizations.localeOf(context),
      items: S.delegate.supportedLocales
          .map((locale) => DropdownMenuItem(
                value: locale,
                child: Text(locale.languageCode.toUpperCase()),
              ))
          .toList(),
      onChanged: (Locale? locale) {
        if (locale != null) {
          MyApp.setLocale(context, locale);
        }
      },
    );
  }
}

class MyApp extends StatefulWidget {
  static void setLocale(BuildContext context, Locale locale) {
    _MyAppState? state = context.findAncestorStateOfType<_MyAppState>();
    state?.setState(() => state._locale = locale);
  }

  @override
  _MyAppState createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  Locale _locale = const Locale('id');
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      locale: _locale,
      // delegasi dll sama seperti sebelumnya
      home: const MyHomePage(),
    );
  }
}
```

Dengan dropdown, pengguna dapat pindah antara Inggris (`en`) dan Indonesia (`id`). Tambahkan bahasa lain cukup menambah file `.arb` dan list di `supportedLocales`.

## 6. Menangani Tanggal & Angka
Gunakan paket `intl` untuk format tanggal, waktu, dan angka.

```yaml
dependencies:
  intl: ^0.18.0
```

```dart
import 'package:intl/intl.dart';

String formattedDate = DateFormat.yMMMMd('id').format(DateTime.now());
String formattedNumber = NumberFormat.currency(locale: 'id', symbol: 'Rp').format(1500000);
```

Output akan menjadi `31 Oktober 2026` dan `Rp1.500.000`.

## 7. Tips & Trik
- **Lint**: Pastikan semua string berada di file `.arb`. Hindari hard‑code teks di widget.
- **Pluralisasi**: Tambahkan kunci `@something` dengan `placeholders` dan `description` untuk plural (contoh `zero`, `one`, `other`).
- **Testing**: Buat unit test untuk memastikan `S.of(context).title` mengembalikan nilai yang tepat pada setiap locale.

## Penutup
Internationalization tidak lagi sulit dengan `flutter_localizations` dan `intl`. Dengan menyiapkan file arb, generate kode, dan menambahkan selector bahasa, aplikasi Anda siap melayani pengguna global.

---

**CTA**: Coba sendiri! Share ke sosial media dan tag @ahsai001.
