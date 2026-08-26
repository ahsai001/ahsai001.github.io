---
layout: post
title: "Flutter #18: JSON Parsing & Model Class — dari `Map<String, dynamic>` ke Objek Dart yang Rapi"
date: 2026-08-27 07:00:00 +0700
tags: [flutter, dart, json, model, parsing, serialization, api, tutorial, indonesia, pemrograman]
---

Di **Flutter #17** kita berhasil ambil data dari API pakai package `http`. Tapi kalau kamu perhatikan, semua data kita akses pakai `dynamic` — `post['title']`, `post['body']`, dll. Nggak ada autocomplete, nggak ada type safety, dan typo kecil bisa bikin crash yang susah di-debug.

Hari ini kita bakal fix semua itu. Kita akan belajar bikin **Model Class** — kelas Dart yang merepresentasikan data JSON secara kuat tipe (*strongly typed*). Setelah ini, setiap kali kamu akses `post.title`, Dart udah tau tipe datanya: `String`. Kalau salah akses, langsung error di compile time — bukan di runtime yang baru ketahuan pas user pakai app.

---

## 1. Kenapa Butuh Model Class?

| Tanpa Model | Dengan Model |
|---|---|
| `post['title']` → `dynamic`, bisa null | `post.title` → `String`, compile-time check |
| Nggak ada autocomplete di IDE | Autocomplete lengkap: `.title`, `.body`, `.userId` |
| Typo = runtime error | Typo = compile error |
| Refactoring susah | Rename field → search & replace satu kali |

Intinya: **Model class** bikin kode lebih aman, lebih cepat ditulis, dan lebih gampang di-maintain.

---

## 2. Model Sederhana: satu Object

Kita mulai dari yang paling basic — parsing satu post dari JSONPlaceholder.

### Model Class

Buat file `models/post.dart`:

```dart
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

  /// Buat Post dari Map (JSON yang udah di-decode)
  factory Post.fromJson(Map<String, dynamic> json) {
    return Post(
      id: json['id'] as int,
      userId: json['userId'] as int,
      title: json['title'] as String,
      body: json['body'] as String,
    );
  }

  /// Ubah Post jadi Map (buat dikirim ke server)
  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'userId': userId,
      'title': title,
      'body': body,
    };
  }
}
```

Beberapa hal penting:

- **`factory Post.fromJson()`** — constructor khusus yang terima `Map<String, dynamic>` dan return `Post`. Ini konvensi yang dipakai di seluruh ekosistem Dart/Flutter.
- **`toJson()`** — kebalikannya: ubah objek jadi Map, berguna kalau mau POST data ke server.
- **`final`** — semua field immutable. Ini best practice biar data nggak berubah di tengah jalan.
- **`as int` / `as String`** — explicit type cast. Kalau JSON punya type beda (misalnya `id` string), Dart langsung kasih error.

### Cara Pakai

Sekarang update `ApiService` biar return `Post` bukan `dynamic`:

```dart
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
}
```

Perhatikan baris `jsonList.map((json) => Post.fromJson(...)).toList()`. Kita loop tiap item di list JSON, dan konversi ke `Post` object. Sekarang di UI:

```dart
final posts = await ApiService.getPosts();
// Autocomplete muncul! ✅
print(posts[0].title);  // "sunt aut facere..."
print(posts[0].id);     // 1 (int)
```

Type-safe, autocomplete-friendly, dan kalau ada field yang salah — error langsung ketahuan di IDE, bukan pas runtime.

---

## 3. Handle Nested JSON

API dunia nyata nggak selalu flat. Kadang ada nested object. Contoh: endpoint `/posts/1/comments` dari JSONPlaceholder return list comment yang punya field `post` di dalamnya.

Buat `models/comment.dart`:

```dart
class Comment {
  final int postId;
  final int id;
  final String name;
  final String email;
  final String body;

  const Comment({
    required this.postId,
    required this.id,
    required this.name,
    required this.email,
    required this.body,
  });

  factory Comment.fromJson(Map<String, dynamic> json) {
    return Comment(
      postId: json['postId'] as int,
      id: json['id'] as int,
      name: json['name'] as String,
      email: json['email'] as String,
      body: json['body'] as String,
    );
  }
}
```

Sekarang model yang lebih realistis — ada nested object. Misalnya API balikin data seperti ini:

```json
{
  "id": 1,
  "title": "Flutter Tips",
  "author": {
    "name": "Andi",
    "email": "andi@example.com"
  }
}
```

Kita perlu dua model — satu buat `Post`, satu buat `Author`:

```dart
// models/author.dart
class Author {
  final String name;
  final String email;

  const Author({required this.name, required this.email});

  factory Author.fromJson(Map<String, dynamic> json) {
    return Author(
      name: json['name'] as String,
      email: json['email'] as String,
    );
  }
}
```

```dart
// models/post_with_author.dart
import 'author.dart';

class PostWithAuthor {
  final int id;
  final String title;
  final Author author;

  const PostWithAuthor({
    required this.id,
    required this.title,
    required this.author,
  });

  factory PostWithAuthor.fromJson(Map<String, dynamic> json) {
    return PostWithAuthor(
      id: json['id'] as int,
      title: json['title'] as String,
      // Nested: parse author object dari Map
      author: Author.fromJson(json['author'] as Map<String, dynamic>),
    );
  }
}
```

