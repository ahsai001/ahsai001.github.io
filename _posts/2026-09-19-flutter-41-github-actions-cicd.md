---
layout: post
title: "Flutter #41: GitHub Actions CI/CD"
date: 2026-09-19 07:00:00 +0700
tags: [flutter, dart, tutorial, indonesia, pemrograman, cicd, github-actions]
---

Selamat pagi! Hari ini kita masuk ke topik yang sering jadi tanda tanya bagi developer: **CI/CD dengan GitHub Actions**. Kalau kamu pernah manual build APK, upload ke Play Store, atau cek test satu-satu di lokal — artikel ini bakal bantu otomatisasiin semuanya.

GitHub Actions gratis untuk repo publik (dan cukup generous untuk private). Kita cuma butuh file YAML di `.github/workflows/`, dan setiap push ke `main` atau pull request bakal jalanin test, build, bahkan deploy otomatis.

---

## Kenapa Perlu CI/CD?

Bayangin skenario ini:
1. Kamu push ke `main` jam 2 pagi.
2. GitHub Actions jalanin `flutter test` — kalau gagal, kamu dapet notifikasi email/Slack.
3. Kalau test lolos, build APK dan AAB otomatis.
4. Artefak (APK/AAB) tersimpan sebagai *artifact* — bisa di-download tim QA.
5. (Opsional) Deploy ke Firebase App Distribution / Play Store Internal Testing.

Semua tanpa nyentuh terminal lokal. **Ini yang namanya *continuous integration* dan *continuous delivery*.**

---

## Struktur Workflow GitHub Actions

File workflow simpan di `.github/workflows/ci.yml`. Format dasar:

```yaml
name: CI/CD Flutter

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    name: Run Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.24.0'
          channel: 'stable'
      - run: flutter pub get
      - run: flutter test --coverage
      - uses: actions/upload-artifact@v4
        with:
          name: coverage
          path: coverage/lcov.info

  build:
    name: Build APK & AAB
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.24.0'
          channel: 'stable'
      - run: flutter pub get
      - run: flutter build apk --release
      - run: flutter build appbundle --release
      - uses: actions/upload-artifact@v4
        with:
          name: release-apk
          path: build/app/outputs/flutter-apk/app-release.apk
      - uses: actions/upload-artifact@v4
        with:
          name: release-aab
          path: build/app/outputs/bundle/release/app-release.aab
```

Penjelasan singkat:
- `on`: trigger kapan workflow jalan (push ke `main`, PR ke `main`).
- `jobs`: daftar job yang dijalanin. `test` jalan dulu, `build` butuh `test` lolos (`needs: test`).
- `runs-on: ubuntu-latest`: pakai runner Linux gratis GitHub.
- `subosito/flutter-action@v2`: action resmi setup Flutter (cache SDK, cepet).
- `actions/upload-artifact@v4`: simpan file hasil build biar bisa di-download dari UI GitHub.

---

## Contoh 1: Workflow Lengkap dengan Code Coverage & Lint

Kita tambah `flutter analyze` dan upload coverage ke Codecov (gratis untuk open source).

```yaml
# .github/workflows/ci.yml
name: CI/CD Flutter

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

env:
  FLUTTER_VERSION: '3.24.0'

jobs:
  analyze:
    name: Analyze & Format
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: ${{ env.FLUTTER_VERSION }}
          channel: 'stable'
      - run: flutter pub get
      - run: dart format --set-exit-if-changed .
      - run: flutter analyze --fatal-infos

  test:
    name: Unit & Widget Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: ${{ env.FLUTTER_VERSION }}
          channel: 'stable'
      - run: flutter pub get
      - run: flutter test --coverage
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v4
        with:
          files: ./coverage/lcov.info
          fail_ci_if_error: false

  build-android:
    name: Build Android (APK + AAB)
    needs: [analyze, test]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: ${{ env.FLUTTER_VERSION }}
          channel: 'stable'
      - run: flutter pub get
      - run: flutter build apk --release --split-per-abi
      - run: flutter build appbundle --release
      - uses: actions/upload-artifact@v4
        with:
          name: android-apks
          path: build/app/outputs/flutter-apk/*.apk
      - uses: actions/upload-artifact@v4
        with:
          name: android-aab
          path: build/app/outputs/bundle/release/app-release.aab

  build-ios:
    name: Build iOS (IPA)
    needs: [analyze, test]
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: ${{ env.FLUTTER_VERSION }}
          channel: 'stable'
      - run: flutter pub get
      - run: flutter build ipa --release --export-options-plist=ExportOptions.plist
      - uses: actions/upload-artifact@v4
        with:
          name: ios-ipa
          path: build/ios/ipa/*.ipa
```

