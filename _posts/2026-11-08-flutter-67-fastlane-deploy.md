---
layout: post
title: "Flutter #67: Fastlane Deploy — Otomatisasi Build & Release ke App Store & Play Store"
date: 2026-11-08 07:00:00 +0700
categories: [flutter]
tags: [flutter, dart, tutorial, indonesia]
---

Mengirim aplikasi ke App Store dan Play Store secara manual itu melelahkan. Build, sign, upload, isi metadata, screenshot, review — lalu ulangi setiap rilis. **Fastlane** mengotomatiskan semua itu. Artikel #67 kali ini kita bahas cara setup Fastlane untuk Flutter, bikin lane deploy otomatis, dan integrasi ke CI/CD.

---

## Apa Itu Fastlane?

Fastlane adalah tool open-source (Ruby) untuk otomatisasi *deployment* mobile app. Fitur utamanya:

| Fitur | Fungsi |
|-------|--------|
| **gym** | Build iOS (.ipa) / Android (.aab) |
| **pilot** | Upload ke TestFlight |
| **deliver** | Upload metadata, screenshot, binary ke App Store Connect |
| **supply** | Upload AAB, metadata, gambar ke Play Console |
| **match** | Manajemen sertifikat & provisioning profile (iOS) |
| **screengrab** | Generate screenshot otomatis (Android) |

---

## 1. Install Fastlane

### macOS (wajib untuk iOS deploy)

```bash
# Via Homebrew (recommended)
brew install fastlane

# Atau via RubyGems
sudo gem install fastlane
```

### Linux / CI (hanya Android)

```bash
sudo gem install fastlane
```

> **Catatan:** Deploy iOS **harus** di macOS (Xcode, sertifikat, App Store Connect). Android bisa di Linux/macOS/Windows.

---

## 2. Inisialisasi Fastlane di Project Flutter

```bash
cd /path/to/flutter-project
fastlane init
```

Pilih platform:
- `ios` → buat folder `ios/fastlane/`
- `android` → buat folder `android/fastlane/`

Struktur hasilnya:

```
ios/
└── fastlane/
    ├── Appfile          # Konfigurasi app identifier, team ID
    ├── Fastfile         # Definisi lane (script deploy)
    └── Matchfile        # Konfigurasi match (opsional)

android/
└── fastlane/
    ├── Appfile
    └── Fastfile
```

---

## 3. Setup Android: Fastfile & Supply

### `android/fastlane/Appfile`

```ruby
package_name("com.ahsai.myapp")
json_key_file("../service-account.json")  # Service account Google Play
```

### Buat Service Account Google Play Console

