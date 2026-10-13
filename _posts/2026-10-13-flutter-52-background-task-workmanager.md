---
layout: post
title: "Flutter #52: Background Task dengan Workmanager"
date: 2026-10-13 07:00:00 +0700
categories: [flutter]
tags: [flutter, dart, tutorial, indonesia]
---

## Pendahuluan
Aplikasi mobile sering membutuhkan tugas yang berjalan di latar belakang, misalnya sinkronisasi data, notifikasi terjadwal, atau pemrosesan berat. Pada Flutter, paket **workmanager** menyediakan API sederhana untuk menjadwalkan pekerjaan (background task) yang tetap berjalan bahkan ketika aplikasi ditutup.

## Instalasi
Tambahkan dependensi di `pubspec.yaml`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  workmanager: ^0.5.0
```
Jalankan `flutter pub get`.

## Registrasi Callback
Workmanager memanggil fungsi **callback** pada isolate terpisah. Fungsi ini **harus** berada di file terpisah atau di `main.dart` sebelum `runApp()`.

```dart
import 'package:workmanager/workmanager.dart';

void callbackDispatcher() {
  Workmanager().executeTask((task, inputData) async {
    switch (task) {
      case "syncTask":
        // TODO: panggil service API untuk sinkronisasi
        print('🔄 Sync task dijalankan');
        break;
      case Workmanager.iOSBackgroundTask:
        print('iOS background fetch');
        break;
    }
    return Future.value(true);
  });
}
```

## Inisialisasi di `main()`

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Workmanager().initialize(
    callbackDispatcher,
    isInDebugMode: true, // hapus di production
  );
  // Jadwalkan tugas periodik tiap 15 menit
  await Workmanager().registerPeriodicTask(
    "1",
    "syncTask",
    frequency: const Duration(minutes: 15),
  );
  runApp(const MyApp());
}
```

> **Catatan:** Android membutuhkan permission `WAKE_LOCK` dan `RECEIVE_BOOT_COMPLETED` di `AndroidManifest.xml`. Workmanager menambahkannya otomatis, cukup sinkronkan `android/app/src/main/AndroidManifest.xml`.

## Menjalankan Tugas Sekali Paksa
Untuk debugging, panggil `registerOneOffTask`:

```dart
await Workmanager().registerOneOffTask(
  "2",
  "syncTask",
  inputData: {"userId": "123"},
);
```

## Menangani Batasan Platform
- **iOS:** Hanya dapat menjalankan tugas selama 30 detik setelah dipanggil oleh sistem. Gunakan `Workmanager.iOSBackgroundTask` untuk fetch background.
- **Android:** Batas minimum interval 15 menit untuk `registerPeriodicTask`. Jika butuh lebih sering, gunakan `AlarmManager` native.

## Debugging & Log
Gunakan `print` dalam callback, lalu lihat log dengan `adb logcat -s Workmanager` (Android) atau Xcode console (iOS).

## Kesimpulan
Workmanager memberi cara cepat menambahkan background job tanpa menulis kode native. Pastikan **tidak** mengakses UI langsung di dalam callback—semua operasi harus asynchronous dan tidak bergantung pada context.

---

*CTA*: Coba sendiri! Share ke sosial media dan tag @ahsai001
