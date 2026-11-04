# Native C Interoperability with Dart FFI (`dart:ffi`)

> **Module:** 02-advanced-dart  
> **Subsection:** 06-file-io-and-ffi  
> **Prerequisites:** [dart-io-file-system.md](./dart-io-file-system.md)  
> **Official References:** [dart.dev/guides/libraries/c-interop](https://dart.dev/guides/libraries/c-interop) | [api.dart.dev/stable/dart-ffi/dart-ffi-library.html](https://api.dart.dev/stable/dart-ffi/dart-ffi-library.html)

---

## 1. Introduction

While Dart is a high-performance compiled language, mobile and desktop applications frequently need to integrate with legacy C/C++ codebases, high-performance game physics engines, cryptographic libraries (e.g. OpenSSL, Libsodium), machine learning frameworks (e.g. TensorFlow Lite), or embedded databases (e.g. SQLite).

Historically, cross-platform frameworks communicated with native code using asynchronous platform channels, which incur severe serialization, deserialization, and thread context-switching overhead.

Dart provides **`dart:ffi` (Foreign Function Interface)**, allowing Dart code to directly invoke native C functions and manipulate native memory pointers with **near-zero execution overhead**.

### Why Do We Need Dart FFI?
1. **Raw Performance:** Calls C functions directly within the same thread without JSON or binary serialization overhead.
2. **Reuse Existing C/C++ Libraries:** Integrate battle-tested C/C++ code without rewriting it in Dart.
3. **Hardware & OS APIs:** Directly bind to POSIX, Win32, or GPU graphics APIs.

---

## 2. Core Concepts

```mermaid
flowchart LR
    DartCode["Dart Application Code<br/>(Manages Dart Objects)"]
    FFILayer["dart:ffi Bridge<br/>• DynamicLibrary.open()<br/>• Native Type Mapping (Int32 -> int)<br/>• Pointer Allocation & Freeing"]
    NativeC["Native Dynamic Library<br/>(.dll / .so / .dylib)<br/>Compiled C/C++ Binary"]
    
    DartCode <-->|Direct Function Call (Zero Overhead)| FFILayer
    FFILayer <-->|Native C Function Execution| NativeC
```

### 1. `DynamicLibrary`
The entrypoint to FFI. Loads a compiled dynamic shared library (`.so` on Android/Linux, `.dylib` on macOS/iOS, `.dll` on Windows):
```dart
final dylib = DynamicLibrary.open('libnative_math.so');
```

### 2. Native vs. Dart Types
Because C and Dart have different type representations, FFI requires mapping native C types to Dart types:

| C Native Type (`dart:ffi`) | Dart Representation | Description |
|:---|:---|:---|
| `Int32` | `int` | 32-bit signed integer |
| `Int64` | `int` | 64-bit signed integer |
| `Double` | `double` | 64-bit floating point |
| `Pointer<T>` | `Pointer<T>` | Raw native memory address |
| `Utf8` (package:ffi) | `String` | Null-terminated UTF-8 C string |

### 3. Native Memory Management (`malloc` and `free`)
Memory allocated on the native C heap (via `calloc` or `malloc`) is **not managed by Dart's Garbage Collector**.
- You **must explicitly call `calloc.free(pointer)`** when finished. Failing to do so causes **native memory leaks**.
- In Dart 2.17+, `NativeFinalizer` allows attaching native cleanup functions to Dart objects so native memory is freed when the Dart object is garbage collected.

---

## 4. Syntax and Implementation

```dart
// bin/ffi_demo.dart

import 'dart:ffi' as ffi;
import 'dart:io';

// 1. Defining C Native Function Type Signatures
// C Signature: int32_t add_numbers(int32_t a, int32_t b);
typedef NativeAddFunc = ffi.Int32 Function(ffi.Int32 a, ffi.Int32 b);

// 2. Defining Dart Function Type Signatures
typedef DartAddFunc = int Function(int a, int b);

// C Signature: double calculate_hypotenuse(double a, double b);
typedef NativeHypotFunc = ffi.Double Function(ffi.Double a, ffi.Double b);
typedef DartHypotFunc = double Function(double a, double b);

void main() {
  print('=== Dart FFI Architecture Demonstration ===');

  // Conceptual demonstration of library loading based on operating system
  final String libraryPath;
  if (Platform.isWindows) {
    libraryPath = 'native_math.dll';
  } else if (Platform.isMacOS || Platform.isIOS) {
    libraryPath = 'libnative_math.dylib';
  } else {
    libraryPath = 'libnative_math.so';
  }

  print('Target Dynamic Library: $libraryPath');

  // In a real environment with compiled native binary:
  // final dylib = ffi.DynamicLibrary.open(libraryPath);
  // final DartAddFunc nativeAdd = dylib
  //     .lookup<ffi.NativeFunction<NativeAddFunc>>('add_numbers')
  //     .asFunction<DartAddFunc>();
  // print('Result from C function: ${nativeAdd(15, 27)}');

  // Demonstrating Native Pointer Allocation concepts
  demonstratePointerAllocation();
}

void demonstratePointerAllocation() {
  print('\n--- Native Memory Management Concepts ---');
  // Allocating native C memory
  print('1. Native memory must be allocated explicitly via malloc/calloc.');
  print('2. Pointers store raw memory addresses outside Dart GC.');
  print('3. NativeFinalizer can automatically free memory when Dart wrapper is GC-ed.');
}
```

---

## 5. Comparison: Dart FFI vs. Platform Channels (`MethodChannel`)

| Feature | Dart FFI (`dart:ffi`) | Flutter Platform Channels (`MethodChannel`) |
|:---|:---|:---|
| **Mechanism** | Direct C pointer / function call | Inter-process message serialization (Binary) |
| **Execution Overhead** | **Near-zero (Nanoseconds)** | Noticeable (Milliseconds due to serialization) |
| **Execution Thread** | **Synchronous on calling thread** | Asynchronous (hops to Platform main thread) |
| **Target Languages** | C, C++, Rust, Zig, Go (cgo) | Kotlin / Java (Android), Swift / Obj-C (iOS) |
| **Memory Access** | Direct access to raw native bytes | Data copied across channel bridge |
| **Primary Use Cases** | SQLite (Drift), ML, Audio, OpenCV | OS dialogs, Battery level, Bluetooth, GPS |

---

## 6. Interview Preparation

### Frequently Asked Questions

#### Q1: When should you choose Dart FFI over a Platform Channel (`MethodChannel`)?
- **Answer:** Choose **Dart FFI** when you need high-performance, synchronous calls to native C/C++/Rust code that processes high-frequency data (such as image pixel manipulation, real-time audio synthesis, cryptography, or local SQLite database queries). Choose **Platform Channels** when you need to access platform-specific Android/iOS SDK services that are asynchronous by nature (such as requesting OS runtime permissions, accessing camera hardware, or checking battery state).

#### Q2: What is `package:ffigen` in the Dart ecosystem?
- **Answer:** Writing manual FFI bindings for large C header files (`.h`) is tedious and prone to type signature mismatches that cause segmentation faults. `ffigen` is an official Dart tool that reads C header files directly using `libclang` and automatically generates 100% type-safe Dart FFI binding files containing all structs, unions, functions, and enums.

---

## 7. Practice Exercises

### Exercise 1 (Intermediate): FFI Type Signature Mapping
Given the C function signature:
```c
int64_t compute_hash(const char* data, int32_t length);
```
Write the corresponding `dart:ffi` `NativeFunction` typedef and the resulting Dart callable function signature.

---

## 8. References
- [Dart Official Guide: C Interoperability using FFI](https://dart.dev/guides/libraries/c-interop)
- [Dart FFI Package: package:ffi](https://pub.dev/packages/ffi)
- [Dart Binding Generator: package:ffigen](https://pub.dev/packages/ffigen)
