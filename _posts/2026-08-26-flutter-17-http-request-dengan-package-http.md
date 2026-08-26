---
layout: post
title: "Flutter #17: HTTP Request dengan Package `http`, Ambil & Kirim Data dari Server"
date: 2026-08-26 07:00:00 +0700
tags: [flutter, dart, http, api, rest, json, network, tutorial, indonesia, pemrograman]
---

Sampai di sini, app Flutter kita masih "tersendiri" — semua data hardcode di dalam kode. Di **Flutter #16** kita kelola state pakai Provider, tapi datanya masih dummy. Padai real-world, data itu datang dari **server**: daftar produk dari API, profil user dari backend, pesanan dari database. Untuk ngambil dan ngirim data itu, kita butuh yang namanya **HTTP request**.

Hari ini kita bakal pakai package **`http`** — package resmi dari tim Flutter untuk melakukan request ke REST API. Kita bakal belajar: GET data, POST data, handle error, dan tampilkan loading state yang user-friendly. Let's go!

---

## 1. Kenapa Package `http`?

Dart punya beberapa opsi buat HTTP request:

| Package | Kelebihan |
|---|---|
| `http` | Ringan, resmi, simpel untuk kebanyakan kasus |
| `dio` | Interceptors, timeout, file download/upload lebih mudah |
| `dio` + `retrofit` | Code generation, strongly typed API |

Untuk belajar fundamental, **`http`** adalah pilihan terbaik. Kode lebih eksplisit — kamu ngerti apa yang terjadi di balik layar. Kalau nanti butuh fitur lanjutan (interceptors, cancel request), barulah migrasi ke `dio`.

---

## 2. Install Package `http`

Tambahkan dependency di `pubspec.yaml`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  http: ^1.2.0
```

Lalu jalankan:

```bash
flutter pub get
```

Done. Package siap dipakai.

---

## 3. GET Request — Ambil Data dari API

Kita bakal pakai [JSONPlaceholder](https://jsonplaceholder.typicode.com/) — API dummy gratis yang cocok buat latihan. Endpoint pertama: ambil daftar posts.

Buat file `services/api_service.dart`:

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

class ApiService {
  static const String baseUrl = 'https://jsonplaceholder.typicode.com';

  /// Ambil semua posts
  static Future<List<dynamic>> getPosts() async {
    final response = await http.get(Uri.parse('$baseUrl/posts'));

    if (response.statusCode == 200) {
      return jsonDecode(response.body) as List<dynamic>;
    } else {
      throw Exception('Gagal load posts: ${response.statusCode}');
    }
  }
}
```

Penjelasan alurnya:

1. **`http.get()`** mengirim HTTP GET ke URL yang diberikan. Hasilnya `Future<http.Response>`.
2. **`response.statusCode`** — 200 berarti sukses. Kode lain (404, 500) berarti ada masalah.
3. **`jsonDecode()`** — mengubah string JSON dari body response jadi `List` atau `Map` Dart.

Coba panggil dari widget:

```dart
import 'package:flutter/material.dart';
import 'services/api_service.dart';

class PostListPage extends StatefulWidget {
  const PostListPage({super.key});

  @override
  State<PostListPage> createState() => _PostListPageState();
}

class _PostListPageState extends State<PostListPage> {
  List<dynamic> _posts = [];
  bool _isLoading = true;
  String? _error;

  @override
  void initState() {
    super.initState();
    _loadPosts();
  }

  Future<void> _loadPosts() async {
    try {
      final posts = await ApiService.getPosts();
      setState(() {
        _posts = posts;
        _isLoading = false;
      });
    } catch (e) {
      setState(() {
        _error = e.toString();
        _isLoading = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    if (_isLoading) {
      return const Scaffold(body: Center(child: CircularProgressIndicator()));
    }
    if (_error != null) {
      return Scaffold(
        body: Center(child: Text('Error: $_error')),
      );
    }
    return Scaffold(
      appBar: AppBar(title: const Text('Posts')),
      body: ListView.builder(
        itemCount: _posts.length,
        itemBuilder: (context, index) {
          final post = _posts[index];
          return ListTile(
            title: Text(post['title']),
            subtitle: Text(
              post['body'].toString().substring(0, 60) + '...',
            ),
          );
        },
      ),
    );
  }
}
```

Jalankan app — kamu bakal lihat daftar 100 post dari JSONPlaceholder. 🎉

---

## 4. POST Request — Kirim Data ke Server

GET buat ngambil, POST buat ngirim. Contoh: bikin post baru.

Tambahkan method di `api_service.dart`:

```dart
/// Buat post baru
static Future<Map<String, dynamic>> createPost({
  required String title,
  required String body,
}) async {
  final response = await http.post(
    Uri.parse('$baseUrl/posts'),
    headers: {'Content-Type': 'application/json'},
    body: jsonEncode({
      'title': title,
      'body': body,
      'userId': 1,
    }),
  );

  if (response.statusCode == 201) {
    return jsonDecode(response.body) as Map<String, dynamic>;
  } else {
    throw Exception('Gagal buat post: ${response.statusCode}');
  }
}
```

