---
layout: post
title: "Flutter #69: Security & Enkripsi — Lindungi Data User dengan Baik"
date: 2026-11-12 07:00:00 +0700
categories: [flutter]
tags: [flutter, dart, tutorial, indonesia]
---

Hai teman-teman! 👋

Kita sudah sampai di artikel ke-69 dari seri 90 hari Flutter. Hari ini kita bahas topik yang **wajib** diaplikasikan di setiap aplikasi production: **Security & Enkripsi**.

Banyak developer yang fokus pada fitur & UI, tapi lupa mengamankan data sensitif user. Ini berbahaya — sekali bocor, kepercayaan user hilang, dan bisa kena sanksi hukum (seperti UU PDP di Indonesia).

---

## 🔒 Kenapa Security Penting?

Sebelum ke kode, pahami dulu *threat model*-nya:

| Threat | Contoh | Dampak |
|--------|--------|--------|
| **Data at Rest** | Database SQLite bocor, file config terbaca | Token, password, PII bocor |
| **Data in Transit** | API tanpa HTTPS, MitM attack | Credential disadap saat kirim |
| **Reverse Engineering** | APK di-decompile, lihat hardcoded secret | API key, secret key bocor |
| **Insecure Storage** | Simpan token di `SharedPreferences` plain text | Siapapun bisa ambil via root/backup |

---

## 🛡️ Layer Pertama: HTTPS Only (Network Security)

Ini *non-negotiable*. Semua komunikasi **wajib** pakai HTTPS.

### 1. `network_security_config.xml` (Android)

Buat file `android/app/src/main/res/xml/network_security_config.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config cleartextTrafficPermitted="false">
        <domain includeSubdomains="true">api.example.com</domain>
        <domain includeSubdomains="true">your-backend.com</domain>
    </domain-config>
    
    <!-- Allow localhost untuk development -->
    <debug-overrides>
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </debug-overrides>
</network-security-config>
```

Daftarkan di `AndroidManifest.xml`:

```xml
<application
    android:networkSecurityConfig="@xml/network_security_config"
    ... >
```

### 2. iOS: `Info.plist` (App Transport Security)

```xml
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <false/>
    <key>NSExceptionDomains</key>
    <dict>
        <key>api.example.com</key>
        <dict>
            <key>NSIncludesSubdomains</key>
            <true/>
            <key>NSExceptionRequiresForwardSecrecy</key>
            <true/>
        </dict>
    </dict>
</dict>
```

> **Tip:** Jangan pernah set `NSAllowsArbitraryLoads = true` di production!

---

## 🔐 Layer Kedua: Enkripsi Data Sensitif (At Rest)

Jangan simpan token, password, atau PII di `SharedPreferences` atau `sqlite` plain text.

### Package: `flutter_secure_storage`

```yaml
dependencies:
  flutter_secure_storage: ^9.0.0
```

Ini menggunakan:
- **Android**: Keystore (hardware-backed jika tersedia)
- **iOS**: Keychain
- **Web**: `window.crypto.subtle` (Web Crypto API)

### Contoh Penggunaan

```dart
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

class SecureStorage {
  static const _storage = FlutterSecureStorage(
    aOptions: AndroidOptions(
      encryptedSharedPreferences: true,
      // Opsional: set custom keystore alias
      keyAlias: 'my_app_key',
    ),
    iOptions: IOSOptions(
      accessibility: KeychainAccessibility.first_unlock_this_device,
    ),
    wOptions: WebOptions(
      // Web: gunakan Web Crypto API
      dbName: 'secure_db',
    ),
  );

  // Simpan token
  Future<void> saveToken(String token) async {
    await _storage.write(key: 'auth_token', value: token);
  }

  // Ambil token
  Future<String?> getToken() async {
    return await _storage.read(key: 'auth_token');
  }

  // Hapus token (logout)
  Future<void> clearToken() async {
    await _storage.delete(key: 'auth_token');
  }

  // Simpan multiple values sekaligus
  Future<void> saveUserSession({
    required String userId,
    required String accessToken,
    required String refreshToken,
  }) async {
    await _storage.writeAll({
      'user_id': userId,
      'access_token': accessToken,
      'refresh_token': refreshToken,
    });
  }
}
```

