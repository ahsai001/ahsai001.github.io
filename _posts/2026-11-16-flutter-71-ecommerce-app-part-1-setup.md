---
layout: post
title: "Flutter #71: E-Commerce App Part 1 - Project Setup & Architecture"
date: 2026-11-16 07:00:00 +0700
categories: [flutter]
tags: [flutter, dart, tutorial, indonesia]
---

Kita memasuki **Fase 6: PROJECT** — rangkaian proyek nyata yang menggabungkan semua skill dari Fase 1-5. Project pertama: **E-Commerce App (4 part)**.

Di part 1 ini, kita akan:
1. Setup struktur project modular (Clean Architecture + Repository Pattern)
2. Konfigurasi dependencies utama (get_it, dio, bloc, hive, dll)
3. Siapkan Theme, Routing, dan Core Module
4. Buat model data dasar (Product, Category, Cart)

---

## 1. Inisialisasi Project

```bash
flutter create ecommerce_app --org com.ahsai --project-name ecommerce_app
cd ecommerce_app
```

Buka `pubspec.yaml` dan tambahkan dependencies:

```yaml
dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.8
  
  # State Management
  flutter_bloc: ^8.1.6
  
  # Dependency Injection
  get_it: ^7.7.0
  injectable: ^2.4.2
  
  # Network
  dio: ^5.5.0+1
  connectivity_plus: ^6.0.3
  
  # Local Storage
  hive: ^2.2.3
  hive_flutter: ^1.1.0
  shared_preferences: ^2.2.3
  
  # Utils
  equatable: ^2.0.5
  intl: ^0.19.0
  cached_network_image: ^3.3.1
  
  # Routing
  go_router: ^14.2.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^4.0.0
  build_runner: ^2.4.11
  injectable_generator: ^2.6.1
  hive_generator: ^2.0.1
```

Jalankan `flutter pub get` lalu generate code:

```bash
flutter pub run build_runner build --delete-conflicting-outputs
```

---

## 2. Struktur Folder (Clean Architecture)

```
lib/
├── core/
│   ├── constants/
│   │   ├── app_constants.dart
│   │   └── api_endpoints.dart
│   ├── di/
│   │   └── injection.dart
│   ├── error/
│   │   ├── exceptions.dart
│   │   └── failures.dart
│   ├── network/
│   │   ├── api_client.dart
│   │   └── network_info.dart
│   ├── router/
│   │   └── app_router.dart
│   ├── theme/
│   │   ├── app_theme.dart
│   │   └── app_colors.dart
│   └── utils/
│       ├── formatters.dart
│       └── validators.dart
├── features/
│   ├── auth/
│   ├── cart/
│   ├── category/
│   ├── home/
│   ├── product/
│   ├── profile/
│   └── checkout/
├── shared/
│   ├── widgets/
│   │   ├── buttons/
│   │   ├── inputs/
│   │   └── loaders/
│   └── models/
│       ├── product.dart
│       ├── category.dart
│       └── cart_item.dart
└── main.dart
```

---

## 3. Core Module Implementation

### 3.1 Constants & API Endpoints

```dart
// lib/core/constants/app_constants.dart
class AppConstants {
  static const String appName = 'AhsaiShop';
  static const String baseUrl = 'https://api.ahsai.my.id/v1';
  static const Duration apiTimeout = Duration(seconds: 30);
  static const int productsPerPage = 20;
}
```

```dart
// lib/core/constants/api_endpoints.dart
class ApiEndpoints {
  static const String products = '/products';
  static const String categories = '/categories';
  static const String auth = '/auth';
  static const String cart = '/cart';
  static const String orders = '/orders';
  static const String users = '/users';
}
```

### 3.2 Error Handling (Failures & Exceptions)

```dart
// lib/core/error/exceptions.dart
class ServerException implements Exception {
  final String message;
  final int? statusCode;
  const ServerException(this.message, {this.statusCode});
  @override String toString() => 'ServerException: $message';
}

class CacheException implements Exception {
  final String message;
  const CacheException(this.message);
  @override String toString() => 'CacheException: $message';
}

class NetworkException implements Exception {
  final String message;
  const NetworkException(this.message);
  @override String toString() => 'NetworkException: $message';
}
```

