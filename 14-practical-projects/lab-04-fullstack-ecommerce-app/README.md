# Lab 04: Full-Stack E-Commerce Storefront (Dio, Freezed, Firebase, and Riverpod)

> **Module:** 14-practical-projects  
> **Lab Number:** 04 (Capstone Architectural Lab)  
> **Prerequisites:** Modules 01 through 13  
> **Estimated Completion Time:** 6 Hours

---

## 1. Project Requirements

Build a production-grade multi-platform E-Commerce mobile application with authentication, product browsing, cart state management, checkout, and automated testing.

### Technical Stack:
1. **Networking:** **Dio** client configured with base options, timeouts, logging interceptors, and error mapping.
2. **Data Modeling:** **Freezed** and **`json_serializable`** for immutable DTOs, union error states, and `copyWith()`.
3. **Authentication & Backend:** **Firebase Auth** session management and **Cloud Firestore** order processing.
4. **State Management:** Modern **Riverpod (2.x / 3.x)** `Notifier` and `AsyncNotifier` providers.
5. **Testing Suite:** Complete unit tests for Cart calculations, widget tests for product cards, and automated integration test workflows.

---

## 2. Architecture & Folder Structure

```
lab-04-fullstack-ecommerce-app/
├── lib/
│   ├── main.dart
│   ├── core/
│   │   ├── network/dio_client.dart          # Configured Dio instance
│   │   └── errors/app_exceptions.dart       # Exception mapping
│   └── features/
│       ├── catalog/
│       │   ├── data/models/product_model.dart # Freezed DTO
│       │   ├── domain/repositories/catalog_repository.dart
│       │   └── presentation/controllers/catalog_controller.dart
│       └── cart/
│           ├── domain/models/cart_item.dart
│           └── presentation/controllers/cart_controller.dart
└── test/
    ├── unit/cart_test.dart
    └── widget/product_tile_test.dart
```

---

## 3. Implementation Steps & Code

### Step 1: Freezed Immutable Product Model (`lib/features/catalog/data/models/product_model.dart`)

```dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'product_model.freezed.dart';
part 'product_model.g.dart';

@freezed
class ProductModel with _$ProductModel {
  const factory ProductModel({
    required String id,
    required String title,
    required String description,
    required double price,
    required String imageUrl,
    @Default(0.0) double rating,
  }) = _ProductModel;

  factory ProductModel.fromJson(Map<String, dynamic> json) =>
      _$ProductModelFromJson(json);
}
```

---

### Step 2: Enterprise Dio Client (`lib/core/network/dio_client.dart`)

```dart
import 'package:dio/dio.dart';
import 'package:flutter/foundation.dart';

class ApiClient {
  late final Dio dio;

  ApiClient({String baseUrl = 'https://dummyjson.com'}) {
    dio = Dio(
      BaseOptions(
        baseUrl: baseUrl,
        connectTimeout: const Duration(seconds: 8),
        receiveTimeout: const Duration(seconds: 8),
        headers: {'Accept': 'application/json'},
      ),
    );

    if (kDebugMode) {
      dio.interceptors.add(
        LogInterceptor(requestBody: false, responseBody: false),
      );
    }
  }
}
```

---

### Step 3: Shopping Cart State Management with Riverpod (`lib/features/cart/presentation/controllers/cart_controller.dart`)

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../../catalog/data/models/product_model.dart';

class CartItem {
  final ProductModel product;
  final int quantity;

  const CartItem({required this.product, this.quantity = 1});

  CartItem copyWith({int? quantity}) => CartItem(
        product: product,
        quantity: quantity ?? this.quantity,
      );
}

class CartState {
  final Map<String, CartItem> items;

  const CartState({this.items = const {}});

  int get totalItemCount =>
      items.values.fold(0, (sum, item) => sum + item.quantity);

  double get totalPrice => items.values
      .fold(0.0, (sum, item) => sum + (item.product.price * item.quantity));

  CartState copyWith({Map<String, CartItem>? items}) =>
      CartState(items: items ?? this.items);
}

class CartNotifier extends Notifier<CartState> {
  @override
  CartState build() => const CartState();

  void addProduct(ProductModel product) {
    final currentMap = Map<String, CartItem>.from(state.items);
    if (currentMap.containsKey(product.id)) {
      currentMap[product.id] = currentMap[product.id]!
          .copyWith(quantity: currentMap[product.id]!.quantity + 1);
    } else {
      currentMap[product.id] = CartItem(product: product);
    }
    state = state.copyWith(items: currentMap);
  }

  void removeProduct(String productId) {
    final currentMap = Map<String, CartItem>.from(state.items);
    currentMap.remove(productId);
    state = state.copyWith(items: currentMap);
  }

  void clear() => state = const CartState();
}

final cartNotifierProvider = NotifierProvider<CartNotifier, CartState>(() {
  return CartNotifier();
});
```

---

### Step 4: Catalog AsyncNotifier (`lib/features/catalog/presentation/controllers/catalog_controller.dart`)

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../../../../core/network/dio_client.dart';
import '../data/models/product_model.dart';

final apiClientProvider = Provider<ApiClient>((ref) => ApiClient());

class CatalogNotifier extends AsyncNotifier<List<ProductModel>> {
  @override
  Future<List<ProductModel>> build() async {
    final client = ref.watch(apiClientProvider);
    final response = await client.dio.get('/products');
    final List rawList = response.data['products'] as List;
    return rawList
        .map((json) => ProductModel.fromJson(json as Map<String, dynamic>))
        .toList();
  }
}

final catalogNotifierProvider =
    AsyncNotifierProvider<CatalogNotifier, List<ProductModel>>(() {
  return CatalogNotifier();
});
```

---

## 4. Automated Testing Suite

### Unit Test: Cart Calculations (`test/unit/cart_test.dart`)

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:ecommerce_app/features/cart/presentation/controllers/cart_controller.dart';
import 'package:ecommerce_app/features/catalog/data/models/product_model.dart';

void main() {
  group('CartNotifier Unit Tests', () {
    test('Calculates total price and item count accurately', () {
      final container = ProviderContainer();
      final notifier = container.read(cartNotifierProvider.notifier);

      const p1 = ProductModel(
        id: '1',
        title: 'Keyboard',
        description: '',
        price: 100.0,
        imageUrl: '',
      );
      const p2 = ProductModel(
        id: '2',
        title: 'Mouse',
        description: '',
        price: 50.0,
        imageUrl: '',
      );

      notifier.addProduct(p1);
      notifier.addProduct(p1); // Quantity = 2
      notifier.addProduct(p2); // Quantity = 1

      final state = container.read(cartNotifierProvider);

      expect(state.totalItemCount, equals(3));
      expect(state.totalPrice, equals(250.0));
    });
  });
}
```

---

## 5. Possible Future Improvements

1. **Payment Gateway Integration:** Integrate Stripe (`flutter_stripe`) or Apple Pay / Google Pay for in-app checkout.
2. **Deep Linking:** Integrate `go_router` with URL path parameters (`/product/:id`) and universal app links for web and mobile sharing.
3. **Push Notifications:** Configure Firebase Cloud Messaging (FCM) background message handlers for order dispatch notifications.
