---
layout: post
title: "Flutter #13: MediaQuery & Orientation Builder — Bikin Layout Responsif"
date: 2026-08-15 07:00:00 +0700
tags: [flutter, dart, mediaquery, orientation, responsive, tutorial, indonesia, pemrograman]
---

Kemarin kita udah bikin app dengan tema & font custom yang rapi. Tapi ada satu masalah yang bakal kamu temui cepat atau lambat: **app kamu jalan di layar yang ukurannya beda-beda**. HP kecil 5 inci, tablet 10 inci, HP lipat, bahkan desktop. Kalau layout kamu ngotot pakai ukuran fixed (misal lebar card `300` pixel), di layar kecil dia bakal overflow, di layar gede dia keliatan sepi.

Solusinya: **desain responsif**. Flutter ngasih dua senjata utama buat ini — **`MediaQuery`** (baca info layar) dan **`OrientationBuilder`** (rebuild otomatis pas layar diputar). Hari ini kita bedah keduanya sampai bisa.

---

## 1. Apa Itu `MediaQuery`?

`MediaQuery` itu kayak "kartu identitas" layar device. Dia nyimpen semua informasi tentang layar & device user: lebar, tinggi, orientasi, text scale, padding (notch/status bar), dan banyak lagi.

Cara bacanya:

```dart
final media = MediaQuery.of(context);

media.size.width;      // lebar layar dalam logical pixel
media.size.height;     // tinggi layar
media.orientation;     // Orientation.portrait / landscape
media.textScaler;      // faktor zoom teks user
media.padding.top;     // tinggi status bar / notch
media.devicePixelRatio; // rasio pixel fisik : logical
```

> ⚠️ **Logical pixel, bukan pixel fisik.** Layar 1080x2400 dengan `devicePixelRatio` 3.0 punya ukuran logical 360x800. Selalu pakai logical pixel di layout, biar konsisten di semua device.

---

## 2. Baca Ukuran Layar — `MediaQuery.sizeOf`

Flutter punya shortcut yang lebih hemat performa: **`MediaQuery.sizeOf(context)`**. Ini cuma ngambil ukuran, tanpa bikin widget jadi dependen ke *semua* properti MediaQuery (jadi lebih sedikit rebuild).

Contoh — bikin card yang lebarnya **selalu 90% dari lebar layar**:

```dart
final size = MediaQuery.sizeOf(context);

Card(
  child: Container(
    width: size.width * 0.9,   // 90% lebar layar
    height: size.height * 0.3, // 30% tinggi layar
    color: Colors.teal.shade50,
    child: const Center(child: Text('Responsif!')),
  ),
)
```

Dengan pola ini, card otomatis menyesuaikan di HP kecil maupun tablet gede. Nggak ada lagi angka `300` yang kaku.

---

## 3. Deteksi Orientasi

`MediaQuery.orientation` ngasih tau layar lagi portrait atau landscape:

```dart
final isPortrait =
    MediaQuery.orientationOf(context) == Orientation.portrait;
```

Tapi ada cara yang lebih Flutter-idiomatic buat nanganin perubahan orientasi: **`OrientationBuilder`**.

---

## 4. `OrientationBuilder` — Rebuild Otomatis Saat Layar Diputar

`OrientationBuilder` bakal nge-*rebuild* widget di dalamnya **setiap kali orientasi berubah**. Kamu nggak perlu dengerin event manual atau panggil `setState`.

```dart
OrientationBuilder(
  builder: (context, orientation) {
    if (orientation == Orientation.portrait) {
      // Susun vertikal — Column
      return const Column(
        children: [
          Icon(Icons.phone_iphone, size: 64),
          Text('Mode Portrait'),
        ],
      );
    } else {
      // Susun horizontal — Row
      return const Row(
        children: [
          Icon(Icons.phone_iphone, size: 64),
          SizedBox(width: 16),
          Text('Mode Landscape'),
        ],
      );
    }
  },
)
```

Pas user muter HP, `OrientationBuilder` otomatis ganti layout dari `Column` ke `Row`. Clean banget, tanpa listener manual.

---

## 5. Breakpoint — Pola Responsif yang Paling Dipakai

Di dunia nyata, kita nggak cuma bedain portrait/landscape. Kita bedain **kategori ukuran layar** pakai *breakpoint*: HP, tablet, desktop. Kombinasikan `MediaQuery.sizeOf` dengan logika sederhana:

```dart
class ResponsiveLayout extends StatelessWidget {
  final Widget mobile;
  final Widget tablet;
  final Widget desktop;
  const ResponsiveLayout({
    super.key,
    required this.mobile,
    required this.tablet,
    required this.desktop,
  });

  // Breakpoint umum
  static const double tabletBreakpoint = 600;
  static const double desktopBreakpoint = 1100;

  static bool isMobile(BuildContext context) =>
      MediaQuery.sizeOf(context).width < tabletBreakpoint;

  static bool isTablet(BuildContext context) =>
      MediaQuery.sizeOf(context).width >= tabletBreakpoint &&
      MediaQuery.sizeOf(context).width < desktopBreakpoint;

  static bool isDesktop(BuildContext context) =>
      MediaQuery.sizeOf(context).width >= desktopBreakpoint;

  @override
  Widget build(BuildContext context) {
    final width = MediaQuery.sizeOf(context).width;
    if (width >= desktopBreakpoint) return desktop;
    if (width >= tabletBreakpoint) return tablet;
    return mobile;
  }
}
```

