# Android Configuration, Gradle Kotlin DSL, and Permissions

> **Module:** 09-platform-integration  
> **Subsection:** 03-android-configuration-and-permissions  
> **Prerequisites:** [09-platform-integration/01-platform-channels-methodchannel/platform-channels-guide.md](../01-platform-channels-methodchannel/platform-channels-guide.md)  
> **Official References:** [developer.android.com/guide/topics/permissions/overview](https://developer.android.com/guide/topics/permissions/overview) | [pub.dev/packages/permission_handler](https://pub.dev/packages/permission_handler)

---

## 1. Introduction

Even though Flutter handles cross-platform UI, the underlying Android platform enforces strict operating system rules:
- Declaration of hardware capabilities inside `AndroidManifest.xml`,
- Runtime user permission grants for dangerous APIs (Camera, Location, Bluetooth, Notifications),
- Modern Gradle build automation using Kotlin Script (`.gradle.kts`),
- Generating cryptographic **SHA-1 and SHA-256 fingerprints** required by Google Play, Firebase, and Google Maps APIs.

---

## 2. Core Concepts: Android Permission Categories

Android categorizes permissions into two major tiers:
1. **Normal / Install-Time Permissions:** Automatically granted by the OS upon app installation if declared in `AndroidManifest.xml` (e.g. `INTERNET`, `ACCESS_NETWORK_STATE`).
2. **Dangerous / Runtime Permissions:** Require active runtime user consent via system dialogs (e.g. `CAMERA`, `ACCESS_FINE_LOCATION`, `POST_NOTIFICATIONS` in Android 13+). Declaring them in the manifest is mandatory but **insufficient**—the app must request them dynamically in Dart before accessing the feature.

---

## 3. Syntax and Implementation

### 1. Declaring Permissions in `AndroidManifest.xml`

File: `android/app/src/main/AndroidManifest.xml`

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <!-- Normal permissions (Auto-granted) -->
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>

    <!-- Dangerous runtime permissions -->
    <uses-permission android:name="android.permission.CAMERA"/>
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
    
    <!-- Android 13+ (API 33) Notification permission -->
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>

    <application
        android:label="Enterprise App"
        android:name="${applicationName}"
        android:icon="@mipmap/ic_launcher">
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:theme="@style/LaunchTheme"
            android:configChanges="orientation|keyboardHidden|keyboard|screenSize|smallestScreenSize|locale|layoutDirection|fontScale|screenLayout|density|uiMode"
            android:hardwareAccelerated="true"
            android:windowSoftInputMode="adjustResize">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
    </application>
</manifest>
```

---

### 2. Modern Gradle Kotlin DSL (`android/app/build.gradle.kts`)

Flutter 3.16+ supports and defaults to Kotlin DSL for Gradle build scripts:

```kotlin
plugins {
    id("com.android.application")
    id("kotlin-android")
    id("dev.flutter.flutter-gradle-plugin")
}

android {
    namespace = "com.example.enterprise"
    compileSdk = 34 // Modern Android 14 standard

    defaultConfig {
        applicationId = "com.example.enterprise"
        minSdk = 24     // Android 7.0 (Nougat) minimum baseline
        targetSdk = 34  // Target Android 14
        versionCode = flutter.versionCode
        versionName = flutter.versionName
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }

    kotlinOptions {
        jvmTarget = "17"
    }
}
```

---

### 3. Runtime Permission Handling with `permission_handler`

```dart
import 'package:flutter/material.dart';
import 'package:permission_handler/permission_handler.dart';

class CameraPermissionHandler {
  Future<bool> requestCameraAccess(BuildContext context) async {
    // 1. Check current status
    var status = await Permission.camera.status;

    if (status.isGranted) {
      return true;
    }

    // 2. Request permission if not determined or denied
    if (status.isDenied) {
      status = await Permission.camera.request();
      if (status.isGranted) return true;
    }

    // 3. User selected "Don't ask again" (permanently denied)
    if (status.isPermanentlyDenied) {
      if (context.mounted) {
        final openSettings = await showDialog<bool>(
          context: context,
          builder: (ctx) => AlertDialog(
            title: const Text('Camera Permission Required'),
            content: const Text('Please enable camera access in App Settings to scan barcodes.'),
            actions: [
              TextButton(onPressed: () => Navigator.pop(ctx, false), child: const Text('Cancel')),
              FilledButton(onPressed: () => Navigator.pop(ctx, true), child: const Text('Open Settings')),
            ],
          ),
        );

        if (openSettings == true) {
          await openAppSettings();
        }
      }
      return false;
    }

    return false;
  }
}
```

---

## 4. Extracting SHA-1 & SHA-256 Fingerprints

Google Sign-In, Firebase App Check, and Google Maps SDK require your debug and release keystore SHA fingerprints.

### Method 1: Using Gradle Wrapper (Fastest)
From your project's `android/` directory:

```bash
# On Windows PowerShell:
.\gradlew signingReport

# On macOS / Linux:
./gradlew signingReport
```

Look for the output:
```
Variant: debug
Config: debug
Store: C:\Users\<user>\.android\debug.keystore
Alias: AndroidDebugKey
MD5:   1B:E4:72:...
SHA1:  23:8B:E8:4A:9C:...
SHA-256: 7F:D4:62:3A:99:...
```

### Method 2: Using JDK `keytool` Directly
```bash
keytool -list -v -keystore "%USERPROFILE%\.android\debug.keystore" -alias androiddebugkey -storepass android -keypass android
```

---

## 5. Interview & Assessment Takeaways

| Interview Question | Concise Verbal Answer |
|:---|:---|
| **What happens if a dangerous permission is declared in `AndroidManifest.xml` but never requested in Dart?** | When the app attempts to invoke the native API (e.g. starting the camera hardware), the Android operating system throws a `SecurityException`, immediately crashing the app. Dangerous permissions must be explicitly requested at runtime. |
| **What is the difference between `PermissionStatus.denied` and `permanentlyDenied`?** | `denied` means the user dismissed the prompt once, but the OS allows the app to ask again on future interactions. `permanentlyDenied` means the user chose "Don't ask again"; the OS will permanently suppress future system prompts, meaning the user must be redirected to `openAppSettings()` to manually grant it. |
| **Why is the SHA-1 fingerprint needed for Firebase and Google Sign-In?** | Android APKs are signed with a private cryptographic certificate. Google APIs use the SHA-1/SHA-256 fingerprint of that certificate to verify that incoming API requests originate strictly from your legitimate application binary and not a malicious third party. |
