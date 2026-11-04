# Practical Machine Coding Rounds and Live Debugging Challenges

> **Module:** 13-interview-and-certification  
> **Subsection:** 04-coding-challenges-and-machine-rounds  
> **Target Audience:** Candidates preparing for 45–60 minute live coding and debugging technical rounds

---

## Challenge 1: Debounced Search Autocomplete with Request Cancellation

### 🎯 Task Description:
Implement a search input that:
1. Debounces keystrokes by 400ms to avoid hammering the search backend.
2. Cancels any in-flight HTTP request if the user types a new character before the previous request finishes.
3. Displays loading indicator, empty state, and matching substring highlights.

### 💻 Production Implementation:

```dart
import 'dart:async';
import 'package:dio/dio.dart';
import 'package:flutter/material.dart';

class DebouncedSearchScreen extends StatefulWidget {
  const DebouncedSearchScreen({super.key});

  @override
  State<DebouncedSearchScreen> createState() => _DebouncedSearchScreenState();
}

class _DebouncedSearchScreenState extends State<DebouncedSearchScreen> {
  final TextEditingController _controller = TextEditingController();
  final Dio _dio = Dio(BaseOptions(baseUrl: 'https://dummyjson.com'));
  
  Timer? _debounceTimer;
  CancelToken? _cancelToken;
  
  List<String> _results = [];
  bool _isLoading = false;
  String _currentQuery = '';

  @override
  void initState() {
    super.initState();
    _controller.addListener(_onSearchChanged);
  }

  void _onSearchChanged() {
    final query = _controller.text.trim();
    if (query == _currentQuery) return;
    _currentQuery = query;

    // 1. Reset debounce timer
    _debounceTimer?.cancel();
    
    if (query.isEmpty) {
      setState(() {
        _results = [];
        _isLoading = false;
      });
      return;
    }

    // 2. Wait 400ms before firing request
    _debounceTimer = Timer(const Duration(milliseconds: 400), () {
      _executeSearch(query);
    });
  }

  Future<void> _executeSearch(String query) async {
    // 3. Cancel any in-flight HTTP request
    _cancelToken?.cancel('Cancelled due to new query entry');
    _cancelToken = CancelToken();

    setState(() => _isLoading = true);

    try {
      final response = await _dio.get(
        '/products/search',
        queryParameters: {'q': query},
        cancelToken: _cancelToken,
      );

      final List products = response.data['products'] as List;
      final titles = products.map((p) => p['title'] as String).toList();

      if (mounted) {
        setState(() {
          _results = titles;
          _isLoading = false;
        });
      }
    } on DioException catch (e) {
      if (CancelToken.isCancel(e)) {
        debugPrint('Previous request cancelled safely: ${e.message}');
      } else {
        if (mounted) setState(() => _isLoading = false);
      }
    }
  }

  @override
  void dispose() {
    _debounceTimer?.cancel();
    _cancelToken?.cancel();
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Live Search Autocomplete')),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(16.0),
            child: TextField(
              controller: _controller,
              decoration: InputDecoration(
                hintText: 'Search products (e.g. phone, perfume)...',
                prefixIcon: const Icon(Icons.search),
                suffixIcon: _isLoading
                    ? const Padding(
                        padding: EdgeInsets.all(12.0),
                        child: CircularProgressIndicator(strokeWidth: 2),
                      )
                    : IconButton(
                        icon: const Icon(Icons.clear),
                        onPressed: () => _controller.clear(),
                      ),
                border: const OutlineInputBorder(),
              ),
            ),
          ),
          Expanded(
            child: _results.isEmpty && !_isLoading
                ? const Center(child: Text('No matching items found.'))
                : ListView.builder(
                    itemCount: _results.length,
                    itemBuilder: (context, index) {
                      return ListTile(
                        leading: const Icon(Icons.shopping_bag_outlined),
                        title: Text(_results[index]),
                        onTap: () {},
                      );
                    },
                  ),
          ),
        ],
      ),
    );
  }
}
```

---

## Challenge 2: Infinite Paginated List with Pull-to-Refresh & Error Retry Footer

### 🎯 Task Description:
Build a paginated social feed that:
1. Loads 15 items per page.
2. Automatically detects when the user scrolls within 200px of the bottom and fetches the next page.
3. Shows a pagination spinner or a "Tap to retry" button if the next page fails to load.

### 💻 Production Implementation:

```dart
import 'package:flutter/material.dart';

class PaginatedFeedScreen extends StatefulWidget {
  const PaginatedFeedScreen({super.key});

  @override
  State<PaginatedFeedScreen> createState() => _PaginatedFeedScreenState();
}

class _PaginatedFeedScreenState extends State<PaginatedFeedScreen> {
  final ScrollController _scrollController = ScrollController();
  final List<String> _items = [];
  
  int _currentPage = 1;
  bool _isLoadingInitial = true;
  bool _isLoadingMore = false;
  bool _hasMoreData = true;
  String? _paginationError;

  @override
  void initState() {
    super.initState();
    _fetchInitialData();
    _scrollController.addListener(_onScroll);
  }

  void _onScroll() {
    // Check if scrolled within 200px of max extent
    if (_scrollController.position.pixels >=
            _scrollController.position.maxScrollExtent - 200 &&
        !_isLoadingMore &&
        _hasMoreData &&
        _paginationError == null) {
      _fetchNextPage();
    }
  }

  Future<void> _fetchInitialData() async {
    setState(() {
      _isLoadingInitial = true;
      _items.clear();
      _currentPage = 1;
      _hasMoreData = true;
      _paginationError = null;
    });

    await Future.delayed(const Duration(seconds: 1)); // Simulated network
    final newItems = List.generate(15, (i) => 'Post #$i (Page 1)');

    setState(() {
      _items.addAll(newItems);
      _isLoadingInitial = false;
    });
  }

  Future<void> _fetchNextPage() async {
    setState(() => _isLoadingMore = true);

    await Future.delayed(const Duration(milliseconds: 1200));

    // Simulate occasional network error on page 3
    if (_currentPage == 2 && _paginationError == null) {
      setState(() {
        _isLoadingMore = false;
        _paginationError = 'Connection timeout fetching page 3';
      });
      return;
    }

    _currentPage++;
    final newItems = List.generate(15, (i) => 'Post #${_items.length + i} (Page $_currentPage)');

    setState(() {
      _items.addAll(newItems);
      _isLoadingMore = false;
      _paginationError = null;
      if (_currentPage >= 4) _hasMoreData = false; // Stop at 4 pages
    });
  }

  @override
  void dispose() {
    _scrollController.removeListener(_onScroll);
    _scrollController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Infinite Paginated Feed')),
      body: _isLoadingInitial
          ? const Center(child: CircularProgressIndicator())
          : RefreshIndicator(
              onRefresh: _fetchInitialData,
              child: ListView.separated(
                controller: _scrollController,
                itemCount: _items.length + 1, // +1 for footer loader/error
                separatorBuilder: (_, __) => const Divider(height: 1),
                itemBuilder: (context, index) {
                  // Render Content Tiles
                  if (index < _items.length) {
                    return ListTile(
                      leading: CircleAvatar(child: Text('${index + 1}')),
                      title: Text(_items[index]),
                    );
                  }

                  // Render Footer
                  if (_paginationError != null) {
                    return Padding(
                      padding: const EdgeInsets.all(16.0),
                      child: Column(
                        children: [
                          Text(_paginationError!, style: const TextStyle(color: Colors.red)),
                          const SizedBox(height: 8),
                          FilledButton.tonal(
                            onPressed: () {
                              setState(() => _paginationError = null);
                              _fetchNextPage();
                            },
                            child: const Text('Retry Page Load'),
                          ),
                        ],
                      ),
                    );
                  }

                  if (_hasMoreData) {
                    return const Padding(
                      padding: EdgeInsets.all(16.0),
                      child: Center(child: CircularProgressIndicator()),
                    );
                  }

                  return const Padding(
                    padding: EdgeInsets.all(24.0),
                    child: Center(child: Text('You have reached the end of the feed!')),
                  );
                },
              ),
            ),
    );
  }
}
```

---

## Challenge 3: Live Debugging — Diagnosing a Broken RenderFlex

### ❌ The Buggy Code Presented by Interviewer:
```dart
Widget buildBrokenCard() {
  return Card(
    child: Row(
      children: [
        Icon(Icons.warning),
        Text('Important Alert Message: Please read this urgently before proceeding to checkout!'),
        Spacer(),
        IconButton(icon: Icon(Icons.close), onPressed: () {}),
      ],
    ),
  );
}
```

### 🔍 Interviewer Question:
*"This card works fine on wide screens, but crashes with yellow-and-black stripes on iPhone SE. Why, and how do you fix it?"*

### 💡 Candidate Solution & Verbal Explanation:
> **The Diagnosis:**  
> "`Text` by default has unbounded maximum width constraints along a `Row`'s main axis. If the text string is long, its intrinsic width exceeds the screen width minus the icons. Because `Row` does not wrap text automatically, it throws a `RenderFlex overflowed by X pixels` error."
>
> **The Fix:**  
> "Wrap the `Text` widget inside `Expanded`. `Expanded` passes a tight constraint bounded by the remaining available space inside the Row, forcing `Text` to wrap gracefully to multiple lines."

#### ✅ Corrected Code:
```dart
Widget buildFixedCard() {
  return Card(
    child: Padding(
      padding: const EdgeInsets.all(12.0),
      child: Row(
        children: [
          const Icon(Icons.warning, color: Colors.orange),
          const SizedBox(width: 8),
          // FIXED: Wrap in Expanded so text wraps into remaining width!
          const Expanded(
            child: Text(
              'Important Alert Message: Please read this urgently before proceeding to checkout!',
            ),
          ),
          const SizedBox(width: 8),
          IconButton(icon: const Icon(Icons.close), onPressed: () {}),
        ],
      ),
    ),
  );
}
```
