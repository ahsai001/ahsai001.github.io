---
layout: post
title: "Flutter #33: Dependency Injection (get_it)"
date: 2026-09-11 07:00:00 +0700
tags: [flutter, dart, dependency-injection, get-it, architecture]
description: "Pelajari Dependency Injection di Flutter menggunakan package get_it. Buat kode lebih modular, mudah di-test, dan tidak bergantung pada implementasi tertentu."
---

Halo Developer! 👋

Di artikel sebelumnya kita sudah belajar **Repository Pattern** untuk memisahkan logic dari UI. Sekarang waktunya lanjut ke topik yang saling berkaitan erat: **Dependency Injection (DI)**.

Ever since kita pakai Repository Pattern, mungkin kamu sadar ada masalah baru — bagaimana caranya `HomePage` dapat akses ke `UserRepository` tanpa harus tahu detail implementasinya? Di sinilah **Dependency Injection** hadir sebagai solusi.

---

## Dependency Injection Itu Apa?

Secara sederhana, Dependency Injection adalah **mengirim dependensi (objek yang dibutuhkan) dari luar** alih-alih membuatnya sendiri di dalam kelas.

**Tanpa DI:**

```dart
class HomePage extends StatefulWidget {
  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  // HomePage harus tahu cara membuat UserRepository
  final _repo = UserRepository(
    apiClient: ApiClient(baseUrl: 'https://api.example.com'),
  );

  @override
  Widget build(BuildContext context) {
    // ...
  }
}
```

Masalahnya? `HomePage` harus tahu `BaseUrl`, harus tahu cara bikin `ApiClient`, harus tahu cara bikin `UserRepository`. Kalau kita ganti base URL, semua halaman yang pakai `UserRepository` harus diubah.

**Dengan DI:**

```dart
class HomePage extends StatefulWidget {
  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  // Ambil dari "service locator" — tidak peduli bagaimana dibuat
  final _repo = getIt<UserRepository>();

  @override
  Widget build(BuildContext context) {
    // ...
  }
}
```

`HomePage` tinggal ambil `UserRepository` yang sudah siap pakai. Mau ganti base URL? Cukup ubah di satu tempat saja.

---

## Kenapa pakai `get_it`?

Flutter sudah punya `InheritedWidget` dan sebenarnya state management seperti Provider juga bisa jadi DI container. Tapi `get_it` punya kelebihan:

1. **Simple** — Hanya `register` dan `get`, tidak ada boilerplate Widget
2. **Bisa dipakai di mana saja** — Bukan hanya di Widget tree, tapi juga di service layer, utility, atau main()
3. **Tidak perlu context** — Tidak seperti Provider yang butuh `Provider.of(context)`
4. **Singleton & Lazy Singleton** — Memastikan hanya ada satu instance (hemat memory)
5. **Ringan** — Package-nya sangat kecil, cocok untuk semua skala project

---

## Setup `get_it`

### 1. Install package

Tambahkan di `pubspec.yaml`:

```yaml
dependencies:
  get_it: ^8.0.3
```

Lalu jalankan:

```bash
flutter pub get
```

### 2. Buat Service Locator file

Buat file baru `lib/core/locator.dart`:

```dart
import 'package:get_it/get_it.dart';
import '../data/repositories/user_repository.dart';
import '../data/services/api_client.dart';
import '../data/services/local_storage.dart';
import '../blocs/user/user_cubit.dart';

final getIt = Get.instance;

void setupLocator() {
  // Services — dependensi "paling dasar"
  getIt.registerLazySingleton<ApiClient>(
    () => ApiClient(baseUrl: 'https://api.example.com'),
  );
  getIt.registerLazySingleton<LocalStorage>(
    () => LocalStorage(),
  );

  // Repository — butuh Services
  getIt.registerLazySingleton<UserRepository>(
    () => UserRepository(
      apiClient: getIt<ApiClient>(),
      storage: getIt<LocalStorage>(),
    ),
  );

  // Cubit/BLoC — butuh Repository
  getIt.registerFactory<UserCubit>(
    () => UserCubit(repository: getIt<UserRepository>()),
  );
}
```

### 3. Panggil di `main()`

```dart
import 'core/locator.dart';

void main() {
  WidgetsFlutterBinding.ensureInitialized();
  setupLocator();
  runApp(const MyApp());
}
```

Sekarang di mana saja di dalam app, kamu bisa ambil dependensi dengan `getIt<T>()`.

---

## Tipe Registrasi di `get_it`

`get_it` menyediakan beberapa cara mendaftarkan service:

### Lazy Singleton

```dart
getIt.registerLazySingleton<ApiClient>(
  () => ApiClient(baseUrl: 'https://api.example.com'),
);
```

Objek **dibuat saat pertama kali diakses** (lazy), dan hanya **satu instance** (singleton). Cocok untuk service yang mahal dibuat tapi tidak harus langsung dibuat saat app start.

### Singleton

```dart
getIt.registerSingleton<LocalStorage>(
  LocalStorage()..init(),
);
```

Objek **dibuat langsung saat registrasi** dan hanya ada satu instance. Cocok untuk service yang harus langsung siap (misal LocalStorage yang perlu `init()` async — tapi kalau async, pakai `registerSingletonAsync`).

### Factory

```dart
getIt.registerFactory<UserCubit>(
  () => UserCubit(repository: getIt<UserRepository>()),
);
```

