---
layout: post
title: "Flutter #56: Performance Optimization"
date: 2026-10-19 07:00:00 +0700
categories: [flutter]
tags: [flutter, dart, tutorial, indonesia]
---

Flutter sudah terkenal cepat karena rendering engine Skia/Impeller yang bypass native UI. Tapi aplikasi kompleks tetap bisa *jank* (drop frame) kalau gak diperhatikan. Hari ini kita bahas teknik optimasi performa yang wajib diketahui developer Flutter level expert.

## 1. Pahami Frame Budget

Target 60 FPS = 16.67 ms per frame. Target 120 FPS (device modern) = 8.33 ms. Semua work (layout, paint, compositing, Dart execution) harus selesai dalam budget ini.

Gunakan **Performance Overlay** untuk profiling:
```dart
// main.dart
MaterialApp(
  showPerformanceOverlay: true, // Aktifkan di debug
  // ...
);
```

Atau via DevTools: **Flutter Inspector → Performance Overlay**.

## 2. Const Constructor & Const Widgets

Gunakan `const` sebisa mungkin. Flutter bisa *reuse* widget instance, skip rebuild, dan skip diffing.

```dart
// ❌ Bad: Rebuild setiap frame
Widget build(BuildContext context) {
  return Container(
    color: Colors.blue,
    child: Text('Hello'),
  );
}

// ✅ Good: Const constructor
const MyWidget = Container(
  color: Colors.blue,
  child: Text('Hello'),
);

// ✅ Good: Const di tree
Widget build(BuildContext context) {
  return const Column(
    children: [
      Text('Static'),
      SizedBox(height: 16),
      Text('Tetap const'),
    ],
  );
}
```

**Aturan:** Kalau widget & semua children immutable → `const`. Kalau ada `setState` / `Provider` / `Bloc` yang berubah → gak bisa const.

## 3. ListView.builder vs ListView

`ListView` bikin semua children sekaligus (O(n) memory). `ListView.builder` lazy-build hanya item yg visible (O(1) memory).

```dart
// ✅ Good untuk list panjang
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) => ItemWidget(item: items[index]),
);

// ✅ Tambahan: cacheExtent untuk preload sebentar sebelum visible
ListView.builder(
  cacheExtent: 500, // pixel
  itemCount: items.length,
  itemBuilder: (context, index) => ItemWidget(item: items[index]),
);
```

## 4. RepaintBoundary — Isolasi Repaint

Widget yang sering berubah (animasi, scroll) jangan bikin parent repaint. Bungkus dengan `RepaintBoundary`.

```dart
// Parent gak repaint saat child berubah
RepaintBoundary(
  child: AnimatedContainer(
    duration: Duration(milliseconds: 300),
    color: isActive ? Colors.red : Colors.blue,
    child: const Text('Animasi'),
  ),
);
```

**Tip:** `ListView` children otomatis dapet `RepaintBoundary` (via `RenderSliverList`). Tapi custom scroll / complex tree butuh manual.

## 5. Opacity & Clipping — Mahal

`Opacity` dan `ClipRect`/`ClipRRect` trigger **offscreen layer** (saveLayer) → GPU cost tinggi.

```dart
// ❌ Bad: Opacity di widget besar
Opacity(
  opacity: 0.5,
  child: LargeComplexWidget(),
);

// ✅ Good: Ubah color/alpha langsung
Container(
  color: Colors.blue.withOpacity(0.5),
  child: LargeComplexWidget(),
);

// ✅ Good: FadeInImage / AnimatedOpacity (sudah optimized)
AnimatedOpacity(
  opacity: visible ? 1.0 : 0.0,
  duration: Duration(milliseconds: 300),
  child: ChildWidget(),
);
```

## 6. Image Caching & Resize

Gambar besar di-resize di runtime = CPU & memory boros. Gunakan `cacheWidth`/`cacheHeight` atau `resizeImage`.

