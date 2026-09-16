---
layout: post
title: "Flutter #39: Firebase Authentication"
date: 2026-09-17 07:00:00 +0700
tags: [flutter, dart, firebase, authentication, tutorial]
---

# Flutter #39: Firebase Authentication

Halo Flutter Developer! 🚀

Setelah belajar setup Firebase & Firestore di artikel kemarin, sekarang waktunya masuk ke fitur yang hampir semua app butuh: **Authentication**. Hari ini kita akan bikin sistem login & register pakai **Firebase Auth** — lengkap dengan Email/Password, Google Sign-In, dan pengecekan state login di seluruh app.

## Kenapa Firebase Auth?

Semua app butuh identitas user. Dari sekedar login sampai role-based access, authentication jadi fondasi keamanan. Firebase Auth hadir dengan berbagai metode login siap pakai:

- **Email & Password** — paling dasar dan universal
- **Google Sign-In** — satu klik, user langsung login pakai akun Google
- **Phone Number** — OTP via SMS
- **Anonymous** — user bisa pakai app dulu tanpa daftar

Kita fokus ke **Email/Password** dan **Google Sign-In** karena paling sering dipakai di production.

## Persiapan

Pastikan kamu sudah setup Firebase seperti di artikel #38. Kalau belum, install dulu package:

```yaml
# pubspec.yaml
dependencies:
  firebase_core: ^3.8.1
  firebase_auth: ^5.4.1
  google_sign_in: ^6.2.2
```

Lalu jalankan:

```bash
flutter pub get
```

## Login dengan Email & Password

### 1. Registrasi User Baru

```dart
import 'package:firebase_auth/firebase_auth.dart';

Future<void> registerEmail(String email, String password) async {
  try {
    final credential = await FirebaseAuth.instance
        .createUserWithEmailAndPassword(
      email: email,
      password: password,
    );
    print('Registrasi berhasil: ${credential.user?.email}');
  } on FirebaseAuthException catch (e) {
    if (e.code == 'weak-password') {
      print('Password terlalu lemah');
    } else if (e.code == 'email-already-in-use') {
      print('Email sudah terdaftar');
    }
  } catch (e) {
    print('Error: $e');
  }
}
```

Cukup panggil `createUserWithEmailAndPassword` — Firebase langsung handle hashing password, generate UID, dan simpan ke backend. Nggak perlu bikin table user sendiri.

### 2. Login User

```dart
Future<void> loginEmail(String email, String password) async {
  try {
    final credential = await FirebaseAuth.instance
        .signInWithEmailAndPassword(
      email: email,
      password: password,
    );
    print('Login berhasil: ${credential.user?.email}');
  } on FirebaseAuthException catch (e) {
    if (e.code == 'user-not-found') {
      print('Email tidak ditemukan');
    } else if (e.code == 'wrong-password') {
      print('Password salah');
    }
  }
}
```

Sama mudahnya — `signInWithEmailAndPassword` melakukan verifikasi kredensial dan return `UserCredential` jika berhasil.

### 3. Logout

```dart
await FirebaseAuth.instance.signOut();
```

Satu baris. Firebase handle session cleanup otomatis.

## Login dengan Google Sign-In

Google Sign-In bikin proses login jadi seamless — user tinggal pilih akun Google mereka.

```dart
import 'package:google_sign_in/google_sign_in.dart';

Future<User?> signInWithGoogle() async {
  final GoogleSignInAccount? googleUser = await GoogleSignIn().signIn();

  if (googleUser == null) return null; // user batal login

  final GoogleSignInAuthentication googleAuth =
      await googleUser.authentication;

  final credential = GoogleAuthProvider.credential(
    accessToken: googleAuth.accessToken,
    idToken: googleAuth.idToken,
  );

  final userCredential =
      await FirebaseAuth.instance.signInWithCredential(credential);

  return userCredential.user;
}
```

**Alurnya:**
1. `GoogleSignIn().signIn()` — tampilkan dialog pilih akun Google
2. Ambil `accessToken` dan `idToken` dari hasilnya
3. Buat `OAuthCredential` dari token tersebut
4. Login ke Firebase pakai credential itu

User nggak perlu daftar ulang — kalau email Google-nya sudah terdaftar di Firebase, langsung login. Kalau belum, Firebase auto-create akun baru.

