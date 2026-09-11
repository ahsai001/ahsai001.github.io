---
layout: post
title: "Flutter #34: Error Handling Best Practice"
date: 2026-09-12 07:00:00 +0700
tags: [flutter, dart, error-handling, debugging, best-practice]
description: "Pelajari cara menangani error di Flutter dengan benar: try-catch, custom exception, centralized error handling, sampai menampilkan pesan error yang ramah ke user."
---

Halo Developer! 👋

Di artikel sebelumnya kita sudah belajar **Dependency Injection** dengan `get_it` supaya kode lebih modular dan mudah di-test. Sekarang kita bahas topik yang sering dianggap remeh padahal paling menentukan kualitas app: **Error Handling**.

App yang bagus bukan yang tidak pernah error — tapi yang **tidak crash diam-diam** dan memberi feedback yang jelas saat ada yang salah. Salah handling, error yang tadinya kecil bisa bikin app force close di tangan user. Yuk kita bedah cara yang benar! 🔧

---

## Kenapa Error Handling Itu Penting?

Coba bayangkan: user lagi asyik checkout, tiba-tiba internet putus. Kalau kode kamu cuma:

```dart
final response = await http.get(uri);
```

...maka app langsung crash dengan `SocketException` di layar. User tidak mengerti kenapa, dan yang paling parah — **kamu sebagai developer juga tidak tahu** karena errornya tidak tercatat di mana pun.

Error handling yang baik punya 3 tujuan:

1. **App tidak crash** — error ditangkap, bukan dibiarkan merambat ke root
2. **User paham situasinya** — muncul pesan yang jelas ("Koneksi bermasalah, coba lagi")
3. **Developer bisa debug** — error tercatat lengkap dengan stack trace

---

## Dasar: try-catch di Dart

Ini fondasinya. Bungkus operasi yang berpotensi gagal, tangkap errornya:

```dart
Future<void> fetchData() async {
  try {
    final response = await http.get(Uri.parse('https://api.example.com/data'));
    
    if (response.statusCode != 200) {
      throw Exception('Gagal memuat data: HTTP ${response.statusCode}');
    }
    
    // Proses response...
    print('Data berhasil dimuat ✅');
  } catch (e) {
    print('Terjadi error: $e');
  } finally {
    // Selalu dieksekusi, mau error atau sukses
    print('Operasi selesai');
  }
}
```

Tiga blok kunci:

- **`try`** — berisi kode yang berpotensi error
- **`catch (e)`** — menangkap error yang terjadi. Bisa tambah `catch (e, stackTrace)` kalau mau ambil stack trace-nya
- **`finally`** — selalu jalan, cocok untuk cleanup (tutup koneksi, hide loading)

> ⚠️ **Pitfall klasik:** menangkap error lalu diam saja (`catch (e) {}`) itu **lebih bahaya** daripada tidak pakai try-catch sama sekali — karena error jadi tidak terlihat di mana pun. Kalau kamu tidak bisa berbuat apa-apa dengan errornya, setidaknya log!

---

## Jangan Tangkap Semua dengan Satu Cara

Dart punya `on` untuk menangkap **tipe error tertentu**:

```dart
try {
  final data = await repository.getData();
  // ...
} on TimeoutException {
  // Khusus timeout — beda penanganan
  showSnackBar('Server terlalu lama merespons. Coba lagi.');
} on SocketException {
  // Khusus masalah koneksi
  showSnackBar('Tidak ada koneksi internet.');
} catch (e) {
  // Error lain yang tidak terduga — fallback
  showSnackBar('Terjadi kesalahan yang tidak diketahui.');
}
```

Pesan error yang **spesifik sesuai penyebabnya** jauh lebih membantu user daripada satu pesan generik untuk semua kasus.

---

## Buat Custom Exception Sendiri

`Exception` generik itu kurang informatif. Di project yang serius, buat exception class sendiri:

```dart
class ApiException implements Exception {
  final String message;
  final int? statusCode;

  ApiException(this.message, {this.statusCode});

  @override
  String toString() => 'ApiException: $message (HTTP $statusCode)';
}

class NetworkException implements Exception {
  final String message;
  NetworkException(this.message);

  @override
  String toString() => 'NetworkException: $message';
}

class CacheException implements Exception {
  final String message;
  CacheException(this.message);

  @override
  String toString() => 'CacheException: $message';
}
```

Sekarang di layer repository, kamu bisa **memetakan error bawaannya** ke exception yang lebih bermakna:

```dart
class UserRepository {
  final ApiClient _api;

  UserRepository(this._api);

  Future<List<User>> getUsers() async {
    try {
      final response = await _api.get('/users');
      return response.data;
    } on SocketException {
      throw NetworkException('Tidak bisa terhubung ke server');
    } on TimeoutException {
      throw NetworkException('Waktu permintaan habis');
    } on ApiException catch (e) {
      // Sudah exception API, teruskan saja
      rethrow;
    } catch (e) {
      throw ApiException('Error tidak terduga: $e');
    }
  }
}
```

Perhatikan `rethrow` — itu cara Dart untuk **melempar ulang** exception yang sudah ditangkap tanpa kehilangan stack trace aslinya. Jangan pakai `throw e;` karena stack trace-nya jadi terpotong.

Dengan pola ini, **UI tidak pernah melihat `SocketException`** — yang dilihat hanya `NetworkException` dan `ApiException` yang sudah jelas artinya.

---

## Pola di UI: State-based + SnackBar

