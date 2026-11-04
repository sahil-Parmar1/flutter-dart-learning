# Module 08: Local Storage and Databases

> **Stage:** Stage 3 (Continued) – Local Persistence, Offline-First & Cryptography  
> **Target Audience:** Intermediate to Senior Flutter Engineers  
> **Prerequisites:** [01-dart-fundamentals/05-oop-and-type-system/classes-and-objects.md](../01-dart-fundamentals/05-oop-and-type-system/classes-and-objects.md) | [07-networking-and-backend/](../07-networking-and-backend/)

---

## Overview

Persisting data on a mobile device is critical for offline-first architecture, caching remote network responses, storing user configuration preferences, and safeguarding authentication tokens.

Selecting the incorrect persistence mechanism can lead to security vulnerabilities (storing passwords or JWTs in plaintext XML) or rendering jank (running blocking disk I/O on the main UI thread).

This module details Flutter's local storage ecosystem across three tiers:
1. **Lightweight & Encrypted Key-Value:** `shared_preferences` (unencrypted settings) vs. `flutter_secure_storage` (hardware-backed Keychain/Keystore cryptography).
2. **High-Performance NoSQL:** Binary key-value document engines with **`Hive` / `hive_ce`** and **`Isar`**.
3. **Structured Relational SQL:** Type-safe, reactive SQLite with **`Drift`** and **`sqflite`**.

---

## Storage Solution Matrix

| Technology | Storage Type | Encryption | Relational Joins? | Speed | Best Used For |
|:---|:---|:---:|:---:|:---:|:---|
| **`shared_preferences`** | Key-Value (XML / Plist) | ❌ Plaintext | ❌ | Fast | Theme mode, onboarding completed flag |
| **`flutter_secure_storage`**| Key-Value (Keystore/Keychain) | ✅ Hardware AES | ❌ | Medium | JWT Access/Refresh tokens, API keys |
| **`Hive / hive_ce`** | NoSQL Binary Box | Optional | ❌ | Ultra-Fast | Offline feed cache, user drafts, cart |
| **`Drift (SQLite)`** | Relational SQL | Optional (SQLCipher) | ✅ Full SQL | High | Complex offline relational databases, message logs |

---

## Directory Roadmap

```
08-local-storage-and-databases/
├── README.md
├── 01-shared-preferences-and-secure-storage/
│   └── preferences-and-secure-storage.md    # SharedPreferences vs FlutterSecureStorage, JWT storage
├── 02-nosql-hive-and-isar/
│   └── hive-and-isar-guide.md               # Hive boxes, TypeAdapters, hive_ce, Isar NoSQL
└── 03-relational-sqlite-and-drift/
    └── sqlite-and-drift-guide.md            # Drift schema DSL, type-safe queries, migrations
```
