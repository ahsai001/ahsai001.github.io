---
layout: post
title: "Flutter #32: Repository Pattern — Clean Architecture"
date: 2026-09-10 07:00:00 +0700
tags: [flutter, dart, clean-architecture, repository-pattern, tutorial]
description: "Pelajari Repository Pattern dan Clean Architecture di Flutter. Pisahkan logic, data, dan UI agar kode mudah di-maintain dan di-scale."
---

Halo Developer! 👋

Di artikel sebelumnya kita sudah belajar **GetX** untuk state management, routing, dan dependency injection. Sekarang waktunya naik level ke arsitektur yang lebih rapi: **Repository Pattern** dalam **Clean Architecture**.

Bayangkan项目（project）kamu tumbuh dari 5 halaman jadi 50 halaman. Tanpa arsitektur yang jelas, semua logic — API call, parsing, UI — menumpuk di satu file. Répot kan? Repository Pattern adalah solusinya.

---

## Kenapa Clean Architecture?

Clean Architecture membagi aplikasi ke dalam **layer** terpisah:

1. **Presentation Layer** — Widget, UI, state management (GetX, Provider, dll)
2. **Domain Layer** — Business logic, entities, repository interface
3. **Data Layer** — Implementasi repository (API, database, cache)

Keuntungannya:
- **Testable** — Kamu bisa unit test logic tanpa jalankan UI
- **Swapable** — Ganti API ke database? Tinggal ganti implementasi, UI tetap sama
- **Readable** — Setiap file punya satu tanggung jawab

---

## Struktur Folder Clean Architecture

```
lib/
├── main.dart
├── core/
│   ├── errors/
│   │   └── exceptions.dart
│   └── network/
│       └── api_service.dart
├── features/
│   └── users/
│       ├── data/
│       │   ├── models/
│       │   │   └── user_model.dart
│       │   ├── repositories/
│       │   │   └── user_repository_impl.dart
│       │   └── datasources/
│       │       ├── user_remote_datasource.dart
│       │       └── user_local_datasource.dart
│       ├── domain/
│       │   ├── entities/
│       │   │   └── user_entity.dart
│       │   ├── repositories/
│       │   │   └── user_repository.dart
│       │   └── usecases/
│       │       └── get_users.dart
│       └── presentation/
│           ├── controllers/
│           │   └── user_controller.dart
│           └── pages/
│               └── user_list_page.dart
```

Terdengar kompleks? Tenang, kita bangun dari yang sederhana dulu.

---

## Contoh 1: Entity & Model

**Entity** adalah definisi data mentah tanpa dependensi ke framework:

```dart
// lib/features/users/domain/entities/user_entity.dart
class UserEntity {
  final int id;
  final String name;
  final String email;
  final String avatarUrl;

  const UserEntity({
    required this.id,
    required this.name,
    required this.email,
    this.avatarUrl = '',
  });
}
```

**Model** adalah representasi data dari API/JSON yang mengonversi ke Entity:

```dart
// lib/features/users/data/models/user_model.dart
import '../domain/entities/user_entity.dart';

class UserModel extends UserEntity {
  const UserModel({
    required super.id,
    required super.name,
    required super.email,
    super.avatarUrl,
  });

  factory UserModel.fromJson(Map<String, dynamic> json) {
    return UserModel(
      id: json['id'] as int,
      name: json['name'] as String,
      email: json['email'] as String,
      avatarUrl: json['avatar_url'] as String? ?? '',
    );
  }

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      'email': email,
      'avatar_url': avatarUrl,
    };
  }
}
```

> **Perbedaan Entity vs Model?** Entity cuma punya field data. Model punya `fromJson()` / `toJson()` dan aware dengan format data spesifik (API response).

---

## Contoh 2: Repository Interface & Implementasi

**Repository Interface** ada di **domain layer** — ini kontrak yang harus dipenuhi:

```dart
// lib/features/users/domain/repositories/user_repository.dart
import '../entities/user_entity.dart';

abstract class UserRepository {
  Future<List<UserEntity>> getUsers();
  Future<UserEntity> getUserById(int id);
}
```

**Repository Implementation** ada di **data layer** — yang eksekusi beneran:

```dart
// lib/features/users/data/repositories/user_repository_impl.dart
import '../datasources/user_remote_datasource.dart';
import '../models/user_model.dart';
import '../../domain/entities/user_entity.dart';
import '../../domain/repositories/user_repository.dart';

class UserRepositoryImpl implements UserRepository {
  final UserRemoteDataSource remoteDataSource;

  UserRepositoryImpl({required this.remoteDataSource});

  @override
  Future<List<UserEntity>> getUsers() async {
    try {
      final List<UserModel> users = await remoteDataSource.fetchUsers();
      return users; // UserModel IS-A UserEntity
    } catch (e) {
      throw ServerException(message: 'Gagal mengambil data user');
    }
  }

  @override
  Future<UserEntity> getUserById(int id) async {
    try {
      final UserModel user = await remoteDataSource.fetchUserById(id);
      return user;
    } catch (e) {
      throw ServerException(message: 'User tidak ditemukan');
    }
  }
}
```