Di Flutter, jangan tampilkan error mentah (`Text('Error: $e')`). Gabungkan dengan state management yang sudah kita pelajari — misalnya Cubit dari artikel #29:

```dart
class HomeCubit extends Cubit<HomeState> {
  final UserRepository _repository;

  HomeCubit(this._repository) : super(HomeInitial());

  Future<void> loadHome() async {
    emit(HomeLoading());
    try {
      final data = await _repository.getHomeData();
      emit(HomeLoaded(data));
    } on NetworkException catch (e) {
      emit(HomeError(e.message));
    } on ApiException catch (e) {
      emit(HomeError('Server error: ${e.message}'));
    } catch (e) {
      emit(HomeError('Terjadi kesalahan tak terduga'));
    }
  }
}
```

Dan di widget, tampilkan UI yang sesuai state-nya:

```dart
class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocBuilder<HomeCubit, HomeState>(
      builder: (context, state) {
        if (state is HomeLoading) {
          return const Center(child: CircularProgressIndicator());
        }
        if (state is HomeError) {
          return Center(
            child: Column(
              mainAxisSize: MainAxisSize.min,
              children: [
                const Icon(Icons.cloud_off, size: 48),
                const SizedBox(height: 12),
                Text(state.message, textAlign: TextAlign.center),
                const SizedBox(height: 16),
                ElevatedButton(
                  onPressed: () => context.read<HomeCubit>().loadHome(),
                  child: const Text('Coba Lagi'),
                ),
              ],
            ),
          );
        }
        final data = (state as HomeLoaded).data;
        return ListView.builder(
          itemCount: data.length,
          itemBuilder: (_, i) => ListTile(title: Text(data[i].title)),
        );
      },
    );
  }
}
```

Lihat polanya: **loading → error (dengan tombol retry) → sukses**. User tidak pernah stuck di layar error tanpa jalan keluar. Tombol "Coba Lagi" itu penting — error handling yang baik selalu memberi opsi pemulihan.

Untuk aksi kecil (misal tombol submit), pakai SnackBar sebagai feedback:

```dart
onPressed: () async {
  try {
    await _authService.login(email, password);
    if (!context.mounted) return;
    Navigator.pushReplacementNamed(context, '/home');
  } on NetworkException catch (e) {
    if (!context.mounted) return;
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text('⚠️ ${e.message}')),
    );
  }
}
```

> 💡 Selalu cek `context.mounted` setelah `await` sebelum pakai `context` — ini mencegah error "deactivated widget" yang sering muncul setelah refactor async.

---

## Tangkap Error Global sebagai Jaring Pengaman

Meskipun semua layer sudah di-handle, tetap ada error yang luput (misal error di widget build). Pasang **jaring pengaman global** di `main()`:

```dart
import 'package:flutter/foundation.dart';

void main() {
  FlutterError.onError = (details) {
    // Error dari framework Flutter (build, layout, dll)
    FlutterError.presentError(details);
    if (kDebugMode) {
      debugPrint(details.toString());
    } else {
      // Kirim ke service monitoring (Firebase Crashlytics, Sentry, dll)
      Crashlytics.instance.recordFlutterError(details);
    }
  };

  PlatformDispatcher.instance.onError = (error, stack) {
    // Error dari luar framework (isolate utama)
    Crashlytics.instance.recordError(error, stack);
    return true; // true = sudah ditangani, app tidak di-kill
  };

  runApp(const MyApp());
}
```

Pola `kDebugMode` di atas penting: di development tampilkan detail selengkat-lengkapnya, di production kirim ke **Crashlytics/Sentry** supaya kamu bisa lihat semua error dari user. Kombinasi local handling + global net + monitoring = error handling lengkap.

---

## Checklist Error Handling

Supaya gampang diingat, ini checklist singkat:

| Aturan | Contoh |
|--------|--------|
| Bungkus operasi yang berpotensi gagal | try-catch di repository/API layer |
| Tangkap error sesuai tipe | `on SocketException`, `on TimeoutException` |
| Buat exception yang bermakna | `NetworkException`, `ApiException` |
| Jangan tangkap lalu diam | Minimal `debugPrint` atau log |
| Tampilkan pesan ramah ke user | SnackBar / state error dengan tombol retry |
| Pasang global error handler | `FlutterError.onError` |
| Kirim error ke monitoring | Crashlytics / Sentry di production |
| Cek `context.mounted` setelah await | Sebelum pakai `context` |

---

## Kesimpulan

Error handling bukan sekadar `try-catch` — ini **pola pikir** tentang bagaimana app kamu bereaksi saat terjadi kegagalan:

1. **Tangkap di layer yang tepat** — repository tahu soal network, UI tidak perlu tahu
2. **Beri konteks** — custom exception dengan pesan yang jelas
3. **User experience matters** — pesan ramah + tombol retry, bukan teks error mentah
4. **Jangan buta** — log dan monitoring supaya error di production terlihat

Di artikel selanjutnya kita akan masuk ke **Unit Testing** — cara otomatis memverifikasi bahwa kode kamu, termasuk logic error handling ini, bekerja dengan benar.

---

Sekarang coba audit project Flutter kamu: cari `catch (e) {}` kosong, ganti pesan error mentah dengan custom exception, dan tambahkan tombol retry di layar error. Perubahan kecil ini yang membedakan app amatir dan profesional. 💪

**Coba sendiri!** Refactor satu halaman di project kamu dengan pola error handling di atas, lalu share ke sosial media dan tag **@ahsai001**! 🚀