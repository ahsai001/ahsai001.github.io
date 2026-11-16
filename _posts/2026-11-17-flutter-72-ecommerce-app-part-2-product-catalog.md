---
layout: post
title: "Flutter #72: E-Commerce App Part 2 - Product Catalog & State Management"
date: 2026-11-17 07:00:00 +0700
categories: [flutter]
tags: [flutter, dart, tutorial, indonesia]
---

Lanjutan dari **Part 1** (setup project, clean architecture, dependency injection), kali ini kita fokus ke **Product Catalog** — menampilkan daftar produk, kategori, pencarian, filter, dan state management yang *scalable*.

---

## Arsitektur State Management untuk Catalog

Di Part 1 kita sudah setup `get_it` dan `Provider`. Sekarang kita pakai **Provider + ChangeNotifier** untuk catalog karena:

| Kebutuhan | Solusi |
|-----------|--------|
| Reactive UI saat data berubah | `ChangeNotifier` + `Consumer` |
| Separation of concerns | Repository pattern (sudah di Part 1) |
| Testable | Mock repository di unit test |
| Caching & pagination | Logic di notifier, UI stay simple |

---

## 1. Domain Model & Repository (Recap)

```dart
// lib/features/catalog/domain/entities/product.dart
class Product {
  final String id;
  final String name;
  final String description;
  final double price;
  final String imageUrl;
  final String categoryId;
  final bool isFeatured;
  final double rating;
  final int reviewCount;
  final DateTime createdAt;

  const Product({
    required this.id,
    required this.name,
    required this.description,
    required this.price,
    required this.imageUrl,
    required this.categoryId,
    this.isFeatured = false,
    this.rating = 0.0,
    this.reviewCount = 0,
    required this.createdAt,
  });

  Product copyWith({
    String? id,
    String? name,
    String? description,
    double? price,
    String? imageUrl,
    String? categoryId,
    bool? isFeatured,
    double? rating,
    int? reviewCount,
    DateTime? createdAt,
  }) => Product(
    id: id ?? this.id,
    name: name ?? this.name,
    description: description ?? this.description,
    price: price ?? this.price,
    imageUrl: imageUrl ?? this.imageUrl,
    categoryId: categoryId ?? this.categoryId,
    isFeatured: isFeatured ?? this.isFeatured,
    rating: rating ?? this.rating,
    reviewCount: reviewCount ?? this.reviewCount,
    createdAt: createdAt ?? this.createdAt,
  );
}
```

```dart
// lib/features/catalog/domain/repositories/product_repository.dart
abstract class ProductRepository {
  Future<Result<List<Product>>> getProducts({
    String? categoryId,
    String? query,
    int page = 1,
    int limit = 20,
  });
  
  Future<Result<List<Category>>> getCategories();
  Future<Result<Product>> getProductDetail(String id);
}
```

`Result<T>` adalah *sealed class* untuk handling success/error tanpa exception (lihat Part 1).

---

## 2. Catalog Notifier — Single Source of Truth

