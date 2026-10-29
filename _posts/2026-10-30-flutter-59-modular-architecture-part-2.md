---
layout: post
title: "Flutter #59: Modular Architecture – Part 2"
date: 2026-10-30 07:00:00 +0700
categories: [flutter]
tags: [flutter, dart, tutorial, indonesia]
---

Modular Architecture adalah cara memecah aplikasi Flutter menjadi modul‑modul kecil yang dapat dikembangkan, diuji, dan dipelihara secara terpisah. Pada bagian **1** kita telah menyiapkan struktur direktori, membuat *feature module* `auth` dan menambahkan *dependency injection* dasar. Pada **Part 2** kita akan menambahkan **routing modular**, **shared core**, serta **testing** untuk tiap modul.

## 1️⃣ Routing Modular dengan `auto_route`

`auto_route` memungkinkan mendefinisikan rute di tiap modul, kemudian menggabungkannya di *app router* utama.

### a. Tambahkan dependensi (hanya satu baris di `pubspec.yaml`)
```yaml
auto_route: ^7.8.0
auto_route_generator: ^7.8.0
build_runner: ^2.4.0
```

### b. Buat file `lib/auth/presentation/router.dart`
```dart
import 'package:auto_route/auto_route.dart';
import 'package:flutter/material.dart';
import 'login_page.dart';
import 'signup_page.dart';

@MaterialAutoRouter(
  replaceInRouteName: 'Page,Route',
  routes: <AutoRoute>[
    AutoRoute(page: LoginPage, initial: true),
    AutoRoute(page: SignUpPage),
  ],
)
class $AuthRouter {}
```

### c. Generate kode
```bash
flutter pub run build_runner build --delete-conflicting-outputs
```

### d. Gabungkan ke *app router* (`lib/router/app_router.dart`)
```dart
import 'package:auto_route/auto_route.dart';
import 'package:my_app/auth/presentation/router.gr.dart'; // generated

@MaterialAutoRouter(
  replaceInRouteName: 'Page,Route',
  routes: <AutoRoute>[
    AutoRoute(page: HomePage),
    // Modul auth dipasang sebagai nested router
    AutoRoute(page: EmptyRouterPage, name: 'AuthRouter', children: AuthRouter().routes),
  ],
)
class $AppRouter {}
```

`EmptyRouterPage` berfungsi sebagai placeholder untuk modul yang memiliki rute internal. Sekarang semua navigasi *auth* berada di modul terpisah.

## 2️⃣ Core Shared – `core` package

Modul `core` berisi **utils**, **constants**, **theme**, dan **error handling** yang dapat dipakai semua modul.

### a. Struktur folder
```
lib/core/
 ├─ constants/
 │   └─ app_constants.dart
 ├─ theme/
 │   └─ app_theme.dart
 ├─ utils/
 │   └─ validators.dart
 └─ extensions/
     └─ string_extensions.dart
```

### b. Contoh `app_theme.dart`
```dart
import 'package:flutter/material.dart';

class AppTheme {
  static final light = ThemeData(
    primarySwatch: Colors.indigo,
    visualDensity: VisualDensity.adaptivePlatformDensity,
  );

  static final dark = ThemeData.dark().copyWith(
    primaryColor: Colors.indigo,
  );
}
```

### c. Menggunakan di modul lain
```dart
import 'package:my_app/core/theme/app_theme.dart';

MaterialApp(
  theme: AppTheme.light,
  darkTheme: AppTheme.dark,
  // ...
);
```

## 3️⃣ Testing Modul Individu

Setiap modul memiliki **unit test** untuk domain & **widget test** untuk UI. Contoh: menguji `AuthRepository`.

### a. Tambahkan `test` folder di modul `auth`
```
auth/test/auth_repository_test.dart
```

### b. Contoh unit test
```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:my_app/auth/data/auth_repository.dart';

void main() {
  test('login returns true for valid credentials', () async {
    final repo = AuthRepository();
    final result = await repo.login('test@example.com', 'Password123');
    expect(result, isTrue);
  });
}
```

### c. Jalankan semua tes
```bash
flutter test
```

Jika semua modul lulus, proyek tetap stabil meski tim menambah fitur baru.

## 4️⃣ Integrasi Semua Modul di `main.dart`
```dart
import 'package:flutter/material.dart';
import 'package:my_app/router/app_router.gr.dart';
import 'package:my_app/core/theme/app_theme.dart';
import 'package:get_it/get_it.dart';

final getIt = GetIt.instance;

void main() async {
  // Registrasi dependency global
  getIt.registerSingleton<AppRouter>(AppRouter());
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  final _appRouter = getIt<AppRouter>();
  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerDelegate: _appRouter.delegate(),
      routeInformationParser: _appRouter.defaultRouteParser(),
      theme: AppTheme.light,
      darkTheme: AppTheme.dark,
    );
  }
}
```

## 5️⃣ Ringkasan & CTA

- **Routing modular** memisahkan UI navigation per fitur.
- **Core shared** menyimpan kode yang dapat dipakai lintas modul.
- **Testing** tiap modul menjamin kualitas ketika aplikasi tumbuh.
- **Dependency injection** (`get_it`) menyederhanakan pengelolaan service.

> **Coba sendiri!** Buat modul `profile`, tambahkan router, dan tes unitnya. Share ke sosial media dan tag **@ahsai001**.

---
