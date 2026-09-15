---
layout: post
title: "Flutter #38: Firebase Setup & Firestore CRUD"
date: 2026-09-16 07:00:00 +0700
tags: [flutter, dart, firebase, firestore, crud, tutorial]
---

# Flutter #38: Firebase Setup & Firestore CRUD

Halo Flutter Developer! 🚀

Setelah belajar testing di artikel sebelumnya, sekarang waktunya masuk ke dunia **backend-as-a-service** yang paling populer di Flutter: **Firebase**. Hari ini kita akan setup Firebase dari nol dan bikin aplikasi lengkap CRUD (Create, Read, Update, Delete) pakai **Cloud Firestore**.

Kenapa Firebase? Karena Firebase itu kayak toolkit super buat mobile developer — authentication, database, storage, push notification, semuanya sudah siap pakai. Nggak perlu bangun server sendiri, nggak perlu config database manual.

## Apa itu Firebase & Firestore?

**Firebase** adalah platform dari Google yang menyediakan backend lengkap untuk aplikasi mobile dan web. Kalau kamu pakai Firebase, kamu nggak perlu khawatir soal server, database, atau authentication — semua sudah managed oleh Google.

**Cloud Firestore** adalah NoSQL database dari Firebase yang menyimpan data dalam bentuk **collections** dan **documents**. Mirip kayak JSON objects yang nested.

Struktur data Firestore:

```
collection "users"
├── document "user_001"
│   ├── name: "Ahmad"
│   ├── email: "ahmad@email.com"
│   └── age: 25
├── document "user_002"
│   ├── name: "Budi"
│   ├── email: "budi@email.com"
│   └── age: 30
```

Kalau dikonversi ke Dart, satu document itu mirip `Map<String, dynamic>`.

## Step 1: Setup Firebase di Project Flutter

### Buat Project Firebase

1. Buka [Firebase Console](https://console.firebase.google.com)
2. Klik **"Add project"**
3. Masukkan nama project (misal: `flutter-crud-app`)
4. Ikuti step sampai selesai

### Install Firebase CLI

```bash
# Install Firebase CLI globally
npm install -g firebase-tools

# Login ke akun Google kamu
firebase login

# Cek apakah sudah terinstall
firebase --version
```

### Setup FlutterFire CLI (Recommended)

Cara paling gampang setup Firebase di Flutter sekarang pakai **FlutterFire CLI**:

```bash
# Install FlutterFire CLI
dart pub global activate flutterfire_cli

# Jalankan di root project Flutter kamu
flutterfire configure
```

FlutterFire CLI akan otomatis:
- Detect Firebase projects yang sudah kamu buat
- Generate file `firebase_options.dart`
- Update `android/app/build.gradle` dan iOS config

## Step 2: Install Dependencies

Tambahkan package Firebase ke `pubspec.yaml`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^3.6.0
  cloud_firestore: ^5.4.0
```

Lalu jalankan:

```bash
flutter pub get
```

## Step 3: Inisialisasi Firebase di main.dart

```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'firebase_options.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Firestore CRUD',
      theme: ThemeData(
        colorSchemeSeed: Colors.blue,
        useMaterial3: true,
      ),
      home: const UserListPage(),
    );
  }
}
```

> **Catatan Penting:** `WidgetsFlutterBinding.ensureInitialized()` wajib dipanggil sebelum `Firebase.initializeApp()`. Kalau lupa, app akan crash.

## Step 4: Model Class untuk Data

Buat file `models/user_model.dart`:

```dart
import 'package:cloud_firestore/cloud_firestore.dart';

class UserModel {
  final String id;
  final String name;
  final String email;
  final int age;

  UserModel({
    required this.id,
    required this.name,
    required this.email,
    required this.age,
  });

  // Convert dari Firestore document ke object Dart
  factory UserModel.fromFirestore(DocumentSnapshot doc) {
    final data = doc.data() as Map<String, dynamic>;
    return UserModel(
      id: doc.id,
      name: data['name'] ?? '',
      email: data['email'] ?? '',
      age: data['age'] ?? 0,
    );
  }