```dart
// lib/features/catalog/presentation/notifiers/catalog_notifier.dart
class CatalogNotifier extends ChangeNotifier {
  final ProductRepository _repository;
  
  CatalogNotifier(this._repository);

  // State
  List<Product> _products = [];
  List<Category> _categories = [];
  Category? _selectedCategory;
  String _searchQuery = '';
  int _currentPage = 1;
  bool _isLoading = false;
  bool _isLoadingMore = false;
  String? _error;
  bool _hasMore = true;

  // Getters
  List<Product> get products => _products;
  List<Category> get categories => _categories;
  Category? get selectedCategory => _selectedCategory;
  String get searchQuery => _searchQuery;
  bool get isLoading => _isLoading;
  bool get isLoadingMore => _isLoadingMore;
  String? get error => _error;
  bool get hasMore => _hasMore;

  // Computed: filtered products
  List<Product> get filteredProducts {
    return _products.where((p) {
      final matchesCategory = _selectedCategory == null || p.categoryId == _selectedCategory!.id;
      final matchesQuery = _searchQuery.isEmpty || 
          p.name.toLowerCase().contains(_searchQuery.toLowerCase()) ||
          p.description.toLowerCase().contains(_searchQuery.toLowerCase());
      return matchesCategory && matchesQuery;
    }).toList();
  }

  // Load initial data
  Future<void> initialize() async {
    await Future.wait([
      _loadCategories(),
      _loadProducts(refresh: true),
    ]);
  }

  Future<void> _loadCategories() async {
    final result = await _repository.getCategories();
    result.when(
      success: (categories) {
        _categories = categories;
        notifyListeners();
      },
      failure: (error) {
        _error = error.message;
        notifyListeners();
      },
    );
  }

  Future<void> _loadProducts({bool refresh = false}) async {
    if (refresh) {
      _currentPage = 1;
      _hasMore = true;
      _isLoading = true;
    } else {
      if (!_hasMore || _isLoadingMore) return;
      _isLoadingMore = true;
    }
    _error = null;
    notifyListeners();

    final result = await _repository.getProducts(
      categoryId: _selectedCategory?.id,
      query: _searchQuery.isEmpty ? null : _searchQuery,
      page: _currentPage,
    );

    result.when(
      success: (newProducts) {
        if (refresh) {
          _products = newProducts;
        } else {
          _products.addAll(newProducts);
        }
        _hasMore = newProducts.length >= 20; // limit default
        _currentPage++;
        _isLoading = false;
        _isLoadingMore = false;
        notifyListeners();
      },
      failure: (error) {
        _error = error.message;
        _isLoading = false;
        _isLoadingMore = false;
        notifyListeners();
      },
    );
  }

  // Public actions
  void selectCategory(Category? category) {
    _selectedCategory = category;
    _loadProducts(refresh: true);
  }

  void setSearchQuery(String query) {
    _searchQuery = query;
    _loadProducts(refresh: true);
  }

  void loadMore() => _loadProducts(refresh: false);

  void clearError() {
    _error = null;
    notifyListeners();
  }
}
```

**Key points:**
- `_products` = data mentah dari API
- `filteredProducts` = computed getter (reactive, no extra state)
- Pagination otomatis via `loadMore()`
- Search & filter trigger `refresh: true` (reset page ke 1)

---

## 3. UI: Catalog Page dengan Search, Filter, Grid

```dart
// lib/features/catalog/presentation/pages/catalog_page.dart
class CatalogPage extends StatelessWidget {
  const CatalogPage({super.key});

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => sl<CatalogNotifier>()..initialize(),
      child: const _CatalogView(),
    );
  }
}

class _CatalogView extends StatelessWidget {
  const _CatalogView();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Produk'),
        actions: [
          IconButton(
            icon: const Icon(Icons.filter_list),
            onPressed: () => _showFilterBottomSheet(context),
          ),
        ],
      ),
      body: Consumer<CatalogNotifier>(
        builder: (context, notifier, _) {
          if (notifier.isLoading && notifier.products.isEmpty) {
            return const Center(child: CircularProgressIndicator());
          }

          if (notifier.error != null && notifier.products.isEmpty) {
            return _ErrorView(
              message: notifier.error!,
              onRetry: () => notifier.initialize(),
            );
          }

          return Column(
            children: [
              _SearchBar(onChanged: notifier.setSearchQuery),
              _CategoryChips(
                categories: notifier.categories,
                selected: notifier.selectedCategory,
                onSelected: notifier.selectCategory,
              ),
              Expanded(
                child: _ProductGrid(
                  products: notifier.filteredProducts,
                  onLoadMore: notifier.loadMore,
                  isLoadingMore: notifier.isLoadingMore,
                  hasMore: notifier.hasMore,
                ),
              ),
            ],
          );
        },
      ),
    );
  }

  void _showFilterBottomSheet(BuildContext context) {
    showModalBottomSheet(
      context: context,
      builder: (_) => const _FilterBottomSheet(),
    );
  }
}
```

