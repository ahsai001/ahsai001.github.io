---
layout: post
title: "Flutter #73: E-Commerce App Part 3 - Keranjang Belanja & Checkout"
date: 2026-11-20 07:00:00 +0700
categories: [flutter]
tags: [flutter, dart, tutorial, indonesia, ecommerce, provider, cart, checkout]
---

Melanjutkan part 2 dimana kita sudah bikin **Product Listing** dan **Product Detail**, hari ini kita fokus ke fitur inti e-commerce: **Keranjang Belanja (Cart)** dan **Checkout Flow**.

Kita pakai **Provider** untuk state management keranjang — simpel, reactive, dan cocok untuk skala menengah.

---

## 1. Model Cart Item

Buat file baru `lib/models/cart_item.dart`:

```dart
import 'product.dart';

class CartItem {
  final Product product;
  int quantity;

  CartItem({required this.product, this.quantity = 1});

  double get totalPrice => product.price * quantity;

  CartItem copyWith({int? quantity}) {
    return CartItem(
      product: product,
      quantity: quantity ?? this.quantity,
    );
  }
}
```

`copyWith` berguna saat update quantity lewat Provider.

---

## 2. Cart Provider

`lib/providers/cart_provider.dart`:

```dart
import 'package:flutter/foundation.dart';
import '../models/cart_item.dart';

class CartProvider with ChangeNotifier {
  final Map<String, CartItem> _items = {};

  Map<String, CartItem> get items => {..._items};

  int get itemCount => _items.values.fold(0, (sum, item) => sum + item.quantity);

  double get totalAmount => _items.values.fold(0.0, (sum, item) => sum + item.totalPrice);

  void addItem(Product product) {
    if (_items.containsKey(product.id)) {
      _items[product.id]!.quantity++;
    } else {
      _items[product.id] = CartItem(product: product);
    }
    notifyListeners();
  }

  void removeItem(String productId) {
    _items.remove(productId);
    notifyListeners();
  }

  void updateQuantity(String productId, int quantity) {
    if (quantity <= 0) {
      removeItem(productId);
      return;
    }
    _items[productId] = _items[productId]!.copyWith(quantity: quantity);
    notifyListeners();
  }

  void clear() {
    _items.clear();
    notifyListeners();
  }
}
```

**Poin penting:**
- Map key = `product.id` biar lookup O(1)
- `notifyListeners()` dipanggil setiap mutasi
- `updateQuantity` handle hapus otomatis kalau quantity ≤ 0

---

## 3. Register Provider di main.dart

```dart
MultiProvider(
  providers: [
    ChangeNotifierProvider(create: (_) => ProductProvider()),
    ChangeNotifierProvider(create: (_) => CartProvider()),
  ],
  child: MyApp(),
)
```

---

## 4. Cart Screen

`lib/screens/cart_screen.dart`:

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../providers/cart_provider.dart';
import '../models/cart_item.dart';
import 'checkout_screen.dart';

class CartScreen extends StatelessWidget {
  const CartScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final cart = context.watch<CartProvider>();
    final items = cart.items.values.toList();

