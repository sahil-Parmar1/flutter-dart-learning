# Lab 02: Responsive & Adaptive Analytics Dashboard

> **Module:** 14-practical-projects  
> **Lab Number:** 02  
> **Prerequisites:** [04-layout-and-widgets/](../../04-layout-and-widgets/) | [05-animations-and-motion/](../../05-animations-and-motion/)  
> **Estimated Completion Time:** 3 Hours

---

## 1. Project Requirements

Build a multiplatform financial and operational analytics dashboard that runs seamlessly across mobile phones, tablets, and desktop browsers.

### Technical Requirements:
1. **Adaptive Navigation Shell:** Dynamically renders a `NavigationBar` on phones (<600px), a `NavigationRail` on tablets (600–840px), and an extended persistent drawer on desktop (>840px).
2. **Coordinated Sliver Viewport:** Use `CustomScrollView` with a collapsing `SliverAppBar`, sticky tab bar, and responsive grid metrics cards.
3. **Custom GPU Chart Rendering:** Paint an interactive financial area curve chart using `CustomPainter`, `Path.quadraticBezierTo`, and gradient shaders.
4. **Theme Adaptation:** Full Material 3 dynamic color scheme support (`ColorScheme.fromSeed`).

---

## 2. Architecture & Folder Structure

```
lab-02-responsive-dashboard/
├── lib/
│   ├── main.dart
│   ├── core/
│   │   ├── responsive/breakpoints.dart   # Screen metric breakpoints
│   │   └── theme/app_theme.dart          # M3 color seeding
│   ├── presentation/
│   │   ├── widgets/
│   │   │   ├── adaptive_shell.dart       # NavRail vs BottomBar switcher
│   │   │   ├── financial_chart.dart      # CustomPainter area curve
│   │   │   └── metric_card.dart          # Metric summary card
│   │   └── views/
│   │       └── dashboard_view.dart       # CustomScrollView + Slivers
└── test/
    └── dashboard_responsive_test.dart    # Tablet & Mobile size widget tests
```

---

## 3. Implementation Steps & Code

### Step 1: Responsive Breakpoints & Shell (`lib/core/responsive/breakpoints.dart`)

```dart
import 'package:flutter/material.dart';

class Breakpoints {
  static const double compact = 600.0;
  static const double medium = 840.0;
}

enum ScreenSize { compact, medium, expanded }

ScreenSize getScreenSize(BuildContext context) {
  final width = MediaQuery.sizeOf(context).width;
  if (width < Breakpoints.compact) return ScreenSize.compact;
  if (width < Breakpoints.medium) return ScreenSize.medium;
  return ScreenSize.expanded;
}
```

---

### Step 2: Custom GPU Financial Area Chart (`lib/presentation/widgets/financial_chart.dart`)

```dart
import 'package:flutter/material.dart';

class FinancialAreaChart extends StatelessWidget {
  final List<double> dataPoints; // Normalized 0.0 to 1.0

  const FinancialAreaChart({super.key, required this.dataPoints});

  @override
  Widget build(BuildContext context) {
    return RepaintBoundary(
      child: CustomPaint(
        size: const Size(double.infinity, 180),
        painter: _ChartPainter(dataPoints: dataPoints),
      ),
    );
  }
}

class _ChartPainter extends CustomPainter {
  final List<double> dataPoints;

  _ChartPainter({required this.dataPoints});

  @override
  void paint(Canvas canvas, Size size) {
    if (dataPoints.length < 2) return;

    final linePaint = Paint()
      ..color = Colors.tealAccent.shade400
      ..style = PaintingStyle.stroke
      ..strokeWidth = 3.5
      ..strokeCap = StrokeCap.round;

    final stepX = size.width / (dataPoints.length - 1);
    final path = Path();
    final fillPath = Path();

    path.moveTo(0, size.height * (1.0 - dataPoints[0]));
    fillPath.moveTo(0, size.height);
    fillPath.lineTo(0, size.height * (1.0 - dataPoints[0]));

    for (var i = 1; i < dataPoints.length; i++) {
      final prevX = (i - 1) * stepX;
      final prevY = size.height * (1.0 - dataPoints[i - 1]);
      final currentX = i * stepX;
      final currentY = size.height * (1.0 - dataPoints[i]);

      // Draw smooth Bézier curve segment
      final controlX = (prevX + currentX) / 2;
      path.quadraticBezierTo(controlX, prevY, currentX, currentY);
      fillPath.quadraticBezierTo(controlX, prevY, currentX, currentY);
    }

    fillPath.lineTo(size.width, size.height);
    fillPath.close();

    // 1. Draw Gradient Fill Area
    final fillPaint = Paint()
      ..shader = LinearGradient(
        begin: Alignment.topCenter,
        end: Alignment.bottomCenter,
        colors: [
          Colors.tealAccent.shade400.withOpacity(0.35),
          Colors.tealAccent.shade400.withOpacity(0.0),
        ],
      ).createShader(Rect.fromLTWH(0, 0, size.width, size.height));

    canvas.drawPath(fillPath, fillPaint);

    // 2. Draw Stroke Path
    canvas.drawPath(path, linePaint);
  }

  @override
  bool shouldRepaint(covariant _ChartPainter oldDelegate) =>
      oldDelegate.dataPoints != dataPoints;
}
```

