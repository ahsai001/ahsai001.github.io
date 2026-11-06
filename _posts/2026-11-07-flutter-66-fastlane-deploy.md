---
layout: post
title: "Flutter #66: Fastlane Deploy — Otomatisasi Build & Release ke Play Store & App Store"
date: 2026-11-07 07:00:00 +0700
categories: [flutter]
tags: [flutter, dart, tutorial, indonesia]
---

Halo semuanya! Kita sudah sampai di artikel ke-66 dari seri 90 hari Flutter. Hari ini kita bahas **Fastlane** — tool wajib untuk mengotomatiskan proses build dan deploy aplikasi Flutter ke Google Play Store dan Apple App Store.

Bayangkan: tanpa Fastlane, setiap release kamu harus manual `flutter build`, buka browser ke Play Console / App Store Connect, upload AAB/IPA, isi changelog, submit review... Ulangi setiap minggu. Dengan Fastlane? Cukup satu command: `fastlane deploy`. Selesai.

---

## Apa Itu Fastlane?

Fastlane adalah open-source tool (Ruby) yang mengotomatiskan tugas-tugas deployment mobile: build, screenshot, metadata, upload ke store, bahkan beta testing via TestFlight / Firebase App Distribution.

**Kenapa pakai Fastlane?**
- **Konsisten**: Build & release identik setiap kali, tidak ada langkah terlewat
- **Cepat**: Dari commit ke store dalam menit, bukan jam
- **CI/CD Ready**: Integrasi mulus dengan GitHub Actions, GitLab CI, Bitrise, Codemagic
- **Multi-platform**: Satu konfigurasi untuk iOS & Android

---

## Setup Fastlane untuk Flutter

### 1. Install Fastlane

```bash
# macOS (via Homebrew)
brew install fastlane

# Atau via RubyGems (cross-platform)
sudo gem install fastlane
```

> **Catatan**: Fastlane butuh Ruby. Di macOS sudah built-in. Di Linux/Windows, install Ruby dulu (pakai `rbenv` atau `rvm` direkomendasikan).

### 2. Inisialisasi di Project Flutter

```bash
cd your_flutter_project
fastlane init
```

Pilih platform: `android` dan/atau `ios`. Fastlane akan membuat folder struktur:

```
your_flutter_project/
├── android/
│   └── fastlane/
│       ├── Appfile
│       └── Fastfile
├── ios/
│   └── fastlane/
│       ├── Appfile
│       └── Fastfile
└── Gemfile (optional, untuk bundler)
```

---

## Konfigurasi Android (Google Play Store)

### 1. Siapkan Service Account Google Play Console

