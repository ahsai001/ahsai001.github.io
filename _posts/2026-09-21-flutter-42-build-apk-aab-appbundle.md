---
layout: post
title: "Flutter #42: Build APK/AAB & AppBundle"
date: 2026-09-21 07:00:00 +0700
tags: [flutter, dart, tutorial, indonesia, pemrograman, release, android]
---

Selamat pagi! Setelah kemarin kita setup CI/CD dengan GitHub Actions, hari ini kita bahas bagian yang paling ditunggu: **build release APK/AAB** sampai siap upload ke Play Store. Ini tahap di mana aplikasimu berhenti jadi "project demo" dan mulai jadi produk beneran.

Banyak developer baru mentok di sini karena signing dan versioning terasa ribet. Padahal kalau dipahami sekali, urutannya cuma tiga langkah: **versioning → signing → build**. Yuk kita bongkar satu-satu.

---

## APK vs AAB: Beda & Kapan Pakai

| Aspek | APK | AAB (App Bundle) |
|-------|-----|------------------|
| Format | Satu file monolitik | Bundle modular |
| Ukuran download | Lebih besar | 15-35% lebih kecil (di-split Play Store) |
| Distribusi | Bebas (WA, website, sideload) | Wajib via Play Store |
| Status di Play | Masih didukung | **Wajib** untuk aplikasi baru |

Aturan simpelnya:

- **Upload ke Play Store** → pakai **AAB**.
- **Bagi-bagi ke teman / testing manual** → pakai **APK**.
- **Testing berbagai device** → APK `--split-per-abi` (pisah armeabi-v7a, arm64-v8a, x86_64).

---

## Versioning: `versionName` & `versionCode`

Buka `pubspec.yaml`:

```yaml
name: my_app
version: 1.2.0+3
```

Format-nya `versionName+versionCode`:

- `1.2.0` → **versionName**, yang dilihat user di Play Store.
- `3` → **versionCode**, integer yang dibaca Android. **Harus naik setiap upload**, tidak boleh sama atau turun.

Flutter otomatis memetakan ini ke `build.gradle`, jadi kamu **cukup edit satu tempat** — jangan hardcode di `build.gradle`:

```gradle
// android/app/build.gradle
defaultConfig {
    versionName flutterVersionName   // diisi dari pubspec.yaml
    versionCode flutterVersionCode.toInteger()
}
```

> **Tips:** Sebelum rilis fitur baru, naikkan versionName (1.2.0 → 1.3.0). Untuk hotfix, cukup naikkan versionCode saja (1.2.0+3 → 1.2.0+4).

---

## Signing: Bikin Keystore & Konfigurasi Release

APK release **wajib** ditandatangani. Kalau tidak, tidak bisa diupload ke Play Store. Bikin keystore sekali seumur project:

```bash
keytool -genkey -v -keystore ~/upload-keystore.jks \
  -keyalg RSA -keysize 2048 -validity 10000 \
  -alias upload
```

Simpan password-nya di `android/key.properties` (dan **jangan commit** file ini — masukkan ke `.gitignore`):

```properties
storePassword=rahasia123
keyPassword=rahasia123
keyAlias=upload
storeFile=/home/ahsai/upload-keystore.jks
```

Lalu sambungkan ke `android/app/build.gradle` (modul app):

```gradle
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}

android {
    signingConfigs {
        release {
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile file(keystoreProperties['storeFile'])
            storePassword keystoreProperties['storePassword']
        }
    }
    buildTypes {
        release {
            signingConfig signingConfigs.release
            minifyEnabled true
            shrinkResources true
        }
    }
}
```

`minifyEnabled` + `shrinkResources` mengaktifkan R8/ProGuard — bikin APK lebih kecil dan kode sedikit lebih sulit di-reverse-engineer.

---

## Build Itu Sendiri

```bash
# AAB untuk Play Store
flutter build appbundle --release

# APK universal (untuk sharing manual)
flutter build apk --release

# APK dipisah per arsitektur (paling ramping)
flutter build apk --release --split-per-abi

# Plus obfuscation + simpan symbol untuk crash report
flutter build appbundle --release \
  --obfuscate \
  --split-debug-info=build/debug-info
```

Hasil build ada di:

- `build/app/outputs/bundle/release/app-release.aab`
- `build/app/outputs/flutter-apk/app-release.apk`
- `build/app/outputs/flutter-apk/app-armeabi-v7a-release.apk` (kalau split)

> **Penting:** Simpan folder `build/debug-info` hasil `--split-debug-info`. Tanpa itu, stack trace dari crash user di Play Console akan tetap ter-obfuscate dan susah dibaca.

---

## Contoh 1: Tampilkan Versi Aplikasi di UI

User sering komplain "coba update dulu" padahal mereka sudah versi terbaru. Solusinya: tampilkan versi aplikasi langsung di halaman About. Pakai `package_info_plus`:

```yaml
# pubspec.yaml
dependencies:
  package_info_plus: ^8.0.0
```

```dart
import 'package:flutter/material.dart';
import 'package:package_info_plus/package_info_plus.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Build Info Demo',
      home: const AboutPage(),
    );
  }
}

class AboutPage extends StatefulWidget {
  const AboutPage({super.key});

  @override
  State<AboutPage> createState() => _AboutPageState();
}

class _AboutPageState extends State<AboutPage> {
  String _info = 'Memuat...';

  @override
  void initState() {
    super.initState();
    _loadPackageInfo();
  }

  Future<void> _loadPackageInfo() async {
    final info = await PackageInfo.fromPlatform();
    if (!mounted) return;
    setState(() {
      _info = '${info.appName} v${info.version} (build ${info.buildNumber})';
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Tentang Aplikasi')),
      body: Center(
        child: Text(_info, style: const TextStyle(fontSize: 18)),
      ),
    );
  }
}
```