Catatan:
- `build-ios` butuh `macos-latest` runner (macOS runner gratis tapi quota terbatas).
- `--split-per-abi` biar APK jadi 3 file (arm64-v8a, armeabi-v7a, x86_64) — ukuran lebih kecil.
- `ExportOptions.plist` perlu disiapin di repo untuk signing iOS (lihat docs Apple).

---

## Contoh 2: Deploy ke Firebase App Distribution

Supaya QA bisa install APK langsung dari Firebase, tambah step deploy:

```yaml
# Tambah di job build-android setelah upload-artifact
- name: Deploy to Firebase App Distribution
  uses: wzieba/Firebase-Distribution-Github-Action@v1
  with:
    appId: ${{ secrets.FIREBASE_APP_ID }}
    token: ${{ secrets.FIREBASE_TOKEN }}
    groups: qa-team
    file: build/app/outputs/flutter-apk/app-release.apk
```

Butuh setup di Firebase Console:
1. Buat project Firebase → App Distribution.
2. Dapatkan `App ID` (format `1:123456789:android:abcdef`).
3. Generate `FIREBASE_TOKEN` via `firebase login:ci` di lokal, simpan ke GitHub Secrets (`Settings → Secrets → Actions`).

---

## Tips & Best Practice

| Tips | Kenapa |
|------|--------|
| Cache `pub` cache | `actions/cache@v4` key `pub-${{ hashFiles('**/pubspec.lock') }}` — cepetin `flutter pub get` |
| Pin Flutter version | Hindari breaking change pas update Flutter stable |
| Fail fast | `flutter analyze --fatal-infos` biar warning jadi error di CI |
| Matrix testing | Test di multiple Flutter version / OS pakai `strategy.matrix` |
| Separate workflow | Pisah `ci.yml` (test+analyze) dan `release.yml` (build+deploy) — biar PR cepet, release manual |

Contoh cache `pub`:

```yaml
- uses: actions/cache@v4
  with:
    path: |
      ~/.pub-cache
      ${{ runner.tool_cache }}/flutter
    key: flutter-${{ env.FLUTTER_VERSION }}-${{ hashFiles('**/pubspec.lock') }}
    restore-keys: |
      flutter-${{ env.FLUTTER_VERSION }}-
```

---

## Debugging Workflow Gagal

1. **Cek log di tab Actions** — klik job yang merah, expand step yang gagal.
2. **Re-run job** — tombol "Re-run jobs" di kanan atas.
3. **SSH ke runner** (hanya private repo / enterprise) — pakai `mxschmitt/action-tmate@master` untuk debug interaktif.
4. **Local act** — `nektos/act` jalanin GitHub Actions di lokal (butuh Docker).

---

## Checklist Sebelum Push

- [ ] `flutter analyze` bersih di lokal
- [ ] `flutter test` semua lolos
- [ ] `pubspec.lock` committed (bukan di `.gitignore`)
- [ ] Secrets di GitHub: `FIREBASE_TOKEN`, `FIREBASE_APP_ID`, dll
- [ ] `ExportOptions.plist` untuk iOS (kalau build iOS)

---

## Kesimpulan

GitHub Actions + Flutter = **otomatisasi gratis & powerful**. Mulai dari workflow sederhana (test + build APK) sampe pipeline lengkap dengan deploy ke Firebase/Play Store. Kuncinya: **mulai kecil**, tambah step seiring butuh.

> **Tips:** Kalau repo private, GitHub Actions gratis 2000 menit/bulan. Cukup untuk project pribadi & tim kecil.

---

Coba sendiri! Buat file `.github/workflows/ci.yml`, push, dan lihat tab **Actions** di repo GitHub kamu. Share ke sosial media dan tag **@ahsai001** kalau berhasil (atau butuh bantuan debug).

---

**Artikel selanjutnya:** Flutter #42 — Build APK/AAB & AppBundle (detail signing, versioning, Play Console).