### Search Bar Component

```dart
// lib/features/catalog/presentation/widgets/search_bar.dart
class _SearchBar extends StatelessWidget {
  final ValueChanged<String> onChanged;
  const _SearchBar({required this.onChanged});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.all(16),
      child: TextField(
        decoration: InputDecoration(
          hintText: 'Cari produk...',
          prefixIcon: const Icon(Icons.search),
          suffixIcon: IconButton(
            icon: const Icon(Icons.clear),
            onPressed: () => onChanged(''),
          ),
          border: OutlineInputBorder(
            borderRadius: BorderRadius.circular(12),
            borderSide: BorderSide.none,
          ),
          filled: true,
          fillColor: Theme.of(context).colorScheme.surfaceContainerHighest,
        ),
        onChanged: onChanged,
      ),
    );
  }
}
```

### Category Chips (Horizontal Scroll)

```dart
// lib/features/catalog/presentation/widgets/category_chips.dart
class _CategoryChips extends StatelessWidget {
  final List<Category> categories;
  final Category? selected;
  final ValueChanged<Category?> onSelected;

  const _CategoryChips({
    required this.categories,
    required this.selected,
    required this.onSelected,
  });

  @override
  Widget build(BuildContext context) {
    final allCategories = [null, ...categories]; // null = "Semua"

    return SizedBox(
      height: 50,
      child: ListView.separated(
        scrollDirection: Axis.horizontal,
        padding: const EdgeInsets.symmetric(horizontal: 16),
        itemCount: allCategories.length,
        separatorBuilder: (_, __) => const SizedBox(width: 8),
        itemBuilder: (context, index) {
          final cat = allCategories[index];
          final isSelected = selected?.id == cat?.id;
          return FilterChip(
            label: Text(cat?.name ?? 'Semua'),
            selected: isSelected,
            onSelected: (_) => onSelected(cat),
            selectedColor: Theme.of(context).colorScheme.primaryContainer,
            labelStyle: TextStyle(
              color: isSelected
                  ? Theme.of(context).colorScheme.onPrimaryContainer
                  : null,
            ),
          );
        },
      ),
    );
  }
}
```

### Product Grid dengan Infinite Scroll

```dart
// lib/features/catalog/presentation/widgets/product_grid.dart
class _ProductGrid extends StatelessWidget {
  final List<Product> products;
  final VoidCallback onLoadMore;
  final bool isLoadingMore;
  final bool hasMore;

  const _ProductGrid({
    required this.products,
    required this.onLoadMore,
    required this.isLoadingMore,
    required this.hasMore,
  });

  @override
  Widget build(BuildContext context) {
    if (products.isEmpty) {
      return const Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Icon(Icons.inventory_2_outlined, size: 64, color: Colors.grey),
            SizedBox(height: 16),
            Text('Tidak ada produk ditemukan'),
          ],
        ),
      );
    }

    return NotificationListener<ScrollNotification>(
      onNotification: (notification) {
        if (notification.metrics.pixels >= 
            notification.metrics.maxScrollExtent - 200) {
          if (hasMore && !isLoadingMore) onLoadMore();
        }
        return false;
      },
      child: GridView.builder(
        padding: const EdgeInsets.all(16),
        gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 2,
          childAspectRatio: 0.7,
          crossAxisSpacing: 12,
          mainAxisSpacing: 12,
        ),
        itemCount: products.length + (hasMore ? 1 : 0),
        itemBuilder: (context, index) {
          if (index >= products.length) {
            return const Center(child: CircularProgressIndicator());
          }
          return _ProductCard(product: products[index]);
        },
      ),
    );
  }
}
```

### Product Card

