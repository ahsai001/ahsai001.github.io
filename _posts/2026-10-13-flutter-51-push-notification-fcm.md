---
layout: post
title: "Flutter #51: Push Notification dengan Firebase Cloud Messaging"
date: 2026-10-13 07:00:00 +0700
categories: [flutter]
tags: [flutter, dart, tutorial, indonesia]
---

Push Notification adalah cara paling efektif untuk menjangkau pengguna meski aplikasi tidak aktif. Di Flutter, **Firebase Cloud Messaging (FCM)** menyediakan solusi lintas‑platform (Android & iOS) yang terintegrasi dengan Firebase. Artikel ini membahas persiapan proyek, konfigurasi Firebase, serta contoh kode lengkap untuk menerima notifikasi di latar belakang.

## 1. Tambahkan Dependency
Buka `pubspec.yaml` dan tambahkan paket berikut:
```yaml
dependencies:
  firebase_core: ^2.15.0
  firebase_messaging: ^14.6.5
```
Jalankan `flutter pub get`.

## 2. Konfigurasi Firebase Console
1. Buat project di https://console.firebase.google.com.
2. Tambahkan aplikasi Android (bundle ID `com.example.app`) dan iOS (bundle ID yang sama).
3. Unduh `google-services.json` (Android) dan `GoogleService‑Info.plist` (iOS).
4. Letakkan file‑file tersebut di folder masing‑masing:
   - Android: `android/app/`
   - iOS: `ios/Runner/`
5. Pada **Project Settings → Cloud Messaging**, catat **Server key** (akan dipakai untuk mengirim notifikasi dari server).

## 3. Android Manifest
Edit `android/app/src/main/AndroidManifest.xml`:
```xml
<manifest ...>
    <application ...>
        <!-- Tambahkan permission -->
        <uses-permission android:name="android.permission.INTERNET" />
        <uses-permission android:name="android.permission.WAKE_LOCK" />
        <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
        <!-- Service untuk handling pesan -->
        <service android:name="com.google.firebase.messaging.FirebaseMessagingService" android:exported="true" />
    </application>
</manifest>
```

## 4. Inisialisasi di Flutter
Buat file `lib/services/push_service.dart`:
```dart
import 'package:firebase_core/firebase_core.dart';
import 'package:firebase_messaging/firebase_messaging.dart';
import 'package:flutter_local_notifications/flutter_local_notifications.dart';

class PushService {
  final FirebaseMessaging _messaging = FirebaseMessaging.instance;
  final FlutterLocalNotificationsPlugin _local = FlutterLocalNotificationsPlugin();

  Future<void> init() async {
    await Firebase.initializeApp();
    // Request permission (iOS)
    await _messaging.requestPermission();
    // Token untuk server
    String? token = await _messaging.getToken();
    print('FCM Token: $token');
    // Foreground handler
    FirebaseMessaging.onMessage.listen(_onMessage);
    // Background handler (top‑level function)
    FirebaseMessaging.onBackgroundMessage(_firebaseMessagingBackgroundHandler);
    // Local notification init
    const AndroidInitializationSettings initSettings = AndroidInitializationSettings('@mipmap/ic_launcher');
    await _local.initialize(const InitializationSettings(android: initSettings));
  }

  void _onMessage(RemoteMessage message) async {
    RemoteNotification? notif = message.notification;
    if (notif != null) {
      await _local.show(
        notif.hashCode,
        notif.title,
        notif.body,
        NotificationDetails(
          android: AndroidNotificationDetails('fcm_channel', 'FCM Notifications', importance: Importance.max),
        ),
      );
    }
  }
}

Future<void> _firebaseMessagingBackgroundHandler(RemoteMessage message) async {
  await Firebase.initializeApp();
  // Bisa simpan data ke local storage atau tampilkan notifikasi dengan plugin lain
}
```

Panggil `PushService().init()` pada `main()` sebelum `runApp()`:
```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await PushService().init();
  runApp(MyApp());
}
```

## 5. Mengirim Notifikasi dari Server
Gunakan API HTTP FCM dengan **Server key** yang didapat di langkah 2. Contoh cURL:
```bash
curl -X POST -H "Authorization: key=YOUR_SERVER_KEY" -H "Content-Type: application/json" \
-d '{
  "to": "DEVICE_FCM_TOKEN",
  "notification": {"title": "Hai!", "body": "Ini contoh push notification"},
  "data": {"click_action": "FLUTTER_NOTIFICATION_CLICK"}
}' https://fcm.googleapis.com/fcm/send
```
Anda dapat membungkusnya dalam Cloud Function atau backend Node.js.

## 6. Penanganan Klik Notifikasi
Tambahkan listener di `main()`:
```dart
FirebaseMessaging.onMessageOpenedApp.listen((RemoteMessage message) {
  // Navigasi ke halaman tertentu berdasarkan data payload
  if (message.data['screen'] != null) {
    Navigator.pushNamed(context, message.data['screen']);
  }
});
```
Pastikan payload `data` berisi kunci `screen`.

## 7. Testing di Emulator & Real Device
- **Android**: Buka **Logcat**, perhatikan token, gunakan cURL di atas.
- **iOS**: Pastikan aplikasi memiliki *Push Notification* capability di Xcode, jalankan di perangkat fisik (simulator tidak menerima APNs).

## 8. Troubleshooting Umum
| Masalah | Penyebab | Solusi |
|---|---|---|
| Token `null` | Permission belum diberikan (iOS) | Panggil `requestPermission()` dan cek `Settings → Notifications` |
| Notifikasi tidak muncul di latar belakang | `onBackgroundMessage` tidak terdaftar di `AndroidManifest` | Tambahkan service atau gunakan `FlutterLocalNotifications` |
| APNs error 403 | Sertifikat APNs tidak cocok | Perbarui **APNs Auth Key** di Firebase console |

## 9. Kesimpulan
FCM memungkinkan **push notification** lintas platform dengan konfigurasi yang relatif sederhana. Tambahkan dependensi, set up Firebase, inisialisasi service, dan kirim pesan dari backend. Dengan contoh kode di atas, Anda sudah siap menambahkan **engagement** ke aplikasi Flutter.

---
Coba sendiri! Share ke sosial media dan tag @ahsai001
