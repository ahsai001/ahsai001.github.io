---
layout: post
title: "Flutter #24: Bottom Navigation & TabBar"
date: 2026-09-01 07:00:00 +0700
tags: [flutter, dart, bottom-navigation, tabbar, ui]
---

Halo developer! 👋

Di artikel #24 ini kita bakal belajar tentang **Bottom Navigation Bar** dan **TabBar** — dua komponen UI paling umum yang hampir pasti ada di setiap app. Instagram pakai bottom nav, WhatsApp juga. Gmail pakai tab bar di inbox-nya.

Topik ini penting banget karena jadi fondasi navigasi multi-halaman dalam satu screen. Kita akan bahas:

1. `BottomNavigationBar` — navigasi di bagian bawah layar
2. `TabBar` + `TabBarView` — tab horizontal di dalam halaman
3. Kombinasi keduanya dalam satu aplikasi

Yuk mulai! 🚀

---

## 1. BottomNavigationBar Dasar

`BottomNavigationBar` adalah widget bawaan Flutter yang menampilkan ikon-ikon di bagian bawah layar. User tap ikon, halaman berubah.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Bottom Nav Demo',
      theme: ThemeData(useMaterial3: true, colorSchemeSeed: Colors.indigo),
      home: const MainScreen(),
    );
  }
}

class MainScreen extends StatefulWidget {
  const MainScreen({super.key});

  @override
  State<MainScreen> createState() => _MainScreenState();
}

class _MainScreenState extends State<MainScreen> {
  int _currentIndex = 0;

  final List<Widget> _pages = [
    const Center(child: Text('🏠 Home', style: TextStyle(fontSize: 24))),
    const Center(child: Text('🔍 Search', style: TextStyle(fontSize: 24))),
    const Center(child: Text('👤 Profile', style: TextStyle(fontSize: 24))),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('My App')),
      body: _pages[_currentIndex],
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _currentIndex,
        onTap: (index) {
          setState(() {
            _currentIndex = index;
          });
        },
        items: const [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Home'),
          BottomNavigationBarItem(icon: Icon(Icons.search), label: 'Search'),
          BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Profile'),
        ],
      ),
    );
  }
}
```

**Yang perlu diperhatikan:**

- **`currentIndex`** — index halaman yang sedang aktif
- **`onTap`** — callback saat user tap, update index dengan `setState`
- **`items`** — daftar item navigasi (ikon + label)
- Body berubah sesuai `_currentIndex`

Kalau cuma 2-3 item, di atas udah cukup. Tapi kalau lebih dari 3 item, gunakan `type: BottomNavigationBarType.fixed` supaya semua label tetap terlihat (default: 4+ item jadi `shifting`).

```dart
BottomNavigationBar(
  type: BottomNavigationBarType.fixed, // semua label selalu tampil
  currentIndex: _currentIndex,
  onTap: (index) => setState(() => _currentIndex = index),
  items: const [
    BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Home'),
    BottomNavigationBarItem(icon: Icon(Icons.search), label: 'Search'),
    BottomNavigationBarItem(icon: Icon(Icons.favorite), label: 'Favorites'),
    BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Profile'),
  ],
)
```

### BottomNavigationBar Tipe 3 dan 4

Ada 3 style utama di Material 3:

- **`BottomNavigationBarType.fixed`** — label selalu tampil, warna statis
- **`BottomNavigationBarType.shifting`** — label muncul saat aktif saja, item lebih lebar
- **Material 3 default** — gaya "navigation bar" dengan background dan indicator

Untuk Material 3, kamu juga bisa pakai `NavigationBar` (widget baru) yang punya animasi halus bawaan:

```dart
NavigationBar(
  selectedIndex: _currentIndex,
  onDestinationSelected: (index) {
    setState(() => _currentIndex = index);
  },
  destinations: const [
    NavigationDestination(icon: Icon(Icons.home_outlined), selectedIcon: Icon(Icons.home), label: 'Home'),
    NavigationDestination(icon: Icon(Icons.search_outlined), selectedIcon: Icon(Icons.search), label: 'Search'),
    NavigationDestination(icon: Icon(Icons.person_outlined), selectedIcon: Icon(Icons.person), label: 'Profile'),
  ],
)
```

> **Tip:** Kalau pakai `useMaterial3: true` (default di Flutter 3.16+), gunakan `NavigationBar` untuk tampilan terkini. `BottomNavigationBar` masih bekerja tapi tampilan agak klasik.

---

## 2. TabBar & TabBarView

`TabBar` cocok untuk navigasi **horizontal** — seperti tab di WhatsApp (Chat, Status, Calls). Biasanya posisi di bawah `AppBar`.

```dart
class TabDemoScreen extends StatelessWidget {
  const TabDemoScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return DefaultTabController(
      length: 3, // jumlah tab
      child: Scaffold(
        appBar: AppBar(
          title: const Text('TabBar Demo'),
          bottom: const TabBar(
            tabs: [
              Tab(icon: Icon(Icons.chat), text: 'Chat'),
              Tab(icon: Icon(Icons.info), text: 'Status'),
              Tab(icon: Icon(Icons.phone), text: 'Calls'),
            ],
          ),
        ),
        body: const TabBarView(
          children: [
            Center(child: Text('💬 Chat List')),
            Center(child: Text('📋 Status Updates')),
            Center(child: Text('📞 Call History')),
          ],
        ),
      ),
    );
  }
}
```

**Key points:**

- **`DefaultTabController`** — membungkus seluruh widget tree, memberikan tab controller secara otomatis. `length` harus sesuai jumlah tab
- **`TabBar`** — taruh di `AppBar.bottom`, berisi `Tab` widgets
- **`TabBarView`** — body yang isinya konten per tab, urutan harus sama dengan `TabBar`

### TabBar dengan Badge / Counter

Mau tambahkan angka notifikasi di tab? Bisa pakai `Badge`:

```dart
TabBar(
  tabs: [
    const Tab(text: 'Chat'),
    const Tab(text: 'Status'),
    Badge(
      label: const Text('5'),
      child: const Tab(text: 'Calls'),
    ),
  ],
)
```

---

## 3. Kombinasi: Bottom Nav + TabBar

Ini pattern paling real-world — misalnya Instagram: bottom nav buat ganti section, tapi di halaman tertentu ada tab bar (like, reels, tags di profil).

```dart
class CombinedScreen extends StatefulWidget {
  const CombinedScreen({super.key});

