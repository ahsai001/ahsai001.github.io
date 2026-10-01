---
layout: post
title: "Flutter #47: Isolate & Concurrency"
date: 2026-10-02 07:00:00 +0700
tags: [flutter, dart, isolate, concurrency, advanced, indonesia]
---

## Kenapa Butuh Isolate?

Dart bersifat **single-threaded**. Event loop menjalankan microtask dan macrotask secara berurutan. Komputasi berat (parsing JSON besar, enkripsi, image processing) akan memblokir UI → aplikasi *jank* / *lag*.

**Isolate** = thread terpisah dengan *memory heap* sendiri. Tidak berbagi memori → *no race condition*. Komunikasi lewat **message passing** (port).

---

## Isolate Dasar: `Isolate.spawn`

```dart
import 'dart:isolate';

void heavyTask(SendPort sendPort) {
  // Simulasi komputasi berat
  int sum = 0;
  for (int i = 0; i < 100_000_000; i++) {
    sum += i;
  }
  sendPort.send(sum); // kirim hasil ke main isolate
}

Future<void> main() async {
  final receivePort = ReceivePort();
  
  // Spawn isolate baru, kirim sendPort-nya
  await Isolate.spawn(heavyTask, receivePort.sendPort);
  
  // Terima hasil
  final result = await receivePort.first;
  print('Hasil dari isolate: $result'); // 4999999950000000
  
  receivePort.close();
}
```

> **Catatan:** Fungsi entry point (`heavyTask`) harus **top-level** atau `static`, bukan closure/lambda.

---

## Pola Request-Response dengan `Completer`

Agar API bersifat async/await:

```dart
Future<int> computeHeavy() {
  final receivePort = ReceivePort();
  final completer = Completer<int>();
  
  Isolate.spawn((SendPort sendPort) {
    int sum = 0;
    for (int i = 0; i < 50_000_000; i++) sum += i;
    sendPort.send(sum);
  }, receivePort.sendPort);
  
  receivePort.listen((message) {
    completer.complete(message as int);
    receivePort.close();
  });
  
  return completer.future;
}

void main() async {
  print('Mulai...');
  final result = await computeHeavy();
  print('Selesai: $result');
}
```

---

## `compute()` Helper (Flutter)

Flutter menyediakan shortcut untuk *one-off* heavy compute:

```dart
import 'package:flutter/foundation.dart';

int fibonacci(int n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

void main() async {
  // Jalan di isolate terpisah otomatis
  final result = await compute(fibonacci, 40);
  print('Fibonacci(40) = $result'); // 102334155
}
```

**Keterbatasan `compute`:**
- Hanya 1 argumen (bisa bungkus ke object/record)
- Tidak cocok untuk isolate *long-running* (streaming, background sync)

---

## Isolate Long-Running: Dua Arah Komunikasi

Butuh `ReceivePort` di **kedua** sisi:

```dart
// main.dart
void main() async {
  final mainReceivePort = ReceivePort();
  
  // Spawn worker
  final isolate = await Isolate.spawn<SendPort>(
    workerEntry, 
    mainReceivePort.sendPort,
  );
  
  // Kirim port worker ke main
  final workerSendPort = await mainReceivePort.first as SendPort;
  
  // Kirim task ke worker
  final replyPort = ReceivePort();
  workerSendPort.send({'task': 'process', 'data': [1,2,3,4,5], 'replyTo': replyPort.sendPort});
  
  final result = await replyPort.first;
  print('Hasil worker: $result');
  
  // Shutdown
  workerSendPort.send({'task': 'shutdown'});
  isolate.kill(priority: Isolate.immediate);
}

void workerEntry(SendPort mainSendPort) {
  final workerReceivePort = ReceivePort();
  mainSendPort.send(workerReceivePort.sendPort); // kirim port worker ke main
  
  workerReceivePort.listen((message) {
    final msg = message as Map;
    if (msg['task'] == 'process') {
      final List<int> data = msg['data'];
      final sum = data.reduce((a, b) => a + b);
      (msg['replyTo'] as SendPort).send(sum);
    } else if (msg['task'] == 'shutdown') {
      workerReceivePort.close();
      Isolate.exit(workerReceivePort.sendPort, 'done');
    }
  });
}
```

---

## Best Practices

| Tips | Penjelasan |
|------|------------|
| **Gunakan `compute` untuk tugas pendek** | Less boilerplate, auto cleanup |
| **Hindari kirim object besar** | Serialisasi/deserialisasi mahal; gunakan `SendPort` + `ByteData` / `Uint8List` |
| **Isolate = biaya startup** | ~1-2ms + memory. Jangan spawn di frame render loop |
| **Gunakan `IsolateNameServer` untuk named port** | Bisa akses isolate dari mana saja via nama string |
| **Error handling** | `Isolate.spawn` return `Future<Isolate>`; tangani `Isolate.exit` & `addOnExitListener` |

---

## Contoh Nyata: Parsing JSON Besar di Background

```dart
import 'dart:convert';
import 'package:flutter/foundation.dart';

List<Post> parsePosts(String jsonString) {
  final parsed = json.decode(jsonString) as List;
  return parsed.map((e) => Post.fromJson(e)).toList();
}

class Post {
  final int id;
  final String title;
  final String body;
  Post({required this.id, required this.title, required this.body});
  factory Post.fromJson(Map<String, dynamic> j) => Post(
    id: j['id'], title: j['title'], body: j['body']
  );
}

Future<List<Post>> fetchAndParsePosts() async {
  // Simulasi fetch JSON besar (mis. 10k item)
  final jsonString = await rootBundle.loadString('assets/posts.json');
  // Parse di isolate
  return compute(parsePosts, jsonString);
}
```

---

## Ringkasan

| Konsep | Kapan Dipakai |
|--------|---------------|
| `Isolate.spawn` | Kontrol penuh, long-running, two-way |
| `compute()` | One-off heavy function, simpel |
| `IsolateNameServer` | Akses isolate global by name |
| `ReceivePort` / `SendPort` | Message passing (single direction) |

Isolate adalah kunci **60 fps UI** saat aplikasi butuh komputasi berat. Gunakan bijak — jangan over-engineer untuk tugas ringan.

---

> **Coba sendiri!** Buat project Flutter, taruh JSON 50k item di `assets/`, bandingkan `json.decode` di main thread vs `compute`. Ukur dengan `flutter run --profile` + DevTools Timeline. Share ke sosial media dan tag @ahsai001.