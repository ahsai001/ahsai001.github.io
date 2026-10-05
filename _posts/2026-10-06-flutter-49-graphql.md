---
layout: post
title: "Flutter #49: Mengintegrasikan GraphQL dengan Flutter"
date: 2026-10-06 08:00:00 +0700
tags: [flutter, graphql, dart, tutorial, indonesia]
---

# Flutter #49: Mengintegrasikan GraphQL dengan Flutter

GraphQL kini menjadi alternatif populer REST API karena fleksibilitasnya dalam memilih data yang dibutuhkan. Di artikel ini kita akan menyiapkan **GraphQL client** di Flutter, membuat query sederhana, dan menampilkan hasilnya dengan widget yang responsif.

## 1. Persiapan Server GraphQL (dummy)

Untuk demo cepat gunakan layanan publik **https://countries.trevorblades.com/**. Endpoint ini menyediakan data negara, kode, dan kapital.

## 2. Tambahkan Dependency

Buka `pubspec.yaml` dan tambahkan paket `graphql_flutter`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  graphql_flutter: ^5.1.0
```

Jalankan `flutter pub get`.

## 3. Inisialisasi GraphQL Client

Buat file `lib/graphql_client.dart`:

```dart
import 'package:graphql_flutter/graphql_flutter.dart';

final HttpLink httpLink = HttpLink('https://countries.trevorblades.com/');

final GraphQLClient client = GraphQLClient(
  cache: GraphQLCache(store: InMemoryStore()),
  link: httpLink,
);
```

## 4. Definisikan Query

Kita ingin menampilkan nama negara dan kapital. Simpan query di file `lib/queries.dart`:

```dart
const String countriesQuery = r'''query GetCountries {
  countries {
    code
    name
    capital
  }
}''';
```

## 5. Widget dengan `Query` Widget

`graphql_flutter` menyediakan widget `Query` yang menangani request dan streaming data.

```dart
import 'package:flutter/material.dart';
import 'package:graphql_flutter/graphql_flutter.dart';
import 'graphql_client.dart';
import 'queries.dart';

class CountriesPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return GraphQLProvider(
      client: ValueNotifier(client),
      child: Scaffold(
        appBar: AppBar(title: Text('Daftar Negara (GraphQL)')),
        body: Query(
          options: QueryOptions(document: gql(countriesQuery)),
          builder: (result, {fetchMore, refetch}) {
            if (result.isLoading) return Center(child: CircularProgressIndicator());
            if (result.hasException) return Center(child: Text('Error: ${result.exception.toString()}'));
            final List countries = result.data!['countries'];
            return ListView.builder(
              itemCount: countries.length,
              itemBuilder: (_, i) {
                final c = countries[i];
                return ListTile(
                  leading: CircleAvatar(child: Text(c['code'])),
                  title: Text(c['name']),
                  subtitle: Text('Ibukota: ${c['capital'] ?? "-"}'),
                );
              },
            );
          },
        ),
      ),
    );
  }
}
```

Widget ini menampilkan loading spinner, menampilkan error bila ada, dan menampilkan daftar negara setelah data tersedia.

## 6. Menambahkan Routing

Di `main.dart` tambahkan route baru:

```dart
import 'countries_page.dart';

void main() async {
  await initHiveForFlutter(); // diperlukan graphql_flutter
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter GraphQL Demo',
      theme: ThemeData(primarySwatch: Colors.blue),
      home: HomeScreen(),
      routes: {
        '/countries': (_) => CountriesPage(),
      },
    );
  }
}
```

Tambahkan tombol di halaman utama untuk menavigasi ke `/countries`.

## 7. Testing Cepat

Jalankan `flutter run`. Jika berhasil, Anda akan melihat daftar semua negara dengan kode ISO dan ibu kotanya.

## 8. Tips Optimasi

* **Cache**: `graphql_flutter` otomatis menyimpan data di memori. Untuk persisten, gunakan `HiveStore`.
* **Error handling**: Periksa `result.exception?.linkException` untuk jaringan, dan `result.exception?.graphqlErrors` untuk error server.
* **Pagination**: Gunakan argumen `first`, `after` di query dan `fetchMore` di widget.

## 9. Kesimpulan

GraphQL memberikan kontrol penuh atas data yang di‑fetch. Dengan `graphql_flutter` integrasi menjadi sangat simpel: definisi client, query, dan widget yang menampilkan hasil. Anda kini dapat memperluas aplikasi Flutter dengan sumber data fleksibel.

---

**Coba sendiri! Share ke sosial media dan tag @ahsai001**
