---
layout: post
title: "Flutter #20: FutureBuilder & AsyncSnapshot — Tampilkan Data Async tanpa Kompleksitas"
date: 2026-08-29 07:00:00 +0700
tags: [flutter, dart, futurebuilder, asyncsnapshot, async, tutorial, indonesia, pemrograman]
---

Di **Flutter #19** kita berhasil fetch data REST API dan tampilkan dalam `ListView`. Tapi kalau kamu perhatikan, kita masih campur-campur — ada `setState()` di `initState()`, ada loading manual, ada error handling di `try-catch` yang pisah dari widget tree.

Hari ini kita akan **deep-dive** ke `FutureBuilder` dan `AsyncSnapshot` — dua fitur Flutter yang dirancang khusus untuk menangani data asynchronous secara *declarative*. Artinya, cukup deskripsikan "kalau loading tampilin ini, kalau error tampilin ini" — Flutter yang urus transisinya.

---

## 1. Kenapa FutureBuilder?

Bayangkan kamu punya data yang belum siap (fetch dari API, baca file, query database). Di **procedural style** (yang kita pakai di #19), kamu harus:

1. Bikin variable `List<Post> posts = []`
2. Set `isLoading = true` di `initState()`
3. Fetch data
4. `setState()` untuk update `isLoading` dan `posts`
5. Di `build()`, cek `isLoading` → tampilin spinner atau list

**Kode jadinya banyak dan repetitive.** Beda di setiap screen.

**FutureBuilder** menyederhanakan itu jadi satu widget yang "subscribe" ke `Future` dan otomatis rebuild tiap state-nya berubah:

```dart
FutureBuilder<List<Post>>(
  future: fetchPosts(),          // Future yang mau di-watch
  builder: (context, snapshot) { // snapshot = AsyncSnapshot
    if (snapshot.connectionState == ConnectionState.waiting) {
      return CircularProgressIndicator();
    }
    if (snapshot.hasError) {
      return Text('Error: ${snapshot.error}');
    }
    // Data ready!
    return ListView(posts: snapshot.data!);
  },
)
```

Tanpa `setState()`, tanpa `isLoading` manual. Clean.

---

## 2. AsyncSnapshot: Apa Isinya?

`AsyncSnapshot<T>` adalah object yang Flutter kasih ke `builder` callback. Isinya semua info tentang status `Future`:

| Property | Tipe | Penjelasan |
|---|---|---|
| `connectionState` | `ConnectionState` | `none`, `waiting`, `active`, `done` |
| `hasData` | `bool` | `true` kalau data udah ada |
| `hasError` | `bool` | `true` kalau Future throw error |
| `data` | `T?` | Data dari Future (null kalau belum ada) |
| `error` | `Object?` | Error dari Future (null kalau nggak error) |

**ConnectionState** adalah yang paling penting:

- **`none`** — Future belum mulai (jarang muncul di production, tapi bisa terjadi kalau `future` null).
- **`waiting`** — Future udah dipanggil tapi belum selesai → **inilah momen loading**.
- **`active`** — Future udah mulai dan masih aktif. Untuk `Future` biasa, ini sama dengan `waiting`. Untuk `Stream`, ini berarti stream masih emit data.
- **`done`** — Future selesai (sukses ATAU error).

> **Tip:** Untuk `Future` biasa, kamu hanya perlu cek `waiting` dan `done`. Cek `hasData` / `hasError` di state `done` untuk bedakan sukses vs error.

---

## 3. Implementasi: FutureBuilder Versi Clean

Kita refactor app dari #19 jadi pakai `FutureBuilder`:

```dart
// screens/post_list_screen.dart
import 'package:flutter/material.dart';
import '../models/post.dart';
import '../services/api_service.dart';
import 'post_detail_screen.dart';

class PostListScreen extends StatelessWidget {
  const PostListScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Posts')),
      body: FutureBuilder<List<Post>>(
        future: ApiService.fetchPosts(),
        builder: (context, AsyncSnapshot<List<Post>> snapshot) {
          // State 1: Loading
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  CircularProgressIndicator(),
                  SizedBox(height: 16),
                  Text('Memuat data...'),
                ],
              ),
            );
          }

          // State 2: Error
          if (snapshot.hasError) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  const Icon(Icons.error_outline, size: 48, color: Colors.red),
                  const SizedBox(height: 16),
                  Text('Gagal memuat data: ${snapshot.error}'),
                  const SizedBox(height: 16),
                  ElevatedButton.icon(
                    onPressed: () {
                      // Trigger rebuild dengan setState di parent,
                      // atau pakai key untuk force rebuild (lihat §5)
                    },
                    icon: const Icon(Icons.refresh),
                    label: const Text('Coba Lagi'),
                  ),
                ],
              ),
            );
          }

          // State 3: Data ready
          final posts = snapshot.data!;
          return RefreshIndicator(
            onRefresh: () async { /* refresh logic */ },
            child: ListView.builder(
              itemCount: posts.length,
              itemBuilder: (context, index) {
                final post = posts[index];
                return Card(
                  margin: const EdgeInsets.symmetric(
                    horizontal: 16, vertical: 8,
                  ),
                  child: ListTile(
                    title: Text(
                      post.title,
                      style: const TextStyle(fontWeight: FontWeight.bold),
                    ),
                    subtitle: Text(
                      post.body,
                      maxLines: 2,
                      overflow: TextOverflow.ellipsis,
                    ),
                    trailing: const Icon(Icons.chevron_right),
                    onTap: () {
                      Navigator.push(
                        context,
                        MaterialPageRoute(
                          builder: (_) => PostDetailScreen(post: post),
                        ),
                      );
                    },
                  ),
                );
              },
            ),
          );
        },
      ),
    );
  }
}
```

Perhatikan: **seluruh logic UI ada di dalam `builder`**. Kita nggak butuh variabel `isLoading`, `errorMessage`, atau `List<Post> posts` di state. Semua dihitung langsung dari `snapshot`.

---

## 4. InitialData: Mencegah "Flash" Loading

Masalah umum: FutureBuilder sebentar tampilin spinner sebelum fetch mulai (beberapa milidetik). Kalau kamu sudah punya data awal (misalnya dari cache), pakai `initialData`:

```dart
FutureBuilder<List<Post>>(
  future: ApiService.fetchPosts(),
  initialData: [],  // ← data default sebelum Future selesai
  builder: (context, snapshot) {
    final posts = snapshot.data ?? [];
    if (posts.isEmpty && snapshot.connectionState == ConnectionState.waiting) {
      return const CircularProgressIndicator();
    }
    return ListView.builder(
      itemCount: posts.length,
      itemBuilder: (context, index) => ListTile(
        title: Text(posts[index].title),
      ),
    );
  },
)
```

`initialData` bikin `snapshot.data` langsung punya nilai dari awal — nggak perlu `snapshot.data ?? []` terus.

---

## 5. Trik: Force Rebuild FutureBuilder

**Masalah klasik:** FutureBuilder itu **stateless** — kalau widget parent rebuild, FutureBuilder buat `Future` baru, tapi builder callback **hanya dipanggil saat connectionState berubah**. Artinya kalau Future yang sama dipanggil ulang, kamu mungkin nggak lihat loading state.

**Solusi: pakai `Key` unik tiap refresh.**

```dart
// Di StatefulWidget parent
class _PostListScreenState extends State<PostListScreen> {
  int _refreshKey = 0;

  void _refresh() {
    setState(() {
      _refreshKey++; // Ganti key → FutureBuilder rebuild total
    });
  }

  @override
  Widget build(BuildContext context) {
    return FutureBuilder<List<Post>>(
      key: ValueKey(_refreshKey), // ← ini kuncinya
      future: ApiService.fetchPosts(),
      builder: (context, snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) {
          return const Center(child: CircularProgressIndicator());
        }
        if (snapshot.hasError) {
          return Center(
            child: ElevatedButton(
              onPressed: _refresh,
              child: const Text('Retry'),
            ),
          );
        }
        final posts = snapshot.data!;
        return RefreshIndicator(
          onRefresh: () async => _refresh(),
          child: ListView.builder(
            itemCount: posts.length,
            itemBuilder: (context, i) => ListTile(
              title: Text(posts[i].title),
            ),
          ),
        );
      },
    );
  }
}
```

Setiap kali `_refresh()` dipanggil, `_refreshKey` berubah → Flutter bikin FutureBuilder baru → Future baru → builder dipanggil ulang dari `ConnectionState.waiting`. **Clean dan predictable.**

---

## 6. Nested FutureBuilder: Fetch Data Dependent

Butuh fetch data bergantung? Misalnya: fetch user → fetch posts by user. Pakai nested `FutureBuilder`:

```dart
FutureBuilder<User>(
  future: ApiService.fetchUser(userId),
  builder: (context, userSnapshot) {
    if (!userSnapshot.hasData) return const CircularProgressIndicator();

    final user = userSnapshot.data!;
    return FutureBuilder<List<Post>>(
      future: ApiService.fetchPostsByUser(user.id),
      builder: (context, postSnapshot) {
        if (postSnapshot.connectionState == ConnectionState.waiting) {
          return const CircularProgressIndicator();
        }
        if (postSnapshot.hasError) return Text('Error: ${postSnapshot.error}');

        return Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text('Posts by ${user.name}',
              style: Theme.of(context).textTheme.headlineSmall),
            Expanded(
              child: ListView.builder(
                itemCount: postSnapshot.data!.length,
                itemBuilder: (context, i) => ListTile(
                  title: Text(postSnapshot.data![i].title),
                ),
              ),
            ),
          ],
        );
      },
    );
  },
)
```

Cara lain yang lebih clean: **gabungkan dua Future jadi satu** di service layer:

```dart
// Di service — single Future, single FutureBuilder
Future<Map<String, dynamic>> fetchUserPosts(int userId) async {
  final user = await fetchUser(userId);
  final posts = await fetchPostsByUser(user.id);
  return {'user': user, 'posts': posts};
}
```

Solusi kedua lebih disarankan — satu FutureBuilder, lebih gampang di-handle.

---

## 7. Common Pitfalls

### ❌ Future di `build()` → infinite loop!

```dart
// JANGAN LAKUKAN INI
@override
Widget build(BuildContext context) {
  return FutureBuilder(
    future: fetchSomething(), // ← Future baru tiap rebuild!
    builder: ...
  );
}
```

Kalau `future` dibuat inline di `build()`, Flutter bikin Future baru setiap kali rebuild. `setState()` → rebuild → Future baru → builder dipanggil → `setState()` lagi → **infinite loop**.

**Fix:** Simpan Future di `initState()` atau di variabel state.

### ❌ Lupa cek `connectionState` sebelum akses `data`

```dart
// JANGAN
final posts = snapshot.data!; // snapshot.data bisa null!

// YA
if (snapshot.connectionState == ConnectionState.done) {
  if (snapshot.hasData) {
    final posts = snapshot.data!;
    // tampilin data
  }
}
```

### ❌ FutureBuilder di dalam `ListView.builder`

`ListView.builder` bisa dispose item yang off-screen. Kalau FutureBuilder ada di dalam item, Future-nya bisa hilang dan di-restart saat user scroll balik. **Pastikan FutureBuilder ada di level widget yang tetap**, bukan di dalam list item.

---

## 8. Checklist

1. **Simpan Future di `initState()`**, bukan di `build()`.
2. **Selalu cek `connectionState`** — `waiting` → spinner, `done` + `hasError` → error UI, `done` + `hasData` → tampilin data.
3. **Pakai `initialData`** kalau punya data default untuk mencegah flash loading.
4. **Pakai `Key`** untuk force rebuild FutureBuilder saat user request refresh.
5. **Gabungkan dependent Futures** jadi satu di service layer — lebih clean dari nested FutureBuilder.
6. **Jangan taruh FutureBuilder di dalam ListView item** — bisa ter-dispose dan restart.
7. **Selalu handle error** — `snapshot.hasError` harus dicek, jangan asumsi Future selalu sukses.

---

## Ringkasan

- **`FutureBuilder<T>`** — widget declarative untuk handle `Future<T>`. tinggal deskripsikan UI untuk setiap state, Flutter yang urus rebuild-nya.
- **`AsyncSnapshot<T>`** — object berisi status Future: `connectionState`, `data`, `error`, `hasData`, `hasError`.
- **Tiga state utama**: `waiting` (loading), `done` + `hasError` (error), `done` + `hasData` (data siap).
- **`Key` pattern** untuk force rebuild saat refresh.
- **Simpan Future di `initState()`** — kesalahan paling umum di FutureBuilder.

Sekarang kamu bisa handle data async di Flutter dengan cara yang clean dan declarative! 🎯

**Coba sendiri!** Bikin app yang fetch data dari API, tampilkan di FutureBuilder dengan tiga state (loading, error, data). Tambahkan tombol retry saat error. Share hasilnya ke sosial media dan tag @ahsai001! 🚀