Perhatikan beberapa hal penting:

- **`headers`** — set `Content-Type: application/json` biar server tau kita kirim JSON.
- **`jsonEncode()`** — mengubah `Map` Dart jadi string JSON.
- **Status 201** — server REST yang benar balikin **201 Created** buat POST sukses (bukan 200).

Pakai di UI dengan form sederhana:

```dart
Future<void> _submitPost() async {
  try {
    final result = await ApiService.createPost(
      title: _titleController.text,
      body: _bodyController.text,
    );
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text('Post berhasil dibuat! ID: ${result['id']}')),
    );
    _titleController.clear();
    _bodyController.clear();
  } catch (e) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text('Error: $e')),
    );
  }
}
```

---

## 5. Handle Error dengan Baik

Kode di atas handle error-nya sederhana. Tapi di production, kamu perlu handle lebih detail:

```dart
import 'package:http/http.dart' as http;

class ApiException implements Exception {
  final int statusCode;
  final String message;
  ApiException(this.statusCode, this.message);

  @override
  String toString() => 'ApiException($statusCode): $message';
}

Future<List<dynamic>> getPostsSafe() async {
  try {
    final response = await http
        .get(Uri.parse('https://jsonplaceholder.typicode.com/posts'))
        .timeout(const Duration(seconds: 10));

    if (response.statusCode == 200) {
      return jsonDecode(response.body) as List<dynamic>;
    }
    throw ApiException(
      response.statusCode,
      _parseErrorMessage(response.body),
    );
  } on http.ClientException {
    throw ApiException(0, 'Tidak ada koneksi internet');
  } on TimeoutException {
    throw ApiException(0, 'Request timeout — server lambat');
  }
}

String _parseErrorMessage(String body) {
  try {
    final json = jsonDecode(body) as Map<String, dynamic>;
    return json['message'] ?? 'Unknown error';
  } catch (_) {
    return 'Response tidak valid';
  }
}
```

Yang diperhatikan di sini:

- **`.timeout()`** — batasi waktu request, jangan sampai user nunggu selamanya.
- **`ClientException`** — terjadi kalau nggak ada koneksi internet.
- **Custom `ApiException`** — bikin error handling lebih rapi, bisa di-catch di UI.

---

## 6. Integrasi dengan Provider

Kalau kamu ikut series Flutter #15 dan #16, kita udah kenal Provider. Sekarang kita gabungkan HTTP + Provider buat bikin app yang beneran fungsional.

Buat `post_provider.dart`:

```dart
import 'package:flutter/foundation.dart';
import '../services/api_service.dart';

class PostProvider extends ChangeNotifier {
  List<dynamic> _posts = [];
  bool _isLoading = false;
  String? _error;

  List<dynamic> get posts => _posts;
  bool get isLoading => _isLoading;
  String? get error => _error;

  Future<void> fetchPosts() async {
    _isLoading = true;
    _error = null;
    notifyListeners();

    try {
      _posts = await ApiService.getPosts();
    } catch (e) {
      _error = e.toString();
    } finally {
      _isLoading = false;
      notifyListeners();
    }
  }

  Future<void> addPost(String title, String body) async {
    try {
      final newPost = await ApiService.createPost(title: title, body: body);
      _posts.insert(0, newPost); // tambah di atas
      notifyListeners();
    } catch (e) {
      _error = e.toString();
      notifyListeners();
    }
  }
}
```

Daftarkan di `main.dart`:

```dart
import 'package:provider/provider.dart';

void main() => runApp(
  ChangeNotifierProvider(
    create: (_) => PostProvider()..fetchPosts(),
    child: const MyApp(),
  ),
);
```

Di UI, tinggal pakai `context.watch<PostProvider>()` — loading, error, dan data semua termanage rapi tanpa `setState` di dalam widget.

---

## Ringkasan

- **`http.get()`** — ambil data dari API, response `statusCode == 200` = sukses.
- **`http.post()`** — kirim data, set `Content-Type` + `jsonEncode()` body, response `201` = sukses.
- **`jsonDecode` / `jsonEncode`** — konversi antara JSON string ↔ Dart object.
- **Timeout & error handling** — jangan pernah anggap request selalu sukses.
- **Gabung dengan Provider** — pisahkan logic API dari UI biar kode bersih dan testable.

HTTP request adalah fondasi dari setiap app yang connect ke internet. Tanpa ini, app cuma bisa tampilin data statis. Dengan HTTP, world is your API! 🌍

**Coba sendiri!** Bikin app sederhana yang nampilin data dari [JSONPlaceholder](https://jsonplaceholder.typicode.com/) — coba endpoint `/users`, `/comments`, atau `/todos`. Share hasilnya ke sosial media dan tag @ahsai001! 🚀