`info.version` = versionName dari `pubspec.yaml`, `info.buildNumber` = versionCode. Kode ini runnable langsung setelah `flutter pub get`.

---

## Contoh 2: Cek Versi Terbaru dari API

Pola paling umum: backend menyimpan versi minimum yang didukung, aplikasi membandingkan saat dibuka. Kalau ketinggalan, tampilkan dialog paksa update.

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;
import 'package:package_info_plus/package_info_plus.dart';

Future<bool> isUpdateRequired(String currentVersion) async {
  final res = await http.get(
    Uri.parse('https://api.example.com/app/version'),
  );
  if (res.statusCode != 200) return false;

  final data = jsonDecode(res.body) as Map<String, dynamic>;
  final minVersion = data['min_supported'] as String;

  // Bandingkan per segmen: "1.10.0" > "1.9.0" (string compare salah di sini!)
  List<int> parse(String v) => v.split('.').map(int.parse).toList();
  final cur = parse(currentVersion);
  final min = parse(minVersion);

  for (var i = 0; i < min.length; i++) {
    if ((i < cur.length ? cur[i] : 0) < min[i]) return true;
    if ((i < cur.length ? cur[i] : 0) > min[i]) return false;
  }
  return false;
}

Future<void> checkForUpdate(BuildContext context) async {
  final info = await PackageInfo.fromPlatform();
  final needsUpdate = await isUpdateRequired(info.version);
  if (!needsUpdate || !context.mounted) return;

  await showDialog<void>(
    context: context,
    barrierDismissible: false,
    builder: (ctx) => AlertDialog(
      title: const Text('Update Tersedia'),
      content: const Text(
        'Versi baru sudah tersedia. Update dulu untuk lanjut pakai aplikasi.',
      ),
      actions: [
        TextButton(
          onPressed: () => Navigator.pop(ctx),
          child: const Text('Nanti'),
        ),
      ],
    ),
  );
}
```

Perhatikan: membandingkan versi pakai string biasa **salah** — `"1.10.0" < "1.9.0"` secara string, padahal secara versi lebih baru. Makanya versi dipecah jadi integer per segmen.

---

## Mengecilkan Ukuran APK

1. **`--split-per-abi`** — hemat paling besar, user cuma download arsitektur device-nya.
2. **`minifyEnabled` + `shrinkResources`** — buang kode & resource tak terpakai.
3. **Ganti PNG ke WebP** — hemat 25-70% per gambar.
4. **Audit dependency** — jalankan `flutter pub deps` dan buang package yang cuma dipakai satu-dua baris kode.
5. **Cek isi APK:**

```bash
flutter build apk --analyze-size --target-platform android-arm64
```

Command ini menghasilkan file JSON berisi rincian ukuran per package — kepakai banget buat ngejar APK yang bengkak tak jelas asalnya.

---

## Upload ke Play Console

1. Buka [play.google.com/console](https://play.google.com/console) → pilih app.
2. **Testing → Internal testing** → Create new release.
3. Upload `.aab` hasil build. Play Console akan menolak kalau versionCode sudah pernah dipakai.
4. Isi release notes singkat (bahasa Indonesia + Inggris lebih baik).
5. Rollout → tunggu review (biasanya beberapa jam untuk internal testing).

Kalau ini rilis pertama, kamu akan diminta mengaktifkan **Play App Signing** — Google menyimpan signing key produksi, sementara kamu upload pakai upload key. Ini fitur bagus: kalau upload key-mu bocor, masih bisa di-reset lewat support.

---

## Checklist Sebelum Rilis

- [ ] `version` di `pubspec.yaml` sudah naik (versionCode **wajib** naik)
- [ ] `key.properties` ada, dan **sudah masuk `.gitignore`**
- [ ] Keystore di-backup di dua tempat (kehilangan ini = tidak bisa update app selamanya)
- [ ] `flutter analyze` bersih, `flutter test` lolos
- [ ] Build pakai `--obfuscate --split-debug-info`, folder debug-info disimpan
- [ ] Test APK release di device fisik (bukan cuma debug mode!)
- [ ] Icon & splash screen sudah pakai asset final
- [ ] Privacy policy URL siap (wajib untuk Play Store)

---

## Kesimpulan

Build release Flutter itu cuma soal disiplin di tiga titik: **versioning** (naikkan versionCode setiap upload), **signing** (keystore sekali bikin, jaga baik-baik), dan **build command** yang tepat (AAB untuk Play, APK split untuk sharing).

Begitu tiga ini beres, rilis berikutnya cuma butuh dua menit: ubah versi, jalankan `flutter build appbundle`, upload. Sisa energinya bisa dipakai buat ngembangin fitur, bukan ngurus konfigurasi.

> **Tips:** Simpan keystore-mu di password manager atau cloud storage terenkripsi. Kalau hilang dan kamu ikut Play App Signing, biasanya masih bisa diselamatkan — kalau tidak, aplikasi itu tamat riwayatnya.

---

Coba sendiri! Build AAB aplikasimu hari ini, test di internal testing Play Console, dan lihat ukuran download-nya turun dibanding APK universal. Share ke sosial media dan tag **@ahsai001** kalau berhasil rilis.

---

**Artikel selanjutnya:** Flutter #43 — Animasi: Implicit vs Explicit (AnimatedContainer, AnimatedOpacity, sampai AnimationController).