### Contoh: Login dengan Secure Storage

```dart
class AuthService {
  final SecureStorage _storage = SecureStorage();
  final Dio _dio = Dio(BaseOptions(baseUrl: 'https://api.example.com'));

  Future<bool> login(String email, String password) async {
    try {
      final response = await _dio.post('/auth/login', data: {
        'email': email,
        'password': password,
      });

      final token = response.data['access_token'];
      final refreshToken = response.data['refresh_token'];
      final userId = response.data['user']['id'].toString();

      // Simpan SECURE
      await _storage.saveUserSession(
        userId: userId,
        accessToken: token,
        refreshToken: refreshToken,
      );

      // Setup interceptor untuk auto-attach token
      _setupAuthInterceptor();
      
      return true;
    } on DioException catch (e) {
      // Handle error
      return false;
    }
  }

  void _setupAuthInterceptor() {
    _dio.interceptors.add(InterceptorsWrapper(
      onRequest: (options, handler) async {
        final token = await _storage.getToken();
        if (token != null) {
          options.headers['Authorization'] = 'Bearer $token';
        }
        return handler.next(options);
      },
    ));
  }
}
```

---

## 🔐 Layer Ketiga: Enkripsi Database (SQLite/Drift)

Kalau pakai database lokal, **enkripsi file database**-nya.

### Package: `sqlcipher_flutter_libs` + `drift`

```yaml
dependencies:
  drift: ^2.12.0
  sqlite3_flutter_libs: ^0.5.0
  # Untuk enkripsi:
  sqlcipher_flutter_libs: ^0.5.0
```

### Konfigurasi Drift dengan SQLCipher

```dart
import 'package:drift/drift.dart';
import 'package:drift/native.dart';
import 'package:sqlcipher_flutter_libs/sqlcipher_flutter_libs.dart';
import 'package:path_provider/path_provider.dart';
import 'package:path/path.dart' as p;

LazyDatabase _openConnection() {
  return LazyDatabase(() async {
    final dir = await getApplicationDocumentsDirectory();
    final file = File(p.join(dir.path, 'app_database.db'));
    
    // Buka dengan SQLCipher (encrypted)
    return NativeDatabase(
      file,
      // Passphrase dari secure storage!
      setup: (db) async {
        final passphrase = await SecureStorage().getDbPassphrase();
        await db.execute("PRAGMA key = '$passphrase';");
      },
    );
  );
}

@DriftDatabase(tables: [Users, Transactions])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());
  
  @override
  int get schemaVersion => 1;
}
```

### Generate & Simpan Passphrase Secure

```dart
class SecureStorage {
  // ... kode sebelumnya ...
  
  static const _dbPassphraseKey = 'db_passphrase';

  Future<String> getDbPassphrase() async {
    String? existing = await _storage.read(key: _dbPassphraseKey);
    if (existing != null) return existing;
    
    // Generate passphrase baru (32 byte = 256 bit)
    final random = Random.secure();
    final bytes = List<int>.generate(32, (_) => random.nextInt(256));
    final passphrase = base64Url.encode(bytes);
    
    await _storage.write(key: _dbPassphraseKey, value: passphrase);
    return passphrase;
  }
}
```

> **Penting:** Passphrase **harus** disimpan di secure storage, bukan hardcoded!

---

## 🔐 Layer Keempat: Jangan Hardcode Secret di Kode

Ini kesalahan klasik: API key, secret key, dll ditulis di file `.dart`.

### ❌ JANGAN LAKUKAN INI:

```dart
// DILARANG KERAS!
class ApiConfig {
  static const apiKey = 'sk_live_abc123...'; // BOCOR SAAT DECOMPILE
  static const secretKey = 'whsec_xyz789...';
}
```

### ✅ LAKUKAN INI:

**1. Gunakan `--dart-define` saat build:**

```bash
flutter build apk --release \
  --dart-define=API_KEY=sk_live_abc123 \
  --dart-define=SECRET_KEY=whsec_xyz789
```

