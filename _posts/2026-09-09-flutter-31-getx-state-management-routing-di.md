---
layout: post
title: "Flutter #31: GetX — State Management, Routing & Dependency Injection"
date: 2026-09-09 07:00:00 +0700
tags: [flutter, dart, tutorial, getx, state-management, routing, dependency-injection, indonesia]
description: "GetX: state management super ringkas, navigasi tanpa context, dan dependency injection built-in. Semua dalam satu package!"
---

# Flutter #31: GetX — State Management, Routing & Dependency Injection

Halo! Di dua artikel terakhir kita belajar **BLoC & Cubit** — state management yang powerful tapi butuh banyak boilerplate. Sekarang kita lihat sisi yang berlawanan: **GetX**.

GetX adalah package all-in-one yang populer di komunitas Flutter. Philosophy-nya: **semua harus simpel**. State management? Tinggal tambah `.obs`. Navigasi? Gak perlu context. Dependency injection? Tinggal `Get.put()`. Semua built-in, gak perlu install package tambahan.

Ada pro dan kontra tentunya — kita bahas semua. Let's go!

---

## 1. Kenapa GetX?

| Masalah | BLoC/Provider | GetX |
|---------|--------------|------|
| Boilerplate | Banyak file (Event, State, Bloc) | Satu file, syntax ringkas |
| Navigasi | Butuk `Navigator` + `context` | `Get.to(() => Page())` tanpa context |
| Dependency Injection | Butuh `get_it` atau manual | `Get.put()` / `Get.find()` built-in |
| Learning curve | Sedang-tinggi | Rendah untuk basic usage |

**When to use GetX:**
- Proyek kecil-menengah yang butuh rapid development
- Tim yang mau fokus fitur, bukan arsitektur
- Prototyping MVP dengan cepat

**When to skip:**
- Proyek besar yang butuh testability & clean architecture
- Tim yang sudah nyaman dengan BLoC
- Project yang butuh pakai Riverpod/Provider karena alasan tertentu

---

## 2. Setup GetX

Tambahkan ke `pubspec.yaml`:

```yaml
dependencies:
  get: ^4.6.6
```

Lalu jalankan:

```bash
flutter pub get
```

Selesai! Satu package, banyak fitur. Tidak perlu install paket terpisah untuk routing atau DI.

---

## 3. Reactive State Management — `.obs`

Ini cara paling simpel di GetX. Tambah `.obs` di belakang variabel:

```dart
import 'package:get/get.dart';

class CounterController extends GetxController {
  // Tambah .obs → jadi reactive
  var count = 0.obs;
  var name = 'Flutter Developer'.obs;

  void increment() => count.value++;
  void decrement() => count.value--;
  void reset() => count.value = 0;

  // Getter reaktif — otomatis update kalau count berubah
  String get statusText => count.value == 0
      ? 'Belum ada aktivitas'
      : 'Total: ${count.value}';
}
```

**Yang perlu diperhatikan:**
- `count` sekarang adalah `RxInt`, bukan `int` biasa
- Akses value pakai `.value` — `count.value`
- `GetX` otomatis deteksi perubahan dan rebuild UI

---

## 4. Reactive UI — `Obx`

`Obx` adalah widget yang **otomatis rebuild** kalau ada variabel `.obs` yang berubah:

```dart
class CounterPage extends StatelessWidget {
  final controller = Get.put(CounterController());

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('GetX Counter')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Obx otomatis rebuild saat count berubah
            Obx(() => Text(
              '${controller.count.value}',
              style: const TextStyle(
                fontSize: 72,
                fontWeight: FontWeight.bold,
              ),
            )),
            const SizedBox(height: 8),
            // Ini juga reactive — statusText bergantung pada count
            Obx(() => Text(
              controller.statusText,
              style: const TextStyle(fontSize: 16, color: Colors.grey),
            )),
            const SizedBox(height: 32),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                ElevatedButton(
                  onPressed: controller.decrement,
                  child: const Icon(Icons.remove),
                ),
                const SizedBox(width: 16),
                ElevatedButton(
                  onPressed: controller.increment,
                  child: const Icon(Icons.add),
                ),
                const SizedBox(width: 16),
                OutlinedButton(
                  onPressed: controller.reset,
                  child: const Text('Reset'),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

**Kelebihan Obx:**
- Gak perlu specify type — `Obx(() => ...)` otomatis tahu
- Granular rebuild — cuma widget di dalam `Obx` yang rebuild
- Tidak perlu `BlocBuilder`, `Consumer`, atau widget wrapper lain

---

## 5. Simple State Management — `GetBuilder`

Kalau gak mau pakai `.obs` dan lebih suka pola imperative, ada `GetBuilder`:

```dart
class CounterController extends GetxController {
  int count = 0; // Tidak pakai .obs!

