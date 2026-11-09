---
layout: post
title: "Flutter #68: Monetization: Admob"
date: 2026-11-10 07:00:00 +0700
categories: [flutter]
tags: [flutter, dart, tutorial, indonesia]
---

Monetisasi aplikasi adalah salah satu alasan utama banyak developer belajar Flutter. Google AdMob adalah platform iklan paling populer untuk aplikasi mobile, dan integrasinya di Flutter cukup straightforward berkat package `google_mobile_ads`. Hari ini kita bahas cara setup AdMob, jenis-jenis iklan, hingga best practice agar revenue maksimal tapi UX tidak terganggu.

---

## 1. Persiapan Akun AdMob

Sebelum kode, pastikan sudah punya:

1. **Google AdMob Account** — daftar di [admob.google.com](https://admob.google.com)
2. **App ID** — buat aplikasi baru di dashboard AdMob, catat `App ID` (format: `ca-app-pub-xxxxxxxxxxxxxxxx~yyyyyyyyyy`)
3. **Ad Unit IDs** — buat unit iklan untuk setiap jenis:
   - **Banner** — `ca-app-pub-xxxxxxxxxxxxxxxx/yyyyyyyyyy`
   - **Interstitial** — `ca-app-pub-xxxxxxxxxxxxxxxx/yyyyyyyyyy`
   - **Rewarded** — `ca-app-pub-xxxxxxxxxxxxxxxx/yyyyyyyyyy`
   - **Native** — `ca-app-pub-xxxxxxxxxxxxxxxx/yyyyyyyyyy`

> ⚠️ **Penting**: Saat development, gunakan **Test IDs** dari Google agar akun tidak dibanned. Test IDs resmi:
> - App ID: `ca-app-pub-3940256099942544~3347511713`
> - Banner: `ca-app-pub-3940256099942544/6300978111`
> - Interstitial: `ca-app-pub-3940256099942544/1033173712`
> - Rewarded: `ca-app-pub-3940256099942544/5224354917`

---

## 2. Setup Dependencies

Tambahkan ke `pubspec.yaml`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  google_mobile_ads: ^5.3.0
```

Jalankan `flutter pub get`.

---

## 3. Konfigurasi Platform

### Android (`android/app/src/main/AndroidManifest.xml`)

Tambahkan `meta-data` di dalam tag `<application>`:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.my_app">

    <application
        android:label="My App"
        android:icon="@mipmap/ic_launcher">

        <!-- AdMob App ID -->
        <meta-data
            android:name="com.google.android.gms.ads.APPLICATION_ID"
            android:value="ca-app-pub-3940256099942544~3347511713" />

        <activity
            android:name=".MainActivity"
            ... >
            ...
        </activity>
    </application>
</manifest>
```

### iOS (`ios/Runner/Info.plist`)

Tambahkan key `GADApplicationIdentifier`:

```xml
<key>GADApplicationIdentifier</key>
<string>ca-app-pub-3940256099942544~3347511713</string>
```

---

## 4. Inisialisasi Mobile Ads

Inisialisasi di `main.dart` sebelum `runApp`:

```dart
import 'package:flutter/material.dart';
import 'package:google_mobile_ads/google_mobile_ads.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  // Inisialisasi AdMob
  await MobileAds.instance.initialize();
  
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'AdMob Demo',
      theme: ThemeData(primarySwatch: Colors.blue),
      home: const HomePage(),
    );
  }
}
```

---

## 5. Banner Ad — Iklan Tetap di Bawah/Atas

Banner cocok untuk revenue passive tanpa ganggu UX.

```dart
import 'package:flutter/material.dart';
import 'package:google_mobile_ads/google_mobile_ads.dart';

class BannerAdWidget extends StatefulWidget {
  final String adUnitId;
  const BannerAdWidget({super.key, required this.adUnitId});

  @override
  State<BannerAdWidget> createState() => _BannerAdWidgetState();
}

class _BannerAdWidgetState extends State<BannerAdWidget> {
  BannerAd? _bannerAd;
  bool _isLoaded = false;

  @override
  void initState() {
    super.initState();
    _loadAd();
  }

  void _loadAd() {
    _bannerAd = BannerAd(
      adUnitId: widget.adUnitId,
      size: AdSize.banner,
      request: const AdRequest(),
      listener: BannerAdListener(
        onAdLoaded: (ad) {
          setState(() => _isLoaded = true);
        },
        onAdFailedToLoad: (ad, error) {
          ad.dispose();
          print('Banner failed to load: $error');
          // Optional: retry setelah delay
          Future.delayed(const Duration(seconds: 30), _loadAd);
        },
      ),
    )..load();
  }

  @override
  void dispose() {
    _bannerAd?.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    if (!_isLoaded || _bannerAd == null) return const SizedBox.shrink();
    
    return Align(
      alignment: Alignment.bottomCenter,
      child: SafeArea(
        child: SizedBox(
          width: _bannerAd!.size.width.toDouble(),
          height: _bannerAd!.size.height.toDouble(),
          child: AdWidget(ad: _bannerAd!),
        ),
      ),
    );
  }
}
```

**Penggunaan di halaman:**

```dart
class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('AdMob Demo')),
      body: Stack(
        children: [
          // Konten utama
          ListView.builder(
            itemCount: 20,
            itemBuilder: (_, i) => ListTile(title: Text('Item $i')),
          ),
          // Banner di bawah
          const BannerAdWidget(
            adUnitId: 'ca-app-pub-3940256099942544/6300978111', // Test ID
          ),
        ],
      ),
    );
  }
}
```

---

## 6. Interstitial Ad — Iklan Full Screen

Interstitial muncul saat transisi halaman (misal: pindah screen, selesai level game). **Jangan tampilkan terlalu sering** — Google menyarankan max 1x per 2-3 menit per user.

```dart
class InterstitialAdManager {
  InterstitialAd? _interstitialAd;
  bool _isLoaded = false;
  final String adUnitId;

  InterstitialAdManager({required this.adUnitId});

  void load() {
    InterstitialAd.load(
      adUnitId: adUnitId,
      request: const AdRequest(),
      adLoadCallback: InterstitialAdLoadCallback(
        onAdLoaded: (ad) {
          _interstitialAd = ad;
          _isLoaded = true;
          _interstitialAd!.fullScreenContentCallback = FullScreenContentCallback(
            onAdDismissedFullScreenContent: (ad) {
              ad.dispose();
              _isLoaded = false;
              load(); // Preload iklan berikutnya
            },
            onAdFailedToShowFullScreenContent: (ad, error) {
              ad.dispose();
              _isLoaded = false;
              load();
            },
          );
        },
        onAdFailedToLoad: (error) {
          print('Interstitial failed to load: $error');
          Future.delayed(const Duration(seconds: 30), load);
        },
      ),
    );
  }

  void show({VoidCallback? onDismissed}) {
    if (_isLoaded && _interstitialAd != null) {
      _interstitialAd!.show(
        onFullScreenContentCallback: FullScreenContentCallback(
          onAdDismissedFullScreenContent: (ad) {
            onDismissed?.call();
          },
        ),
      );
      _isLoaded = false; // Prevent double show
    } else {
      onDismissed?.call(); // Langsung lanjut kalau iklan belum siap
    }
  }

  void dispose() => _interstitialAd?.dispose();
}
```

**Contoh penggunaan saat navigasi:**

```dart
class GameScreen extends StatefulWidget {
  const GameScreen({super.key});

  @override
  State<GameScreen> createState() => _GameScreenState();
}

class _GameScreenState extends State<GameScreen> {
  late final InterstitialAdManager _interstitialAd;

  @override
  void initState() {
    super.initState();
    _interstitialAd = InterstitialAdManager(
      adUnitId: 'ca-app-pub-3940256099942544/1033173712', // Test ID
    )..load();
  }

  @override
  void dispose() {
    _interstitialAd.dispose();
    super.dispose();
  }

  void _onLevelComplete() {
    _interstitialAd.show(onDismissed: () {
      Navigator.pushReplacement(
        context,
        MaterialPageRoute(builder: (_) => const NextLevelScreen()),
      );
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: ElevatedButton(
          onPressed: _onLevelComplete,
          child: const Text('Selesai Level →'),
        ),
      ),
    );
  }
}
```

---

## 7. Rewarded Ad — Iklan Bayar User

Rewarded paling user-friendly: user *pilih* nonton iklan untuk dapat reward (koin, extra life, unlock konten). Retensi & revenue biasanya tertinggi.

```dart
class RewardedAdManager {
  RewardedAd? _rewardedAd;
  bool _isLoaded = false;
  final String adUnitId;

  RewardedAdManager({required this.adUnitId});

  void load() {
    RewardedAd.load(
      adUnitId: adUnitId,
      request: const AdRequest(),
      rewardedAdLoadCallback: RewardedAdLoadCallback(
        onAdLoaded: (ad) {
          _rewardedAd = ad;
          _isLoaded = true;
          _rewardedAd!.fullScreenContentCallback = FullScreenContentCallback(
            onAdDismissedFullScreenContent: (ad) {
              ad.dispose();
              _isLoaded = false;
              load();
            },
          );
        },
        onAdFailedToLoad: (error) {
          print('Rewarded failed to load: $error');
          Future.delayed(const Duration(seconds: 30), load);
        },
      ),
    );
  }

  void show({required Function(RewardItem) onUserEarnedReward}) {
    if (_isLoaded && _rewardedAd != null) {
      _rewardedAd!.show(
        onUserEarnedReward: (ad, reward) {
          onUserEarnedReward(reward);
        },
      );
      _isLoaded = false;
    }
  }

  bool get isLoaded => _isLoaded;
  void dispose() => _rewardedAd?.dispose();
}
```

**UI Contoh — Tombol "Tonton Iklan untuk 100 Koin":**

```dart
class ShopScreen extends StatefulWidget {
  const ShopScreen({super.key});

  @override
  State<ShopScreen> createState() => _ShopScreenState();
}

class _ShopScreenState extends State<ShopScreen> {
  late final RewardedAdManager _rewardedAd;
  int _coins = 0;

  @override
  void initState() {
    super.initState();
    _rewardedAd = RewardedAdManager(
      adUnitId: 'ca-app-pub-3940256099942544/5224354917', // Test ID
    )..load();
  }

  @override
  void dispose() {
    _rewardedAd.dispose();
    super.dispose();
  }

  void _watchAdForCoins() {
    if (!_rewardedAd.isLoaded) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('Iklan belum siap, coba lagi sebentar')),
      );
      return;
    }

    _rewardedAd.show(onUserEarnedReward: (reward) {
      setState(() => _coins += reward.amount.toInt());
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Selamat! Kamu dapat ${reward.amount} koin')),
      );
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Toko')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Koin: $_coins', style: const TextStyle(fontSize: 32)),
            const SizedBox(height: 24),
            ElevatedButton.icon(
              onPressed: _watchAdForCoins,
              icon: const Icon(Icons.card_giftcard),
              label: const Text('Tonton Iklan +100 Koin'),
              style: ElevatedButton.styleFrom(
                padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 12),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## 8. Native Ad — Iklan Custom UI

Native ad memungkinkan kamu desain tampilan iklan sendiri (match dengan desain app). Butuh setup lebih kompleks di dashboard AdMob (buat template native). Contop dasar:

```dart
class NativeAdWidget extends StatefulWidget {
  final String adUnitId;
  final Widget Function(NativeAd ad) builder;
  const NativeAdWidget({super.key, required this.adUnitId, required this.builder});

  @override
  State<NativeAdWidget> createState() => _NativeAdWidgetState();
}

class _NativeAdWidgetState extends State<NativeAdWidget> {
  NativeAd? _nativeAd;
  bool _isLoaded = false;

  @override
  void initState() {
    super.initState();
    _loadAd();
  }

  void _loadAd() {
    _nativeAd = NativeAd(
      adUnitId: widget.adUnitId,
      factoryId: 'ad_factory_id', // Harus match dengan yang di dashboard
      request: const AdRequest(),
      listener: NativeAdListener(
        onAdLoaded: (ad) => setState(() => _isLoaded = true),
        onAdFailedToLoad: (ad, error) {
          ad.dispose();
          Future.delayed(const Duration(seconds: 30), _loadAd);
        },
      ),
    )..load();
  }

  @override
  void dispose() {
    _nativeAd?.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    if (!_isLoaded || _nativeAd == null) return const SizedBox.shrink();
    return widget.builder(_nativeAd!);
  }
}
```

---

## 9. Best Practice Monetisasi

| Tips | Alasan |
|------|--------|
| **Gunakan Test IDs saat dev** | Hindari banned akun |
| **Jangan spam interstitial** | Max 1x per 2-3 menit per user |
| **Rewarded > Interstitial > Banner** | Revenue per impression tertinggi di rewarded |
| **Preload iklan** | Load di `initState`, show saat butuh — hindari lag |
| **Dispose dengan benar** | Prevent memory leak |
| **Frequency capping** | Batasi frekuensi tampil per user per hari |
| **A/B test posisi** | Coba banner atas vs bawah, interstitial after level vs before |
| **Respect user** | Jangan blok konten utama, berikan opsi skip (rewarded) |

---

## 10. Error Handling Umum

```dart
// Listener error codes berguna untuk debug:
onAdFailedToLoad: (ad, error) {
  print('Error code: ${error.code}'); 
  print('Error domain: ${error.domain}');
  print('Error message: ${error.message}');
  
  // Common codes:
  // 0: ERROR_CODE_INTERNAL_ERROR
  // 1: ERROR_CODE_INVALID_REQUEST  
  // 2: ERROR_CODE_NETWORK_ERROR
  // 3: ERROR_CODE_NO_FILL (Tidak ada iklan tersedia - normal di region tertentu)
}
```

---

## 11. Testing Checklist Sebelum Release

- [ ] Semua Test IDs diganti ke **Real Ad Unit IDs**
- [ ] `AndroidManifest.xml` & `Info.plist` pakai **Real App ID**
- [ ] Test di device fisik (bukan emulator) — iklan butuh Google Play Services
- [ ] Test banner, interstitial, rewarded semua muncul normal
- [ ] Test `dispose()` dipanggil saat screen di-pop/dispose
- [ ] Cek frequency capping tidak terlalu agresif
- [ ] Pastikan app tidak crash kalau iklan gagal load (handle `onAdFailedToLoad`)

---

## Kesimpulan

AdMob di Flutter pakai `google_mobile_ads` cukup powerful. Kunci sukses:
1. **Pilih jenis iklan tepat** untuk konteks app (rewarded untuk game, banner untuk utility)
2. **Preload & dispose** dengan benar
3. **Jangan mengganggu UX** — user yang seneng = retention tinggi = revenue jangka panjang

Artikel selanjutnya: **Flutter #69: Security & Enkripsi** — cara aman menyimpan data sensitif, encrypt local storage, dan best practice keamanan aplikasi.

---

**Coba sendiri!** Share ke sosial media dan tag @ahsai001