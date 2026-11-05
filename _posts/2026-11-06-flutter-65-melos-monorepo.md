---
layout: post
title: "Flutter #65: Melos Monorepo - Kelola Banyak Package Flutter dalam Satu Repo"
date: 2026-11-06 07:00:00 +0700
categories: [flutter]
tags: [flutter, dart, tutorial, indonesia]
---

Kita sudah sampai di artikel ke-65 seri tutorial Flutter! 🎉 Hari ini kita bahas **Melos** — tool wajib kalau kamu mau manage multiple Dart/Flutter packages dalam satu repository (monorepo).

---

## Kenapa Butuh Monorepo?

Bayangkan kamu punya project Flutter yang gede: ada `core` package (utils, models, theme), `feature_auth`, `feature_chat`, `feature_payment`, dan `app` sebagai entry point. Kalau dipisah repo, bakal ribet:
- Versioning manual tiap package
- Dependency hell saat update shared code
- CI/CD terfragmentasi

**Monorepo** solve ini: satu repo, banyak package, versioning terpusat, shared tooling.

---

## Apa Itu Melos?

> **Melos** = tool manajemen monorepo untuk Dart/Flutter (seperti `npm workspaces` tapi lebih powerful).

Fitur utamanya:
- 🔗 **Link local packages** otomatis (ga perlu `path: ../package` manual)
- 📦 **Versioning & publishing** otomatis ke pub.dev
- 🛠 **Scripts** yang bisa jalan di semua package sekaligus (`melos run test`)
- 🌲 **Dependency graph** visualisasi

---

## Setup Melos di Project Flutter

### 1. Install Melos

```bash
dart pub global activate melos
# atau via Homebrew
brew install melos
```

Pastikan `$HOME/.pub-cache/bin` ada di PATH.

### 2. Inisialisasi Project

```bash
mkdir my_flutter_monorepo
cd my_flutter_monorepo
melos init
```

Ini bikin `melos.yaml` — file konfigurasi utama.

---

## Struktur Monorepo Contoh

```
my_flutter_monorepo/
├── melos.yaml              # Konfigurasi Melos
├── pubspec.yaml            # Root pubspec (optional)
├── packages/
│   ├── core/               # Shared package
│   │   ├── pubspec.yaml
│   │   └── lib/
│   ├── feature_auth/
│   │   ├── pubspec.yaml
│   │   └── lib/
│   ├── feature_chat/
│   │   ├── pubspec.yaml
│   │   └── lib/
│   └── feature_payment/
│       ├── pubspec.yaml
│       └── lib/
└── apps/
    └── my_app/             # Entry point Flutter app
        ├── pubspec.yaml
        └── lib/
```

---

## Konfigurasi `melos.yaml` Lengkap

```yaml
name: my_flutter_monorepo
repository: https://github.com/username/my_flutter_monorepo

# Folder yang berisi packages
packages:
  - packages/*
  - apps/*

# Scripts yang bisa dijalankan: `melos run <script>`
scripts:
  # Install deps semua package sekaligus
  bootstrap:
    run: flutter pub get
    package_filters:
      dir_exists: test

  # Run test semua package
  test:
    run: flutter test
    package_filters:
      dir_exists: test

  # Analyze code semua package
  analyze:
    run: flutter analyze

  # Format code
  format:
    run: dart format .

  # Build semua package
  build:
    run: flutter build apk --release
    package_filters:
      dir_exists: android

  # Version bump & changelog
  version:
    run: melos version --yes

  # Publish ke pub.dev (dry-run dulu)
  publish:
    run: melos publish --dry-run

# Konfigurasi versioning
version:
  # Gunakan conventional commits untuk tentuin version bump
  # feat: -> minor, fix: -> patch, BREAKING CHANGE: -> major
  command: melos version --yes

# Git hooks (optional)
command:
  bootstrap:
    run: melos bootstrap
    description: "Install dependencies for all packages"
```

---

## Contoh `pubspec.yaml` Package

### `packages/core/pubspec.yaml`

```yaml
name: core
version: 1.0.0
description: Shared utilities, models, and theme for the app

environment:
  sdk: ">=3.2.0 <4.0.0"

dependencies:
  flutter:
    sdk: flutter
  equatable: ^2.0.5
  intl: ^0.19.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  very_good_analysis: ^5.1.0
```

### `packages/feature_auth/pubspec.yaml`

```yaml
name: feature_auth
version: 1.0.0
description: Authentication feature module

environment:
  sdk: ">=3.2.0 <4.0.0"

dependencies:
  flutter:
    sdk: flutter
  core:
    path: ../core  # Akan di-link otomatis oleh Melos
  flutter_bloc: ^8.1.3
  equatable: ^2.0.5

dev_dependencies:
  flutter_test:
    sdk: flutter
  bloc_test: ^9.1.5
  mocktail: ^1.0.0
```

### `apps/my_app/pubspec.yaml`

```yaml
name: my_app
version: 1.0.0+1
description: Main Flutter application

environment:
  sdk: ">=3.2.0 <4.0.0"

dependencies:
  flutter:
    sdk: flutter
  core:
    path: ../../packages/core
  feature_auth:
    path: ../../packages/feature_auth
  feature_chat:
    path: ../../packages/feature_chat
  feature_payment:
    path: ../../packages/feature_payment
  flutter_bloc: ^8.1.3
  go_router: ^12.1.1

dev_dependencies:
  flutter_test:
    sdk: flutter
  integration_test:
    sdk: flutter
```

> **Note:** Path-dependency (`path: ../core`) masih ditulis di pubspec, tapi **Melos akan rewrite ke dependency lokal yang linked** saat `melos bootstrap`.

---

## Workflow Harian dengan Melos

