---
layout: post
title: "Flutter #21: StreamBuilder & Stream — Tangani Data Real-time dengan Mudah"
date: 2026-08-30 07:00:00 +0700
tags: [flutter, dart, streambuilder, stream, tutorial, indonesia, pemrograman]
---

Welcome to **Flutter #21**! 🎉

Di **Flutter #19** kita fetch data REST API dan tampilkan dalam `ListView`. Di **#20** kita mendalam ke `FutureBuilder` dan `AsyncSnapshot` — widget deklaratif yang menjaga UI bersih saat data asinkron datang dari `Future`. Tapi begitulah dunia nyata: bukan semua data datang dari `Future` sekadar sekali.

Beberapa data datang * terus — seperti notifikasi chat, sensor GPS, hasil validasi form user mengetik, atau stream dari server yang terus menerus mengirim event. Di sinilah **`StreamBuilder`** hadir, dan itulah yang kita bahas hari ini.

---

## 1. Apa itu Stream dan Kenapa Perlu StreamBuilder?

Bayangkan kasus berikut: kamu bikin chat aplikasi. Pengirim mengirim pesan, penerima terima. Tapi data ini nggak datang dari `Future` sekali — pesan bisa datang kapan saja, berulang kali, sampai user menutup aplikasi.

**`Stream`** adalah abstraksi untuk urutan event yang datang seiring waktu. Berbeda dengan `Future` yang:
- **Future**: satu nilai, datang sekali, lalu selesai.
- **Stream**: banyak nilai, datang terus-menerus (atau sampai dihentikan), bisa 0, 1, atau lebih event.

**`StreamBuilder`** adalah widget yang "mendengarkan" Stream dan membangun UI berdasarkan event-nya. Konsepnya mirip dengan `FutureBuilder`, tapi untuk aliran data terus-menerus.

---

## 2. Perbandingan Cepat: Future vs Stream

| Aspek | `Future<T>` | `Stream<T>` |
|---|---|---|
| **Jumlah nilai** | Satu nilai saja | Banyak nilai |
| **Waktu** | Datakali sekali, lalu selesai | Bisa nggak berakhir, bisa berhenti kapan saja |
| **Contoh** | Fetch data API sekali | Live GPS tracking, chat messages, tombol counter |
| **Widget** | `FutureBuilder` | `StreamBuilder` |

---

## 3. Konsep Dasar: StreamController

Sekali-kali kamu perlu menciptakan Stream sendiri. Tools utamanya adalah **`StreamController`**.

```dart
import 'dart:async';
import 'package:flutter/material.dart';

class CounterService {
  // Membuat controller yang akan mengontrol stream
  final _controller = StreamController<int>();

  // Method untuk menambah bilangan ke stream
  void addNumber(int number) {
    _controller.sink.add(number);
  }

  // Method untuk menghentikan stream
  void dispose() {
    _controller.close();
  }
}
```

Penting diingat: **`_controller.sink.add(number)`** itu yang memicu widget untuk rebuild. Setiap kali dipanggil, `StreamBuilder` akan dapat event baru dan update UI.

---

## 4. Implementasi: StreamBuilder Sederhana

Struktur `StreamBuilder` mirip dengan `FutureBuilder`:

```dart
StreamBuilder<int>(
  stream: counterService.stream,
  builder: (context, snapshot) {
    // State: menunggu (waiting)
    if (snapshot.connectionState == ConnectionState.waiting) {
      return const Center(child: CircularProgressIndicator());
    }

    // State: ada error
    if (snapshot.hasError) {
      return Center(child: Text('Error: ${snapshot.error}'));
    }

    // State: ada data
    if (!snapshot.hasData || snapshot.data == null) {
      return const Center(child: Text('Tidak ada data'));
    }

    // Data siap dipakai
    final count = snapshot.data!;
    return Text('Angka: $count', style: const TextStyle(fontSize: 24));
  },
),
```

Tiga state utama yang harus dicek:
1. **`ConnectionState.waiting`** — Stream baru mulai atau belum ada event
2. **`ConnectionState.done`** — Stream sudah tertutup (dispose)
3. **`ConnectionState.active`** — Stream sedang emit event (default untuk Stream non-stop)

---

## 5. Contoh Praktis: Real-time Counter dari Stream

Mari bangun aplikasi counter sederhana yang bisa di-increment dari Stream, bukan dari `setState` langsung.

