---
layout: post
title: "Flutter #46: Platform Channel (Native)"
date: 2026-09-29 10:00:00 +0700
tags: [flutter, platform-channel, native, android, ios]
---

Platform Channel memungkinkan Flutter berkomunikasi dengan kode native (Kotlin/Java untuk Android, Swift/Objective‑C untuk iOS). Ini penting ketika kamu butuh API yang belum tersedia di paket pub.dev – misalnya sensor spesifik, Bluetooth low‑energy, atau integrasi SDK proprietary.

## 1️⃣ Konsep dasar

* **MethodChannel** – mengirim panggilan satu arah dan menunggu hasil.
* **EventChannel** – aliran data terus‑menerus dari native ke Dart.
* **BasicMessageChannel** – pesan dua arah bebas tipe (String, JSON, byte‑array).

Setiap channel diidentifikasi string unik, misalnya `"com.example/sensor"`. Pastikan tidak ada duplikasi antara Android & iOS.

## 2️⃣ Membuat MethodChannel di Flutter

```dart
import 'package:flutter/services.dart';

class SensorChannel {
  static const _channel = MethodChannel('com.example/sensor');

  // Meminta nilai sensor dari native
  static Future<double> getTemperature() async {
    final result = await _channel.invokeMethod('getTemperature');
    return (result as num).toDouble();
  }
}
```

> **Catatan:** `invokeMethod` mengembalikan `dynamic`. Selalu cast dengan hati‑hati, gunakan `try/catch` untuk menangkap `PlatformException`.

## 3️⃣ Implementasi Android (Kotlin)

Buka `android/app/src/main/kotlin/com/example/yourapp/MainActivity.kt` dan tambahkan:

```kotlin
import io.flutter.embedding.android.FlutterActivity
import io.flutter.embedding.engine.FlutterEngine
import io.flutter.plugin.common.MethodChannel

class MainActivity: FlutterActivity() {
    private val CHANNEL = "com.example/sensor"
    override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)
        MethodChannel(flutterEngine.dartExecutor.binaryMessenger, CHANNEL).setMethodCallHandler { call, result ->
            when (call.method) {
                "getTemperature" -> {
                    val temp = getDeviceTemperature() // implementasi kamu
                    result.success(temp)
                }
                else -> result.notImplemented()
            }
        }
    }

    private fun getDeviceTemperature(): Double {
        // Contoh stub, biasanya pakai sensor manager
        return 36.5
    }
}
```

Jika memakai Java, ubah sintaksnya namun logika tetap sama.

## 4️⃣ Implementasi iOS (Swift)

Edit `ios/Runner/AppDelegate.swift`:

```swift
import UIKit
import Flutter

@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
  private let channel = "com.example/sensor"

  override func application(_ application: UIApplication,
                            didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    let controller = window?.rootViewController as! FlutterViewController
    let methodChannel = FlutterMethodChannel(name: channel,
                                             binaryMessenger: controller.binaryMessenger)
    methodChannel.setMethodCallHandler { (call, result) in
      if call.method == "getTemperature" {
        let temp = self.getDeviceTemperature()
        result(temp)
      } else {
        result(FlutterMethodNotImplemented)
      }
    }
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }

  private func getDeviceTemperature() -> Double {
    // Stub – biasanya pakai Core Motion atau sensor lain
    return 36.5
  }
}
```

## 5️⃣ Menggunakan di UI

```dart
class TempPage extends StatefulWidget { @override _TempPageState createState()=>_TempPageState(); }
class _TempPageState extends State<TempPage> {
  double? _temp;
  @override void initState(){ super.initState(); _loadTemp(); }
  Future<void> _loadTemp() async {
    try { final t = await SensorChannel.getTemperature(); setState(()=>_temp=t); }
    on PlatformException catch(e){ debugPrint('Error: ${e.message}'); }
  }
  @override Widget build(BuildContext context) => Scaffold(
    appBar: AppBar(title: Text('Sensor Temperature')),
    body: Center(child: _temp==null ? CircularProgressIndicator() : Text('Temp: $_temp°C')),
  );
}
```

## 6️⃣ Debugging & Tips

* **Logcat / Xcode console** – gunakan `println` di native untuk melacak panggilan.
* **`flutter logs`** – menampilkan semua log, berguna saat `PlatformException` terjadi.
* **Versi Flutter** – pastikan minimal `2.5.0` karena API `configureFlutterEngine` stabil sejak versi itu.
* **Null‑Safety** – semua kode Dart harus `await` dan `?` bila hasil dapat `null`.
* **Threading** – jangan lakukan pekerjaan berat di handler; gunakan `HandlerThread` (Android) atau `DispatchQueue.global()` (iOS) lalu kirimkan kembali ke UI thread lewat `result.success`.

## 7️⃣ Kesimpulan

Platform Channel membuka pintu ke ekosistem native tanpa meninggalkan satu baris Flutter. Dengan tiga tipe channel, kamu dapat mengirim data sederhana, aliran kontinu, atau bahkan file biner. Selalu **pisahkan** logika native dalam kelas terpisah, **uji** dengan unit test pada sisi Dart, dan **debug** native secara independen.

---
Coba sendiri! Share ke sosial media dan tag @ahsai001