```dart
// lib/core/error/failures.dart
import 'package:equatable/equatable.dart';

abstract class Failure extends Equatable {
  final String message;
  const Failure(this.message);
  @override List<Object?> get props => [message];
}

class ServerFailure extends Failure {
  const ServerFailure(super.message);
}

class CacheFailure extends Failure {
  const CacheFailure(super.message);
}

class NetworkFailure extends Failure {
  const NetworkFailure(super.message);
}

class ValidationFailure extends Failure {
  final Map<String, String> errors;
  const ValidationFailure(this.errors) : super('Validation failed');
  @override List<Object?> get props => [message, errors];
}
```

### 3.3 Network Layer (Dio Client)

```dart
// lib/core/network/api_client.dart
import 'package:dio/dio.dart';
import 'package:injectable/injectable.dart';
import '../constants/app_constants.dart';
import '../constants/api_endpoints.dart';
import '../error/exceptions.dart';

@LazySingleton()
class ApiClient {
  final Dio _dio;

  ApiClient() : _dio = Dio(BaseOptions(
    baseUrl: AppConstants.baseUrl,
    connectTimeout: AppConstants.apiTimeout,
    receiveTimeout: AppConstants.apiTimeout,
    headers: {'Content-Type': 'application/json'},
  )) {
    _dio.interceptors.add(LogInterceptor(
      requestBody: true,
      responseBody: true,
      logPrint: (obj) => print('[DIO] $obj'),
    ));
    _dio.interceptors.add(InterceptorsWrapper(
      onError: (error, handler) {
        if (error.type == DioExceptionType.connectionTimeout ||
            error.type == DioExceptionType.receiveTimeout) {
          return handler.reject(NetworkException('Koneksi timeout'));
        }
        if (error.response?.statusCode != null) {
          return handler.reject(ServerException(
            error.response?.data['message'] ?? 'Server error',
            statusCode: error.response?.statusCode,
          ));
        }
        return handler.next(error);
      },
    ));
  }

  Dio get dio => _dio;

  // Helper methods
  Future<Response> get(String path, {Map<String, dynamic>? query}) =>
      _dio.get(path, queryParameters: query);

  Future<Response> post(String path, {dynamic data}) =>
      _dio.post(path, data: data);

  Future<Response> put(String path, {dynamic data}) =>
      _dio.put(path, data: data);

  Future<Response> delete(String path) => _dio.delete(path);
}
```

```dart
// lib/core/network/network_info.dart
import 'package:connectivity_plus/connectivity_plus.dart';
import 'package:injectable/injectable.dart';

@LazySingleton()
class NetworkInfo {
  final Connectivity _connectivity;
  NetworkInfo(this._connectivity);

  Future<bool> get isConnected async {
    final result = await _connectivity.checkConnectivity();
    return result != ConnectivityResult.none;
  }
}
```

### 3.4 Dependency Injection Setup

```dart
// lib/core/di/injection.dart
import 'package:get_it/get_it.dart';
import 'package:injectable/injectable.dart';
import 'injection.config.dart';

final getIt = GetIt.instance;

@InjectableInit(
  initializerName: 'init',
  preferRelativeImports: true,
  asExtension: false,
)
Future<void> configureDependencies() async {
  await getIt.init();
}
```

```dart
// lib/core/di/injection.config.dart (auto-generated)
// Jangan edit manual - di-generate oleh build_runner
```

### 3.5 Theme & Colors

```dart
// lib/core/theme/app_colors.dart
import 'package:flutter/material.dart';

class AppColors {
  // Primary
  static const Color primary = Color(0xFF6366F1); // Indigo
  static const Color primaryLight = Color(0xFF818CF8);
  static const Color primaryDark = Color(0xFF4F46E5);
  
  // Secondary
  static const Color secondary = Color(0xFFEC4899); // Pink
  static const Color secondaryLight = Color(0xFFF472B6);
  
  // Neutral
  static const Color background = Color(0xFFFAFAFA);
  static const Color surface = Color(0xFFFFFFFF);
  static const Color textPrimary = Color(0xFF111827);
  static const Color textSecondary = Color(0xFF6B7280);
  static const Color divider = Color(0xFFE5E7EB);
  
  // Status
  static const Color success = Color(0xFF10B981);
  static const Color warning = Color(0xFFF59E0B);
  static const Color error = Color(0xFFEF4444);
  
  // Gradients
  static const LinearGradient primaryGradient = LinearGradient(
    colors: [primary, primaryDark],
    begin: Alignment.topLeft,
    end: Alignment.bottomRight,
  );
}
```