Objek **dibuat baru setiap kali diakses**. Cocok untuk Cubit/BLoC karena tiap halaman butuh Cubit yang terpisah.

### Perbandingan singkat

| Tipe | Kapan dibuat | Jumlah instance |
|------|-------------|-----------------|
| `registerSingleton` | Saat registrasi | 1 |
| `registerLazySingleton` | Saat pertama kali diakses | 1 |
| `registerFactory` | Setiap kali diakses | Banyak (baru tiap kali) |

---

## Contoh Lengkap: UserCubit dengan DI

Mari kita lihat bagaimana DI mengalir dari data layer sampai UI.

### UserCubit

```dart
import 'package:flutter_bloc/flutter_bloc.dart';
import '../../data/repositories/user_repository.dart';

class UserCubit extends Cubit<UserState> {
  final UserRepository _repository;

  UserCubit({required UserRepository repository})
      : _repository = repository,
        super(UserInitial());

  Future<void> loadUsers() async {
    emit(UserLoading());
    try {
      final users = await _repository.getUsers();
      emit(UserLoaded(users));
    } catch (e) {
      emit(UserError(e.toString()));
    }
  }
}
```

Tidak ada import `ApiClient`, tidak ada import `LocalStorage`. `UserCubit` hanya peduli pada `UserRepository`. Inilah kekuatan DI — **loose coupling**.

### Menggunakan di Widget

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:get_it/get_it.dart';
import '../../blocs/user/user_cubit.dart';

class UserListPage extends StatelessWidget {
  const UserListPage({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (_) => getIt<UserCubit>()..loadUsers(),
      child: Scaffold(
        appBar: AppBar(title: const Text('Users')),
        body: BlocBuilder<UserCubit, UserState>(
          builder: (context, state) {
            if (state is UserLoading) {
              return const Center(child: CircularProgressIndicator());
            }
            if (state is UserError) {
              return Center(child: Text('Error: ${state.message}'));
            }
            if (state is UserLoaded) {
              return ListView.builder(
                itemCount: state.users.length,
                itemBuilder: (_, index) {
                  final user = state.users[index];
                  return ListTile(
                    title: Text(user.name),
                    subtitle: Text(user.email),
                  );
                },
              );
            }
            return const SizedBox.shrink();
          },
        ),
      ),
    );
  }
}
```

Perhatikan baris `create: (_) => getIt<UserCubit>()..loadUsers()`. Widget tidak tahu cara `UserCubit` dibuat — ia hanya meminta ke `getIt`.

### Bonus: Lazy singleton untuk Cubit

Kalau kamu ingin Cubit-nya singleton (shared across app), ganti registrasi:

```dart
// Di locator.dart — pakai singleton bukan factory
getIt.registerLazySingleton<UserCubit>(
  () => UserCubit(repository: getIt<UserRepository>()),
);
```

Sekarang di mana pun kamu panggil `getIt<UserCubit>()`, hasilnya selalu instance yang sama.

---

## Testing dengan DI

Salah satu manfaat terbesar DI: **mudah di-test**. Untuk unit test, tinggal register mock:

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:get_it/get_it.dart';
import 'package:mockito/mockito.dart';

// Mock Repository
class MockUserRepository extends Mock implements UserRepository {}

void main() {
  final getIt = Get.instance;

  setUp(() {
    // Reset sebelum tiap test
    getIt.reset();

    // Register mock
    getIt.registerLazySingleton<UserRepository>(
      () => MockUserRepository(),
    );
  });

  test('loadUsers emits Loaded state', () async {
    final mockRepo = getIt<UserRepository>();
    when(mockRepo.getUsers()).thenAnswer(
      (_) async => [
        User(name: 'Ahmad', email: 'ahmad@test.com'),
      ],
    );

    final cubit = getIt<UserCubit>();
    await cubit.loadUsers();

    expect(cubit.state, isA<UserLoaded>());
  });
}
```

`UserCubit` tidak berubah sedikit pun — kita hanya mengganti `UserRepository` dengan versi mock. Inilah loose coupling yang dijanjikan DI.

---

## Kapan Harus Pakai DI?

| Situasi | Butuh DI? |
|---------|-----------|
| App sederhana (1-3 halaman) | ❌ Mungkin overkill |
| Multi-page dengan shared service | ✅ Sangat membantu |
| Unit testing | ✅ Membuat testing jauh lebih mudah |
| Banyak service (API, storage, auth, analytics) | ✅ Wajib pakai |
| Bekerja dalam tim / project besar | ✅ Standar industri |

---

## Kesimpulan

Dependency Injection dengan `get_it` adalah tool sederhana tapi sangat powerful untuk project Flutter:

1. **Loose coupling** — Widget tidak perlu tahu cara dependensi dibuat
2. **Satu titik registrasi** — Ganti implementasi di satu tempat saja
3. **Siap untuk testing** — Tinggal swap dengan mock
4. **Ringan dan simple** — Tidak ada boilerplate berlebih

Di artikel selanjutnya kita akan lanjut ke **Error Handling Best Practice** — bagaimana menangani error secara elegan di seluruh layer aplikasi Flutter.

---

Sekarang kamu sudah paham bagaimana menghubungkan semua layer di Flutter tanpa menghardcode dependensi di mana-mana. 🔗

**Coba sendiri!** Buat project Flutter baru, setup `get_it`, dan refactor kode yang sebelumnya hardcode dependency. Share ke sosial media dan tag **@ahsai001**! 🚀