    return Scaffold(
      appBar: AppBar(
        title: const Text('Keranjang'),
        actions: [
          if (items.isNotEmpty)
            IconButton(
              icon: const Icon(Icons.delete_sweep),
              onPressed: () => _showClearDialog(context, cart),
            ),
        ],
      ),
      body: items.isEmpty
          ? _buildEmptyCart()
          : _buildCartList(items, cart),
      bottomNavigationBar: items.isEmpty ? null : _buildCheckoutBar(context, cart),
    );
  }

  Widget _buildEmptyCart() {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Icon(Icons.shopping_cart_outlined, size: 80, color: Colors.grey[400]),
          const SizedBox(height: 16),
          Text('Keranjang kosong', style: Theme.of(context).textTheme.headlineSmall?.copyWith(color: Colors.grey[600])),
          const SizedBox(height: 8),
          Text('Yuk belanja!', style: TextStyle(color: Colors.grey[500])),
          const SizedBox(height: 24),
          ElevatedButton.icon(
            icon: const Icon(Icons.shopping_bag),
            label: const Text('Mulai Belanja'),
            onPressed: () => Navigator.pushReplacementNamed(context, '/products'),
          ),
        ],
      ),
    );
  }

  Widget _buildCartList(List<CartItem> items, CartProvider cart) {
    return ListView.builder(
      padding: const EdgeInsets.all(16),
      itemCount: items.length,
      itemBuilder: (ctx, i) => _CartItemTile(item: items[i], cart: cart),
    );
  }

  Widget _buildCheckoutBar(BuildContext context, CartProvider cart) {
    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: Colors.white,
        boxShadow: [BoxShadow(color: Colors.black12, blurRadius: 8, offset: const Offset(0, -2))],
      ),
      child: SafeArea(
        child: Row(
          children: [
            Expanded(
              child: Column(
                mainAxisSize: MainAxisSize.min,
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text('Total', style: TextStyle(color: Colors.grey[600], fontSize: 14)),
                  Text(
                    'Rp ${cart.totalAmount.toStringAsFixed(0)}',
                    style: const TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
                  ),
                ],
              ),
            ),
            ElevatedButton(
              style: ElevatedButton.styleFrom(
                padding: const EdgeInsets.symmetric(horizontal: 32, vertical: 16),
                shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
              ),
              onPressed: () => Navigator.push(
                context,
                MaterialPageRoute(builder: (_) => const CheckoutScreen()),
              ),
              child: const Text('Checkout', style: TextStyle(fontSize: 16)),
            ),
          ],
        ),
      ),
    );
  }

  void _showClearDialog(BuildContext context, CartProvider cart) {
    showDialog(
      context: context,
      builder: (_) => AlertDialog(
        title: const Text('Kosongkan Keranjang?'),
        content: const Text('Semua item akan dihapus.'),
        actions: [
          TextButton(onPressed: () => Navigator.pop(context), child: const Text('Batal')),
          TextButton(
            onPressed: () {
              cart.clear();
              Navigator.pop(context);
            },
            child: const Text('Hapus', style: TextStyle(color: Colors.red)),
          ),
        ],
      ),
    );
  }
}

class _CartItemTile extends StatelessWidget {
  final CartItem item;
  final CartProvider cart;

  const _CartItemTile({required this.item, required this.cart});

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.only(bottom: 12),
      child: Padding(
        padding: const EdgeInsets.all(12),
        child: Row(
          children: [
            ClipRRect(
              borderRadius: BorderRadius.circular(8),
              child: Image.network(
                item.product.imageUrl,
                width: 70,
                height: 70,
                fit: BoxFit.cover,
                errorBuilder: (_, __, ___) => Container(
                  width: 70,
                  height: 70,
                  color: Colors.grey[200],
                  child: const Icon(Icons.image_not_supported),
                ),
              ),
            ),
            const SizedBox(width: 12),
            Expanded(
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(item.product.name, style: const TextStyle(fontWeight: FontWeight.w600, fontSize: 16)),
                  const SizedBox(height: 4),
                  Text('Rp ${item.product.price.toStringAsFixed(0)}', style: TextStyle(color: Colors.grey[600])),
                ],
              ),
            ),
            _QuantityControl(item: item, cart: cart),
          ],
        ),
      ),
    );
  }
}

class _QuantityControl extends StatelessWidget {
  final CartItem item;
  final CartProvider cart;

  const _QuantityControl({required this.item, required this.cart});