```dart
// ✅ Resize di decode (memory efficient)
Image.network(
  url,
  cacheWidth: 300, // decode ke 300px width
  cacheHeight: 200,
);

// ✅ Atau pakai resizeImage untuk asset
Image.asset(
  'assets/large.png',
  width: 100,
  height: 100,
  // Flutter auto downscale kalau width/height < intrinsic
);
```

**Pro tip:** Gunakan `precacheImage` di `initState` untuk preload hero image.

```dart
@override
void initState() {
  super.initState();
  precacheImage(NetworkImage(heroUrl), context);
}
```

## 7. Build Methods Ringan

Jangan taruh logic berat di `build`. Pindah ke `compute` (isolate) atau `Future.microtask`.

```dart
// ❌ Bad: Heavy computation di build
Widget build(BuildContext context) {
  final sorted = heavySort(items); // Block UI thread
  return ListView.builder(...);
}

// ✅ Good: Compute di isolate
Future<List<Item>> _sortInBackground(List<Item> items) async {
  return compute(heavySort, items);
}

Widget build(BuildContext context) {
  return FutureBuilder(
    future: _sortInBackground(items),
    builder: (context, snapshot) {
      if (!snapshot.hasData) return const CircularProgressIndicator();
      return ListView.builder(...);
    },
  );
}
```

## 8. Keys — Preserve State

Pakai `Key` yang stabil (bukan `UniqueKey` di list) agar Flutter reuse element tree.

```dart
// ❌ Bad: Key berubah tiap rebuild
ListView.builder(
  itemBuilder: (_, i) => ItemWidget(key: UniqueKey(), item: items[i]),
);

// ✅ Good: Key berdasarkan ID stabil
ListView.builder(
  itemBuilder: (_, i) => ItemWidget(key: ValueKey(items[i].id), item: items[i]),
);
```

## 9. Profile Mode — Bukan Debug/Release

Debug mode: asserts, checks, unoptimized. Release: full optimize tapi gak bisa profiling. **Profile mode** = optimized + profiling tools.

```bash
flutter run --profile
# atau
flutter build apk --profile
```

Buka DevTools → **Timeline** → rekam scroll/interaksi → cari *red bars* (frame > 16ms).

## 10. Impeller & Shader Compilation

Flutter 3.16+ pakai **Impeller** (iOS default, Android opt-in). Impeller compile shader di build time (AOT), gak di runtime → *zero jank* pertama kali.

```yaml
# android/app/build.gradle (Android)
flutter {
  // ...
  experimental-impeller = true
}
```

**Warm-up shader** (pre-Impeller / fallback):
```dart
// main.dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Future.wait([
    // Prewarm routes
    precacheImage(AssetImage('assets/bg.png'), context),
    // ...
  ]);
  runApp(MyApp());
}
```

## Checklist Cepat Sebelum Release

| Area | Check |
|------|-------|
| Const widgets | ✅ Semua static widget `const` |
| List뷰 | ✅ `ListView.builder` + `cacheExtent` |
| Repaint | ✅ `RepaintBoundary` di animasi/scroll |
| Opacity | ✅ Hindari `Opacity` widget besar |
| Images | ✅ `cacheWidth`/`cacheHeight` / `precacheImage` |
| Build | ✅ Logic berat ke `compute` / isolate |
| Keys | ✅ `ValueKey(id)` stabil |
| Profiling | ✅ Test di `--profile` mode, DevTools Timeline |
| Impeller | ✅ Aktifkan di Android (`experimental-impeller`) |

## Kesimpulan

Performa Flutter bukan sihir — bisa diukur & dioptimasi. Pola paling impact:
1. **Const everywhere** — skip rebuild & diff
2. **Lazy lists** — `ListView.builder`
3. **Isolate heavy work** — `compute()`
4. **Profile mode + DevTools Timeline** — temukan bottleneck real, bukan dugaan

Coba sendiri! Share ke sosial media dan tag @ahsai001