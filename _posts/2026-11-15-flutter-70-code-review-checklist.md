---
layout: post
title: "Flutter #70: Code Review Checklist — Panduan Lengkap Kualitas Kode Tim"
date: 2026-11-15 07:00:00 +0700
categories: [flutter]
tags: [flutter, dart, tutorial, indonesia]
---

Sudah sampai di artikel ke-70 — terakhir dari seri Flutter 90 hari! 🎉 Hari ini kita bahas **Code Review Checklist**, panduan praktis agar setiap Pull Request (PR) yang masuk ke repositori tim Flutter kamu konsisten, aman, dan maintainable.

Code review bukan cuma "nyari bug" — tapi budaya *shared ownership*, mentoring junior, dan jaga kualitas jangka panjang. Checklist ini dipakai tim AhsaLabs dan bisa kamu adaptasi langsung.

---

## Kenapa Butuh Checklist?

Tanpa checklist, review jadi *ad-hoc*:
- Reviewer A cek logic, reviewer B cek formatting, reviewer C cek test
- Hal penting kelewat (misal: `dispose()` tidak dipanggil, hardcoded string, missing test)
- Feedback jadi inconsistens — developer bingus mana yang *must fix* vs *nice to have*

Checklist bikin review **terstruktur, cepat, dan adil**.

---

## Checklist Code Review Flutter (Versi AhsaLabs)

### 1. Arsitektur & Struktur

| ✅ Item | Deskripsi |
|--------|-----------|
| **Layer separation** | Business logic di `domain/`, data di `data/`, UI di `presentation/` — tidak campur aduk |
| **Dependency rule** | Inner layer tidak import outer layer (Clean Architecture) |
| **Single Responsibility** | Setiap class/widget punya 1 alasan berubah |
| **Feature-first folders** | Struktur folder per fitur, bukan per layer (lebih scalable) |

```dart
// ❌ SALAH: Logic di widget
class UserProfile extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final user = await UserRepository().getUser(); // Network call di build!
    return Text(user.name);
  }
}

// ✅ BENAR: Logic di provider/bloc, widget cuma tampilkan
class UserProfile extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);
    return userAsync.when(
      data: (user) => Text(user.name),
      loading: () => CircularProgressIndicator(),
      error: (e, _) => Text('Error: $e'),
    );
  }
}
```

### 2. Dart & Flutter Best Practices

| ✅ Item | Deskripsi |
|--------|-----------|
| **`const` constructors** | Gunakan `const` di mana mungkin (perf + catch bug compile-time) |
| **`late` & `final`** | Hindari `var` tanpa inisialisasi; pakai `late final` untuk DI |
| **Null safety** | Tidak ada `!` paksa tanpa guard; gunakan `?.` atau `??` |
| **Immutable models** | Data class pakai `freezed` atau `@immutable` + `Equatable` |
| **No `BuildContext` async gap** | Jika `await` lalu pakai `context`, cek `mounted` dulu |

```dart
// ❌ SALAH: context bisa tidak valid setelah await
Future<void> _submit() async {
  await api.save(data);
  ScaffoldMessenger.of(context).showSnackBar(...); // Crash kalau user close screen
}

// ✅ BENAR: cek mounted
Future<void> _submit() async {
  await api.save(data);
  if (!mounted) return;
  ScaffoldMessenger.of(context).showSnackBar(...);
}
```

### 3. State Management

| ✅ Item | Deskripsi |
|--------|-----------|
| **Provider scope** | Provider dibungkus di scope terkecil (tidak global sembarangan) |
| **Dispose/cleanup** | `StreamController`, `Timer`, `AnimationController` selalu `dispose()` |
| **No logic di `build`** | `build` murni UI; logic di `initState`, `effect`, atau notifier |
| **Selector/filter** | Pakai `select()` / `watch((p) => p.specificField)` biar rebuild minimal |