  @override
  Widget build(BuildContext context) {
    return Container(
      decoration: BoxDecoration(
        border: Border.all(color: Colors.grey[300]!),
        borderRadius: BorderRadius.circular(8),
      ),
      child: Row(
        mainAxisSize: MainAxisSize.min,
        children: [
          IconButton(
            icon: const Icon(Icons.remove, size: 20),
            onPressed: () => cart.updateQuantity(item.product.id, item.quantity - 1),
            constraints: const BoxConstraints(minWidth: 40, minHeight: 40),
          ),
          Text('${item.quantity}', style: const TextStyle(fontSize: 16, fontWeight: FontWeight.w500)),
          IconButton(
            icon: const Icon(Icons.add, size: 20),
            onPressed: () => cart.updateQuantity(item.product.id, item.quantity + 1),
            constraints: const BoxConstraints(minWidth: 40, minHeight: 40),
          ),
        ],
      ),
    );
  }
}
```

---

## 5. Checkout Screen

`lib/screens/checkout_screen.dart`:

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../providers/cart_provider.dart';
import 'order_success_screen.dart';

class CheckoutScreen extends StatefulWidget {
  const CheckoutScreen({super.key});

  @override
  State<CheckoutScreen> createState() => _CheckoutScreenState();
}

class _CheckoutScreenState extends State<CheckoutScreen> {
  final _formKey = GlobalKey<FormState>();
  final _nameController = TextEditingController();
  final _phoneController = TextEditingController();
  final _addressController = TextEditingController();
  final _notesController = TextEditingController();
  bool _isProcessing = false;

  @override
  void dispose() {
    _nameController.dispose();
    _phoneController.dispose();
    _addressController.dispose();
    _notesController.dispose();
    super.dispose();
  }

  Future<void> _submitOrder() async {
    if (!_formKey.currentState!.validate()) return;

    setState(() => _isProcessing = true);

    // Simulasi API call
    await Future.delayed(const Duration(seconds: 2));

    if (!mounted) return;

    final cart = context.read<CartProvider>();
    cart.clear();

    Navigator.pushReplacement(
      context,
      MaterialPageRoute(builder: (_) => const OrderSuccessScreen()),
    );
  }

  @override
  Widget build(BuildContext context) {
    final cart = context.watch<CartProvider>();

    return Scaffold(
      appBar: AppBar(title: const Text('Checkout')),
      body: Form(
        key: _formKey,
        child: ListView(
          padding: const EdgeInsets.all(16),
          children: [
            _buildSectionTitle('Detail Pesanan'),
            ...cart.items.values.map((item) => _OrderItemRow(item: item)),
            const Divider(height: 32),
            _buildSectionTitle('Alamat Pengiriman'),
            TextFormField(
              controller: _nameController,
              decoration: const InputDecoration(labelText: 'Nama Lengkap', border: OutlineInputBorder()),
              validator: (v) => v!.isEmpty ? 'Wajib diisi' : null,
            ),
            const SizedBox(height: 12),
            TextFormField(
              controller: _phoneController,
              decoration: const InputDecoration(labelText: 'Nomor HP', border: OutlineInputBorder()),
              keyboardType: TextInputType.phone,
              validator: (v) => v!.isEmpty ? 'Wajib diisi' : null,
            ),
            const SizedBox(height: 12),
            TextFormField(
              controller: _addressController,
              decoration: const InputDecoration(labelText: 'Alamat Lengkap', border: OutlineInputBorder()),
              maxLines: 3,
              validator: (v) => v!.isEmpty ? 'Wajib diisi' : null,
            ),
            const SizedBox(height: 12),
            TextFormField(
              controller: _notesController,
              decoration: const InputDecoration(labelText: 'Catatan (opsional)', border: OutlineInputBorder()),
              maxLines: 2,
            ),
            const SizedBox(height: 24),
            _buildSectionTitle('Ringkasan Pembayaran'),
            _SummaryRow(label: 'Subtotal', value: 'Rp ${cart.totalAmount.toStringAsFixed(0)}'),
            _SummaryRow(label: 'Ongkir', value: 'Rp 15.000'),
            const Divider(),
            _SummaryRow(
              label: 'Total Bayar',
              value: 'Rp ${(cart.totalAmount + 15000).toStringAsFixed(0)}',
              isTotal: true,
            ),
            const SizedBox(height: 24),
            SizedBox(
              width: double.infinity,
              height: 52,
              child: ElevatedButton(
                onPressed: _isProcessing ? null : _submitOrder,
                style: ElevatedButton.styleFrom(
                  shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
                ),
                child: _isProcessing
                    ? const SizedBox(
                        width: 24,
                        height: 24,
                        child: CircularProgressIndicator(strokeWidth: 2, color: Colors.white),
                      )
                    : const Text('Bayar Sekarang', style: TextStyle(fontSize: 16)),
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildSectionTitle(String title) {
    return Padding(
      padding: const EdgeInsets.only(bottom: 12),
      child: Text(title, style: const TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
    );
  }
}

class _OrderItemRow extends StatelessWidget {
  final CartItem item;

  const _OrderItemRow({required this.item});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.only(bottom: 8),
      child: Row(
        children: [
          ClipRRect(
            borderRadius: BorderRadius.circular(6),
            child: Image.network(item.product.imageUrl, width: 50, height: 50, fit: BoxFit.cover),
          ),
          const SizedBox(width: 12),
          Expanded(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(item.product.name, style: const TextStyle(fontWeight: FontWeight.w500)),
                Text('${item.quantity} x Rp ${item.product.price.toStringAsFixed(0)}', style: TextStyle(color: Colors.grey[600], fontSize: 12)),
              ],
            ),
          ),
          Text('Rp ${item.totalPrice.toStringAsFixed(0)}', style: const TextStyle(fontWeight: FontWeight.w600)),
        ],
      ),
    );
  }
}

class _SummaryRow extends StatelessWidget {
  final String label;
  final String value;
  final bool isTotal;

  const _SummaryRow({required this.label, required this.value, this.isTotal = false});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 6),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Text(label, style: TextStyle(fontSize: isTotal ? 16 : 14, fontWeight: isTotal ? FontWeight.bold : FontWeight.normal)),
          Text(value, style: TextStyle(fontSize: isTotal ? 16 : 14, fontWeight: isTotal ? FontWeight.bold : FontWeight.normal)),
        ],
      ),
    );
  }
}
```

