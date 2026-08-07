---
layout: post
title: "Flutter #5: StatelessWidget vs StatefulWidget & setState"
date: 2026-08-07 07:00:00 +0700
tags: [flutter, dart, tutorial, indonesia, pemrograman, state, statefulwidget]
---

Halo, teman-teman! Setelah kemarin kita ngulik widget dasar — `Text`, `Container`, `Row`, `Column` — sekarang saatnya masuk ke konsep yang bikin aplikasi Flutter jadi **hidup**: State! Kamu pasti pernah liat tombol yang berubah warna pas diklik, counter yang naik angkanya, atau form yang nampilin error pas input kosong. Semua itu berkat **State Management**. Dan pondasi pertamanya: `StatelessWidget` vs `StatefulWidget`.

---

## Apa Itu "State"?

Sederhananya, **state** adalah "keadaan saat ini" dari sebuah widget. Bayangin kamu punya lampu — state-nya bisa `hidup` atau `mati`. Begitu juga di Flutter:

- Teks yang berubah setelah user ngetik → state
- Counter yang bertambah setelah tombol ditekan → state
- Checkbox yang dicentang → state
- Halaman yang berpindah → state

Nah, Flutter membagi widget menjadi dua jenis berdasarkan apakah mereka punya state yang bisa berubah:

| Tipe Widget | State Berubah? | Contoh |
|-------------|---------------|--------|
| `StatelessWidget` | ❌ Tidak | Text, Icon, Image (statis) |
| `StatefulWidget` | ✅ Ya | Checkbox, Slider, TextField, Counter |

---

## StatelessWidget — Widget yang "Beku"

`StatelessWidget` adalah widget yang **gak bisa berubah** setelah dibuat. Begitu dia di-build, semua propertinya final — kayak foto yang sudah dijepret.

**Kapan pakai StatelessWidget?** Kalau UI kamu cuma bergantung pada data yang dikasih dari luar dan gak akan berubah sendiri.

Contoh sederhana: widget ucapan selamat datang.

```dart
import 'package:flutter/material.dart';

class WelcomeCard extends StatelessWidget {
  final String username;

  const WelcomeCard({Key? key, required this.username}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(20),
      margin: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: Colors.indigo.shade50,
        borderRadius: BorderRadius.circular(12),
      ),
      child: Row(
        children: [
          Icon(Icons.waving_hand, color: Colors.amber.shade700, size: 32),
          const SizedBox(width: 12),
          Text(
            'Halo, $username! Selamat datang kembali.',
            style: const TextStyle(fontSize: 18),
          ),
        ],
      ),
    );
  }
}
```

Widget ini nerima `username` dari constructor, lalu langsung render. Gak ada yang berubah sendiri. `build()` cuma dipanggil sekali saat widget dibuat.

**Ciri StatelessWidget:**
- Cuma punya method `build()`.
- Semua data masuk lewat constructor, bersifat `final`.
- Ringan, performanya bagus — gak ada overhead state.

---

## StatefulWidget — Widget yang "Hidup"

Nah, kalau `StatefulWidget` ini widget yang **state-nya bisa berubah seiring waktu**. FLutter mengawinkan `StatefulWidget` dengan class pasangannya: `State`. Jadi ada dua class:

1. **StatefulWidget** — immutable, cuma bungkus, nerima data dari luar.
2. **State\<T\>** — mutable, tempat logic dan data yang berubah berada.

Contoh paling klasik: **counter app**.

```dart
import 'package:flutter/material.dart';

class CounterScreen extends StatefulWidget {
  const CounterScreen({Key? key}) : super(key: key);

  @override
  State<CounterScreen> createState() => _CounterScreenState();
}

class _CounterScreenState extends State<CounterScreen> {
  int _counter = 0;

  void _increment() {
    setState(() {
      _counter++;
    });
  }

  void _decrement() {
    setState(() {
      if (_counter > 0) _counter--;
    });
  }

  void _reset() {
    setState(() {
      _counter = 0;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Belajar StatefulWidget'),
        centerTitle: true,
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(
              'Kamu sudah klik sebanyak:',
              style: TextStyle(fontSize: 18, color: Colors.grey.shade600),
            ),
            const SizedBox(height: 12),
            Text(
              '$_counter',
              style: const TextStyle(
                fontSize: 72,
                fontWeight: FontWeight.bold,
                color: Colors.indigo,
              ),
            ),
            const SizedBox(height: 32),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                ElevatedButton.icon(
                  onPressed: _decrement,
                  icon: const Icon(Icons.remove),
                  label: const Text('Kurang'),
                ),
                const SizedBox(width: 16),
                ElevatedButton.icon(
                  onPressed: _increment,
                  icon: const Icon(Icons.add),
                  label: const Text('Tambah'),
                ),
              ],
            ),
            const SizedBox(height: 12),
            TextButton(
              onPressed: _counter > 0 ? _reset : null,
              child: const Text('Reset'),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Magic-nya `setState()`

`setState()` adalah rahasia di balik kenapa UI Flutter bisa berubah. Begitu kamu panggil:

```dart
setState(() {
  _counter++; // ubah data di sini
});
```

Yang terjadi:
1. Flutter menjalankan callback di dalam `setState()` — jadi `_counter` bertambah.
2. Flutter otomatis memanggil ulang `build()`.
3. UI di-rebuild hanya dengan data terbaru.
4. Hasilnya: layar menampilkan angka counter yang baru!

**Tanpa `setState()`,** meskipun nilai `_counter` berubah di memori, Flutter gak akan tahu kalau UI perlu direfresh. Angka tetap seperti semula.

> ⚠️ **Jangan taruh logic berat di dalam `setState()`!** Isinya sebaiknya cuma assignment variabel ringan. Kalau kamu manggil API atau baca file di dalam `setState()`, UI bakal freeze.

---

## Lifecycle StatefulWidget (Penting!)

StatefulWidget punya siklus hidup yang perlu kamu kenali sejak awal:

```
createState()    →  initState()   →  build()
                                      ↓
                              setState() → build()
                                      ↓
                              dispose()