1. Buka [Google Cloud Console](https://console.cloud.google.com/)
2. Buat project baru / pilih existing
3. Enable **Google Play Android Developer API**
4. Buat **Service Account** → beri role **Project Owner** (atau minimal **Release Manager** di Play Console)
5. Download JSON key → simpan di `android/fastlane/google-play-key.json`
6. Di Play Console: **Users & permissions** → invite service account email dengan role **Release Manager**

### 2. Edit `android/fastlane/Appfile`

```ruby
package_name("com.example.myapp")
json_key_file("fastlane/google-play-key.json")
```

### 3. Edit `android/fastlane/Fastfile`

```ruby
default_platform(:android)

platform :android do
  desc "Build & deploy ke Internal Testing Track"
  lane :deploy_internal do
    gradle(task: "clean bundleRelease")
    upload_to_play_store(
      track: "internal",
      aab: "../build/app/outputs/bundle/release/app-release.aab",
      skip_upload_apk: true,
      skip_upload_metadata: true,
      skip_upload_images: true,
      skip_upload_screenshots: true,
      release_status: "draft",
      changes_not_sent_for_review: true
    )
  end

  desc "Build & deploy ke Production"
  lane :deploy_production do
    gradle(task: "clean bundleRelease")
    upload_to_play_store(
      track: "production",
      aab: "../build/app/outputs/bundle/release/app-release.aab",
      release_status: "completed",
      rollout: "0.1" # 10% dulu, naikkan bertahap
    )
  end

  desc "Build saja, tidak upload"
  lane :build_only do
    gradle(task: "clean bundleRelease")
  end
end
```

---

## Konfigurasi iOS (App Store Connect)

### 1. Siapkan App Store Connect API Key

1. Buka [App Store Connect](https://appstoreconnect.apple.com/) → **Users & Access** → **Keys**
2. Generate **API Key** dengan role **App Manager** atau **Admin**
3. Catat: `Key ID`, `Issuer ID`, download `.p8` private key
4. Simpan `.p8` di `ios/fastlane/AuthKey_XXXXXX.p8`

### 2. Edit `ios/fastlane/Appfile`

```ruby
app_identifier("com.example.myapp")
apple_id("your@appleid.com")
team_id("YOUR_TEAM_ID") # Dari Apple Developer Portal
itc_team_id("YOUR_ITC_TEAM_ID") # Dari App Store Connect
```

### 3. Edit `ios/fastlane/Fastfile`

```ruby
default_platform(:ios)

platform :ios do
  desc "Build & upload ke TestFlight"
  lane :beta do
    # Sync certificates & profiles via match (recommended)
    # match(type: "appstore", readonly: true)
    
    build_app(
      scheme: "Runner",
      workspace: "Runner.xcworkspace",
      configuration: "Release",
      export_method: "app-store",
      output_directory: "./build/ios",
      output_name: "Runner.ipa"
    )
    
    upload_to_testflight(
      api_key_path: "./AuthKey_XXXXXX.p8",
      api_key_id: "YOUR_KEY_ID",
        api_issuer_id: "YOUR_ISSUER_ID",
      skip_waiting_for_build_processing: true
    )
  end

  desc "Build & submit ke App Store"
  lane :release do
    # match(type: "appstore", readonly: true)
    
    build_app(
      scheme: "Runner",
      workspace: "Runner.xcworkspace",
      configuration: "Release",
      export_method: "app-store",
      output_directory: "./build/ios",
      output_name: "Runner.ipa"
    )
    
    upload_to_app_store(
      api_key_path: "./AuthKey_XXXXXX.p8",
      api_key_id: "YOUR_KEY_ID",
      api_issuer_id: "YOUR_ISSUER_ID",
      submit_for_review: true,
      automatic_release: true,
      force: true,
      skip_metadata: false,
      skip_screenshots: false,
      precheck_include_in_app_purchases: false
    )
  end
end
```

> **Pro Tip**: Gunakan `match` (Fastlane's cert management) untuk handle signing certificate & provisioning profile otomatis via Git repo terenkripsi. Baca docs: [fastlane.tools/match](https://docs.fastlane.tools/actions/match/)

---

## Menjalankan Fastlane

```bash
# Android
cd android
bundle exec fastlane deploy_internal   # ke Internal Testing
bundle exec fastlane deploy_production # ke Production

# iOS
cd ios
bundle exec fastlane beta    # ke TestFlight
bundle exec fastlane release # ke App Store
```

---

## Integrasi dengan GitHub Actions (CI/CD)

Buat file `.github/workflows/deploy.yml`:

```yaml
name: Deploy to Stores

on:
  push:
    tags:
      - 'v*' # Trigger saat push tag v1.0.0, v1.0.1, dst.

jobs:
  deploy-android:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.24.0'
      - name: Install dependencies
        run: flutter pub get
      - name: Build AAB
        run: flutter build appbundle --release
      - name: Setup Fastlane
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.2'
          bundler-cache: true
      - name: Deploy to Play Store Internal
        run: |
          cd android
          bundle exec fastlane deploy_internal
        env:
          PLAY_STORE_JSON_KEY: ${{ secrets.PLAY_STORE_JSON_KEY }}
          # Atau gunakan file key yang sudah di-commit (encrypt dulu dengan git-crypt/sops)

  deploy-ios:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.24.0'
      - name: Install dependencies
        run: flutter pub get
      - name: Build IPA
        run: flutter build ipa --release
      - name: Setup Fastlane
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.2'
          bundler-cache: true
      - name: Deploy to TestFlight
        run: |
          cd ios
          bundle exec fastlane beta
        env:
          APP_STORE_CONNECT_API_KEY: ${{ secrets.APP_STORE_CONNECT_API_KEY }}
          APP_STORE_CONNECT_ISSUER_ID: ${{ secrets.APP_STORE_CONNECT_ISSUER_ID }}
          APP_STORE_CONNECT_KEY_ID: ${{ secrets.APP_STORE_CONNECT_KEY_ID }}
```

> **Keamanan**: JANGAN commit service account JSON / `.p8` key ke repo publik. Gunakan **GitHub Secrets** + inject ke file saat runtime, atau gunakan `git-crypt` / `sops` untuk encrypt file sensitif.

---

## Tips & Best Practices

| Tips | Penjelasan |
|------|------------|
| **Versioning otomatis** | Gunakan `flutter_version` plugin atau script untuk bump `pubspec.yaml` version sebelum build |
| **Changelog otomatis** | Integrasi dengan `github_changelog_generator` atau conventional commits |
| **Screenshot otomatis** | `fastlane snapshot` + `frameit` untuk generate screenshot di semua device size |
| **Beta testing** | Gunakan Firebase App Distribution (`firebase_app_distribution` action) untuk internal tester sebelum ke store |
| **Rollout bertahap** | Play Store: mulai `rollout: "0.1"` (10%), naikkan via Play Console atau Fastlane `upload_to_play_store(rollout: "1.0")` |
| **Fastfile modular** | Pisahkan lane per environment: `deploy_dev`, `deploy_staging`, `deploy_prod` |

---

## Contoh: Auto Version Bump + Build + Deploy

Tambahkan di `android/fastlane/Fastfile`:

```ruby
lane :bump_and_deploy do
  # Bump version code (otomatis +1 dari Play Store)
  current_version = google_play_track_version_codes(track: "internal").first
  new_version_code = current_version + 1
  
  # Update pubspec.yaml version (butuh script terpisah atau plugin)
  # sh("cd ../.. && dart run flutter_version:main --build-number #{new_version_code}")
  
  gradle(task: "clean bundleRelease")
  upload_to_play_store(
    track: "internal",
    version_code: new_version_code,
    aab: "../build/app/outputs/bundle/release/app-release.aab"
  )
end
```

---

## Kesimpulan

Fastlane mengubah deployment mobile dari **proses manual yang rawan error** jadi **pipeline otomatis yang reliable**. Investasi setup awal ~1-2 hari, tapi menghemat puluhan jam setiap release.

**Langkah selanjutnya**: Coba setup Fastlane di project Flutter kamu hari ini. Mulai dari Android Internal Testing track dulu, lalu iOS TestFlight. Kalau sudah lancar, integrasikan ke GitHub Actions biar *push tag → auto deploy*.

---

Coba sendiri! Share ke sosial media dan tag @ahsai001 🚀