---

## 6. Order Success Screen

`lib/screens/order_success_screen.dart`:

```dart
import 'package:flutter/material.dart';

class OrderSuccessScreen extends StatelessWidget {
  const OrderSuccessScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Padding(
          padding: const EdgeInsets.all(32),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Container(
                width: 100,
                height: 100,
                decoration: BoxDecoration(
                  color: Colors.green[100],
                  shape: BoxShape.circle,
                ),
                child: const Icon(Icons.check_circle, size: 60, color: Colors.green),
              ),
              const SizedBox(height: 24),
              Text(
                'Pesanan Berhasil!',
                style: Theme.of(context).textTheme.headlineMedium?.copyWith(fontWeight: FontWeight.bold),
                textAlign: TextAlign.center,
              ),
              const SizedBox(height: 12),
              Text(
                'Terima kasih telah berbelanja. Pesanan Anda sedang diproses dan akan segera dikirim.',
                style: TextStyle(color: Colors.grey[600], fontSize: 16),
                textAlign: TextAlign.center,
              ),
              const SizedBox(height: 32),
              ElevatedButton.icon(
                icon: const Icon(Icons.shopping_bag),
                label: const Text('Kembali Belanja'),
                style: ElevatedButton.styleFrom(
                  padding: const EdgeInsets.symmetric(horizontal: 32, vertical: 16),
                  shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
                ),
                onPressed: () => Navigator.pushNamedAndRemoveUntil(context, '/products', (route) => false),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

---

## 7. Tambah Route & Entry Point

Di `main.dart` tambahkan route:

```dart
routes: {
  '/': (context) => const HomeScreen(),
  '/products': (context) => const ProductListScreen(),
  '/product/:id': (context) => const ProductDetailScreen(),
  '/cart': (context) => const CartScreen(),
  '/checkout': (context) => const CheckoutScreen(),
  '/order-success': (context) => const OrderSuccessScreen(),
},
```

Di `ProductDetailScreen`, tambahkan tombol "Tambah ke Keranjang":

```dart
ElevatedButton.icon(
  icon: const Icon(Icons.add_shopping_cart),
  label: const Text('Tambah ke Keranjang'),
  style: ElevatedButton.styleFrom(
    minimumSize: const Size(double.infinity, 52),
    shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
  ),
  onPressed: () {
    context.read<CartProvider>().addItem(product);
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text('${product.name} ditambahkan ke keranjang')),
    );
  },
),
```

---

## 8. Bottom Navigation - Cart Badge

Update `BottomNavigationBar` di `HomeScreen` supaya nampak badge jumlah item:

```dart
BottomNavigationBarItem(
  icon: Stack(
    clipBehavior: Clip.none,
    children: [
      const Icon(Icons.shopping_cart),
      Consumer<CartProvider>(
        builder: (_, cart, __) => cart.itemCount > 0
            ? Positioned(
                right: -8,
                top: -8,
                child: Container(
                  padding: const EdgeInsets.all(2),
                  decoration: BoxDecoration(
                    color: Colors.red,
                    borderRadius: BorderRadius.circular(10),
                  ),
                  constraints: const BoxConstraints(minWidth: 16, minHeight: 16),
                  child: Text(
                    '${cart.itemCount}',
                    style: const TextStyle(color: Colors.white, fontSize: 10),
                    textAlign: TextAlign.center,
                  ),
                ),
              )
            : const SizedBox(),
      ),
    ],
  ),
  label: 'Keranjang',
),
```

---

## Ringkasan Part 3

| Fitur | File | Key Point |
|-------|------|-----------|
| Cart Model | `cart_item.dart` | `copyWith` untuk update immutable |
| Cart State | `cart_provider.dart` | Map<String, CartItem>, notifyListeners |
| Cart UI | `cart_screen.dart` | Quantity control, total, clear dialog |
| Checkout | `checkout_screen.dart` | Form validation, summary, loading state |
| Success | `order_success_screen.dart` | Clear cart, push replacement |

---

## Part 4 Preview

Part 4 (artikel #74) kita bahas:
- **Order History** — simpan riwayat pesanan ke local (SharedPreferences/SQLite)
- **User Profile** — kelola alamat, riwayat, logout
- **Admin Panel** — kelola produk, lihat order masuk

Stay tuned!

---

**Coba sendiri!** Share ke sosial media dan tag **@ahsai001**