```

| Method | Kapan Dipanggil | Gunanya |
|--------|----------------|---------|
| `initState()` | Sekali, sebelum `build()` pertama | Inisialisasi data, listener, controller |
| `build()` | Setiap kali widget dirender | Membangun UI |
| `setState()` | Kamu panggil manual saat data berubah | Trigger rebuild UI |
| `dispose()` | Saat widget dihancurkan dari widget tree | Bersihin controller, listener, subscription |

Contoh dengan `initState` dan `dispose`:

```dart
class TimerWidget extends StatefulWidget {
  const TimerWidget({Key? key}) : super(key: key);

  @override
  State<TimerWidget> createState() => _TimerWidgetState();
}

class _TimerWidgetState extends State<TimerWidget> {
  int _seconds = 0;

  @override
  void initState() {
    super.initState();
    // Mulai timer pas widget pertama kali dibuat
    Future.delayed(const Duration(seconds: 1), _tick);
  }

  void _tick() {
    if (!mounted) return; // cek apakah widget masih ada
    setState(() => _seconds++);
    Future.delayed(const Duration(seconds: 1), _tick);
  }

  @override
  void dispose() {
    // Gak perlu apa-apa di sini karena pakai Future, bukan Timer
    // Tapi kalau pakai Timer.periodic, wajib cancel di sini!
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Text('Waktu: $_seconds detik');
  }
}
```

Pro tip: selalu panggil `super.initState()` di awal dan `super.dispose()` di akhir.

---

## Kapan Pakai yang Mana?

**Gunakan StatelessWidget kalau:**
- UI cuma menampilkan data yang gak berubah (label, logo, header statis)
- Widget cuma nge-wrap child tanpa logic internal

**Gunakan StatefulWidget kalau:**
- Ada data yang bisa berubah karena interaksi user (counter, form, checkbox)
- Ada animasi, timer, atau efek berbasis waktu
- Widget punya lifecycle event yang perlu di-handle (`initState`, `dispose`)

---

## Tips Biar Gak Salah Paham

1. **Stateless ≠ gak bisa update sama sekali.** Kalau parent-nya rebuild (misal lewat `setState` di parent), `StatelessWidget` juga ikut dirender ulang dengan data baru.
2. **Minimalkan StatefulWidget.** Makin banyak state, makin kompleks debugging-nya. Kalau bisa pakai `Stateless`, pakai aja.
3. **`setState()` bukan satu-satunya cara.** Nanti di artikel lanjutan kita bakal bahas Provider, BLoC, GetX — cara yang lebih scalable buat state management.

---

## Kesimpulan

Hari ini kita belajar:
- **State** = data yang bisa berubah di widget
- **StatelessWidget** = widget statis, gak bisa berubah sendiri
- **StatefulWidget** = widget dinamis, state-nya bisa diubah lewat `setState()`
- **setState()** = cara Flutter untuk rebuild UI saat data berubah
- **Lifecycle** = `initState` → `build` ↔ `setState` → `dispose`

Di artikel berikutnya, kita masuk ke **Layouting**: cara mengatur padding, margin, `SizedBox`, `Expanded`, dan `Flexible` supaya aplikasi kamu rapi di semua ukuran layar. Stay tuned!

---

Coba sendiri counter app di atas, tambahin fitur kayak multiply atau auto-increment, terus share ke sosial media dan tag [@ahsai001](https://ahsai.my.id)! Sampai jumpa besok. Happy coding! 🚀
