---
layout: post
title: "Flutter #53: Deep Linking"
date: 2026-10-14 07:00:00 +0700
categories: [flutter]
tags: [flutter, dart, tutorial, indonesia]
---

Deep Linking memungkinkan aplikasi Flutter Anda dibuka langsung ke halaman tertentu lewat URL, baik dari browser, email, atau aplikasi lain. Pada artikel ini kita bahas cara menyiapkan deep link di Android & iOS, menghubungkannya ke route Flutter, dan contoh implementasi sederhana.

## 1. Apa Itu Deep Linking?
Deep link adalah URL khusus (contoh: `myapp://product/42`) yang ketika diklik akan meluncurkan aplikasi Anda dan menavigasi ke layar tertentu. Ini berbeda dengan *Universal Links* (iOS) atau *App Links* (Android) yang menggunakan skema HTTP/HTTPS.

## 2. Tambahkan Skema URL ke Android & iOS
### Android
Edit `android/app/src/main/AndroidManifest.xml` dan tambahkan intent filter di dalam `<activity>` utama:

```xml
<intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="myapp" />
</intent-filter>
```

### iOS
Buka Xcode, pilih *Runner* → *Info* → *URL Types* dan tambahkan `myapp` sebagai **Identifier** dan **URL Schemes**.

## 3. Buat Route Named di Flutter
Di `main.dart` definisikan route yang akan dipanggil:

```dart
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Deep Link Demo',
      initialRoute: '/',
      onGenerateRoute: (settings) {
        if (settings.name?.startsWith('/product/') == true) {
          final id = settings.name!.split('/').last;
          return MaterialPageRoute(builder: (_) => ProductPage(id: id));
        }
        return null; // fallback to default routes
      },
      routes: {
        '/': (_) => HomePage(),
      },
    );
  }
}

class HomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: Text('Home')),
        body: Center(child: Text('Home Screen')),
      );
}

class ProductPage extends StatelessWidget {
  final String id;
  const ProductPage({required this.id});
  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: Text('Produk $id')),
        body: Center(child: Text('Detail produk dengan ID $id')),
      );
}
```
Kode di atas menangkap URL `/product/42` dan menampilkan `ProductPage` dengan ID yang diparsing.

## 4. Menangkap Deep Link di Runtime
Gunakan paket `uni_links` untuk mendengarkan link yang masuk saat aplikasi sudah berjalan.

```dart
import 'package:flutter/material.dart';
import 'package:uni_links/uni_links.dart';
import 'dart:async';

class LinkHandler extends StatefulWidget {
  @override
  _LinkHandlerState createState() => _LinkHandlerState();
}

class _LinkHandlerState extends State<LinkHandler> {
  StreamSubscription? _sub;

  @override
  void initState() {
    super.initState();
    _sub = uriLinkStream.listen((Uri? uri) {
      if (uri != null && uri.scheme == 'myapp') {
        final path = uri.path; // e.g. /product/42
        Navigator.of(context).pushNamed(path);
      }
    });
  }

  @override
  void dispose() {
    _sub?.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Container(); // invisible widget
}
```
Tambahkan `LinkHandler()` di atas widget `MaterialApp` agar listener aktif.

## 5. Uji Coba di Emulator
### Android
```bash
adb shell am start -a android.intent.action.VIEW -d "myapp://product/7"
```
### iOS (Simulator)
Buka *Safari* di simulator, ketik `myapp://product/7` lalu tekan *Go*.

Jika semua langkah di atas benar, aplikasi akan langsung membuka halaman `ProductPage` dengan ID yang diberikan.

## 6. Tips & Trik
- Gunakan `https://` + Universal Links untuk integrasi yang lebih mulus di iOS (perlu file `apple-app-site-association`).
- Selalu validasi parameter yang diterima dari URL untuk menghindari crash.
- Simpan fallback route (`/`) bila link tidak dikenali.

## 7. Ringkasan
Deep linking meningkatkan konversi dan pengalaman pengguna karena mereka dapat langsung mengakses konten spesifik. Dengan menambahkan skema URL ke native konfigurasi, mendefinisikan route di Flutter, dan mendengarkan link dengan `uni_links`, Anda sudah siap mengimplementasikan deep link.

---

Coba sendiri! Share ke sosial media dan tag @ahsai001
