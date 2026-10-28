---
layout: post
title: "Flutter #58: Modular Architecture (Part 1) - Memisahkan Fitur ke Module Terpisah"
date: 2026-10-29 07:00:00 +0700
categories: [flutter]
tags: [flutter, dart, tutorial, indonesia]
---

Besarnya aplikasi Flutter sering membuat codebase jadi sulit dikelola. Solusi klasik: **Modular Architecture** — memecah fitur besar jadi *module* yang independen, punya `pubspec.yaml` sendiri, bisa di-*import* sebagai package lokal. Part 1 ini fokus pada **struktur folder, cara membuat module, dan wiring via `pubspec.yaml` path dependency**.

---

## Kenapa Modular?

| Masalah monolith | Solusi modular |
|------------------|----------------|
| Semua kode di `lib/` bercampur | Tiap fitur punya folder `packages/fitur_x/` |
| Dependency graph kacau | Dependency eksplisit di `pubspec.yaml` tiap module |
| Build lambat, hot-reload semua | Hanya module yang berubah yang rebuild |
| Tim tabrakan merge | Tim bisa kerja paralel di module berbeda |

---

## Struktur Project Contoh

```
my_app/
├── pubspec.yaml                 # root app
├── lib/
│   └── main.dart                # entry point
└── packages/
    ├── core/                    # shared utilities, theme, di
    │   ├── pubspec.yaml
    │   └── lib/
    │       └── core.dart
    ├── auth/                    # fitur login/register
    │   ├── pubspec.yaml
    │   └── lib/
    │       └── auth.dart
    └── dashboard/               # fitur dashboard
        ├── pubspec.yaml
        └── lib/
            └── dashboard.dart
```

> **Tip:** Taruh semua module di folder `packages/` (bukan `modules/` agar konsisten dengan *melos* nanti).

---

## 1. Buat Module `core`

`packages/core/pubspec.yaml`

```yaml
name: core
version: 0.0.1
publish_to: none

environment:
  sdk: ">=3.3.0 <4.0.0"

dependencies:
  flutter:
    sdk: flutter
  freezed_annotation: ^2.4.1
  equatable: ^2.0.5

dev_dependencies:
  build_runner: ^2.4.9
  freezed: ^2.5.2
```

`packages/core/lib/core.dart`

```dart
library core;

export 'src/theme/app_theme.dart';
export 'src/di/service_locator.dart';
export 'src/utils/extensions.dart';
```

Isi file asli di `packages/core/lib/src/...` (biar `core.dart` tetap bersih).

---

## 2. Buat Module `auth`

`packages/auth/pubspec.yaml`

```yaml
name: auth
version: 0.0.1
publish_to: none

environment:
  sdk: ">=3.3.0 <4.0.0"

dependencies:
  flutter:
    sdk: flutter
  core:
    path: ../core
  firebase_auth: ^5.0.0
  google_sign_in: ^6.2.0
```

`packages/auth/lib/auth.dart`

```dart
library auth;

export 'src/presentation/pages/login_page.dart';
export 'src/presentation/bloc/auth_bloc.dart';
export 'src/data/repositories/auth_repository_impl.dart';
```

`auth` **depends on** `core` via `path: ../core`. Jadi `core` bisa dipakai tanpa publish ke pub.dev.

---

## 3. Daftarkan Module di Root `pubspec.yaml`

`my_app/pubspec.yaml`

```yaml
name: my_app
version: 1.0.0+1
publish_to: none

environment:
  sdk: ">=3.3.0 <4.0.0"

dependencies:
  flutter:
    sdk: flutter
  core:
    path: packages/core
  auth:
    path: packages/auth
  dashboard:
    path: packages/dashboard
```

Jalankan:

```bash
flutter pub get
```

Sekarang `import 'package:auth/auth.dart';` bisa dipakai di `lib/main.dart`.

---

## 4. Contoh Penggunaan di `main.dart`

```dart
import 'package:flutter/material.dart';
import 'package:core/core.dart';
import 'package:auth/auth.dart';

void main() {
  setupServiceLocator(); // dari core
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Modular Demo',
      theme: appTheme, // dari core
      home: const LoginPage(), // dari auth
    );
  }
}
```

---

## 5. Aturan Dependency (Best Practice)

1. **Core** → *tidak* bergantung ke module lain.
2. **Feature modules** (auth, dashboard) → boleh depend ke `core` & module lain **hanya kalau butuh**.
3. **Jangan** bikin circular dependency (`auth` → `dashboard` → `auth`).
4. Gunakan **interface/abstract class** di `core` lalu implementasi di feature module (Dependency Inversion).

---

## 6. Build & Test Module Terpisah

```bash
cd packages/auth
flutter test          # test unit/auth bloc
flutter analyze       # static analysis hanya module ini
```

Ini mempercepat CI karena hanya module yang berubah dijalankan.

---

## Ringkasan Part 1

- Pisahkan fitur ke `packages/<feature>/` dengan `pubspec.yaml` sendiri.
- Pakai `path:` dependency di root `pubspec.yaml`.
- `core` jadi fondasi shared (theme, DI, utils).
- Setiap module bisa di-*test* & *analyze* independen.

---

**Next (Part 2):** Kita akan bahas **Melos** untuk mengelola multi-package (bootstrap, versioning, publish), **code generation** (freezed, json_serializable) di level module, dan **feature flag** untuk mengaktifkan/menonaktifkan module di build flavor.

> Coba sendiri! Buat project baru, taruh `core` + `auth`, jalanin `flutter pub get`, lalu import `LoginPage` di `main.dart`. Share ke sosial media dan tag **@ahsai001**.