```dart
import 'dart:async';
import 'package:flutter/material.dart';

// Service yang mengelola Stream-nya
class CounterService {
  final _controller = StreamController<int>.broadcast();
  int _counter = 0;

  // Tambah angka ke stream
  void increment() {
    _counter++;
    _controller.sink.add(_counter);
  }

  // Reset angka
  void reset() {
    _counter = 0;
    _controller.sink.add(_counter);
  }

  // Ambil stream untuk dipakai di widget
  Stream<int> get stream => _controller.stream;

  // Penting: tutup controller saat widget dispose
  void dispose() {
    _controller.close();
  }
}

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: const CounterScreen(),
    );
  }
}

class CounterScreen extends StatefulWidget {
  const CounterScreen({super.key});
  @override
  State<CounterScreen> createState() => _CounterScreenState();
}

class _CounterScreenState extends State<CounterScreen> {
  late CounterService _counterService;
  late StreamSubscription<int> _subscription;

  @override
  void initState() {
    super.initState();
    _counterService = CounterService();
    // Dengar stream perubahan
    _subscription = _counterService.stream.listen((count) {
      // Akumulasi di sini jika butuh, tapi kita pakai StreamBuilder
    });
  }

  @override
  void dispose() {
    _counterService.dispose();
    _subscription.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('StreamBuilder Counter')),
      body: Center(
        child: StreamBuilder<int>(
          stream: _counterService.stream,
          builder: (context, snapshot) {
            if (snapshot.connectionState == ConnectionState.waiting) {
              return const CircularProgressIndicator();
            }
            if (snapshot.hasError) {
              return Text('Error: ${snapshot.error}');
            }
            if (snapshot.hasData) {
              return Text(
                'Angka: ${snapshot.data}',
                style: const TextStyle(fontSize: 48, fontWeight: FontWeight.bold),
              );
            }
            return const Text('Tidak ada data');
          },
        ),
      ),
      floatingActionButton: Column(
        mainAxisAlignment: MainAxisAlignment.end,
        children: [
          FloatingActionButton(
            onPressed: () => _counterService.increment(),
            tooltip: 'Increment',
            child: const Icon(Icons.add),
          ),
          const SizedBox(height: 16),
          FloatingActionButton(
            onPressed: () => _counterService.reset(),
            tooltip: 'Reset',
            child: const Icon(Icons.refresh),
          ),
        ],
      ),
    );
  }
}
```

**Kunci utama:** Setiap kali `_counterService.increment()` dipanggil, `stream` akan emit event baru, dan `StreamBuilder` akan rebuild otomatis menampilkan angka terbaru. Kita nggak butuh `setState` sama sekali!

---

## 6. Contoh Lanjut: Stream dengan ListView

Stream juga bisa dipakai untuk menampilkan data yang terus bertambah, misalnya log aktivitas atau feed notifikasi yang datang dari server.

```dart
StreamBuilder<List<String>>(
  stream: notificationService.notifications,
  builder: (context, snapshot) {
    if (snapshot.connectionState == ConnectionState.waiting) {
      return const Center(child: CircularProgressIndicator());
    }
    if (snapshot.hasError) {
      return Center(child: Text('Error: ${snapshot.error}'));
    }
    final notifications = snapshot.data ?? [];
    
    return ListView.builder(
      itemCount: notifications.length,
      itemBuilder: (context, index) {
        final note = notifications[index];
        return ListTile(
          title: Text(note),
          leading: const Icon(Icons.notifications),
        );
      },
    );
  },
),
```

---

## 7. Best Practice: Jangan Lupa Dispose

Ini paling sering dilupakan pemula. Setiap kali membuat `StreamController`, wajib dipasang `dispose()` saat widget dihancurkan:

```dart
@override
void dispose() {
  _controller.close(); // Atau _controller.close() kalau sudah pakai sink
  super.dispose();
}
```

Kalau lupa, akan memory leak karena stream masih merasional event meski widget sudah hilang.

---

## 8. Common Pitfalls

### ❌ `setState` di dalam `builder` StreamBuilder

```dart
// JANGAN LAKUKAN INI
StreamBuilder(
  builder: (context, snapshot) {
    return ElevatedButton(
      onPressed: () => setState(() {}), // Anti-pattern!
      child: Text('Update'),
    );
  },
),
```

Kalau butuh interaksi di tombol, panggil method dari luar StreamBuilder, bukan pakai `setState` di dalam builder.

### ❌ Lupa batalkan subscription (`cancel()`)

```dart
// Jangan lupa di dispose()
@override
void dispose() {
  _subscription.cancel(); // Batalkan stream subscription
  super.dispose();
}
```

Kalau nggak dibatalkan, stream masih merujuk ke widget yang sudah dihancurkan.

### ❌ Buat StreamController di dalam `build()`

```dart
// JANGAN: Setiap build membikin controller baru
@override
Widget build(BuildContext context) {
  final _controller = StreamController<int>(); // BURU! 
  // Akan selalu membikin object baru tiap rebuild
}
```

Simpan `StreamController` di `initState()` atau di variabel state agar hanya dibuat sekali.

---

## 9. Ringkasan

- **`Stream`** = urutan event yang datang seiring waktu (beda `Future` yang sekadar satu nilai)
- **`StreamBuilder`** = widget deklaratif untuk mendengarkan Stream dan rebuild UI berdasarkan event-nya
- **Tiga state** — `waiting` (loading), `hasError` (error), `hasData` (data siap)
- **`StreamController`** = cara utama menciptakan dan mengontrol Stream
- **`dispose()`** = jangan lupa menutup controller untuk mencegah memory leak
- **`listen()`** vs **`StreamBuilder`** — `listen()` untuk logic sisi server, `StreamBuilder` untuk UI

StreamBuilder dan Stream adalah fondasi untuk fitur real-time di Flutter. Setelahnya kita bahas **State Management lanjutannya: BLoC pattern**, mana yang masih pakai Stream secara dalaman.

---

## Ringkasan Cepat: Perbedaan Utama

| Konsep | Future | Stream |
|---|---|---|
| **Nilai** | Satu kali | Banyak |
| **Waktu** | Satu titik | Seiring waktu |
| **Widget** | `FutureBuilder` | `StreamBuilder` |
| **Controller** | `Future` | `StreamController` |

---

**Coba sendiri!** Bikin counter real-time pake `StreamBuilder` seperti contoh di atas. Coba tambah fitur reset dan lihat bagaimana stream emit event-nya di UI. Cobain juga bikin `StreamBuilder` yang nampilin daftar notifikasi yang terus bertambah. Share hasilnya ke sosial media dan tag @ahsai001! 🚀