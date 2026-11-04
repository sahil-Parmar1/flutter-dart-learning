# Platform Channels: MethodChannel, EventChannel, and Native Interop

> **Module:** 09-platform-integration  
> **Subsection:** 01-platform-channels-methodchannel  
> **Prerequisites:** [03-flutter-fundamentals/01-flutter-architecture-and-engine/layered-architecture.md](../../03-flutter-fundamentals/01-flutter-architecture-and-engine/layered-architecture.md)  
> **Official References:** [docs.flutter.dev/platform-integration/platform-channels](https://docs.flutter.dev/platform-integration/platform-channels) | [api.flutter.dev/flutter/services/MethodChannel-class.html](https://api.flutter.dev/flutter/services/MethodChannel-class.html)

---

## 1. Introduction

Flutter does not compile Dart code into Java, Kotlin, Swift, or Objective-C. Instead, the Flutter C++ Engine runs side-by-side with the host operating system.

When your application requires native device capabilities that have no pure Dart implementation, it communicates across the **Platform Channel** bridge using asynchronous message passing.

---

## 2. Core Concepts: The Three Channel Types

| Channel Type | Direction | Data Paradigm | Use Case |
|:---|:---:|:---:|:---|
| **`BasicMessageChannel`** | Bi-directional | Request-Response or Fire-and-Forget | Passing custom binary or string payloads |
| **`MethodChannel`** | Bi-directional (usually Dart -> Native) | RPC (Remote Procedure Call) | One-time operations: getting battery %, triggering haptics, biometric prompt |
| **`EventChannel`** | Unidirectional (Native -> Dart) | Continuous Streaming (`Stream<T>`) | Continuous hardware sensors: accelerometer, pedometer, network connectivity changes |

### The `BinaryMessenger` and `StandardMessageCodec`
Under the hood, all platform channels use Flutter's **`BinaryMessenger`**. Data types (`String`, `int`, `Map`, `List`, `bool`) are encoded into compact binary buffers using the **`StandardMessageCodec`** and sent across the C++ engine boundary.

---

## 3. Syntax and Implementation

### Complete Real-World Implementation: Device Battery Percentage

#### 1. Dart Side (`battery_service.dart`)

```dart
import 'package:flutter/services.dart';

class BatteryService {
  static const MethodChannel _methodChannel =
      MethodChannel('com.example.enterprise/battery');
  
  static const EventChannel _eventChannel =
      EventChannel('com.example.enterprise/charging_state');

  /// 1. Invoke native method call
  Future<int> getBatteryLevel() async {
    try {
      final int? batteryLevel =
          await _methodChannel.invokeMethod<int>('getBatteryLevel');
      return batteryLevel ?? -1;
    } on PlatformException catch (e) {
      throw 'Failed to get battery level: ${e.message}';
    } on MissingPluginException {
      throw 'Plugin not implemented on this platform.';
    }
  }

  /// 2. Subscribe to continuous charging status stream
  Stream<String> watchChargingState() {
    return _eventChannel
        .receiveBroadcastStream()
        .map((dynamic event) => event as String);
  }
}
```

---

#### 2. Android Host Implementation in Kotlin (`MainActivity.kt`)

```kotlin
package com.example.enterprise

import android.content.Context
import android.content.ContextWrapper
import android.content.Intent
import android.content.IntentFilter
import android.os.BatteryManager
import android.os.Build.VERSION
import android.os.Build.VERSION_CODES
import io.flutter.embedding.android.FlutterActivity
import io.flutter.embedding.engine.FlutterEngine
import io.flutter.plugin.common.EventChannel
import io.flutter.plugin.common.MethodChannel

class MainActivity: FlutterActivity() {
    private val METHOD_CHANNEL = "com.example.enterprise/battery"
    private val EVENT_CHANNEL = "com.example.enterprise/charging_state"

    override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)

        // 1. MethodChannel Handler
        MethodChannel(flutterEngine.dartExecutor.binaryMessenger, METHOD_CHANNEL)
            .setMethodCallHandler { call, result ->
                if (call.method == "getBatteryLevel") {
                    val batteryLevel = getBatteryPercentage()
                    if (batteryLevel != -1) {
                        result.success(batteryLevel)
                    } else {
                        result.error("UNAVAILABLE", "Battery level not available.", null)
                    }
                } else {
                    result.notImplemented()
                }
            }

        // 2. EventChannel Stream Handler
        EventChannel(flutterEngine.dartExecutor.binaryMessenger, EVENT_CHANNEL)
            .setStreamHandler(object : EventChannel.StreamHandler {
                override fun onListen(arguments: Any?, events: EventChannel.EventSink?) {
                    val status = if (isDeviceCharging()) "CHARGING" else "DISCHARGING"
                    events?.success(status)
                }

                override fun onCancel(arguments: Any?) {
                    // Clean up broadcast receivers
                }
            })
    }

    private fun getBatteryPercentage(): Int {
        val intent = ContextWrapper(applicationContext).registerReceiver(
            null,
            IntentFilter(Intent.ACTION_BATTERY_CHANGED)
        )
        return (intent?.getIntExtra(BatteryManager.EXTRA_LEVEL, -1) ?: -1)
    }

    private fun isDeviceCharging(): Boolean {
        val intent = ContextWrapper(applicationContext).registerReceiver(
            null,
            IntentFilter(Intent.ACTION_BATTERY_CHANGED)
        )
        val status = intent?.getIntExtra(BatteryManager.EXTRA_STATUS, -1) ?: -1
        return status == BatteryManager.BATTERY_STATUS_CHARGING ||
               status == BatteryManager.BATTERY_STATUS_FULL
    }
}
```

---

#### 3. iOS Host Implementation in Swift (`AppDelegate.swift`)

```swift
import UIKit
import Flutter

@main
@objc class AppDelegate: FlutterAppDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    let controller : FlutterViewController = window?.rootViewController as! FlutterViewController
    
    // 1. Setup MethodChannel
    let batteryChannel = FlutterMethodChannel(
      name: "com.example.enterprise/battery",
      binaryMessenger: controller.binaryMessenger
    )
    
    batteryChannel.setMethodCallHandler({
      (call: FlutterMethodCall, result: @escaping FlutterResult) -> Void in
      guard call.method == "getBatteryLevel" else {
        result(FlutterMethodNotImplemented)
        return
      }
      
      UIDevice.current.isBatteryMonitoringEnabled = true
      let batteryLevel = Int(UIDevice.current.batteryLevel * 100)
      
      if batteryLevel >= 0 {
        result(batteryLevel)
      } else {
        result(FlutterError(code: "UNAVAILABLE", message: "Battery info unavailable", details: nil))
      }
    })

    GeneratedPluginRegistrant.register(with: self)
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }
}
```

---

## 4. Threading and Performance Considerations

> ⚠️ **CRITICAL THREADING RULE:**  
> Platform channel messages arrive on the native platform's **UI / Main thread** (Android Main Looper / iOS Main Queue).  
> If you perform long-running blocking tasks (e.g. image processing, heavy SQL queries, network requests) directly inside your native `setMethodCallHandler`, you will freeze the native OS host and cause **ANR (Application Not Responding)** crashes!  
> **Solution:** Dispatch heavy operations onto background dispatchers (Kotlin Coroutines `Dispatchers.IO` or Swift `DispatchQueue.global()`) and post the result back to `result.success()` on the main thread.

---

## 5. Interview & Assessment Takeaways

| Interview Question | Concise Verbal Answer |
|:---|:---|
| **What is the difference between `MethodChannel` and `EventChannel`?** | `MethodChannel` is intended for discrete request-response Remote Procedure Calls (RPCs), returning a `Future<T>`. `EventChannel` is designed for continuous, asynchronous streaming from native platform to Dart, returning a `Stream<T>`. |
| **Are Platform Channel calls synchronous or asynchronous?** | They are strictly **asynchronous**. Messages are serialized into binary buffers and passed across the C++ engine boundary via the `BinaryMessenger`, returning a Dart `Future`. |
| **What happens if you execute heavy computational logic inside a native `MethodCallHandler`?** | It blocks the platform's main UI thread, freezing native gesture dispatching and triggering an ANR crash on Android or watchdog termination on iOS. Heavy tasks must be offloaded to native background worker threads. |
