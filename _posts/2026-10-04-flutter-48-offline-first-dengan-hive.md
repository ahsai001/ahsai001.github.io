---
layout: post
title: "Flutter #48: Offline‑First dengan Hive"
date: 2026-10-04 08:00:00 +0700
tags: [flutter, hive, offline, database, dart]
---

## Pendahuluan
Aplikasi mobile sering dihadapkan pada **koneksi tidak stabil**. Pengguna berharap data tetap dapat diakses meski offline. Di Flutter, paket **Hive** menyediakan basis data NoSQL ringan, cepat, dan sepenuhnya **offline‑first**.

## Kenapa Hive?
- **Performansi**: Baca/tulis dalam milidetik, tanpa JNI.
- **Ringkas**: Tidak butuh SQLite schema, cukup tipe primitive atau model yang di‑serialize.
- **Cross‑platform**: Web, Android, iOS, macOS.
- **Enkripsi**: Opsional, cocok untuk data sensitif.

## Instalasi
```bash
flutter pub add hive hive_flutter
flutter pub add build_runner hive_generator --dev
```
> `hive_flutter` meng‑inisialisasi Hive pada direktori aplikasi.

## Buat Model
Gunakan **type adapters** agar Hive mengerti objek Dart.
```dart
import 'package:hive/hive.dart';

part 'note.g.dart';

@HiveType(typeId: 0)
class Note extends HiveObject {
  @HiveField(0)
  String title;

  @HiveField(1)
  String content;

  @HiveField(2)
  DateTime createdAt;

  Note({required this.title, required this.content, DateTime? createdAt})
      : this.createdAt = createdAt ?? DateTime.now();
}
```
Jalankan generator:
```bash
flutter packages pub run build_runner build --delete-conflicting-outputs
```
File `note.g.dart` otomatis dibuat.

## Inisialisasi Hive
```dart
import 'package:hive_flutter/hive_flutter.dart';
import 'note.dart';

Future<void> main() async {
  await Hive.initFlutter();
  Hive.registerAdapter(NoteAdapter());
  await Hive.openBox<Note>('notes');
  runApp(MyApp());
}
```
Box `notes` akan menyimpan semua catatan.

## CRUD Praktis
```dart
final box = Hive.box<Note>('notes');

// CREATE
await box.add(Note(title: 'Halo', content: 'Ini contoh note'));

// READ ALL
final allNotes = box.values.toList();

// UPDATE (index based)
final note = box.getAt(0);
note?.title = 'Ubah Judul';
await note?.save();

// DELETE
await box.deleteAt(0);
```
Semua operasi **lokal**, tidak memerlukan jaringan.

## Sinkronisasi dengan Server (Opsional)
Saat koneksi tersedia, kumpulkan perubahan dengan flag `isSynced`. Kirim ke API, lalu set flag `true`. Contoh sederhana:
```dart
Future<void> syncNotes() async {
  final unsynced = box.values.where((n) => !(n as dynamic).isSynced);
  for (var note in unsynced) {
    await upload(note);
    (note as dynamic).isSynced = true;
    await note.save();
  }
}
```
Anda dapat memanggil `syncNotes` pada `ConnectivityResult.mobile` atau `wifi`.

## Enkripsi (Jika Data Sensitif)
```dart
await Hive.initFlutter('path/to/app');
await Hive.openBox<Note>('secureNotes', encryptionCipher: HiveAesCipher(utf8.encode('my‑secret‑key')));
```
Kunci harus **32‑byte** untuk AES‑256.

## Penutup
Dengan Hive, aplikasi Anda tetap **responsif** meski jaringan terputus. Simpan data penting secara lokal, sinkronkan ketika jaringan kembali. Eksperimen dengan **lazy boxes**, **watchers**, dan **binary adapters** untuk performa maksimum.

**Coba sendiri! Share ke sosial media dan tag @ahsai001**
