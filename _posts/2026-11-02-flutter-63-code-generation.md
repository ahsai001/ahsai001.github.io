---
layout: post
title: "Flutter #63: Code Generation dengan build_runner"
date: 2026-11-02 07:00:00 +0700
categories: [flutter]
tags: [flutter, dart, tutorial, indonesia]
---

Flutter semakin produktif bila memanfaatkan **code generation**. Dengan `build_runner` dan paket seperti `json_serializable` atau `freezed`, kita mengurangi boilerplate, meningkatkan konsistensi, dan menghindari error manual.

## Kenapa Pakai Code Generation?
- **Rapi**: Model data hanya definisi satu kali.
- **Konsisten**: Semua kelas dibuat dengan pola yang sama.
- **Terintegrasi**: IDE dapat men‑generate kode secara otomatis saat menyimpan.

## 1. Siapkan Project
Buka terminal di root project Flutter Anda, lalu jalankan:

```bash
flutter pub add build_runner json_serializable
```

> `build_runner` menjalankan generator, `json_serializable` menghasilkan `fromJson`/`toJson`.

## 2. Buat Model `User`
Buat file `lib/models/user.dart` dengan anotasi:

```dart
import 'package:json_annotation/json_annotation.dart';

part 'user.g.dart'; // ← file yang akan di‑generate

@JsonSerializable()
class User {
  final String id;
  final String name;
  final int age;

  User({required this.id, required this.name, required this.age});

  // Boilerplate dibuang, generator buat otomatis
  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
  Map<String, dynamic> toJson() => _$UserToJson(this);
}
```

Simpan file, lalu jalankan generator:

```bash
flutter pub run build_runner build --delete-conflicting-outputs
```

Akan muncul `user.g.dart` dengan implementasi `_$UserFromJson` dan `_$UserToJson`.

## 3. Contoh Penggunaan
```dart
void main() {
  final json = {'id':'u123','name':'Budi','age':30};
  final user = User.fromJson(json);
  print('Halo ${user.name}, umur ${user.age}');

  final serialized = user.toJson();
  print('JSON kembali: $serialized');
}
```

Output:
```
Halo Budi, umur 30
JSON kembali: {id: u123, name: Budi, age: 30}
```

## 4. Meng‑generate Immutable Model dengan `freezed`
Untuk model **immutable** dan **copyWith**, gunakan paket `freezed`.

```bash
flutter pub add freezed_annotation freezed
flutter pub add --dev build_runner freezed
```

Buat file `lib/models/post.dart`:

```dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'post.freezed.dart';
part 'post.g.dart';

@freezed
class Post with _$Post {
  const factory Post({
    required String id,
    required String title,
    String? content,
  }) = _Post;

  factory Post.fromJson(Map<String, dynamic> json) => _$PostFromJson(json);
}
```

Run generator lagi:

```bash
flutter pub run build_runner build --delete-conflicting-outputs
```

Sekarang Anda dapat memakai:
```dart
final p1 = Post(id:'p1', title:'Hello');
final p2 = p1.copyWith(title:'Updated');
print(p2);
```

## 5. Tips Praktis
- **Jalankan otomatis**: Tambahkan script di `pubspec.yaml`:
```yaml
scripts:
  gen: flutter pub run build_runner build --delete-conflicting-outputs
```
- **Watch mode** selama development:
```bash
flutter pub run build_runner watch
```
- **Hindari konflik**: Selalu gunakan `--delete-conflicting-outputs`.

## Kesimpulan
Code generation mempercepat pengembangan Flutter, mengurangi boilerplate, dan menjaga konsistensi model data. Mulai dengan `json_serializable` untuk API, lalu jelajahi `freezed` untuk immutable dan utility lainnya.

**Coba sendiri! Share ke sosial media dan tag @ahsai001**