```dart
// ❌ SALAH: Rebuild seluruh tree saat 1 field berubah
final user = ref.watch(userProvider); // user: UserModel (nama, email, avatar, settings...)

// ✅ BENAR: Hanya rebuild saat nama berubah
final userName = ref.watch(userProvider.select((u) => u.name));
```

### 4. UI & Widget

| ✅ Item | Deskripsi |
|--------|-----------|
| **Widget kecil** | Pecah widget besar jadi komponen reusable (Single Responsibility) |
| **`const` widget** | Widget statis dijadikan `const` (contoh: `const SizedBox(height: 16)`) |
| **Key yang tepat** | `ValueKey` untuk list item unik, `GlobalKey` hanya kalau butuh akses state |
| **Accessibility** | `semanticLabel` pada `IconButton`, `Image`, custom widget |
| **Responsive** | Pakai `LayoutBuilder`, `MediaQuery`, `Flexible` — tidak hardcode width |

```dart
// ✅ Widget kecil, reusable, const-friendly
class AppButton extends StatelessWidget {
  final String label;
  final VoidCallback? onPressed;
  final bool loading;

  const AppButton({
    super.key,
    required this.label,
    this.onPressed,
    this.loading = false,
  });

  @override
  Widget build(BuildContext context) {
    return FilledButton(
      onPressed: loading ? null : onPressed,
      child: loading
          ? const SizedBox(
              width: 20,
              height: 20,
              child: CircularProgressIndicator(strokeWidth: 2),
            )
          : Text(label),
    );
  }
}
```

### 5. Network & Data

| ✅ Item | Deskripsi |
|--------|-----------|
| **Error handling** | Semua `try-catch` return `Result/Either` atau throw custom exception |
| **Timeout** | HTTP client punya timeout (connect + receive) — tidak hang forever |
| **Retry logic** | Idempotent request (GET) pakai retry exponential backoff |
| **Caching strategy** | Cache-read-first untuk data jarang berubah (pakai `hydrated_bloc` / `riverpod_cache`) |
| **No sensitive data di log** | Token, password, PII tidak di-print ke console |

```dart
// ✅ Wrapper Result pattern (bisa pakai package `dartz` atau buat sendiri)
sealed class Result<T> {
  const Result();
}

class Success<T> extends Result<T> {
  final T value;
  const Success(this.value);
}

class Failure<T> extends Result<T> {
  final AppException error;
  const Failure(this.error);
}

// Penggunaan di repository
Future<Result<User>> getUser(String id) async {
  try {
    final response = await _client.get('/users/$id');
    return Success(User.fromJson(response.data));
  } on DioException catch (e) {
    return Failure(NetworkException.fromDio(e));
  }
}
```

### 6. Testing

| ✅ Item | Deskripsi |
|--------|-----------|
| **Unit test** | Logic bisnis (notifier, usecase, repository) — coverage ≥ 80% |
| **Widget test** | Critical flow (login, checkout, form validasi) — pakai `pumpWidget` |
| **Golden test** | UI pixel-perfect (design system component) |
| **Integration test** | End-to-end happy path (CI jalan di device/emulator) |
| **Test naming** | `should_[expected]_when_[condition]` — readable |

```dart
// ✅ Nama test jelas
test('should_return_user_when_api_succeeds', () async {
  // arrange
  when(mockClient.get('/users/1')).thenAnswer((_) async => Response(data: {'id': 1, 'name': 'Ahsa'}, statusCode: 200));
  
  // act
  final result = await repository.getUser('1');
  
  // assert
  expect(result, isA<Success<User>>());
  expect((result as Success<User>).value.name, 'Ahsa');
});
```

### 7. Performance

| ✅ Item | Deskripsi |
|--------|-----------|
| **ListView.builder** | Untuk list panjang — tidak `Column` + `List.generate` |
| **Image caching** | `CachedNetworkImage` atau `precacheImage` — tidak reload berulang |
| **`const` & `RepaintBoundary`** | Isolate repaint area yang sering berubah (animasi, chart) |
| **Profile mode test** | Cek `flutter run --profile` sebelum merge — no jank di scroll/animasi |

