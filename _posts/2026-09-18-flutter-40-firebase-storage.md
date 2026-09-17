---
layout: post
title: "Flutter #40: Firebase Storage"
date: 2026-09-18 07:00:00 +0700
tags: [flutter, dart, firebase, storage, tutorial]
---

# Flutter #40: Firebase Storage

Halo Flutter Developer! 🚀

Di artikel #38 kita sudah belajar Firestore untuk nyimpen data teks, dan #39 kita bikin authentication. Tapi pertanyaan besarnya: **file-nya ditaruh di mana?** Foto profil, dokumen PDF, video tutorial — semua itu nggak cocok disimpan sebagai string di database. Nah, hari ini kita pakai **Firebase Storage**: tempat nyimpen file di cloud, lengkap dengan upload progress, download URL, dan security rules.

## Kenapa Nggak Simpan File di Firestore?

Firestore punya batas dokumen **1 MB**. Sebuah foto dari kamera HP saja sudah 2–5 MB. Kalau kamu paksakan simpan dalam bentuk Base64, kamu bukan cuma melanggar batas, tapi juga:

- Boros bandwidth (Base64 bikin ukuran naik ~33%)
- Bikin query lambat karena setiap read ikut menarik seluruh file
- Biaya baca membengkak (Firestore menagih per dokumen yang dibaca)

**Aturan emasnya:** simpan *metadata* di Firestore, simpan *file*-nya di Storage. Yang kamu simpan di Firestore cuma URL-nya.

```dart
// ✅ Pola yang benar
// Firestore: { nama: 'Ahmad', fotoUrl: 'https://firebasestorage.googleapis.com/...' }
// Storage:   users/abc123/profile.jpg
```

## Setup Awal

Tambahkan package-nya:

```yaml
# pubspec.yaml
dependencies:
  firebase_core: ^3.8.1
  firebase_storage: ^12.3.7
  image_picker: ^1.1.2
```

```bash
flutter pub get
```

Buka **Firebase Console → Storage → Get Started**, pilih lokasi bucket (pilih `asia-southeast2` untuk Indonesia, lebih dekat = lebih cepat). Selesai. Secara default rules-nya masih mode test — nanti kita perbaiki di bagian akhir.

## Upload File dengan Progress

Ini inti hari ini. Kuncinya adalah `UploadTask`, yang bisa kita dengerin progresnya seperti Stream:

```dart
import 'dart:io';
import 'package:firebase_storage/firebase_storage.dart';

class StorageService {
  final _storage = FirebaseStorage.instance;

  /// Upload file dan kembalikan download URL-nya.
  Future<String?> uploadFile({
    required File file,
    required String userId,
    void Function(double progress)? onProgress,
  }) async {
    try {
      // Path unik: user/{uid}/{timestamp}_{namafile}
      final ext = file.path.split('.').last;
      final filename = '${DateTime.now().millisecondsSinceEpoch}.$ext';
      final ref = _storage.ref('users/$userId/$filename');

      final task = ref.putFile(
        file,
        SettableMetadata(
          contentType: 'image/$ext',
          customMetadata: {'uploadedBy': userId},
        ),
      );

      // Dengarkan progress upload
      task.snapshotEvents.listen((snapshot) {
        final progress = snapshot.bytesTransferred / snapshot.totalBytes;
        onProgress?.call(progress);
      });

      final snapshot = await task;
      return await snapshot.ref.getDownloadURL();
    } on FirebaseException catch (e) {
      print('Upload gagal [${e.code}]: ${e.message}');
      return null;
    }
  }
}
```

Perhatikan `snapshotEvents`. Setiap kali ada byte terkirim, kita dapat notifikasi — ini yang bikin progress bar di UI kamu bergerak real-time.

## Integrasi dengan Image Picker + UI

Sekarang kita rangkai semuanya jadi satu halaman yang bisa dipakai:

```dart
import 'dart:io';
import 'package:flutter/material.dart';
import 'package:image_picker/image_picker.dart';

class UploadFotoPage extends StatefulWidget {
  final String userId;
  const UploadFotoPage({super.key, required this.userId});

  @override
  State<UploadFotoPage> createState() => _UploadFotoPageState();
}

class _UploadFotoPageState extends State<UploadFotoPage> {
  final _service = StorageService();
  final _picker = ImagePicker();

  File? _file;
  double _progress = 0;
  bool _uploading = false;
  String? _downloadUrl;

  Future<void> _pilihGambar() async {
    final picked = await _picker.pickImage(
      source: ImageSource.gallery,
      imageQuality: 75, // kompres biar upload cepat
      maxWidth: 1200,
    );
    if (picked != null) {
      setState(() {
        _file = File(picked.path);
        _downloadUrl = null;
      });
    }
  }

  Future<void> _upload() async {
    if (_file == null) return;
    setState(() {
      _uploading = true;
      _progress = 0;
    });

    final url = await _service.uploadFile(
      file: _file!,
      userId: widget.userId,
      onProgress: (p) => setState(() => _progress = p),
    );

    setState(() {
      _uploading = false;
      _downloadUrl = url;
    });

    if (url != null) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('Upload berhasil! 🎉')),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Upload Foto Profil')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            if (_file != null)
              ClipRRect(
                borderRadius: BorderRadius.circular(12),
                child: Image.file(_file!, height: 200, fit: BoxFit.cover),
              ),
            const SizedBox(height: 16),
            ElevatedButton.icon(
              onPressed: _uploading ? null : _pilihGambar,
              icon: const Icon(Icons.photo_library),
              label: const Text('Pilih Gambar'),
            ),
            const SizedBox(height: 8),
            if (_uploading) ...[
              LinearProgressIndicator(value: _progress),
              const SizedBox(height: 4),
              Text('${(_progress * 100).toStringAsFixed(0)}%'),
            ],
            if (!_uploading && _file != null)
              ElevatedButton(
                onPressed: _upload,
                child: const Text('Upload Sekarang'),
              ),
            if (_downloadUrl != null)
              Padding(
                padding: const EdgeInsets.only(top: 16),
                child: SelectableText('URL: $_downloadUrl'),
              ),
          ],
        ),
      ),
    );
  }
}
```