### 1. Bootstrap (Install Semua Deps)

```bash
melos bootstrap
```

Ini akan:
- `flutter pub get` di semua package
- Link local packages symlink otomatis
- Generate `pubspec_overrides.yaml` kalau perlu

### 2. Development: Jalankan Test/Analyze Semua

```bash
# Test semua package yang punya folder test/
melos run test

# Analyze semua
melos run analyze

# Format kode
melos run format
```

### 3. Filter Package (Jangan Jalanin di Semua)

```bash
# Hanya package yang nama mengandung "feature"
melos run test --scope="feature*"

# Hanya package yang punya folder "android"
melos run build --dir-exists=android

# Kecualikan package tertentu
melos run test --ignore="feature_payment"
```

### 4. Versioning & Changelog Otomatis

Commit message pakai **Conventional Commits**:

```bash
git commit -m "feat(auth): add biometric login support"
git commit -m "fix(core): handle null date parsing"
git commit -m "BREAKING CHANGE(api): rename UserModel to UserEntity"
```

Lalu:

```bash
melos version
```

Ini akan:
- Bump version tiap package berdasarkan commit
- Generate `CHANGELOG.md` otomatis
- Commit dan tag git

### 5. Publish ke pub.dev

```bash
# Dry-run dulu (aman)
melos publish --dry-run

# Publish beneran
melos publish
```

---

## Contoh Kode Nyata: Shared Theme di `core`

```dart
// packages/core/lib/src/theme/app_theme.dart
import 'package:flutter/material.dart';

class AppTheme {
  static ThemeData get lightTheme => ThemeData(
        useMaterial3: true,
        colorScheme: ColorScheme.fromSeed(
          seedColor: const Color(0xFF2563EB), // Blue 600
          brightness: Brightness.light,
        ),
        textTheme: const TextTheme(
          headlineLarge: TextStyle(fontWeight: FontWeight.bold),
          headlineMedium: TextStyle(fontWeight: FontWeight.w600),
          bodyLarge: TextStyle(fontSize: 16),
          bodyMedium: TextStyle(fontSize: 14),
        ),
        elevatedButtonTheme: ElevatedButtonThemeData(
          style: ElevatedButton.styleFrom(
            padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 12),
            shape: RoundedRectangleBorder(
              borderRadius: BorderRadius.circular(12),
            ),
          ),
        ),
        cardTheme: CardThemeData(
          elevation: 2,
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(16),
          ),
        ),
      );

  static ThemeData get darkTheme => ThemeData(
        useMaterial3: true,
        colorScheme: ColorScheme.fromSeed(
          seedColor: const Color(0xFF3B82F6), // Blue 500
          brightness: Brightness.dark,
        ),
        // ... konfigurasi dark theme
      );
}
```

Dipakai di `feature_auth`:

```dart
// packages/feature_auth/lib/src/presentation/pages/login_page.dart
import 'package:core/theme/app_theme.dart';
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

class LoginPage extends StatelessWidget {
  const LoginPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Theme(
      data: AppTheme.lightTheme,
      child: Scaffold(
        appBar: AppBar(title: const Text('Login')),
        body: const Center(child: _LoginForm()),
      ),
    );
  }
}

class _LoginForm extends StatelessWidget {
  const _LoginForm();

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.all(24),
      child: Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          Text('Welcome Back', style: Theme.of(context).textTheme.headlineMedium),
          const SizedBox(height: 24),
          TextField(
            decoration: const InputDecoration(
              labelText: 'Email',
              prefixIcon: Icon(Icons.email_outlined),
            ),
          ),
          const SizedBox(height: 16),
          TextField(
            obscureText: true,
            decoration: const InputDecoration(
              labelText: 'Password',
              prefixIcon: Icon(Icons.lock_outline),
            ),
          ),
          const SizedBox(height: 24),
          SizedBox(
            width: double.infinity,
            child: ElevatedButton(
              onPressed: () {},
              child: const Text('Login'),
            ),
          ),
        ],
      ),
    );
  }
}
```

---

## Tips & Best Practices

| Tips | Penjelasan |
|------|------------|
| **Gunakan `package_filters`** | Hanya jalankan script di package yang relevan (misal `dir_exists: test`) |
| **Conventional Commits** | Wajib biar versioning & changelog otomatis jalan |
| **CI/CD dengan Melos** | Di GitHub Actions: `melos run test`, `melos run analyze` |
| **Jangan over-split** | Package terlalu kecil bikin maintenance overhead. Group by feature/domain |
| **Shared `analysis_options.yaml`** | Taruh di root, include di tiap package: `include: ../../analysis_options.yaml` |

---

## CI/CD GitHub Actions Contoh

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dart-lang/setup-dart@v1
        with:
          sdk: '3.2.0'
      - run: dart pub global activate melos
      - run: melos bootstrap
      - run: melos run analyze
      - run: melos run test
      - run: melos run format -- --set-exit-if-changed
```

---

## Kesimpulan

Melos bikin monorepo Flutter **bener-bener maintainable**:
- ✅ Satu command jalanin test/analyze/build semua package
- ✅ Versioning & publish otomatis pakai conventional commits
- ✅ Local package linking otomatis (no more `path: ../../..` manual)
- ✅ Filtering package biar CI cepet

Kalau project Flutter kamu mulai gede dan punya shared code di banyak feature — **waktunya migrasi ke Melos**.

---

## Referensi

- [Melos Documentation](https://melos.invertase.dev/)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [Very Good Analysis](https://pub.dev/packages/very_good_analysis) — lint rules recommended

---

Coba sendiri! Setup Melos di project existing atau bikin monorepo baru. Share ke sosial media dan tag **@ahsai001** 🚀