### 8. Security & Privacy

| ✅ Item | Deskripsi |
|--------|-----------|
| **No hardcoded secret** | API key, token di `--dart-define` atau `Env` — tidak di `lib/` |
| **Certificate pinning** | Production app wajib SSL pinning (pakai `dio` + `flutter_secure_storage`) |
| **Biometric/Auth** | Data sensitif (saldo, riwayat medis) butuh auth ulang |
| **Data minimization** | Hanya minta permission yang benar-benar dipakai |

### 9. Git & CI/CD

| ✅ Item | Deskripsi |
|--------|-----------|
| **Conventional Commits** | `feat:`, `fix:`, `refactor:`, `test:`, `chore:` — otomatis changelog |
| **Branch naming** | `feat/user-profile`, `fix/login-crash`, `refactor/api-layer` |
| **PR size** | < 400 baris perubahan (kecuali generated code) — pecah kalau besar |
| **Self-review dulu** | Author cek own PR sebelum assign reviewer (cek diff, jalanin test) |
| **CI pass** | `flutter analyze`, `flutter test`, `flutter build apk --release` semua hijau |

---

## Template PR Description (Copy-Paste)

```markdown
## 📝 Deskripsi
[Jelaskan apa yang diubah & kenapa — 2-3 kalimat]

## ✅ Checklist Author (Self-Review)
- [ ] `flutter analyze` — 0 error, 0 warning baru
- [ ] `flutter test` — semua pass, coverage tidak turun
- [ ] `flutter build apk --release` — build sukses
- [ ] Tidak ada hardcoded string (pakai `AppLocalizations`)
- [ ] Tidak ada `print()` / `debugPrint()` di production code
- [ ] Widget baru punya `const` constructor kalau memungkinkan
- [ ] Logic baru punya unit test
- [ ] UI baru punya widget test / golden test

## 🧪 Cara Test Manual
1. [Langkah 1]
2. [Langkah 2]
3. [Expected result]

## 📸 Screenshot / Video (UI changes)
[Drag-drop screenshot atau link Loom]

## 🔗 Related Issue / Task
Closes #123
```

---

## Tips Reviewer: Efektif & Santai

1. **Pakai label**: `🔴 Must Fix` (bug, security, breaking) vs `🟡 Suggestion` (style, naming, optimasi) vs `🟢 Nitpick` (formatting, typo)
2. **Tanya, jangan perintah**: *"Apakah sengaja pakai `late` di sini?*" lebih baik dari *"Hapus `late`"*
3. **Approve kalau sudah bersih** — jangan nunggu "perfect"
4. **Pair review** untuk PR kompleks — review bareng 15 menit lebih cepat dari async 3 hari

---

## Otomatisasi (Biar Hidup Lebih Mudah)

| Tool | Fungsi |
|------|--------|
| **`very_good_analysis`** | Lint rule ketat (include `prefer_const_constructors`, `avoid_print`, dll) |
| **`dcmm` (Dart Code Metrics)** | Deteksi complexity, unused code, long method — config di `analysis_options.yaml` |
| **`lefthook` / `husky`** | Pre-commit hook: `flutter analyze`, `dart format --set-exit-if-changed .` |
| **GitHub Actions** | Required status checks: analyze, test, build — block merge kalau merah |
| **Dependabot / Renovate** | Auto-update dependency + PR otomatis |

---

## Kesimpulan

Code review checklist itu **living document** — evolusi seiring tim berkembang. Mulai dari yang simpel (10 item), tambahin seiring butuh. Yang penting: **konsisten dipakai**.

Sampai di sini seri **Flutter 90 Hari** selesai! 🎉

> **Coba sendiri!** Share ke sosial media dan tag **@ahsai001**

---

*Artikel ini bagian dari seri **Flutter 90 Hari** — dari pemula hingga mahir. Baca artikel sebelumnya: [Flutter #69: Security & Enkripsi](/blog/2026/11/14/flutter-69-security-encryption/)*