```dart
// lib/features/catalog/presentation/widgets/product_card.dart
class _ProductCard extends StatelessWidget {
  final Product product;
  const _ProductCard({required this.product});

  @override
  Widget build(BuildContext context) {
    return Card(
      clipBehavior: Clip.antiAlias,
      child: InkWell(
        onTap: () => context.push('/product/${product.id}'),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Image
            AspectRatio(
              aspectRatio: 1,
              child: CachedNetworkImage(
                imageUrl: product.imageUrl,
                fit: BoxFit.cover,
                placeholder: (_, __) => const Center(
                  child: CircularProgressIndicator(strokeWidth: 2),
                ),
                errorWidget: (_, __, ___) => const Icon(Icons.image_not_supported),
              ),
            ),
            // Info
            Expanded(
              child: Padding(
                padding: const EdgeInsets.all(8),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text(
                      product.name,
                      maxLines: 2,
                      overflow: TextOverflow.ellipsis,
                      style: Theme.of(context).textTheme.titleSmall?.copyWith(
                        fontWeight: FontWeight.w600,
                      ),
                    ),
                    const SizedBox(height: 4),
                    Row(
                      children: [
                        const Icon(Icons.star, size: 14, color: Colors.amber),
                        const SizedBox(width: 2),
                        Text(
                          '${product.rating.toStringAsFixed(1)} (${product.reviewCount})',
                          style: Theme.of(context).textTheme.bodySmall,
                        ),
                      ],
                    ),
                    const Spacer(),
                    Text(
                      NumberFormat.currency(
                        locale: 'id_ID', symbol: 'Rp ', decimalDigits: 0,
                      ).format(product.price),
                      style: Theme.of(context).textTheme.titleMedium?.copyWith(
                        color: Theme.of(context).colorScheme.primary,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                  ],
                ),
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

## 4. Product Detail Page

```dart
// lib/features/catalog/presentation/pages/product_detail_page.dart
class ProductDetailPage extends StatelessWidget {
  final String productId;
  const ProductDetailPage({super.key, required this.productId});

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => ProductDetailNotifier(sl<ProductRepository>(), productId),
      child: Consumer<ProductDetailNotifier>(
        builder: (context, notifier, _) {
          if (notifier.isLoading) {
            return const Scaffold(body: Center(child: CircularProgressIndicator()));
          }
          if (notifier.error != null) {
            return _ErrorView(message: notifier.error!, onRetry: notifier.load);
          }
          return _ProductDetailView(product: notifier.product!);
        },
      ),
    );
  }
}

class _ProductDetailView extends StatelessWidget {
  final Product product;
  const _ProductDetailView({required this.product});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomScrollView(
        slivers: [
          SliverAppBar(
            expandedHeight: 300,
            flexibleSpace: FlexibleSpaceBar(
              background: CachedNetworkImage(
                imageUrl: product.imageUrl,
                fit: BoxFit.cover,
              ),
            ),
          ),
          SliverToBoxAdapter(
            child: Padding(
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(product.name, style: Theme.of(context).textTheme.headlineSmall),
                  const SizedBox(height: 8),
                  Row(
                    children: [
                      const Icon(Icons.star, color: Colors.amber),
                      Text(' ${product.rating} (${product.reviewCount} ulasan)'),
                    ],
                  ),
                  const SizedBox(height: 16),
                  Text(
                    NumberFormat.currency(locale: 'id_ID', symbol: 'Rp ', decimalDigits: 0)
                        .format(product.price),
                    style: Theme.of(context).textTheme.headlineMedium?.copyWith(
                      color: Theme.of(context).colorScheme.primary,
                    ),
                  ),
                  const SizedBox(height: 24),
                  Text('Deskripsi', style: Theme.of(context).textTheme.titleLarge),
                  const SizedBox(height: 8),
                  Text(product.description),
                  const SizedBox(height: 100), // space for bottom bar
                ],
              ),
            ),
          ),
        ],
      ),
      bottomNavigationBar: _AddToCartBar(product: product),
    );
  }
}
```

### Bottom Add-to-Cart Bar

```dart
class _AddToCartBar extends ConsumerWidget {
  final Product product;
  const _AddToCartBar({required this.product});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Cart notifier dari feature cart (akan dibahas Part 3)
    final cart = ref.watch(cartNotifierProvider);
    