  void increment() {
    count++;
    update(); // Manual trigger rebuild
  }

  void decrement() {
    count--;
    update();
  }

  void reset() {
    count = 0;
    update();
  }
}
```

UI-nya pakai `GetBuilder`:

```dart
GetBuilder<CounterController>(
  init: CounterController(), // Controller baru
  builder: (c) => Text(
    '${c.count}',
    style: const TextStyle(fontSize: 72),
  ),
)
```

**Kapan pakai `GetBuilder` vs `Obx`?**

| Aspek | `Obx` + `.obs` | `GetBuilder` |
|-------|----------------|--------------|
| Syntax | `count.value` | `count` biasa |
| Update trigger | Otomatis | Manual (`update()`) |
| Performance | Sangat granular | Lebih kasar |
| Controls | Tidak perlu | `update(['id'])` untuk partial rebuild |

`Obx` lebih modern dan direkomendasikan untuk sebagian besar kasus.

---

## 6. Navigasi Tanpa Context — `Get.to()` & `Get.off()`

Ini salah satu fitur paling populer. Navigasi tanpa perlu `Navigator.of(context)`:

```dart
// Push biasa (ada tombol back)
Get.to(() => DetailPage());

// Push & hapus halaman sebelumnya
Get.off(() => HomePage());

// Push & hapus SEMUA halaman sebelumnya
Get.offAll(() => LoginPage());

// Navigasi dengan named route
Get.toNamed('/detail/123');

// Back biasa
Get.back();

// Replace current route
Get.offNamed('/home');
```

**Contoh lengkap:**

```dart
// Halaman daftar
class UserListPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Daftar User')),
      body: ListView(
        children: [
          ListTile(
            title: const Text('Ahmad'),
            subtitle: const Text('Flutter Developer'),
            onTap: () {
              // Push ke DetailPage — gak perlu context!
              Get.to(() => const DetailPage(
                name: 'Ahmad',
                role: 'Flutter Developer',
              ));
            },
          ),
          ListTile(
            title: const Text('Siti'),
            subtitle: const Text('Backend Developer'),
            onTap: () => Get.to(() => const DetailPage(
              name: 'Siti',
              role: 'Backend Developer',
            )),
          ),
        ],
      ),
    );
  }
}
```

**Kelebihan navigasi GetX:**
- Gak perlu `BuildContext` — bisa navigate dari controller/service
- Syntax ringkas — satu baris
- Built-in transition animation: `Get.to(() => Page(), transition: Transition.cupertino)`
- SnackBar & dialog juga tanpa context: `Get.snackbar(...)`, `Get.defaultDialog(...)`

---

## 7. Routing dengan Named Routes

Untuk proyek yang lebih besar, pakai named routes:

```dart
// routes.dart
abstract class AppRoutes {
  static const home = '/home';
  static const counter = '/counter';
  static const detail = '/detail';
  static const login = '/login';
}

// Binding — daftarkan controller per route
class HomeBinding extends Bindings {
  @override
  void dependencies() {
    Get.lazyPut(() => CounterController());
  }
}

// main.dart
void main() {
  runApp(GetMaterialApp(
    initialRoute: AppRoutes.home,
    getPages: [
      GetPage(
        name: AppRoutes.home,
        page: () => const HomePage(),
        binding: HomeBinding(),
        transition: Transition.fade,
      ),
      GetPage(
        name: AppRoutes.counter,
        page: () => const CounterPage(),
        transition: Transition.leftToRight,
      ),
      GetPage(
        name: AppRoutes.detail,
        page: () => const DetailPage(),
      ),
    ],
  ));
}
```

Navigasi tinggal panggil:

```dart
// Dari mana saja — controller, service, widget
Get.toNamed('/detail/123');
Get.toNamed('/counter', arguments: {'from': 'home'});
```

**`GetPage` setup:**
- `name` — path route (bisa parameter: `/detail/:id`)
- `page` — widget yang ditampilkan
- `binding` — controller yang di-load saat route aktif
- `transition` — animasi transisi

---

## 8. Dependency Injection — `Get.put()`, `Get.lazyPut()`, `Get.find()`

GetX punya DI built-in, gak perlu `get_it`:

### `Get.put()` — Langsung buat & simpan instance

```dart
// Controller langsung dibuat saat ini
final controller = Get.put(CounterController());