Sekarang kamu bisa bikin halaman yang tampil beda di tiap device:

```dart
ResponsiveLayout(
  mobile: const _MobileHome(),    // daftar vertikal 1 kolom
  tablet: const _TabletHome(),    // grid 2 kolom
  desktop: const _DesktopHome(),  // grid 4 kolom + sidebar
)
```

Ini fondasi dari hampir semua app profesional. Sekali bikin, dipakai di semua halaman.

---

## 6. Jangan Lupa `SafeArea`

Ini kesalahan klasik pemula: konten ketutupan status bar / notch. `SafeArea` otomatis nge-*push* konten keluar dari area berbahaya:

```dart
Scaffold(
  body: SafeArea(
    child: Column(
      children: const [
        Text('Konten aman di sini'),
        // nggak ketutup notch / status bar
      ],
    ),
  ),
)
```

`SafeArea` juga punya parameter `minimum` kalau kamu mau kasih jarak ekstra (misal `EdgeInsets.all(16)`).

---

## 7. Contoh Lengkap: Dashboard Responsif

Gabungin semuanya — deteksi orientasi + breakpoint + SafeArea:

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MaterialApp(home: DashboardScreen()));

class DashboardScreen extends StatelessWidget {
  const DashboardScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('📱 Dashboard Responsif')),
      body: SafeArea(
        child: OrientationBuilder(
          builder: (context, orientation) {
            final isPortrait = orientation == Orientation.portrait;
            // Di landscape, kita pindahkan info ke samping (Row)
            // Di portrait, semuanya tersusun vertikal (Column)
            final items = [
              const _StatCard(label: 'Pengguna', value: '1.2K'),
              const _StatCard(label: 'Order', value: '348'),
              const _StatCard(label: 'Pendapatan', value: 'Rp 5,4jt'),
            ];

            if (isPortrait) {
              return Column(children: items);
            } else {
              return Row(
                children: items
                    .map((c) => Expanded(child: c))
                    .toList(),
              );
            }
          },
        ),
      ),
    );
  }
}

class _StatCard extends StatelessWidget {
  final String label;
  final String value;
  const _StatCard({required this.label, required this.value});

  @override
  Widget build(BuildContext context) {
    // Card menyesuaikan lebar parent-nya
    return Card(
      margin: const EdgeInsets.all(8),
      child: Padding(
        padding: const EdgeInsets.all(20),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Text(
              value,
              style: const TextStyle(
                fontSize: 28,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 8),
            Text(label),
          ],
        ),
      ),
    );
  }
}
```

**Jalankan dan coba:** muter HP kamu (atau pakai emulator rotate). Dashboard otomatis berubah dari tumpukan vertikal jadi baris horizontal. Kalau kamu tes di window desktop lalu resize, breakpoint-nya juga bisa diterapin pakai `ResponsiveLayout` di atas.

---

## 8. Best Practice

- **Pakai `MediaQuery.sizeOf` / `orientationOf`** daripada `MediaQuery.of` — lebih hemat rebuild karena cuma subscribe ke properti yang dibutuhkan.
- **Hindari ukuran fixed** (`width: 300`) untuk elemen utama. Pakai persentase, `Expanded`, `Flexible`, atau breakpoint.
- **`OrientationBuilder` untuk perubahan orientasi**, `ResponsiveLayout` (breakpoint) untuk perbedaan device. Keduanya saling melengkapi.
- **Selalu bungkus konten dengan `SafeArea`** di halaman yang menyentuh tepi layar.
- **Test di banyak ukuran** — Flutter DevTools punya fitur *device preview* buat simulasi berbagai layar tanpa HP fisik.
- **Jangan hardcode `MediaQuery.padding`** manual kalau bisa pakai `SafeArea` — lebih bersih dan anti-error.

---

## Ringkasan

- **`MediaQuery`** = sumber info layar (ukuran, orientasi, padding, text scale)
- **`MediaQuery.sizeOf(context)`** = baca ukuran layar secara efisien
- **`OrientationBuilder`** = rebuild otomatis saat orientasi berubah
- **Breakpoint** = pola klasik membedakan mobile/tablet/desktop
- **`SafeArea`** = lindungi konten dari notch & status bar
- **Responsif = jangan pakai ukuran fixed, pakai persentase & breakpoint**

Layout responsif adalah pembeda antara app "jadi" dan app **profesional**. User nggak bakal mikir "kok layoutnya ancur di HP gue?" — dan itu justru tanda kerja bagus.

Besok kita tutup Fase Fundamental dengan **Debugging & Flutter DevTools** — belajar nemuin bug kayak detektif. Stay tuned! 🐛

**Coba sendiri! Bikin dashboard sederhana yang berubah layout saat HP diputar, lalu test di beberapa ukuran layar. Share hasilnya ke sosial media dan tag @ahsai001!**