**Remote DataSource** bertugas komunikasi langsung dengan API:

```dart
// lib/features/users/data/datasources/user_remote_datasource.dart
import 'dart:convert';
import 'package:http/http.dart' as http;
import '../models/user_model.dart';

class UserRemoteDataSource {
  final String baseUrl;

  UserRemoteDataSource({this.baseUrl = 'https://jsonplaceholder.typicode.com'});

  Future<List<UserModel>> fetchUsers() async {
    final response = await http.get(Uri.parse('$baseUrl/users'));
    if (response.statusCode == 200) {
      final List<dynamic> jsonList = json.decode(response.body);
      return jsonList.map((json) => UserModel.fromJson(json)).toList();
    }
    throw Exception('Gagal fetch users');
  }

  Future<UserModel> fetchUserById(int id) async {
    final response = await http.get(Uri.parse('$baseUrl/users/$id'));
    if (response.statusCode == 200) {
      return UserModel.fromJson(json.decode(response.body));
    }
    throw Exception('User tidak ditemukan');
  }
}
```

---

## Menggunakan Repository di Controller (GetX)

Sekarang kita hubungkan ke **presentation layer** pakai GetX:

```dart
// lib/features/users/presentation/controllers/user_controller.dart
import 'package:get/get.dart';
import '../../domain/entities/user_entity.dart';
import '../../domain/repositories/user_repository.dart';

class UserController extends GetxController {
  final UserRepository userRepository;

  UserController({required this.userRepository});

  final users = <UserEntity>[].obs;
  final isLoading = false.obs;
  final errorMessage = ''.obs;

  @override
  void onInit() {
    super.onInit();
    fetchUsers();
  }

  Future<void> fetchUsers() async {
    isLoading.value = true;
    errorMessage.value = '';
    try {
      final result = await userRepository.getUsers();
      users.assignAll(result);
    } catch (e) {
      errorMessage.value = 'Terjadi kesalahan: $e';
    } finally {
      isLoading.value = false;
    }
  }
}
```

---

## Wiring: Connect Everything

Di `main.dart` atau file setup:

```dart
import 'package:get/get.dart';
import 'features/users/data/datasources/user_remote_datasource.dart';
import 'features/users/data/repositories/user_repository_impl.dart';
import 'features/users/presentation/controllers/user_controller.dart';
import 'features/users/presentation/pages/user_list_page.dart';

void main() {
  // DataSource
  final remoteDataSource = UserRemoteDataSource();

  // Repository (interface → impl)
  final userRepository = UserRepositoryImpl(
    remoteDataSource: remoteDataSource,
  );

  // Controller
  Get.put<UserController>(
    UserController(userRepository: userRepository),
  );

  runApp(const MyApp());
}
```

> Karena `UserRepositoryImpl` implements `UserRepository`, kita bisa pakai **interface** di controller. Kalau nanti mau pakai local database, tinggal buat `UserRepositoryLocalImpl` dan ganti — controller tidak perlu diubah.

---

## Alur Visual Clean Architecture

```
┌─────────────────────────────────────────────┐
│              PRESENTATION                    │
│  UserListPage → UserController (GetX)       │
│  (Widget)       (state + logic)             │
└────────────────────┬────────────────────────┘
                     │ calls
┌────────────────────▼────────────────────────┐
│              DOMAIN                          │
│  GetUserUseCase → UserRepository (abstract)  │
│  UserEntity                                 │
└────────────────────┬────────────────────────┘
                     │ implements
┌────────────────────▼────────────────────────┐
│              DATA                            │
│  UserRepositoryImpl → RemoteDataSource       │
│  UserModel (fromJson/toJson)                │
│  API / Database / Cache                     │
└─────────────────────────────────────────────┘
```

Arah dependency: **Presentation → Domain ← Data**. Domain tidak tahu dari mana data datang.

---

## Best Practice

1. **Interface di Domain, Implementasi di Data** — agar domain layer bersih dari dependensi teknis
2. **Satu Repository per Feature** — `UserRepository`, `ProductRepository`, `OrderRepository`
3. **Jangan skip Entity** — meski mirip Model, Entity menjaga domain tetap independen
4. **Exception handling di layer Data** — jangan biarkan raw exception bocor ke UI
5. ** pakai `Either` atau Result type** untuk error handling yang lebih rapi (bisa pakai `fpdart` package)

---

## Kesimpulan

| Konsep | Layer | Tugas |
|--------|-------|-------|
| Entity | Domain | Definisi data murni |
| Repository Interface | Domain | Kontrak akses data |
| Repository Impl | Data | Eksekusi akses data |
| DataSource | Data | Komunikasi langsung (API/DB) |
| Controller | Presentation | State + logic |
| Widget | Presentation | UI |

Repository Pattern memang butuh setup lebih banyak di awal, tapi investasi ini terbayar saat project tumbuh. Kode jadi terstruktur, mudah di-test, dan siap di-scale.

---

Coba sendiri! Buat mini project dengan Clean Architecture dari awal. Mulai dari feature sederhana seperti User List pakai API public seperti [JSONPlaceholder](https://jsonplaceholder.typicode.com). Share ke sosial media dan tag **@ahsai001** 🚀