Kode ini runnable penuh — cukup ganti `widget.userId` dengan UID dari Firebase Auth artikel kemarin. Setelah upload selesai, simpan `_downloadUrl` ke Firestore:

```dart
await FirebaseFirestore.instance
    .collection('users')
    .doc(widget.userId)
    .update({'fotoUrl': url});
```

## Download & Caching Gambar

Untuk menampilkan gambar dari Storage, **jangan** pakai `Image.network` biasa. Pakai `cached_network_image` supaya gambar nggak di-download ulang setiap kali halaman dibuka:

```yaml
dependencies:
  cached_network_image: ^3.4.1
```

```dart
CachedNetworkImage(
  imageUrl: user.fotoUrl,
  placeholder: (_, __) => const CircularProgressIndicator(),
  errorWidget: (_, __, ___) => const Icon(Icons.broken_image),
)
```

Kalau butuh baca file sebagai binary (misal PDF atau video), pakai `getData()`:

```dart
final bytes = await FirebaseStorage.instance
    .ref('dokumen/laporan.pdf')
    .getData(10 * 1024 * 1024); // batasi 10 MB
```

Selalu beri `maxSize` di `getData()` — ini mencegah app kamu crash karena kehabisan memori saat file besar.

## Hapus File

Hapus file sama seperti akses path-nya. Tapi ingat: kalau kamu upload file baru dan lupa hapus yang lama, storage kamu akan menumpuk file yatim (orphan file) yang tetap dihitung biaya:

```dart
Future<void> hapusFile(String pathGoogleStorage) async {
  try {
    await FirebaseStorage.instance.refFromURL(pathGoogleStorage).delete();
  } on FirebaseException catch (e) {
    if (e.code == 'object-not-found') return; // sudah terhapus, aman
    rethrow;
  }
}
```

Tips produksi: setiap kali user ganti foto profil, panggil `hapusFile(user.fotoUrlLama)` **setelah** upload baru sukses.

## Security Rules — Wajib!

Default rules Firebase Storage yang masih mode test akan kedaluwarsa dalam 30 hari, dan lebih bahaya lagi: siapa pun bisa upload apa pun. Ganti dengan ini:

```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    // Foto profil: hanya pemilik yang bisa tulis
    match /users/{userId}/{allPaths=**} {
      allow read: if true;
      allow write: if request.auth != null
                   && request.auth.uid == userId
                   && request.resource.size < 5 * 1024 * 1024
                   && request.resource.contentType.matches('image/.*');
    }
  }
}
```

Tiga hal penting di rules ini:

1. **`request.auth.uid == userId`** — user hanya bisa nulis ke folder miliknya sendiri. Ini pertahanan utama.
2. **`size < 5 MB`** — batasi ukuran di sisi server. Validasi di client saja tidak cukup, orang bisa hit API langsung.
3. **`contentType.matches('image/.*')`** — tolak file yang bukan gambar. Tanpa ini, seseorang bisa upload script atau file executable ke bucket kamu.

## Kesalahan Umum

**Upload jalan terus tanpa progress.** Biasanya karena `snapshotEvents` dilisten *setelah* task selesai. Pastikan listen dulu, baru `await task`.

**Download URL selalu sama setelah update.** Karena nama file-nya sama. Storage melakukan caching di CDN. Solusinya pakai nama unik (timestamp) seperti di kode kita, atau tambahkan query token.

**`permission-denied` padahal rules kelihatan benar.** Cek ulang apakah user benar-benar login (`FirebaseAuth.instance.currentUser`) dan path-nya cocok persis dengan pola di rules.

**File numpuk dan biaya naik.** Firebase Storage menagih per GB tersimpan + per GB keluar. Bersihkan file yatim, dan kompres gambar di client (`imageQuality: 75`) sebelum upload.

## Kesimpulan

Sekarang kamu punya pipeline lengkap: **Auth** (siapa user-nya) → **Storage** (upload file-nya) → **Firestore** (simpan URL-nya). Ini adalah tulang punggung hampir semua aplikasi modern.

Poin penting hari ini:

- Simpan **metadata di Firestore**, **file di Storage**
- Gunakan `UploadTask.snapshotEvents` untuk progress real-time
- Kompres gambar sebelum upload — hemat bandwidth dan biaya
- **Security rules bukan opsional** — validasi tipe, ukuran, dan kepemilikan di server
- Pakai `cached_network_image`, jangan `Image.network` mentah

Minggu depan kita lanjut ke **#41: GitHub Actions CI/CD** — bikin app kamu build dan test otomatis setiap kali push. Sampai jumpa!

Coba sendiri! Share ke sosial media dan tag @ahsai001 🚀
