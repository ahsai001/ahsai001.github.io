---
title: "Flutter #50: WebSocket Chat – Membuat Aplikasi Chat Real‑Time"
date: 2026-10-09 08:00:00 +0800
categories: [flutter]
tags: [flutter, dart, websocket, chat, realtime]
---

## Pendahuluan
WebSocket memungkinkan komunikasi dua arah antara klien dan server tanpa harus melakukan *polling* berulang. Pada tutorial ini kita buat **chat sederhana** yang menampilkan pesan secara real‑time menggunakan paket `web_socket_channel`.

## Persiapan
1. Tambahkan dependensi di `pubspec.yaml`:
```yaml
dependencies:
  flutter:
    sdk: flutter
  web_socket_channel: ^3.0.0
```
2. Jalankan `flutter pub get`.

## Membuat Server Echo
Untuk demonstrasi gunakan server **echo** sederhana dengan `dart:io`. Simpan di folder `tool/echo_server.dart`.
```dart
import 'dart:io';
import 'dart:convert';

Future<void> main() async {
  final server = await HttpServer.bind(InternetAddress.loopbackIPv4, 4040);
  print('WebSocket echo listening on ws://localhost:4040');
  await for (var request in server) {
    if (WebSocketTransformer.isUpgradeRequest(request)) {
      final ws = await WebSocketTransformer.upgrade(request);
      ws.listen((msg) => ws.add(msg)); // echo kembali
    } else {
      request.response
        ..statusCode = HttpStatus.notFound
        ..close();
    }
  }
}
```
Jalankan dengan `dart tool/echo_server.dart`. Server akan meng‑echo pesan yang diterima.

## UI Flutter
Buat widget `ChatPage` yang terhubung ke WebSocket.
```dart
import 'package:flutter/material.dart';
import 'package:web_socket_channel/web_socket_channel.dart';

class ChatPage extends StatefulWidget {
  const ChatPage({Key? key}) : super(key: key);
  @override
  _ChatPageState createState() => _ChatPageState();
}

class _ChatPageState extends State<ChatPage> {
  final _channel = WebSocketChannel.connect(
    Uri.parse('ws://10.0.2.2:4040'), // Android emulator localhost
  );
  final _controller = TextEditingController();
  final List<String> _messages = [];

  @override
  void initState() {
    super.initState();
    _channel.stream.listen((msg) {
      setState(() => _messages.add(msg as String));
    });
  }

  @override
  void dispose() {
    _channel.sink.close();
    _controller.dispose();
    super.dispose();
  }

  void _send() {
    if (_controller.text.isNotEmpty) {
      _channel.sink.add(_controller.text);
      _controller.clear();
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('WebSocket Chat')),
      body: Column(
        children: [
          Expanded(
            child: ListView.builder(
              itemCount: _messages.length,
              itemBuilder: (_, i) => ListTile(title: Text(_messages[i])),
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(8.0),
            child: Row(
              children: [
                Expanded(
                  child: TextField(
                    controller: _controller,
                    decoration: const InputDecoration(hintText: 'Ketik pesan...'),
                    onSubmitted: (_) => _send(),
                  ),
                ),
                IconButton(icon: const Icon(Icons.send), onPressed: _send),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```
Tambahkan route di `main.dart`:
```dart
routes: {'/chat': (_) => const ChatPage()},
```
Buka halaman `/chat` pada emulator atau perangkat.

## Uji Coba
1. Jalankan server `dart tool/echo_server.dart`.
2. Jalankan aplikasi Flutter (`flutter run`).
3. Buka dua emulator/device, masuk ke `/chat`.
4. Kirim pesan, lihat muncul di kedua sisi.

## Penanganan Error & Reconnect
Untuk produksi gunakan paket `flutter_websocket` atau tambahkan logika **retry** saat koneksi terputus.

## Ringkasan
Dengan `web_socket_channel` Anda dapat menambahkan fitur chat real‑time tanpa kerumitan. Ide ini dapat dikembangkan menjadi grup chat, notifikasi, atau integrasi dengan backend seperti **Firebase Realtime Database**.

---
Coba sendiri! Share ke sosial media dan tag @ahsai001