**2. Akses di kode:**

```dart
class ApiConfig {
  static const apiKey = String.fromEnvironment('API_KEY');
  static const secretKey = String.fromEnvironment('SECRET_KEY');
  
  // Validasi di runtime
  static void validate() {
    if (apiKey.isEmpty || secretKey.isEmpty) {
      throw StateError('Environment variables not set!');
    }
  }
}
```

**3. Untuk CI/CD (GitHub Actions):**

```yaml
# .github/workflows/build.yml
jobs:
  build:
    steps:
      - name: Build APK
        run: |
          flutter build apk --release \
            --dart-define=API_KEY=${{ secrets.API_KEY }} \
            --dart-define=SECRET_KEY=${{ secrets.SECRET_KEY }}
```

---

## 🔐 Layer Kelima: Certificate Pinning (Anti-MitM)

Mencegah attacker intercept traffic meski pakai HTTPS, dengan memvalidasi certificate server.

### Package: `dio` + custom `HttpClientAdapter`

```dart
import 'package:dio/dio.dart';
import 'package:http_client_adapter/http_client_adapter.dart';

class PinnedDioAdapter extends DefaultHttpClientAdapter {
  final List<String> _allowedSha256Fingerprints;

  PinnedDioAdapter(this._allowedSha256Fingerprints);

  @override
  Future<HttpClient> createHttpClient(BaseOptions options) async {
    final client = await super.createHttpClient(options);
    
    client.badCertificateCallback = (cert, host, port) {
      // Hanya izinkan certificate dengan fingerprint yang dikenal
      final sha256 = cert.sha256; // butuh extension/package
      return _allowedSha256Fingerprints.contains(sha256);
    };
    
    return client;
  }
}

// Usage
final dio = Dio();
dio.httpClientAdapter = PinnedDioAdapter([
  'sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=', // Fingerprint prod
]);
```

> **Catatan:** Certificate pinning butuh maintenance — saat cert server diganti, app akan error kalau fingerprint tidak di-update. Pertimbangkan *risk vs effort*.

---

## 🔐 Layer Keenam: Obfuscation & Minification (Release Build)

Sulitkan reverse engineering dengan meng-obfuscate kode Dart.

### Aktifkan di `build.gradle` (Android)

```gradle
android {
    buildTypes {
        release {
            // Obfuscate Dart code
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
```

### Flutter Obfuscation (Dart 2.12+)

```bash
flutter build apk --release --obfuscate --split-debug-info=/path/to/debug_info
```

- `--obfuscate`: Rename simbol jadi `a`, `b`, `c`...
- `--split-debug-info`: Simpan mapping file untuk debugging crash nanti (simpan aman!)

### iOS: Bitcode & Strip Symbols

Di Xcode:
- **Enable Bitcode**: Yes (App Store butuh ini)
- **Strip Style**: All Symbols (Release)

---

## 🔐 Layer Ketujuh: Biometric Auth (Opsional tapi Direkomendasikan)

Tambah lapisan proteksi sebelum user akses data sensitif.

### Package: `local_auth`

```yaml
dependencies:
  local_auth: ^2.1.0
```

### Implementasi

```dart
import 'package:local_auth/local_auth.dart';

class BiometricAuth {
  final LocalAuthentication _auth = LocalAuthentication();

  Future<bool> authenticate({String reason = 'Autentikasi untuk akses data'}) async {
    try {
      // Cek apakah device support biometric
      final canCheck = await _auth.canCheckBiometrics;
      final isDeviceSupported = await _auth.isDeviceSupported;
      
      if (!canCheck || !isDeviceSupported) {
        return false; // Fallback ke PIN/password
      }

      // List biometric yang tersedia
      final available = await _auth.getAvailableBiometrics();
      if (available.isEmpty) return false;

      // Minta autentikasi
      return await _auth.authenticate(
        localizedReason: reason,
        options: const AuthenticationOptions(
          biometricOnly: true, // Hanya biometric, bukan device credential
          stickyAuth: true,    // Tetap minta auth kalau app ke background
        ),
      );
    } on PlatformException catch (e) {
      // Handle error (user cancel, dll)
      return false;
    }
  }
}
```