## Cek Status Login di Seluruh App

Bagian terpenting: cara tahu user sudah login atau belum, dan react perubahan state secara real-time.

### Menggunakan `stream`

```dart
import 'package:firebase_auth/firebase_auth.dart';

class AuthGate extends StatelessWidget {
  const AuthGate({super.key});

  @override
  Widget build(BuildContext context) {
    return StreamBuilder<User?>(
      stream: FirebaseAuth.instance.authStateChanges(),
      builder: (context, snapshot) {
        // Loading
        if (snapshot.connectionState == ConnectionState.waiting) {
          return const Scaffold(
            body: Center(child: CircularProgressIndicator()),
          );
        }

        // Sudah login → ke Home
        if (snapshot.hasData) {
          return HomePage(user: snapshot.data!);
        }

        // Belum login → ke Login
        return const LoginPage();
      },
    );
  }
}
```

`authStateChanges()` return `Stream<User?>` yang emit setiap ada perubahan login state. User login? Stream emit `User`. User logout? Stream emit `null`. Widget otomatis rebuild.

### Implementasi di main.dart

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'My App',
      home: const AuthGate(),
    );
  }
}
```

Tanpa `if-else` manual. Tanpa `SharedPreferences` untuk simpan status login. Firebase handle semuanya — bahkan kalau user kill app dan buka lagi, `authStateChanges()` langsung emit state terbaru.

## Kirim Email Verifikasi

Produk Firebase Auth free tier punya limit email verification. Aktifkan untuk keamanan:

```dart
Future<void> sendVerification() async {
  final user = FirebaseAuth.instance.currentUser;
  if (user != null && !user.emailVerified) {
    await user.sendEmailVerification();
    print('Email verifikasi dikirim ke ${user.email}');
  }
}
```

Setelah user klik link di email, kamu bisa cek:

```dart
bool isVerified = FirebaseAuth.instance.currentUser?.emailVerified ?? false;
```

## Reset Password

```dart
Future<void> resetPassword(String email) async {
  try {
    await FirebaseAuth.instance.sendPasswordResetEmail(email: email);
    print('Link reset password dikirim');
  } on FirebaseAuthException catch (e) {
    print('Error: ${e.message}');
  }
}
```

User tinggal klik link di email, set password baru, dan bisa login lagi.

## Tips Production

**1. Selalu handle error.** Setiap method Firebase Auth bisa throw `FirebaseAuthException` — jangan pernah pakai `.catchError` tanpa cek `e.code`:

```dart
} on FirebaseAuthException catch (e) {
  switch (e.code) {
    case 'user-not-found':
      showMessage('Email tidak terdaftar');
      break;
    case 'wrong-password':
      showMessage('Password salah');
      break;
    case 'invalid-email':
      showMessage('Format email tidak valid');
      break;
    default:
      showMessage('Terjadi kesalahan: ${e.message}');
  }
}
```

**2. Jangan simpan password di Firestore.** Firebase Auth sudah handle semuanya. Yang kamu simpan di Firestore hanya `uid`, `email`, `displayName` — metadata user.

**3. Pakai `user.uid` sebagai document ID.** Kalau kamu butuh profile user di Firestore:

```dart
await FirebaseFirestore.instance
    .collection('users')
    .doc(user.uid)
    .set({
      'email': user.email,
      'displayName': user.displayName ?? '',
      'createdAt': FieldValue.serverTimestamp(),
    });
```

## Ringkasan

| Metode | Kapan Pakai | Kompleksitas |
|--------|-------------|-------------|
| Email/Password | App universal, semua platform | ⭐ |
| Google Sign-In | User base Android-heavy | ⭐⭐ |
| Phone Number | App yang butuh verifikasi real | ⭐⭐ |
| Anonymous | App casual, registrasi nanti | ⭐ |

Firebase Auth itu powered by backend Google — scalenya auto, security-nya managed, dan pricing-nya free untuk 50K user/bulan. Untuk kebanyakan app indie, kamu nggak akan bayar sepeser pun.

Di artikel selanjutnya (#40), kita akan belajar **Firebase Storage** — upload dan manage file/image di cloud.

---

Coba sendiri! Bikin project Flutter baru, setup Firebase Auth, dan coba login pakai email atau Google. Share hasilnya ke sosial media dan tag **@ahsai001**! 🎉