1. Buka [Google Cloud Console](https://console.cloud.google.com/)
2. Buat project / pilih existing
3. IAM → Service Accounts → Create Service Account
4. Beri role: **Service Account User** + **Project Viewer**
5. Buka Play Console → Setup → API Access → Link project Cloud
6. Grant access: **Release Manager** (bisa upload AAB, manage tracks)
7. Download JSON key → simpan di `android/fastlane/service-account.json`

> **JANGAN commit file JSON key ke Git!** Masukkan ke `.gitignore` dan inject via CI secret.

### `android/fastlane/Fastfile`

```ruby
default_platform(:android)

platform :android do
  desc "Build release AAB"
  lane :build do
    gradle(
      task: "bundle",
      build_type: "Release",
      project_dir: "./android/",
      properties: {
        "android.injected.signing.store.file" => ENV["ANDROID_KEYSTORE_PATH"],
        "android.injected.signing.store.password" => ENV["ANDROID_KEYSTORE_PASSWORD"],
        "android.injected.signing.key.alias" => ENV["ANDROID_KEY_ALIAS"],
        "android.injected.signing.key.password" => ENV["ANDROID_KEY_PASSWORD"],
      }
    )
  end

  desc "Deploy ke Play Store (internal testing)"
  lane :deploy_internal do
    build
    supply(
      track: "internal",
      aab: "./android/app/build/outputs/bundle/release/app-release.aab",
      json_key: "./service-account.json",
      skip_upload_metadata: false,
      skip_upload_images: false,
      skip_upload_screenshots: false
    )
  end

  desc "Deploy ke Production"
  lane :deploy_production do
    build
    supply(
      track: "production",
      aab: "./android/app/build/outputs/bundle/release/app-release.aab",
      json_key: "./service-account.json"
    )
  end
end
```

---

## 4. Setup iOS: Match, Gym & Deliver

### `ios/fastlane/Appfile`

```ruby
app_identifier("com.ahsai.myapp")
apple_id("your-apple-id@email.com")
team_id("YOUR_TEAM_ID")  # Dari Apple Developer Portal
```

### `ios/fastlane/Matchfile` (opsional tapi recommended)

```ruby
git_url("git@github.com:your-org/fastlane-certs.git")
storage_mode("git")
type("appstore")  # atau "adhoc", "enterprise", "development"
app_identifier(["com.ahsai.myapp"])
username("your-apple-id@email.com")
team_id("YOUR_TEAM_ID")
```

> **Match** menyimpan sertifikat & provisioning profile di repo Git private (terenkripsi). Semua developer & CI pakai yang sama → no more "profile not found" error.

### `ios/fastlane/Fastfile`

```ruby
default_platform(:ios)

platform :ios do
  before_all do
    setup_ci if ENV["CI"]  # Setup keychain di CI
    match(type: "appstore", readonly: true) if ENV["MATCH_READONLY"] == "true"
  end

  desc "Build IPA untuk App Store"
  lane :build do
    increment_build_number(
      build_number: latest_testflight_build_number + 1,
      xcodeproj: "./ios/Runner.xcodeproj"
    )
    gym(
      scheme: "Runner",
      workspace: "./ios/Runner.xcworkspace",
      configuration: "Release",
      export_method: "app-store",
      output_directory: "./build/ios",
      output_name: "Runner.ipa"
    )
  end

  desc "Upload ke TestFlight"
  lane :beta do
    build
    pilot(
      ipa: "./build/ios/Runner.ipa",
      skip_waiting_for_build_processing: true
    )
  end

  desc "Deploy ke App Store"
  lane :release do
    build
    deliver(
      ipa: "./build/ios/Runner.ipa",
      submit_for_review: false,  # Manual review di App Store Connect
      force: true,
      skip_metadata: false,
      skip_screenshots: false
    )
  end
end
```

---

## 5. Keystore & Sertifikat: Simpan Aman

### Android Keystore

```bash
# Generate keystore (hanya sekali)
keytool -genkey -v -keystore upload-keystore.jks \
  -keyalg RSA -keysize 2048 -validity 10000 \
  -alias upload -storepass STORE_PASS -keypass KEY_PASS
```

Simpan env vars di CI (GitHub Actions, GitLab CI, Bitrise, dsb):

| Variable | Contoh |
|----------|--------|
| `ANDROID_KEYSTORE_PATH` | `upload-keystore.jks` |
| `ANDROID_KEYSTORE_PASSWORD` | `********` |
| `ANDROID_KEY_ALIAS` | `upload` |
| `ANDROID_KEY_PASSWORD` | `********` |
| `PLAY_STORE_JSON_KEY` | (base64 encoded JSON) |

### iOS Certificates (via Match)

```bash
# Pertama kali: generate & simpan ke repo certs
fastlane match appstore

# Di CI: readonly=true (hanya download, tidak generate baru)
fastlane match appstore readonly:true
```

---

## 6. Integrasi ke GitHub Actions (CI/CD)

### `.github/workflows/deploy.yml`

```yaml
name: Deploy Flutter App

on:
  workflow_dispatch:
    inputs:
      platform:
        description: 'Platform to deploy'
        required: true
        type: choice
        options: [android, ios, both]
      track:
        description: 'Release track'
        required: false
        default: 'internal'
        type: choice
        options: [internal, alpha, beta, production]

jobs:
  deploy-android:
    if: github.event.inputs.platform == 'android' || github.event.inputs.platform == 'both'
    runs-on: ubuntu-latest
    timeout-minutes: 30
    steps:
      - uses: actions/checkout@v4

      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.24.0'
          channel: 'stable'

      - name: Install dependencies
        run: flutter pub get

      - name: Decode keystore
        run: |
          echo "${{ secrets.ANDROID_KEYSTORE_BASE64 }}" | base64 -d > android/upload-keystore.jks

      - name: Decode Play Store JSON
        run: |
          echo "${{ secrets.PLAY_STORE_JSON_KEY }}" | base64 -d > android/fastlane/service-account.json

      - name: Setup Ruby & Fastlane
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.2'
          bundler-cache: true

      - name: Deploy Android
        run: |
          cd android
          bundle exec fastlane deploy_${{ github.event.inputs.track }}
        env:
          ANDROID_KEYSTORE_PATH: ../upload-keystore.jks
          ANDROID_KEYSTORE_PASSWORD: ${{ secrets.ANDROID_KEYSTORE_PASSWORD }}
          ANDROID_KEY_ALIAS: ${{ secrets.ANDROID_KEY_ALIAS }}
          ANDROID_KEY_PASSWORD: ${{ secrets.ANDROID_KEY_PASSWORD }}

  deploy-ios:
    if: github.event.inputs.platform == 'ios' || github.event.inputs.platform == 'both'
    runs-on: macos-latest
    timeout-minutes: 45
    steps:
      - uses: actions/checkout@v4

      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.24.0'

      - name: Install dependencies
        run: flutter pub get

      - name: Setup Ruby & Fastlane
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.2'
          bundler-cache: true

      - name: Install CocoaPods
        run: cd ios && pod install

      - name: Decode Match certs (SSH key)
        run: |
          echo "${{ secrets.MATCH_GIT_SSH_KEY }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan github.com >> ~/.ssh/known_hosts

      - name: Deploy iOS
        run: |
          cd ios
          bundle exec fastlane ${{ github.event.inputs.track == 'production' && 'release' || 'beta' }}
        env:
          MATCH_PASSWORD: ${{ secrets.MATCH_PASSWORD }}
          MATCH_READONLY: "true"
          APP_STORE_CONNECT_API_KEY_KEY_ID: ${{ secrets.APP_STORE_CONNECT_API_KEY_KEY_ID }}
          APP_STORE_CONNECT_API_KEY_ISSUER_ID: ${{ secrets.APP_STORE_CONNECT_API_KEY_ISSUER_ID }}
          APP_STORE_CONNECT_API_KEY_PRIVATE_KEY: ${{ secrets.APP_STORE_CONNECT_API_KEY_PRIVATE_KEY }}
```

> **Tips:** Gunakan **App Store Connect API Key** (bukan password Apple ID) untuk CI — lebih aman & tidak butuh 2FA.

---

## 7. Metadata & Screenshot Otomatis

Fastlane juga manage metadata (deskripsi, kata kunci, URL support, dll) dan screenshot.

### Struktur folder metadata

```
android/fastlane/metadata/android/
├── en-US/
│   ├── title.txt
│   ├── short_description.txt
│   ├── full_description.txt
│   └── changelogs/
│       └── 67.txt
├── id-ID/
│   ├── title.txt
│   └── ...
└── images/
    ├── phoneScreenshots/
    │   ├── 1.png
    │   └── 2.png
    └── featureGraphic.png
```

### `ios/fastlane/deliver/` (mirip struktur)

```bash
# Generate template metadata
fastlane deliver init
```

Lalu edit file `.txt` dan taruh screenshot di folder `images/`. Fastlane `deliver` & `supply` akan upload otomatis.

---

## 8. Lane Praktis untuk Workflow Harian

| Perintah | Fungsi |
|----------|--------|
| `fastlane android build` | Build AAB saja |
| `fastlane android deploy_internal` | Build + upload ke Internal Testing |
| `fastlane android deploy_production` | Build + upload ke Production |
| `fastlane ios build` | Build IPA (increment build number otomatis) |
| `fastlane ios beta` | Build + upload ke TestFlight |
| `fastlane ios release` | Build + upload ke App Store (draft) |

---

## 9. Troubleshooting Umum

| Error | Solusi |
|-------|--------|
| `Keystore not found` | Pastikan path keystore benar & env var ter-set di CI |
| `Invalid JSON key` | Pastikan service account punya role Release Manager di Play Console |
| `Match: repo not accessible` | Setup SSH key di CI, `ssh-keyscan github.com` |
| `Provisioning profile expired` | `fastlane match nuke` lalu `fastlane match appstore` regenerate |
| `App Store Connect: 2FA required` | Gunakan API Key (bukan Apple ID password) |

---

## 10. Best Practices Ringkas

1. **Semua secret di CI** — tidak ada keystore, JSON key, password di repo
2. **Match readonly di CI** — hindari regenerate sertifikat otomatis
3. **Increment build number otomatis** — pakai `latest_testflight_build_number` (iOS) atau `google_play_track_version_codes` (Android)
4. **Test di Internal/Alpha dulu** — jangan langsung Production
5. **Changelog otomatis** — generate dari commit message conventional commits
6. **Screenshot via screengrab** — untuk Android, otomatis di device farm

---

## Kesimpulan

Fastlane mengubah proses release dari **manual 1-2 jam** jadi **otomatis 10-15 menit**. Setup awal butuh effort (sertifikat, service account, CI secrets), tapi sekali jalan — setiap rilis tinggal:

```bash
# Deploy Android internal
fastlane android deploy_internal

# Deploy iOS TestFlight
fastlane ios beta
```

Atau tekan tombol **Run workflow** di GitHub Actions. Done.

Artikel selanjutnya (#68): **Flutter Web SEO** — cara bikin Flutter web bisa di-index Google dengan benar.

---

**Coba sendiri!** Setup Fastlane di project Flutter kamu, push ke GitHub Actions, dan rasakan kemudahan deploy otomatis. Share pengalamanmu ke sosial media dan tag **@ahsai001** 🚀