### Integrasi di App

```dart
class AppLockWrapper extends StatefulWidget {
  final Widget child;
  const AppLockWrapper({required this.child});

  @override
  State<AppLockWrapper> createState() => _AppLockWrapperState();
}

class _AppLockWrapperState extends State<AppLockWrapper> {
  final BiometricAuth _biometric = BiometricAuth();
  bool _unlocked = false;

  @override
  void initState() {
    super.initState();
    _checkAuth();
  }

  Future<void> _checkAuth() async {
    // Cek apakah user sudah enable biometric di settings app
    final enabled = await SecureStorage().getBiometricEnabled();
    if (!enabled) {
      setState(() => _unlocked = true);
      return;
    }

    final success = await _biometric.authenticate();
    setState(() => _unlocked = success);
  }

  @override
  Widget build(BuildContext context) {
    if (!_unlocked) {
      return Scaffold(
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              const Icon(Icons.fingerprint, size: 64, color: Colors.grey),
              const SizedBox(height: 16),
              const Text('Autentikasi diperlukan'),
              const SizedBox(height: 24),
              ElevatedButton(
                onPressed: _checkAuth,
                child: const Text('Coba Lagi'),
              ),
            ],
          ),
        ),
      );
    }
    return widget.child;
  }
}
```

---

## 📋 Checklist Security Sebelum Release

| ✅ Item | Status |
|---------|--------|
| Semua API pakai HTTPS (valid cert) | ☐ |
| `network_security_config` & ATS dikonfigurasi | ☐ |
| Token/credential di `flutter_secure_storage` | ☐ |
| Database lokal dienkripsi (SQLCipher) | ☐ |
| Tidak ada secret hardcoded (pakai `--dart-define`) | ☐ |
| Certificate pinning (jika perlu) | ☐ |
| Build release dengan `--obfuscate` | ☐ |
| Debug info disimpan terpisah & aman | ☐ |
| Biometric auth untuk fitur sensitif | ☐ |
| Logging sensitif DISABLED di release | ☐ |
| Error handling tidak bocorkan stack trace ke user | ☐ |

---

## 🎯 Best Practice Ringkas

```dart
// 1. SELALU validasi input
final email = EmailValidator.validate(input) ? input : throw FormatException('Email invalid');

// 2. JANGAN log sensitif
logger.d('Login attempt for: $email'); // OK
logger.d('Password: $password');        // DILARANG!

// 3. Gunakan const constructor untuk immutable data
class User {
  final String id;
  final String email;
  const User({required this.id, required this.email});
}

// 4. Rate limiting di client side (opsional tapi bagus)
final limiter = RateLimiter(maxRequests: 5, window: Duration(minutes: 1));

// 5. Sanitasi output (XSS prevention di WebView)
final safeHtml = HtmlEscape().convert(userInput);
```

---

## 🏁 Kesimpulan

Security bukan fitur *optional* — itu **fondasi** aplikasi production. Yang harus kamu ingat:

1. **HTTPS everywhere** — tidak ada alasan pakai HTTP
2. **Secure storage** untuk token & credential — jangan `SharedPreferences`
3. **Enkripsi database** — SQLCipher untuk SQLite/Drift
4. **No hardcoded secrets** — gunakan `--dart-define` + CI/CD secrets
5. **Obfuscation** — sulitkan reverse engineering
6. **Biometric** — tambah lapisan user-facing security
7. **Audit berkala** — dependency update, cert rotation, pen testing

---

## 💡 Tantangan Hari Ini

Coba implementasikan **satu** dari berikut di project kamu:
1. Ganti `SharedPreferences` → `flutter_secure_storage` untuk token
2. Tambah `network_security_config.xml` & ATS
3. Setup `--dart-define` untuk API key di GitHub Actions
4. Aktifkan `--obfuscate` di build release

**Coba sendiri! Share ke sosial media dan tag @ahsai001** 🚀

---

*Artikel ini bagian dari seri **90 Hari Flutter**. Artikel selanjutnya: **#70: Code Review Checklist** — checklist lengkap sebelum merge ke main branch.*