```dart
// lib/core/theme/app_theme.dart
import 'package:flutter/material.dart';
import 'app_colors.dart';

class AppTheme {
  static ThemeData get lightTheme => ThemeData(
    useMaterial3: true,
    colorScheme: ColorScheme.fromSeed(
      seedColor: AppColors.primary,
      brightness: Brightness.light,
    ),
    scaffoldBackgroundColor: AppColors.background,
    appBarTheme: const AppBarTheme(
      elevation: 0,
      centerTitle: true,
      backgroundColor: AppColors.surface,
      foregroundColor: AppColors.textPrimary,
      titleTextStyle: TextStyle(
        fontSize: 20,
        fontWeight: FontWeight.w600,
        color: AppColors.textPrimary,
      ),
    ),
    cardTheme: CardThemeData(
      elevation: 0,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(16),
        side: const BorderSide(color: AppColors.divider),
      ),
      color: AppColors.surface,
    ),
    elevatedButtonTheme: ElevatedButtonThemeData(
      style: ElevatedButton.styleFrom(
        backgroundColor: AppColors.primary,
        foregroundColor: Colors.white,
        elevation: 0,
        padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 14),
        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
        textStyle: const TextStyle(fontSize: 16, fontWeight: FontWeight.w600),
      ),
    ),
    inputDecorationTheme: InputDecorationTheme(
      filled: true,
      fillColor: AppColors.surface,
      contentPadding: const EdgeInsets.symmetric(horizontal: 16, vertical: 16),
      border: OutlineInputBorder(
        borderRadius: BorderRadius.circular(12),
        borderSide: const BorderSide(color: AppColors.divider),
      ),
      enabledBorder: OutlineInputBorder(
        borderRadius: BorderRadius.circular(12),
        borderSide: const BorderSide(color: AppColors.divider),
      ),
      focusedBorder: OutlineInputBorder(
        borderRadius: BorderRadius.circular(12),
        borderSide: const BorderSide(color: AppColors.primary, width: 2),
      ),
      errorBorder: OutlineInputBorder(
        borderRadius: BorderRadius.circular(12),
        borderSide: const BorderSide(color: AppColors.error),
      ),
      labelStyle: const TextStyle(color: AppColors.textSecondary),
      hintStyle: const TextStyle(color: AppColors.textSecondary),
    ),
    textTheme: const TextTheme(
      headlineLarge: TextStyle(fontSize: 32, fontWeight: FontWeight.bold, color: AppColors.textPrimary),
      headlineMedium: TextStyle(fontSize: 28, fontWeight: FontWeight.bold, color: AppColors.textPrimary),
      titleLarge: TextStyle(fontSize: 22, fontWeight: FontWeight.w600, color: AppColors.textPrimary),
      titleMedium: TextStyle(fontSize: 18, fontWeight: FontWeight.w600, color: AppColors.textPrimary),
      bodyLarge: TextStyle(fontSize: 16, color: AppColors.textPrimary),
      bodyMedium: TextStyle(fontSize: 14, color: AppColors.textSecondary),
      labelLarge: TextStyle(fontSize: 14, fontWeight: FontWeight.w600, color: AppColors.textPrimary),
    ),
  );
}
```

### 3.6 Routing (GoRouter)

```dart
// lib/core/router/app_router.dart
import 'package:go_router/go_router.dart';
import 'package:injectable/injectable.dart';

@LazySingleton()
class AppRouter {
  final GoRouter router;

  AppRouter() : router = GoRouter(
    initialLocation: '/',
    routes: [
      GoRoute(
        path: '/',
        name: 'home',
        builder: (context, state) => const HomePage(),
        routes: [
          GoRoute(
            path: 'product/:id',
            name: 'product-detail',
            builder: (context, state) => ProductDetailPage(
              productId: state.pathParameters['id']!,
            ),
          ),
          GoRoute(
            path: 'category/:id',
            name: 'category-products',
            builder: (context, state) => CategoryProductsPage(
              categoryId: state.pathParameters['id']!,
            ),
          ),
        ],
      ),
      GoRoute(
        path: '/cart',
        name: 'cart',
        builder: (context, state) => const CartPage(),
      ),
      GoRoute(
        path: '/profile',
        name: 'profile',
        builder: (context, state) => const ProfilePage(),
      ),
      GoRoute(
        path: '/checkout',
        name: 'checkout',
        builder: (context, state) => const CheckoutPage(),
      ),
      GoRoute(
        path: '/login',
        name: 'login',
        builder: (context, state) => const LoginPage(),
      ),
      GoRoute(
        path: '/register',
        name: 'register',
        builder: (context, state) => const RegisterPage(),
      ),
    ],
  );
}
```