  @override
  State<CombinedScreen> createState() => _CombinedScreenState();
}

class _CombinedScreenState extends State<CombinedScreen> {
  int _bottomIndex = 0;

  // Tab 0: Home page (punya tab bar sendiri)
  Widget _buildHomePage() {
    return DefaultTabController(
      length: 2,
      child: Column(
        children: [
          const TabBar(
            tabs: [
              Tab(text: 'For You'),
              Tab(text: 'Following'),
            ],
          ),
          Expanded(
            child: TabBarView(
              children: [
                Center(
                  child: ListView.builder(
                    itemCount: 5,
                    itemBuilder: (ctx, i) => ListTile(
                      leading: const CircleAvatar(child: Icon(Icons.person)),
                      title: Text('Post For You #$i'),
                    ),
                  ),
                ),
                const Center(child: Text('Following feed')),
              ],
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildProfilePage() {
    return DefaultTabController(
      length: 3,
      child: Column(
        children: [
          // Profil header
          const Padding(
            padding: EdgeInsets.all(16),
            child: Column(
              children: [
                CircleAvatar(radius: 40, child: Icon(Icons.person, size: 40)),
                SizedBox(height: 8),
                Text('Ahmad Saifullah', style: TextStyle(fontWeight: FontWeight.bold, fontSize: 18)),
                Text('Flutter Developer'),
              ],
            ),
          ),
          const TabBar(
            tabs: [
              Tab(icon: Icon(Icons.grid_view)),
              Tab(icon: Icon(Icons.bookmark)),
              Tab(icon: Icon(Icons.person)),
            ],
          ),
          Expanded(
            child: TabBarView(
              children: [
                // Grid posts
                GridView.builder(
                  padding: const EdgeInsets.all(4),
                  gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(crossAxisCount: 3, crossAxisSpacing: 4, mainAxisSpacing: 4),
                  itemCount: 9,
                  itemBuilder: (ctx, i) => Container(
                    color: Colors.blue.shade100,
                    child: Center(child: Text('${i + 1}')),
                  ),
                ),
                const Center(child: Text('Bookmarked posts')),
                const Center(child: Text('Tagged posts')),
              ],
            ),
          ),
        ],
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('MyApp')),
      body: [
        _buildHomePage(),
        const Center(child: Text('🔍 Search')),
        _buildProfilePage(),
      ][_bottomIndex],
      bottomNavigationBar: NavigationBar(
        selectedIndex: _bottomIndex,
        onDestinationSelected: (i) => setState(() => _bottomIndex = i),
        destinations: const [
          NavigationDestination(icon: Icon(Icons.home_outlined), selectedIcon: Icon(Icons.home), label: 'Home'),
          NavigationDestination(icon: Icon(Icons.search_outlined), selectedIcon: Icon(Icons.search), label: 'Search'),
          NavigationDestination(icon: Icon(Icons.person_outlined), selectedIcon: Icon(Icons.person), label: 'Profile'),
        ],
      ),
    );
  }
}
```

**Pattern di atas:**

1. `NavigationBar` di bawah mengontrol halaman utama (Home, Search, Profile)
2. Home page punya `TabBar` sendiri (For You, Following)
3. Profile page juga punya tab (Posts, Bookmarks, Tagged)
4. Setiap tab section pakai `DefaultTabController` terpisah

---

## 4. Tips Praktis

### TabController Manual (Advanced)

Kalau kamu perlu kontrol programmatic (misalnya auto-switch tab setelah action), pakai `TabController` manual:

```dart
class ManualTabScreen extends StatefulWidget {
  const ManualTabScreen({super.key});

  @override
  State<ManualTabScreen> createState() => _ManualTabScreenState();
}

class _ManualTabScreenState extends State<ManualTabScreen>
    with SingleTickerProviderStateMixin {
  late TabController _tabController;

  @override
  void initState() {
    super.initState();
    _tabController = TabController(length: 3, vsync: this);
  }

  @override
  void dispose() {
    _tabController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Manual TabController'),
        bottom: TabBar(
          controller: _tabController,
          tabs: const [
            Tab(text: 'Tab 1'),
            Tab(text: 'Tab 2'),
            Tab(text: 'Tab 3'),
          ],
        ),
      ),
      body: TabBarView(
        controller: _tabController,
        children: const [
          Center(child: Text('Page 1')),
          Center(child: Text('Page 2')),
          Center(child: Text('Page 3')),
        ],
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          // Pindah ke tab berikutnya
          _tabController.animateTo(
            (_tabController.index + 1) % 3,
          );
        },
        child: const Icon(Icons.arrow_forward),
      ),
    );
  }
}
```

### IndexedStack — Preservasi State

Kalau pakai `setState` dengan list pages, setiap ganti tab state akan reset. Pakai `IndexedStack` supaya state tetap terjaga:

```dart
body: IndexedStack(
  index: _currentIndex,
  children: _pages,
),
```

---

## Kesimpulan

| Komponen | Gunanya |
|---|---|
| `NavigationBar` / `BottomNavigationBar` | Navigasi utama (Home, Search, Profile) |
| `TabBar` + `TabBarView` | Sub-navigasi dalam satu halaman |
| `DefaultTabController` | Controller otomatis untuk tab |
| `IndexedStack` | Preserve state saat switch tab |

**Next:** Di artikel #25 kita bahas **Drawer & Navigation Drawer** — cara bikin menu slide-out dari kiri. Stay tuned! 🎯

---

Coba sendiri! Share ke sosial media dan tag @ahsai001 🚀