  // Convert dari object Dart ke Map untuk disimpan ke Firestore
  Map<String, dynamic> toMap() {
    return {
      'name': name,
      'email': email,
      'age': age,
    };
  }
}
```

Pattern ini penting banget — `fromFirestore` untuk baca data, `toMap()` untuk tulis data. Dengan model class, kode jadi lebih rapi dan type-safe.

## Step 5: Implementasi CRUD

### CREATE — Tambah Data Baru

```dart
Future<void> addUser(String name, String email, int age) async {
  await FirebaseFirestore.instance.collection('users').add({
    'name': name,
    'email': email,
    'age': age,
  });
}
```

Firestore otomatis generate document ID kalau pakai `add()`. Kalau mau custom ID, pakai `doc('custom_id').set(...)`.

### READ — Baca Data

```dart
// Baca semua data (realtime)
Stream<List<UserModel>> getUsers() {
  return FirebaseFirestore.instance
      .collection('users')
      .snapshots()
      .map((snapshot) => snapshot.docs
          .map((doc) => UserModel.fromFirestore(doc))
          .toList());
}
```

Pakai `snapshots()` buat real-time updates — kalau ada data baru atau berubah, stream otomatis emit data terbaru. Nggak perlu refresh manual!

### UPDATE — Edit Data

```dart
Future<void> updateUser(String docId, Map<String, dynamic> data) async {
  await FirebaseFirestore.instance
      .collection('users')
      .doc(docId)
      .update(data);
}
```

### DELETE — Hapus Data

```dart
Future<void> deleteUser(String docId) async {
  await FirebaseFirestore.instance
      .collection('users')
      .doc(docId)
      .delete();
}
```

## Step 6: Halaman Lengkap dengan UI

```dart
import 'package:flutter/material.dart';
import 'package:cloud_firestore/cloud_firestore.dart';

class UserListPage extends StatelessWidget {
  const UserListPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Firestore CRUD')),
      body: StreamBuilder<QuerySnapshot>(
        stream: FirebaseFirestore.instance
            .collection('users')
            .orderBy('name')
            .snapshots(),
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(child: CircularProgressIndicator());
          }

          if (!snapshot.hasData || snapshot.data!.docs.isEmpty) {
            return const Center(
              child: Text('Belum ada data. Tambah yuk!'),
            );
          }

          final docs = snapshot.data!.docs;

          return ListView.builder(
            itemCount: docs.length,
            itemBuilder: (context, index) {
              final data = docs[index].data() as Map<String, dynamic>;
              final docId = docs[index].id;

              return ListTile(
                title: Text(data['name'] ?? ''),
                subtitle: Text('${data['email']} • Umur: ${data['age']}'),
                trailing: Row(
                  mainAxisSize: MainAxisSize.min,
                  children: [
                    IconButton(
                      icon: const Icon(Icons.edit),
                      onPressed: () {
                        // TODO: Navigasi ke halaman edit
                      },
                    ),
                    IconButton(
                      icon: const Icon(Icons.delete, color: Colors.red),
                      onPressed: () async {
                        await FirebaseFirestore.instance
                            .collection('users')
                            .doc(docId)
                            .delete();
                      },
                    ),
                  ],
                ),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          // TODO: Navigasi ke halaman tambah data
        },
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

## Firestore Security Rules

Jangan lupa set **security rules** di Firebase Console! Untuk development, bisa pakai rules sementara:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{document} {
      allow read, write: if true;
    }
  }
}
```

> ⚠️ **WARNING:** Rules `allow read, write: if true` hanya untuk testing! Sebelum production, WAJIB setup rules yang proper supaya data tidak bisa diakses sembarang orang.

## Tip Penting

1. **Selalu pakai `await`** — Operasi Firestore itu asynchronous. Kalau lupa await, operasi bisa gagal tanpa error yang jelas.

2. **Error handling wajib** — Bungkus semua operasi Firestore di `try-catch`:

```dart
try {
  await FirebaseFirestore.instance.collection('users').add(userData);
  if (context.mounted) {
    ScaffoldMessenger.of(context).showSnackBar(
      const SnackBar(content: Text('Berhasil disimpan!')),
    );
  }
} catch (e) {
  if (context.mounted) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text('Error: $e')),
    );
  }
}
```

3. **Pakai indexes** — Kalau query pakai `orderBy` atau `where` dengan beberapa field, Firestore mungkin minta kamu buat index. Link error-nya akan muncul di console — klik aja.

4. **Data modeling** — Firestore lebih mahal kalau data terlalu banyak di satu collection. Kalau data relasional, pertimbangkan untuk embed atau pakai subcollections.

## Penutup

Hari ini kita sudah belajar:

- ✅ Setup Firebase di project Flutter
- ✅ Install & inisialisasi Firebase packages
- ✅ Buat model class untuk Firestore
- ✅ Operasi CRUD lengkap (Create, Read, Update, Delete)
- ✅ Real-time updates dengan StreamBuilder
- ✅ Security rules basics

Firebase itu powerful banget, dan Firestore sebagai database utama sangat cocok untuk hampir semua jenis aplikasi. Di artikel selanjutnya kita akan bahas **Firebase Authentication** — cara bikin login & register user!

**Coba sendiri!** Bikin aplikasi CRUD sederhana pakai Firestore. Share ke sosial media dan tag **@ahsai001** 🔥

---

*Artikel #38 dari 90 Hari Flutter — Fase 3: Advanced*