---

## 4. Shared Models (Hive Ready)

```dart
// lib/shared/models/product.dart
import 'package:hive/hive.dart';
import 'package:equatable/equatable.dart';

part 'product.g.dart';

@HiveType(typeId: 0)
class Product extends Equatable {
  @HiveField(0)
  final String id;
  
  @HiveField(1)
  final String name;
  
  @HiveField(2)
  final String description;
  
  @HiveField(3)
  final double price;
  
  @HiveField(4)
  final double? discountPrice;
  
  @HiveField(5)
  final String imageUrl;
  
  @HiveField(6)
  final List<String> galleryImages;
  
  @HiveField(7)
  final String categoryId;
  
  @HiveField(8)
  final String categoryName;
  
  @HiveField(9)
  final int stock;
  
  @HiveField(10)
  final double rating;
  
  @HiveField(11)
  final int reviewCount;
  
  @HiveField(12)
  final bool isFeatured;
  
  @HiveField(13)
  final DateTime createdAt;

  const Product({
    required this.id,
    required this.name,
    required this.description,
    required this.price,
    this.discountPrice,
    required this.imageUrl,
    required this.galleryImages,
    required this.categoryId,
    required this.categoryName,
    required this.stock,
    required this.rating,
    required this.reviewCount,
    required this.isFeatured,
    required this.createdAt,
  });

  double get effectivePrice => discountPrice ?? price;
  bool get hasDiscount => discountPrice != null && discountPrice! < price;
  int get discountPercentage => hasDiscount 
      ? ((1 - effectivePrice / price) * 100).round() 
      : 0;

  factory Product.fromJson(Map<String, dynamic> json) => Product(
    id: json['id'] as String,
    name: json['name'] as String,
    description: json['description'] as String,
    price: (json['price'] as num).toDouble(),
    discountPrice: json['discount_price'] != null 
        ? (json['discount_price'] as num).toDouble() 
        : null,
    imageUrl: json['image_url'] as String,
    galleryImages: List<String>.from(json['gallery_images'] ?? []),
    categoryId: json['category_id'] as String,
    categoryName: json['category_name'] as String,
    stock: json['stock'] as int,
    rating: (json['rating'] as num?)?.toDouble() ?? 0.0,
    reviewCount: json['review_count'] as int? ?? 0,
    isFeatured: json['is_featured'] as bool? ?? false,
    createdAt: DateTime.parse(json['created_at'] as String),
  );

  Map<String, dynamic> toJson() => {
    'id': id,
    'name': name,
    'description': description,
    'price': price,
    'discount_price': discountPrice,
    'image_url': imageUrl,
    'gallery_images': galleryImages,
    'category_id': categoryId,
    'category_name': categoryName,
    'stock': stock,
    'rating': rating,
    'review_count': reviewCount,
    'is_featured': isFeatured,
    'created_at': createdAt.toIso8601String(),
  };

  @override
  List<Object?> get props => [
    id, name, description, price, discountPrice, imageUrl,
    galleryImages, categoryId, categoryName, stock, rating,
    reviewCount, isFeatured, createdAt,
  ];
}
```

```dart
// lib/shared/models/category.dart
import 'package:hive/hive.dart';
import 'package:equatable/equatable.dart';

part 'category.g.dart';

@HiveType(typeId: 1)
class Category extends Equatable {
  @HiveField(0)
  final String id;
  
  @HiveField(1)
  final String name;
  
  @HiveField(2)
  final String? imageUrl;
  
  @HiveField(3)
  final int productCount;
  
  @HiveField(4)
  final int sortOrder;

  const Category({
    required this.id,
    required this.name,
    this.imageUrl,
    required this.productCount,
    required this.sortOrder,
  });

  factory Category.fromJson(Map<String, dynamic> json) => Category(
    id: json['id'] as String,
    name: json['name'] as String,
    imageUrl: json['image_url'] as String?,
    productCount: json['product_count'] as int? ?? 0,
    sortOrder: json['sort_order'] as int? ?? 0,
  );

  Map<String, dynamic> toJson() => {
    'id': id,
    'name': name,
    'image_url': imageUrl,
    'product_count': productCount,
    'sort_order': sortOrder,
  };

  @override
  List<Object?> get props => [id, name, imageUrl, productCount, sortOrder];
}
```

