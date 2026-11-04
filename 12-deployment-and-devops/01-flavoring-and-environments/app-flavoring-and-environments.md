# App Flavoring and Environment Configuration: Dev, Staging, and Prod

> **Module:** 12-deployment-and-devops  
> **Subsection:** 01-flavoring-and-environments  
> **Prerequisites:** [09-platform-integration/03-android-configuration-and-permissions/android-permissions-and-gradle.md](../../09-platform-integration/03-android-configuration-and-permissions/android-permissions-and-gradle.md)  
> **Official References:** [docs.flutter.dev/deployment/flavors](https://docs.flutter.dev/deployment/flavors)

---

## 1. Introduction

In professional mobile app development, you never test features against live production financial databases or production customer records. Enterprise teams maintain at least three isolated environments:
1. **Development (`dev`):** Connects to local mock servers or development cloud instances; verbose logging enabled; distinct debug app icon; allows co-existence alongside production on the same physical device (`applicationId: com.example.app.dev`).
2. **Staging (`stage`):** Mirrors production hardware and data schemas for QA acceptance testing.
3. **Production (`prod`):** Connects to live production servers; debug logs stripped; optimized release binaries (`applicationId: com.example.app`).

This configuration separation is achieved using **Flavors** (Android) and **Schemes** (iOS).

---

## 2. Compile-Time Secrets: `--dart-define-from-file`

> 💡 **Modern Best Practice (Flutter 3.7+):**  
> Do not check API keys or server secrets into Git repositories! Use `--dart-define-from-file` to inject environment variables securely during build time.

### Configuration JSON Files (`config/`):
File: `config/dev.json`
```json
{
  "ENVIRONMENT": "development",
  "API_BASE_URL": "https://dev-api.enterprise.com",
  "ENABLE_LOGGING": true
}
```

File: `config/prod.json`
```json
{
  "ENVIRONMENT": "production",
  "API_BASE_URL": "https://api.enterprise.com",
  "ENABLE_LOGGING": false
}
```

### Reading in Dart Code (`lib/core/config/app_environment.dart`):
```dart
class AppEnvironment {
  static const String environment =
      String.fromEnvironment('ENVIRONMENT', defaultValue: 'development');

  static const String apiBaseUrl =
      String.fromEnvironment('API_BASE_URL', defaultValue: 'https://dev-api.enterprise.com');

  static const bool enableLogging =
      bool.fromEnvironment('ENABLE_LOGGING', defaultValue: true);

  static bool get isProduction => environment == 'production';
}
```

---

## 3. Platform Configuration

### 1. Android Configuration (`android/app/build.gradle.kts`)

```kotlin
android {
    ...
    flavorDimensions += "default"

    productFlavors {
        create("dev") {
            dimension = "default"
            applicationIdSuffix = ".dev"
            resValue("string", "app_name", "Enterprise (Dev)")
            versionNameSuffix = "-dev"
        }
        create("stage") {
            dimension = "default"
            applicationIdSuffix = ".stage"
            resValue("string", "app_name", "Enterprise (Stage)")
            versionNameSuffix = "-stage"
        }
        create("prod") {
            dimension = "default"
            resValue("string", "app_name", "Enterprise")
        }
    }
}
```

---

### 2. iOS Scheme Configuration in Xcode

1. Open `ios/Runner.xcworkspace` in Xcode.
2. In Project Settings under **Configurations**, duplicate `Debug`, `Profile`, and `Release` into:
   - `Debug-dev`, `Profile-dev`, `Release-dev`
   - `Debug-prod`, `Profile-prod`, `Release-prod`
3. Under **Manage Schemes**, create matching schemes: `dev` and `prod`.
4. Update the **Bundle Identifier** for `dev` to `com.example.enterprise.dev`.

---

## 4. Build and Run Commands

```bash
# Run Development Flavor on connected device:
flutter run --flavor dev --target lib/main_dev.dart --dart-define-from-file=config/dev.json

# Build Production Android App Bundle:
flutter build appbundle --flavor prod --target lib/main_prod.dart --dart-define-from-file=config/prod.json

# Build Production iOS IPA:
flutter build ipa --flavor prod --target lib/main_prod.dart --dart-define-from-file=config/prod.json
```

---

## 5. Interview & Assessment Takeaways

| Interview Question | Concise Verbal Answer |
|:---|:---|
| **Why is `applicationIdSuffix = ".dev"` used in Android flavors?** | It changes the final package identifier on the device (e.g. `com.example.app.dev`). This allows QA engineers and developers to install both the development build and production build simultaneously on the same test phone without overwriting each other. |
| **What is the advantage of `String.fromEnvironment()` over loading a `.env` file from assets at runtime?** | `String.fromEnvironment()` injects values at **compile time**. The Dart compiler can evaluate dead code and completely tree-shake unused branches (e.g. `if (AppEnvironment.isProduction)` blocks), and secrets are embedded directly into machine instructions rather than stored as plaintext asset files inside the APK. |