Sekarang aksesnya jadi super clean:

```dart
final post = PostWithAuthor.fromJson(json);
print(post.author.name);   // "Andi"
print(post.author.email);  // "andi@example.com"
```

---

## 4. Handle Null dengan Safety

Di dunia nyata, field bisa aja null. Mungkin server nggak kirim `description`, atau field `avatarUrl` belum diisi user.

```dart
class UserProfile {
  final String name;
  final String? email;      // nullable — boleh null
  final String? avatarUrl;  // nullable — boleh null
  final int age;

  const UserProfile({
    required this.name,
    this.email,
    this.avatarUrl,
    required this.age,
  });

  factory UserProfile.fromJson(Map<String, dynamic> json) {
    return UserProfile(
      name: json['name'] as String,
      email: json['email'] as String?,       // cast ke nullable
      avatarUrl: json['avatarUrl'] as String?,
      age: json['age'] as int,
    );
  }
}
```

Di UI, handle null-nya:

```dart
final profile = UserProfile.fromJson(json);

// Opsi 1: null-aware operator
final displayEmail = profile.email ?? 'Belum diisi';

// Opsi 2: Conditional
if (profile.avatarUrl != null) {
  return Image.network(profile.avatarUrl!); // ! untuk assert non-null
}
```

Kunci di sini: **jangan pernah asumsi semua field pasti ada di JSON**. Selalu pakai nullable type (`String?`) untuk field yang mungkin nggak dikirim server.

---

## 5. JSON Serialization Manual vs Code Generation

Sampai sekarang kita nulis `fromJson` dan `toJson` manual. Untuk project kecil, ini fine. Tapi kalau model-nya 20+ field, nulis manual jadi melelahkan dan rawan typo.

### Opsi Manual (yang kita pakai sekarang)

```dart
// Cocok untuk: < 10 field, project kecil, belajar
factory Post.fromJson(Map<String, dynamic> json) => Post(
  id: json['id'] as int,
  title: json['title'] as String,
  // ... field by field
);
```

### Opsi Code Generation: `json_serializable` + `build_runner`

```yaml
# pubspec.yaml
dependencies:
  json_annotation: ^4.8.0

dev_dependencies:
  json_serializable: ^6.7.0
  build_runner: ^2.4.0
```

```dart
import 'package:json_annotation/json_annotation.dart';
part 'post.g.dart'; // file generated

@JsonSerializable()
class Post {
  final int id;
  final int userId;
  final String title;
  final String body;

  const Post({required this.id, required this.userId, required this.title, required this.body});

  factory Post.fromJson(Map<String, dynamic> json) => _$PostFromJson(json);
  Map<String, dynamic> toJson() => _$PostToJson(this);
}
```

Jalankan generator:

```bash
flutter pub run build_runner build
```

**`build_runner`** bakal auto-generate file `post.g.dart` yang isi `_$PostFromJson` dan `_$PostToJson`. Nggak perlu nulis satu pun field manual.

Untuk pemula: **mulai dari manual dulu**. Pahami cara kerjanya sebelum pakai generator. Kalau udah nyaman dan project-nya makin besar, barulah migrasi ke `json_serializable`.

---

## 6. Best Practice Ringkas

1. **Satu model = satu file.** `models/post.dart`, `models/comment.dart` — jangan gabung semua di satu file.
2. **Factory constructor `fromJson`** — ini konvensi resmi Dart. Selalu pakai nama ini.
3. **`toJson()` method** — perlu kalau kamu POST/PUT data ke server.
4. **Field `final`** — immutable data lebih aman, lebih mudah di-debug.
5. **Null safety** — pakai `String?` untuk field opsional, jangan pernah pakai `dynamic` cuma karena takut null.
6. **Naming konsisten** — kalau JSON pakai `snake_case` (`user_id`), konvert ke `camelCase` (`userId`) di model. Pakai library `json_annotation` `@JsonKey(name: 'user_id')` untuk handle ini.

---

## Ringkasan

- **Model class** mengubah JSON `dynamic` jadi Dart object yang strongly typed.
- **`factory fromJson(Map<String, dynamic>)`** — konstruktor standar untuk parsing JSON ke object.
- **`toJson()`** — mengubah object kembali jadi Map untuk dikirim ke server.
- **Nested JSON** — parse sub-object dengan panggil `SubModel.fromJson()` di dalam parent model.
- **Null safety** — field opsional wajib pakai `?` type, handle di UI pakai `??` atau null check.
- **Manual vs code gen** — mulai manual, pindah ke `json_serializable` kalau model udah besar.

Sekarang data di app kita bukan lagi `dynamic` misterius — melainkan objek Dart yang rapi, type-safe, dan gampang di-maintain. Langkah kecil, dampak besar untuk kualitas kode! 💪

**Coba sendiri!** Buat model class untuk endpoint `/users` dari JSONPlaceholder — ada field `name`, `email`, `phone`, `website`, dan nested `address.city`. Practice nested parsing! Share hasilnya ke sosial media dan tag @ahsai001! 🚀
