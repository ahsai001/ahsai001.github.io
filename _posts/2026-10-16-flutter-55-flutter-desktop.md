---
layout: post
title: "Flutter #55: Flutter Desktop — Bangun Aplikasi Desktop Native"
date: 2026-10-16 07:00:00 +0700
categories: [flutter]
tags: [flutter, dart, tutorial, indonesia]
---

Flutter Desktop memungkinkan kamu membangun aplikasi Windows, macOS, dan Linux dari kode yang sama dengan mobile dan web. Sekarang sudah **stable** sejak Flutter 3.0 — artinya siap produksi.

---

## Mengaktifkan Desktop Support

Pastikan Flutter SDK versi terbaru:

```bash
flutter upgrade
flutter config --enable-windows-desktop
flutter config --enable-macos-desktop
flutter config --enable-linux-desktop
```

Cek device yang tersedia:

```bash
flutter devices
```

Kamu akan melihat `windows`, `macos`, atau `linux` di daftar device.

---

## Struktur Project Desktop

Buat project baru atau tambahkan ke project existing:

```bash
flutter create my_desktop_app
cd my_desktop_app
```

Folder platform desktop otomatis dibuat:
- `windows/` — CMake, runner, manifest
- `macos/` — Xcode project, entitlements
- `linux/` — CMake, GTK, .desktop file

---

## Contoh 1: Window Size & Title Kustom

Di `main.dart`, atur ukuran jendela dan title sebelum `runApp`:

```dart
import 'package:flutter/material.dart';
import 'package:window_manager/window_manager.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  // Konfigurasi window manager
  await windowManager.ensureInitialized();
  
  WindowOptions windowOptions = const WindowOptions(
    size: Size(1024, 768),
    minimumSize: Size(800, 600),
    center: true,
    backgroundColor: Colors.transparent,
    titleBarStyle: TitleBarStyle.hidden, // custom title bar
  );
  
  windowManager.waitUntilReadyToShow(windowOptions, () async {
    await windowManager.show();
    await windowManager.focus();
  });
  
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});
  
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Desktop App',
      theme: ThemeData(
        useMaterial3: true,
        colorSchemeSeed: Colors.indigo,
      ),
      home: const HomePage(),
    );
  }
}

class HomePage extends StatelessWidget {
  const HomePage({super.key});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Flutter Desktop'),
        centerTitle: true,
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.computer, size: 80, color: Colors.indigo),
            const SizedBox(height: 16),
            const Text(
              'Hello Flutter Desktop!',
              style: TextStyle(fontSize: 28, fontWeight: FontWeight.w600),
            ),
            const SizedBox(height: 8),
            Text(
              'Running on ${Platform.operatingSystem}',
              style: TextStyle(fontSize: 16, color: Colors.grey[600]),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Dependency `window_manager`** (di `pubspec.yaml`):
```yaml
dependencies:
  window_manager: ^0.3.8
```

Package ini memberikan kontrol penuh: resize, minimize, maximize, fullscreen, custom title bar, dll.

---

## Contoh 2: Menu Bar Native (macOS/Windows)

Gunakan `bitsdojo_window` untuk menu bar custom yang terasa native:

```dart
// lib/main.dart
import 'package:bitsdojo_window/bitsdojo_window.dart';

void main() {
  runApp(const MyApp());
  
  // Setup window buttons (close, minimize, maximize)
  doWhenWindowReady(() {
    final win = appWindow;
    win.minSize = const Size(800, 600);
    win.size = const Size(1024, 768);
    win.alignment = Alignment.center;
    win.title = "My Flutter Desktop App";
    win.show();
  });
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});
  
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData(useMaterial3: true, colorSchemeSeed: Colors.blue),
      home: const DesktopShell(),
    );
  }
}

class DesktopShell extends StatelessWidget {
  const DesktopShell({super.key});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      // Custom title bar dengan window buttons
      appBar: PreferredSize(
        preferredSize: const Size.fromHeight(kToolbarHeight),
        child: WindowTitleBarBox(
          child: Row(
            children: [
              Expanded(
                child: MoveWindow(
                  child: Container(
                    padding: const EdgeInsets.symmetric(horizontal: 16),
                    color: Theme.of(context).colorScheme.surface,
                    child: const Text('My App', style: TextStyle(fontWeight: FontWeight.w600)),
                  ),
                ),
              ),
              const WindowButtons(),
            ],
          ),
        ),
      ),
      body: const Center(child: Text('Content goes here')),
    );
  }
}
```

**Dependency**:
```yaml
dependencies:
  bitsdojo_window: ^0.1.5
```

---

## Build & Distribusi

### Windows (MSIX / EXE)
```bash
flutter build windows --release
# Output: build/windows/x64/runner/Release/
```

Buat installer dengan **MSIX** (Windows Store) atau **Inno Setup** / **NSIS** untuk EXE tradisional.

### macOS (App Bundle / DMG)
```bash
flutter build macos --release
# Output: build/macos/Build/Products/Release/
```

Codesign & notarize untuk distribusi di luar App Store:
```bash
codesign --deep --force --verify --verbose --sign "Developer ID Application: Your Name" build/macos/Build/Products/Release/your_app.app
# Lalu buat DMG dengan create-dmg atau appdmg
```

### Linux (AppImage / Snap / Flatpak / DEB)
```bash
flutter build linux --release
# Output: build/linux/x64/release/bundle/
```

Gunakan **AppImage** untuk portable, atau **Snapcraft** / **Flatpak** untuk store.

---

## Tips Khusus Desktop

| Fitur | Solusi |
|-------|--------|
| Keyboard shortcuts | `Shortcuts` + `Actions` widget atau `FocusableActionDetector` |
| Context menu (right-click) | `PopupMenuButton` atau `flutter_context_menu` |
| Drag & drop files | `desktop_drop` package |
| System tray | `system_tray` / `window_manager` |
| Native dialogs | `file_picker`, `path_provider` |
| Auto-update | `upgrader` (GitHub Releases) atau custom |

---

## Perbedaan Mobile vs Desktop

| Aspek | Mobile | Desktop |
|-------|--------|---------|
| Input | Touch | Mouse, keyboard, trackpad |
| Window | Fullscreen | Resizable, multi-window |
| Menu | Drawer/BottomNav | Menu bar, context menu |
| Density | High DPI | Variable DPI |
| Distribution | App Store/Play Store | Website, Store, Package manager |

---

## Kesimpulan

Flutter Desktop sudah **production-ready**. Kode UI sama, tinggal handle:
1. Window management (`window_manager`, `bitsdojo_window`)
2. Native menu & shortcuts
3. Build & signing per platform
4. Distribusi (MSIX, DMG, AppImage, Snap)

Mulai migrasi app mobile ke desktop hari ini — codebase 90% reusable!

---

Coba sendiri! Share ke sosial media dan tag @ahsai001