    return SafeArea(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Row(
          children: [
            // Quantity selector
            Container(
              decoration: BoxDecoration(
                border: Border.all(color: Colors.grey.shade300),
                borderRadius: BorderRadius.circular(8),
              ),
              child: Row(
                children: [
                  IconButton(
                    icon: const Icon(Icons.remove),
                    onPressed: () => cart.decrement(product),
                  ),
                  Text('${cart.getQuantity(product.id)}'),
                  IconButton(
                    icon: const Icon(Icons.add),
                    onPressed: () => cart.increment(product),
                  ),
                ],
              ),
            ),
            const SizedBox(width: 12),
            // Add to cart button
            Expanded(
              child: FilledButton.icon(
                icon: const Icon(Icons.shopping_cart),
                label: const Text('Tambah ke Keranjang'),
                onPressed: () => cart.addItem(product),
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

## 5. Routing Setup (GoRouter)

```dart
// lib/core/routing/app_router.dart
final appRouter = GoRouter(
  routes: [
    GoRoute(
      path: '/catalog',
      builder: (_, __) => const CatalogPage(),
    ),
    GoRoute(
      path: '/product/:id',
      builder: (_, state) => ProductDetailPage(productId: state.pathParameters['id']!),
    ),
    // Cart, checkout, dll. akan ditambah di Part 3-4
  ],
);
```

---

## 6. Testing: Unit Test untuk CatalogNotifier

```dart
// test/features/catalog/catalog_notifier_test.dart
void main() {
  late MockProductRepository mockRepo;
  late CatalogNotifier notifier;

  setUp(() {
    mockRepo = MockProductRepository();
    notifier = CatalogNotifier(mockRepo);
  });

  test('initialize loads categories and products', () async {
    when(mockRepo.getCategories()).thenAnswer((_) async => Result.success([category1]));
    when(mockRepo.getProducts(page: 1)).thenAnswer((_) async => Result.success([product1]));

    await notifier.initialize();

    expect(notifier.categories, [category1]);
    expect(notifier.products, [product1]);
    expect(notifier.isLoading, false);
  });

  test('selectCategory refreshes products', () async {
    when(mockRepo.getProducts(categoryId: 'cat1', page: 1))
        .thenAnswer((_) async => Result.success([product2]));

    notifier.selectCategory(category1);

    await untilCalled(mockRepo.getProducts(categoryId: 'cat1', page: 1));
    expect(notifier.products, [product2]);
  });

  test('search query triggers refresh', () async {
    when(mockRepo.getProducts(query: 'baju', page: 1))
        .thenAnswer((_) async => Result.success([product3]));

    notifier.setSearchQuery('baju');

    await untilCalled(mockRepo.getProducts(query: 'baju', page: 1));
    expect(notifier.searchQuery, 'baju');
  });
}
```

---

## 7. Checklist Part 2

| Fitur | Status |
|-------|--------|
| Product list dengan grid | ✅ |
| Kategori horizontal chips | ✅ |
| Search real-time | ✅ |
| Infinite scroll pagination | ✅ |
| Product detail dengan SliverAppBar | ✅ |
| Bottom add-to-cart bar | ✅ |
| Unit test notifier | ✅ |
| Error & empty states | ✅ |

---

## Selanjutnya: Part 3 — Cart & Checkout

Di **Part 3** kita bangun:
- Cart state management (persist ke local storage)
- Quantity management, subtotal, total
- Checkout flow: alamat, pengiriman, pembayaran
- Integrasi Midtrans / Xendit (payment gateway Indonesia)

**File baru:** `cart_notifier.dart`, `cart_page.dart`, `checkout_page.dart`, `payment_service.dart`.

---

Coba sendiri! Share ke sosial media dan tag **@ahsai001**