// Atau di dalam controller lain
class AuthController extends GetxController {
  // Simpan UserService ke dalam GetX DI
  final UserService _userService = Get.put(UserService());
}
```

### `Get.lazyPut()` — Buat instance saat pertama kali dipanggil

```dart
// TIDAK langsung dibuat — tunggu sampai pertama kali find
Get.lazyPut(() => ApiClient());
Get.lazyPut(() => UserRepository());
Get.lazyPut(() => AuthController());
```

### `Get.find()` — Cari instance yang sudah ada

```dart
// Cari controller yang sudah di-put
final controller = Get.find<CounterController>();
controller.increment();
```

### `Get.delete()` — Hapus instance

```dart
// Hapus dari memory (misal saat logout)
Get.delete<AuthController>();
```

---

## 9. Contoh Nyata: Auth Flow dengan GetX

Ini contoh lengkap bagaimana GetX handle real-world scenario:

```dart
// === auth_service.dart ===
class AuthService extends GetxService {
  final isLoggedIn = false.obs;
  final user = Rxn<Map<String, dynamic>>();

  Future<bool> login(String email, String password) async {
    try {
      // Simulasi API call
      await Future.delayed(const Duration(seconds: 2));

      // Simulasi berhasil
      isLoggedIn.value = true;
      user.value = {
        'name': 'Ahmad Saifullah',
        'email': email,
        'role': 'Developer',
      };
      return true;
    } catch (e) {
      return false;
    }
  }

  void logout() {
    isLoggedIn.value = false;
    user.value = null;
    Get.offAllNamed('/login'); // Logout & balik ke login
  }
}
```

**Inisialisasi di `main.dart`:**

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  // Daftarkan AuthService — akan live selama app running
  await Get.putAsync(() => AuthService().init());

  runApp(GetMaterialApp(
    initialRoute: '/login',
    getPages: [
      GetPage(name: '/login', page: () => const LoginPage()),
      GetPage(name: '/home', page: () => const HomePage()),
    ],
  ));
}
```

**Login page:**

```dart
class LoginPage extends StatelessWidget {
  const LoginPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text('Login', style: TextStyle(fontSize: 32)),
            const SizedBox(height: 24),
            const TextField(
              decoration: InputDecoration(
                labelText: 'Email',
                border: OutlineInputBorder(),
              ),
            ),
            const SizedBox(height: 16),
            const TextField(
              obscureText: true,
              decoration: InputDecoration(
                labelText: 'Password',
                border: OutlineInputBorder(),
              ),
            ),
            const SizedBox(height: 24),
            ElevatedButton(
              onPressed: () async {
                final auth = Get.find<AuthService>();
                final success = await auth.login('test@email.com', '123');

                if (success) {
                  Get.offAllNamed('/home');
                  Get.snackbar(
                    'Berhasil!',
                    'Selamat datang kembali! 🎉',
                    snackPosition: SnackPosition.BOTTOM,
                  );
                } else {
                  Get.snackbar(
                    'Gagal',
                    'Email atau password salah',
                    snackPosition: SnackPosition.BOTTOM,
                  );
                }
              },
              child: const Text('Masuk'),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Semua tanpa `Navigator.push`, tanpa `Provider.of`, tanpa `BlocProvider`.** GetX handle semuanya dari dalam.

---

## 10. GetX vs BLoC vs Provider — Perbandingan Final

| Aspek | Provider | BLoC | GetX |
|-------|----------|------|------|
| Boilerplate | Sedikit | Banyak | Sedikit |
| Testability | Baik | Sangat baik | Cukup (hati-hati global) |
| Navigasi | `Navigator` | `Navigator` | `Get.to()` built-in |
| DI | Manual | Manual | Built-in |
| Community | Sangat besar | Sangat besar | Besar tapi polarizing |
| Best untuk | Medium-large | Large enterprise | Small-medium / MVP |

**Pendapat jujur:** GetX itu *productive banget* untuk MVP dan proyek kecil. Tapi untuk tim besar yang butuh testability ketat, BLoC atau Provider lebih safe. Yang penting: **konsisten**. Pilih satu, kuasai, apply dengan konsisten di proyek.

---

## Kesimpulan

Hari ini kita belajar:

1. **`.obs`** — variabel reaktif yang auto-detect perubahan
2. **`Obx`** — widget yang auto-rebuild saat variabel `.obs` berubah
3. **`GetBuilder`** — alternatif simple state management dengan `update()`
4. **Navigasi tanpa context** — `Get.to()`, `Get.off()`, `Get.offAll()`
5. **Named Routes** — `GetPage` + `getPages` untuk routing terstruktur
6. **Dependency Injection** — `Get.put()`, `Get.lazyPut()`, `Get.find()`
7. **`GetxService`** — service yang live selama app running

Di artikel #32 kita lanjut ke **Repository Pattern & Clean Architecture** — bagaimana mengorganisasi kode supaya scalable dan mudah di-maintain. See you! 🚀

---

**Coba sendiri!** Bikin aplikasi sederhana dengan 3 halaman (Login → Home → Profile), pakai GetX untuk state management, navigasi, dan dependency injection. Share hasilnya ke sosial media dan tag **@ahsai001** 💪
