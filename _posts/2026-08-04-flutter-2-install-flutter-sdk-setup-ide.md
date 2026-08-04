---
layout: post
title: "Flutter #2: Install Flutter SDK & Setup IDE (VS Code / Android Studio)"
date: 2026-08-04 07:00:00 +0700
tags: [flutter, dart, mobile, tutorial]
---

## Pengantar

Di artikel pertama kita sudah kenalan dengan Flutter dan alasan kenapa framework ini layak dipelajari. Sekarang saatnya aksi: **install Flutter SDK, setup IDE, dan jalankan project pertama!** Tenang, step-by-step-nya bakal gampang banget — bahkan kalau kamu belum pernah setup environment development sebelumnya.

---

## Langkah 1: Install Flutter SDK

### Windows

1. Download Flutter SDK dari [flutter.dev](https://docs.flutter.dev/get-started/install/windows)
2. Ekstrak ZIP ke `C:\src\flutter` (hindari folder seperti `C:\Program Files` yang butuh permission khusus)
3. Tambahkan Flutter ke **Environment Variables → Path**: `C:\src\flutter\bin`
4. Buka Command Prompt atau PowerShell, jalankan:

```bash
flutter doctor
```

### macOS

```bash
# Pakai Homebrew (paling gampang)
brew install --cask flutter

# Atau download manual dari flutter.dev
```

### Linux

```bash
# Download & ekstrak ke ~/development/
cd ~/development
wget https://storage.googleapis.com/flutter_infra_release/releases/stable/linux/flutter_linux_latest.tar.xz
tar xf flutter_linux_latest.tar.xz

# Tambahkan ke PATH (masukkan ke ~/.bashrc atau ~/.zshrc)
export PATH="$PATH:$HOME/development/flutter/bin"

# Reload
source ~/.bashrc
```

> **PENTING**: Setelah install, selalu jalankan `flutter doctor` — command ini akan kasih tahu apa saja yang masih kurang.

---

## Langkah 2: Pahami Output `flutter doctor`

```bash
$ flutter doctor
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel stable, 3.x)
[✓] Android toolchain - develop for Android devices
[✓] Chrome - develop for the web
[✓] Android Studio
[✓] VS Code
[✓] Connected device
```

Setiap tanda centang hijau artinya siap. Kalau ada tanda silang `[✗]` atau tanda seru `[!]`, Flutter akan kasih petunjuk apa yang perlu kamu install/fix. **Jangan skip ini!** Banyak error di kemudian hari gara-gara `flutter doctor` gak bersih.

### Hal yang biasanya perlu diinstall:

| Item | Solusi |
|------|--------|
| `[✗] Android toolchain` | Install Android Studio, lalu buka SDK Manager |
| `[✗] Android license` | Jalankan `flutter doctor --android-licenses` |
| `[!] Chrome` | Install Google Chrome (untuk web development) |
| `[✗] Visual Studio` | Hanya untuk Windows desktop development |

---

## Langkah 3: Setup IDE

Kamu bisa pilih **VS Code** (ringan, rekomendasi untuk pemula) atau **Android Studio** (fitur lengkap, lebih berat).

### VS Code (Rekomendasi)

1. Install [VS Code](https://code.visualstudio.com/)
2. Buka Extensions (`Ctrl+Shift+X`), cari dan install:
   - **Flutter** (otomatis juga install Dart extension)

```dart
// Setelah install extension, test dengan:
// Ctrl+Shift+P → ketik "Flutter: New Project"
```

### Android Studio

1. Install [Android Studio](https://developer.android.com/studio)
2. Saat setup, pastikan centang **Android SDK**, **Android SDK Platform-Tools**, dan **Android Emulator**
3. Buka **SDK Manager** → tab **SDK Tools** → centang **Android SDK Command-line Tools**
4. Install plugin Flutter: **File → Settings → Plugins → cari "Flutter"**

---

## Langkah 4: Setup Emulator & Device

### Opsi A: Android Emulator (Built-in)

```bash
# Di terminal, list emulator yang tersedia
flutter emulators

# Jalankan emulator tertentu
flutter emulators --launch Pixel_7_API_34
```

Atau lewat Android Studio: **AVD Manager → Create Virtual Device → pilih device → pilih system image → Finish → Play**.

### Opsi B: Chrome (untuk Web)

Paling simpel: `flutter run` otomatis detect Chrome dan jalankan versi web.

### Opsi C: Real Device (Android)

1. Aktifkan **Developer Options** di HP (Settings → About Phone → tap "Build Number" 7x)
2. Aktifkan **USB Debugging**
3. Colok HP ke laptop pakai kabel USB
4. Accept "Allow USB debugging" dialog di HP
5. Verify: `flutter devices`

---

## Langkah 5: Buat Project Pertama & Jalankan!

```bash
# Buat project baru
flutter create nama_aplikasi_pertamamu

# Masuk ke folder project
cd nama_aplikasi_pertamamu

# Jalankan!
flutter run
```

Kalau berhasil, kamu akan lihat aplikasi **counter default Flutter** — tombol `+` di kanan bawah, tiap tap angka bertambah.

### Struktur file yang terbentuk:

```bash
nama_aplikasi_pertamamu/
├── lib/
│   └── main.dart          ← Tempat kamu ngoding!
├── android/               ← Native Android project
├── ios/                   ← Native iOS project
├── web/                   ← Web config
├── test/                  ← Unit test
├── pubspec.yaml           ← Dependencies & asset config
└── README.md
```

Coba buka `lib/main.dart` — isinya kode Dart yang menghasilkan tampilan aplikasi counter tadi. Kita akan bahas struktur project lebih detail di artikel berikutnya.

---

## Tips & Troubleshooting

### Error: `cmdline-tools component is missing`
Jalanin: **Android Studio → SDK Manager → SDK Tools → centang "Android SDK Command-line Tools" → Apply**

### Error: `Unable to locate Android SDK`
Pastikan `ANDROID_HOME` environment variable ter-set ke lokasi SDK (biasanya `C:\Users\NAMA\AppData\Local\Android\Sdk` di Windows, `/Users/NAMA/Library/Android/sdk` di macOS).

### Error: Emulator gak muncul
Coba restart ADB: `adb kill-server && adb start-server`

### Flutter gak kedetect di terminal (Windows)
Tutup dan buka ulang Command Prompt setelah set PATH. Atau restart PC.

---

## Latihan Hari Ini

1. **Install Flutter** sampai `flutter doctor` minimal 4 centang hijau (Flutter, Chrome, VS Code/Android Studio, Connected device)
2. **Buat project baru**, ganti teks "You have pushed the button..." di `lib/main.dart` jadi kalimat versimu sendiri, lalu jalankan
3. **Screenshot** hasil `flutter doctor` dan aplikasi pertamamu — share ke sosial media, tag @ahsai001!

---

## Apa Selanjutnya?

Besok kita akan bahas **Struktur Project Flutter & File Penting** — kamu akan paham fungsi setiap folder dan file yang digenerate oleh `flutter create`, termasuk `pubspec.yaml`, `lib/`, dan cara mengorganisir kode dengan benar.

> 🚀 Install Flutter sekarang juga! 15 menit aja udah bisa lihat aplikasi jalan. Share pengalaman pertamamu — tag @ahsai001!

---

*Seri "Flutter dari Nol sampai Expert" — artikel ke-2*
