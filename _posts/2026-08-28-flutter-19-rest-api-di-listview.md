---
layout: post
title: "Flutter #19: REST API di ListView — Tampilkan Data API dalam Daftar yang Rapi"
date: 2026-08-28 07:00:00 +0700
tags: [flutter, dart, rest-api, listview, jsonplaceholder, tutorial, indonesia, pemrograman]
---

Di **Flutter #17** kita ambil data dari API pakai `http`, dan di **#18** kita bungkus data JSON ke dalam Model Class yang rapi. Tapi semua data masih kita `print()` ke console — nggak ada yang muncul di layar HP.

Hari ini kita akan **gabungkan semuanya**: fetch data REST API → parse ke Model Class → tampilkan dalam `ListView` yang bisa di-scroll. Ini adalah pola paling umum di hampir semua app production: news feed, product list, chat messages, dll.

---

## 1. Goal Hari Ini

Kita akan bikin app yang:
1. Fetch 20 post dari JSONPlaceholder API saat pertama dibuka
2. Tampilkan dalam `ListView` card-card yang rapi
3. Tap card → detail post
4. Handle loading state & error state

**API yang dipakai:** `https://jsonplaceholder.typicode.com/posts` (gratis, butuh setup, cocok untuk belajar).

---

## 2. Persiapan: Model & Service

Kalau kamu masih punya code dari Flutter #18, tinggal pakai. Kalau belum, ini ringkasan model dan service-nya:

```dart
// models/post.dart
class Post {
  final int id;
  final int userId;
  final String title;
  final String body;

  const Post({
    required this.id,
    required this.userId,
    required this.title,
    required this.body,
  });

  factory Post.fromJson(Map<String, dynamic> json) {
    return Post(
      id: json['id'] as int,
      userId: json['userId'] as int,
      title: json['title'] as String,
      body: json['body'] as String,
    );
  }
}
```

```dart
// services/api_service.dart
import 'dart:convert';
import 'package:http/http.dart' as http;
import '../models/post.dart';

class ApiService {
  static const String baseUrl = 'https://jsonplaceholder.typicode.com';

  static Future<List<Post>> getPosts() async {
    final response = await http.get(Uri.parse('$baseUrl/posts'));
    if (response.statusCode == 200) {
      final List<dynamic> jsonList = jsonDecode(response.body);
      return jsonList
          .map((json) => Post.fromJson(json as Map<String, dynamic>))
          .toList();
    } else {
      throw Exception('Gagal load posts: ${response.statusCode}');
    }
  }

  static Future<Post> getPostDetail(int id) async {
    final response = await http.get(Uri.parse('$baseUrl/posts/$id'));
    if (response.statusCode == 200) {
      return Post.fromJson(jsonDecode(response.body) as Map<String, dynamic>);
    } else {
      throw Exception('Gagal load post: ${response.statusCode}');
    }
  }
}
```

Perhatikan kita tambah `getPostDetail(id)` — biar bisa fetch satu post untuk halaman detail.

---

## 3. Tampilkan dalam ListView — Versi Dasar

```dart
import 'package:flutter/material.dart';
import '../models/post.dart';
import '../services/api_service.dart';

class PostListScreen extends StatefulWidget {
  const PostListScreen({super.key});

  @override
  State<PostListScreen> createState() => _PostListScreenState();
}

class _PostListScreenState extends State<PostListScreen> {
  late Future<List<Post>> _futurePosts;

  @override
  void initState() {
    super.initState();
    _futurePosts = ApiService.getPosts();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Blog Posts')),
      body: FutureBuilder<List<Post>>(
        future: _futurePosts,
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(child: CircularProgressIndicator());
          }
          if (snapshot.hasError) {
            return Center(
              child: Column(
                mainAxisSize: MainAxisSize.min,
                children: [
                  const Icon(Icons.error_outline, size: 48, color: Colors.red),
                  const SizedBox(height: 16),
                  Text('Error: ${snapshot.error}'),
                  const SizedBox(height: 12),
                  ElevatedButton(
                    onPressed: () {
                      setState(() {
                        _futurePosts = ApiService.getPosts();
                      });
                    },
                    child: const Text('Coba Lagi'),
                  ),
                ],
              ),
            );
          }

          final posts = snapshot.data!;
          return ListView.builder(
            itemCount: posts.length,
            itemBuilder: (context, index) {
              final post = posts[index];
              return ListTile(
                title: Text(post.title),
                subtitle: Text(
                  post.body,
                  maxLines: 2,
                  overflow: TextOverflow.ellipsis,
                ),
                trailing: const Icon(Icons.chevron_right),
              );
            },
          );
        },
      ),
    );
  }
}
```

Beberapa hal penting di kode ini:

- **`FutureBuilder`** — widget yang listen ke `Future` dan rebuild UI berdasarkan state-nya: `waiting`, `hasData`, atau `hasError`.
- **`ListView.builder`** — list yang efficient: widget cuma dibuat untuk item yang visible di layar. Beda dengan `ListView(children: [...])` yang render semua item sekaligus (jelek untuk list panjang).
- **`initState()`** — fetch data di sini, bukan di `build()`. Kalau fetch di `build()`, tiap rebuild akan fetch ulang terus-menerus.
- **Retry button** — kalau error, user bisa tap "Coba Lagi" tanpa restart app.

---

## 4. Versi Card — Tampilan Lebih Menarik

`ListItem` terlalu plain. Mari upgrade ke `Card` dengan layout yang lebih informatif:

```dart
ListView.builder(
  itemCount: posts.length,
  itemBuilder: (context, index) {
    final post = posts[index];
    return Card(
      margin: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
      elevation: 2,
      child: InkWell(
        onTap: () {
          Navigator.push(
            context,
            MaterialPageRoute(
              builder: (_) => PostDetailScreen(post: post),
            ),
          );
        },
        child: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              // Header: avatar + user ID
              Row(
                children: [
                  CircleAvatar(
                    child: Text('U${post.userId}'),
                  ),
                  const SizedBox(width: 12),
                  Text(
                    'User ${post.userId}',
                    style: Theme.of(context).textTheme.bodySmall,
                  ),
                ],
              ),
              const SizedBox(height: 12),
              // Title
              Text(
                post.title,
                style: Theme.of(context).textTheme.titleMedium?.copyWith(
                  fontWeight: FontWeight.bold,
                ),
              ),
              const SizedBox(height: 8),
              // Body preview
              Text(
                post.body,
                maxLines: 3,
                overflow: TextOverflow.ellipsis,
                style: Theme.of(context).textTheme.bodyMedium,
              ),
            ],
          ),
        ),
      ),
    );
  },
)
```

Sekarang tiap item tampil sebagai card dengan avatar user ID, title bold, dan body preview max 3 baris. Tap card → pindah ke detail screen.

---

## 5. Halaman Detail Post

```dart
class PostDetailScreen extends StatelessWidget {
  final Post post;
  const PostDetailScreen({super.key, required this.post});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Post #${post.id}'),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Header
            Chip(
              avatar: const Icon(Icons.person, size: 18),
              label: Text('User ${post.userId}'),
            ),
            const SizedBox(height: 12),
            // Title
            Text(
              post.title,
              style: Theme.of(context).textTheme.headlineSmall?.copyWith(
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 16),
            const Divider(),
            const SizedBox(height: 16),
            // Body
            Text(
              post.body,
              style: Theme.of(context).textTheme.bodyLarge?.copyWith(
                height: 1.6,
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

Karena body post bisa panjang, kita pakai `SingleChildScrollView` supaya bisa di-scroll. Jangan lupa `CrossAxisAlignment.start` biar text align ke kiri.

---

## 6. Pull to Refresh

User expect kalau list bisa di-refresh dengan swipe ke bawah. Tambahin `RefreshIndicator`:

```dart
RefreshIndicator(
  onRefresh: () async {
    setState(() {
      _futurePosts = ApiService.getPosts();
    });
    await _futurePosts;
  },
  child: FutureBuilder<List<Post>>(
    future: _futurePosts,
    builder: (context, snapshot) {
      // ... kode FutureBuilder seperti biasa
    },
  ),
)
```

`RefreshIndicator` wrap `FutureBuilder` kita. Saat user swipe-down, `onRefresh` dipanggil, data di-fetch ulang, dan FutureBuilder rebuild otomatis.

---

## 7. Pola Error & Loading di Production

Di dunia nyata, kamu sering butuh tiga state yang ditampilkan ke user:

```dart
enum ViewState { idle, loading, loaded, error }

// ... di dalam State class:
ViewState _viewState = ViewState.idle;
List<Post> _posts = [];
String _errorMessage = '';

Future<void> _loadPosts() async {
  setState(() {
    _viewState = ViewState.loading;
  });

  try {
    final posts = await ApiService.getPosts();
    setState(() {
      _posts = posts;
      _viewState = ViewState.loaded;
    });
  } catch (e) {
    setState(() {
      _errorMessage = e.toString();
      _viewState = ViewState.error;
    });
  }
}

// Di build():
Widget _buildBody() {
  switch (_viewState) {
    case ViewState.loading:
      return const Center(child: CircularProgressIndicator());
    case ViewState.error:
      return Center(
        child: Text('Error: $_errorMessage'),
      );
    case ViewState.loaded:
      return RefreshIndicator(
        onRefresh: _loadPosts,
        child: ListView.builder(
          itemCount: _posts.length,
          itemBuilder: (context, index) => _buildPostCard(_posts[index]),
        ),
      );
    default:
      return const SizedBox.shrink();
  }
}
```

Pola `enum ViewState` ini lebih eksplisit daripada cuma andalkan `FutureBuilder`. Di tutorial selanjutnya (Provider/BLoC), pattern ini bakal jadi fondasi state management yang lebih canggih.

---

## 8. Checklist & Best Practice

1. **Fetch di `initState()`**, bukan di `build()` — mencegah fetch berulang tiap rebuild.
2. **Pakai `ListView.builder()`** — lazy loading, cuma render item yang visible.
3. **Tangani tiga state**: loading, error, loaded — user harus selalu tau apa yang terjadi.
4. **Tombol retry** — kalau error, user bisa coba lagi tanpa restart app.
5. **Pull to refresh** — wrap list dengan `RefreshIndicator`.
6. **Data di-pass ke detail screen** — nggak perlu fetch ulang kalau data udah ada di memori.
7. **`const` constructor** — hemat rebuild, optimize performance.

---

## Ringkasan

- **`FutureBuilder`** adalah widget utama untuk menampilkan data async di Flutter.
- **`ListView.builder`** — list efficient yang cuma render item visible.
- **Tiga state** — loading (spinner), error (message + retry), loaded (list).
- **`RefreshIndicator`** — pull-to-refresh native yang user expect di mobile.
- **Model Class** dari #18 bikin akses data jadi type-safe: `post.title`, bukan `post['title']`.

Sekarang app kamu udah bisa tampilin data dari internet secara real-time! 🌐

**Coba sendiri!** Tambahkan search bar di atas list — filter post berdasarkan title pakai `TextField` + `setState`. Share hasilnya ke sosial media dan tag @ahsai001! 🚀