---

### Step 3: Coordinated Dashboard View (`lib/presentation/views/dashboard_view.dart`)

```dart
import 'package:flutter/material.dart';
import '../widgets/financial_chart.dart';

class DashboardView extends StatelessWidget {
  const DashboardView({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomScrollView(
        slivers: [
          // 1. Collapsing App Bar with Integrated Chart
          SliverAppBar(
            expandedHeight: 260.0,
            pinned: true,
            flexibleSpace: FlexibleSpaceBar(
              title: const Text('Executive Revenue Metrics'),
              background: Container(
                decoration: BoxDecoration(
                  gradient: LinearGradient(
                    colors: [Colors.indigo.shade900, Colors.deepPurple.shade800],
                  ),
                ),
                padding: const EdgeInsets.only(top: 60, left: 16, right: 16, bottom: 24),
                child: const FinancialAreaChart(
                  dataPoints: [0.2, 0.45, 0.3, 0.7, 0.55, 0.85, 0.95],
                ),
              ),
            ),
          ),

          // 2. Responsive Metrics Grid
          SliverPadding(
            padding: const EdgeInsets.all(16.0),
            sliver: SliverGrid(
              gridDelegate: const SliverGridDelegateWithMaxCrossAxisExtent(
                maxCrossAxisExtent: 260, // Auto-adapts columns based on width
                mainAxisSpacing: 16,
                crossAxisSpacing: 16,
                childAspectRatio: 1.4,
              ),
              delegate: SliverChildListDelegate(
                const [
                  _MetricCard(title: 'Active Users', value: '42,910', delta: '+12.4%'),
                  _MetricCard(title: 'Gross Revenue', value: '\$189,400', delta: '+8.1%'),
                  _MetricCard(title: 'Churn Rate', value: '1.2%', delta: '-0.4%'),
                  _MetricCard(title: 'Server Latency', value: '28 ms', delta: '-5 ms'),
                ],
              ),
            ),
          ),

          // 3. Activity Feed List
          SliverToBoxAdapter(
            child: Padding(
              padding: const EdgeInsets.symmetric(horizontal: 16.0, vertical: 8.0),
              child: Text('Live Transaction Log', style: Theme.of(context).textTheme.titleLarge),
            ),
          ),
          SliverList(
            delegate: SliverChildBuilderDelegate(
              (context, index) => ListTile(
                leading: CircleAvatar(child: Text('${index + 1}')),
                title: Text('Transaction #TX-9021-$index'),
                subtitle: const Text('Settled via Automated Clearing House (ACH)'),
                trailing: Text('+\$${(index + 1) * 24}.00', style: const TextStyle(fontWeight: FontWeight.bold)),
              ),
              childCount: 20,
            ),
          ),
        ],
      ),
    );
  }
}

class _MetricCard extends StatelessWidget {
  final String title;
  final String value;
  final String delta;

  const _MetricCard({required this.title, required this.value, required this.delta});

  @override
  Widget build(BuildContext context) {
    return Card(
      elevation: 2,
      child: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(title, style: const TextStyle(color: Colors.grey)),
            const SizedBox(height: 6),
            Text(value, style: const TextStyle(fontSize: 22, fontWeight: FontWeight.bold)),
            const SizedBox(height: 4),
            Text(delta, style: const TextStyle(color: Colors.green, fontWeight: FontWeight.w600)),
          ],
        ),
      ),
    );
  }
}
```

---

## 4. Automated Testing: Multi-Screen Dimension Tests

File: `test/dashboard_responsive_test.dart`

```dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:responsive_dashboard/core/responsive/breakpoints.dart';

void main() {
  testWidgets('Correctly classifies Mobile vs Desktop breakpoints', (WidgetTester tester) async {
    // 1. Simulate Mobile Width (400px)
    tester.view.physicalSize = const Size(400, 800);
    tester.view.devicePixelRatio = 1.0;

    await tester.pumpWidget(
      MaterialApp(
        home: Builder(
          builder: (context) {
            final size = getScreenSize(context);
            expect(size, ScreenSize.compact);
            return const SizedBox();
          },
        ),
      ),
    );

    // 2. Simulate Desktop 1080p Monitor (1920px)
    tester.view.physicalSize = const Size(1920, 1080);
    await tester.pumpWidget(
      MaterialApp(
        home: Builder(
          builder: (context) {
            final size = getScreenSize(context);
            expect(size, ScreenSize.expanded);
            return const SizedBox();
          },
        ),
      ),
    );

    // Reset view bounds
    addTearDown(tester.view.resetPhysicalSize);
  });
}
```

---

## 5. Possible Future Improvements

1. **Interactive Touch Tooltip:** Add a `GestureDetector` to `FinancialAreaChart` to track touch coordinates and display hover tooltips showing exact data values.
2. **Keyboard Shortcuts:** Integrate `Shortcuts` and `Actions` widgets to allow desktop users to navigate tabs via `Cmd/Ctrl + 1/2/3`.