```dart
// lib/shared/models/cart_item.dart
import 'package:hive/hive.dart';
import 'package:equatable/equatable.dart';
import 'product.dart';

part 'cart_item.g.dart';

@HiveType(typeId: 2)
class CartItem extends Equatable {
  @HiveField(0)
  final String id;
  
  @HiveField(1)
  final Product product;
  
  @HiveField(2)
  final int quantity;
  
  @HiveField(3)
  final DateTime addedAt;

  const CartItem({
    required this.id,
    required this.product,
    required this.quantity,
    required this.addedAt,
  });

  double get subtotal => product.effectivePrice * quantity;

  CartItem copyWith({int? quantity}) => CartItem(
    id: id,
    product: product,
    quantity: quantity ?? this.quantity,
    addedAt: addedAt,
  );

  factory CartItem.fromProduct(Product product, {int quantity = 1}) => CartItem(
    id: '${product.id}_${DateTime.now().millisecondsSinceEpoch}',
    product: product,
    quantity: quantity,
    addedAt: DateTime.now(),
  );

  Map<String, dynamic> toJson() => {
    'id': id,
    'product_id': product.id,
    'quantity': quantity,
    'added_at': addedAt.toIso8601String(),
  };

  @override
  List<Object?> get props => [id, product, quantity, addedAt];
}
```

---

## 5. Main Entry Point

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:hive_flutter/hive_flutter.dart';
import 'package:get_it/get_it.dart';

import 'core/di/injection.dart';
import 'core/router/app_router.dart';
import 'core/theme/app_theme.dart';
import 'shared/models/product.dart';
import 'shared/models/category.dart'
import 'shared/models/cart_item.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  // Initialize Hive
  await Hive.initFlutter();
  Hive.registerAdapter(ProductAdapter());
  Hive.registerAdapter(CategoryAdapter());
  Hive.registerAdapter(CartItemAdapter());
  await Hive.openBox<Product>('products_cache');
  await Hive.openBox<Category>('categories_cache');
  await Hive.openBox<CartItem>('cart_box');
  
  // Configure Dependencies
  await configureDependencies();
  
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    final router = getIt<AppRouter>().router;
    
    return MaterialApp.router(
      title: 'AhsaiShop',
      debugShowCheckedModeBanner: false,
      theme: AppTheme.lightTheme,
      routerConfig: router,
    );
  }
}
```

---

## 6. Placeholder Pages (Untuk Routing)

Buat file minimal di `lib/features/home/presentation/pages/home_page.dart`:

```dart
// lib/features/home/presentation/pages/home_page.dart
import 'package:flutter/material.dart';

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('AhsaiShop')),
      body: const Center(child: Text('Home Page - Coming Soon')),
      bottomNavigationBar: BottomNavigationBar(
        items: const [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Home'),
          BottomNavigationBarItem(icon: Icon(Icons.category), label: 'Kategori'),
          BottomNavigationBarItem(icon: Icon(Icons.shopping_cart), label: 'Keranjang'),
          BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Profil'),
        ],
      ),
    );
  }
}
```

Buat placeholder serupa untuk `ProductDetailPage`, `CategoryProductsPage`, `CartPage`, `ProfilePage`, `CheckoutPage`, `LoginPage`, `RegisterPage`.

---

## 7. Verifikasi Setup

Jalankan project:

```bash
flutter run
```

Seharusnya muncul halaman Home dengan bottom navigation. Semua routing, theme, DI, dan model sudah siap.

---

## Kesimpulan Part 1

Di part 1 ini kita sudah:
- ✅ Setup Clean Architecture structure
- ✅ Konfigurasi dependencies modern (BLoC, GetIt, Dio, Hive, GoRouter)
- ✅ Core module: Constants, Error Handling, Network, DI, Theme, Router
- ✅ Shared Models dengan Hive adapters untuk caching offline
- ✅ Main entry point dengan Hive initialization

**Part 2** akan membahas: **Repository Pattern, Data Sources, dan BLoC untuk Home/Produk**.

---

Coba sendiri! Share ke sosial media dan tag @ahsai001