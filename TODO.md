# TODO.md — Android 12 (API 31) Compatibility Guide

## Introduction

This document is a step-by-step guide for making SeekerClaw fully compatible with **Android 12 (API level 31)** as the minimum SDK version, down from the original Android 14 (API 34) requirement. The goal is to broaden device support while preserving all core features: Node.js runtime, AI agent (Claude), Solana wallet, Telegram bot, device controls, web search, skills, and cron jobs.

---

## 1. Global Build Changes

- [x] Update `minSdk` from `34` to `31` in `app/build.gradle.kts`
- [ ] Keep `compileSdk = 35` (required for latest AndroidX APIs and lint checks)
- [ ] Keep `targetSdk = 35` (ensures latest behavior on modern devices)
- [ ] Verify JDK 17 compatibility remains (no changes needed)

**File:** `app/build.gradle.kts` line 36

---

## 2. Dependency Adjustments

All current dependencies support API 31+. No downgrades are required:

- [x] `androidx.core:core-ktx:1.15.0` — supports minSdk 19+
- [x] `androidx.lifecycle:*:2.8.7` — supports minSdk 21+
- [x] `androidx.activity:activity-compose:1.9.3` — supports minSdk 21+
- [x] `androidx.compose:compose-bom:2024.12.01` — supports minSdk 21+
- [x] `androidx.navigation:navigation-compose:2.8.5` — supports minSdk 21+
- [x] `androidx.camera:camera-*:1.4.1` — supports minSdk 21+
- [x] `com.google.mlkit:barcode-scanning:17.3.0` — supports minSdk 21+
- [x] `org.nanohttpd:nanohttpd:2.3.1` — pure Java, no API requirements
- [x] `com.solanamobile:mobile-wallet-adapter-clientlib-ktx:2.0.3` — supports minSdk 23+
- [x] `org.sol4k:sol4k:0.4.2` — pure Kotlin/JVM
- [x] `com.google.firebase:firebase-bom:33.7.0` — supports minSdk 21+
- [x] `nodejs-mobile v18.20.4` — native ARM64 binary, no API-level dependency

---

## 3. Foreground Service and Background Execution

Android 14 (API 34) introduced mandatory foreground service type declaration. On API 31–33, foreground services work without specific type enforcement.

- [x] Add version-gated `startForeground` call in `OpenClawService.kt`:
  - API 34+ (`UPSIDE_DOWN_CAKE`): Use `ServiceCompat.startForeground()` with `FOREGROUND_SERVICE_TYPE_SPECIAL_USE`
  - API 31–33: Use standard `startForeground(id, notification)` (no type required)
- [x] Add `tools:targetApi="34"` to `FOREGROUND_SERVICE_SPECIAL_USE` permission in `AndroidManifest.xml`
- [x] Keep `foregroundServiceType="specialUse"` in manifest service declaration (ignored gracefully on API <34)
- [x] Verify `startForegroundService()` call (API 26+, safe for minSdk 31)
- [x] Verify `START_STICKY` behavior works across API 31–35

**Files:**
- `app/src/main/java/com/seekerclaw/app/service/OpenClawService.kt`
- `app/src/main/AndroidManifest.xml`

---

## 4. Permissions and Manifest Changes

### POST_NOTIFICATIONS (API 33+)

The `POST_NOTIFICATIONS` permission was introduced in API 33 (TIRAMISU). On API 31–32, notification permission is automatically granted.

- [x] Guard `POST_NOTIFICATIONS` permission check in `SetupScreen.kt` with `Build.VERSION.SDK_INT >= TIRAMISU`
- [x] Guard runtime permission request in `SetupScreen.kt` notification dialog
- [x] Guard permission status in `ConfigManager.writePlatformMd()` — report "granted" on API <33

### FOREGROUND_SERVICE_SPECIAL_USE (API 34+)

- [x] Add `tools:targetApi="34"` annotation to suppress lint warnings on API <34

### Exported Components (API 31+ requirement)

Android 12 requires explicit `android:exported` on all components with intent-filters.

- [x] `MainActivity` — `exported="true"` ✓
- [x] `OpenClawService` — `exported="false"` ✓
- [x] `SolanaAuthActivity` — `exported="false"` ✓
- [x] `CameraCaptureActivity` — `exported="false"` ✓
- [x] `QrScannerActivity` — `exported="false"` ✓
- [x] `BootReceiver` — `exported="false"` ✓

### PendingIntent Mutability (API 31+ requirement)

Android 12 requires `FLAG_MUTABLE` or `FLAG_IMMUTABLE` on all PendingIntents.

- [x] `OpenClawService.createNotification()` — uses `FLAG_IMMUTABLE` ✓
- [x] `OpenClawService.createSetupNotification()` — uses `FLAG_IMMUTABLE` ✓

**Files:**
- `app/src/main/AndroidManifest.xml`
- `app/src/main/java/com/seekerclaw/app/ui/setup/SetupScreen.kt`
- `app/src/main/java/com/seekerclaw/app/config/ConfigManager.kt`

---

## 5. Device Access Components

### SMS (`AndroidBridge.handleSms`)

- [x] `context.getSystemService(SmsManager::class.java)` — available from API 31 (S), safe with new minSdk

### Clipboard, Contacts, Phone, Location, TTS, Camera, Battery, Storage, Network

- [x] All use APIs available since API 21–26, no changes needed for API 31 minSdk

### Application.getProcessName() (SeekerClawApplication)

- [x] Available since API 28 (P), safe for minSdk 31

### enableEdgeToEdge() (MainActivity, QrScannerActivity)

- [x] AndroidX Compat method, works on all API levels

### setShowWhenLocked / setTurnScreenOn (CameraCaptureActivity)

- [x] Available since API 27 (O_MR1), safe for minSdk 31

**File:** `app/src/main/java/com/seekerclaw/app/bridge/AndroidBridge.kt`

---

## 6. Node.js Runtime Compatibility

- [x] `nodejs-mobile v18.20.4` ships pre-compiled ARM64 `libnode.so` — no API-level dependency
- [ ] Test Node.js startup and Telegram bot on Android 12 device/emulator
- [ ] Verify JNI bridge (`native-lib.so` + `libnode.so`) loads correctly on Android 12
- [ ] Test long-running stability (24h+) on Android 12

**Notes:**
- nodejs-mobile is compiled against a low NDK API level for broad compatibility
- The JNI bridge (`native-lib.cpp`) uses standard POSIX APIs
- No Android-version-specific native code

---

## 7. Solana and Crypto Integrations

- [x] `mobile-wallet-adapter-clientlib-ktx:2.0.3` supports API 23+
- [x] `sol4k:0.4.2` is pure Kotlin/JVM (no Android API requirements)
- [ ] Verify MWA (Mobile Wallet Adapter) works with Phantom/Solflare on Android 12
- [ ] Seed Vault is Seeker-specific hardware — software fallback via MWA works on any device
- [ ] Test Jupiter swap flow end-to-end on Android 12

---

## 8. Testing and Debugging

- [ ] Create Android 12 (API 31) emulator in Android Studio
- [ ] Test full setup flow (QR scan, manual entry, notification permission skip on API <33)
- [ ] Test foreground service lifecycle (start, stop, restart, boot receiver)
- [ ] Test all AndroidBridge endpoints (battery, SMS, camera, location, etc.)
- [ ] Test Telegram bot connectivity and message handling
- [ ] Test Solana wallet connect and transaction signing
- [ ] Test 24h continuous operation with battery optimization exempt
- [ ] Test app behavior with aggressive battery saving enabled
- [ ] Run all unit tests: `./gradlew testDappStoreDebugUnitTest`

---

## 9. Potential Breaking Changes / Known Limitations

| Feature | API 31–32 | API 33 | API 34+ |
|---------|-----------|--------|---------|
| Foreground service type | Not enforced | Not enforced | Required (specialUse) |
| POST_NOTIFICATIONS | Auto-granted | Runtime permission | Runtime permission |
| Background service stability | Good (no type enforcement) | Good | Best (explicit type) |
| Battery killing (OEM) | More aggressive on some OEMs | Better | Best on stock Android |
| Notification channels | Supported | Supported | Supported |

### Risks on Android 12–13:
1. **OEM battery killing**: Samsung, Xiaomi, Huawei may kill background services more aggressively. Mitigation: battery optimization exemption prompt + user education.
2. **Foreground service without type**: On API 31–33, the service runs as a generic foreground service. This works but provides less system-level guarantees than typed services on API 34+.
3. **No notification permission dialog on API <33**: Notifications are auto-granted, which is actually better UX.

---

## 10. Final Steps

1. Build debug APK:
   ```bash
   ./gradlew assembleDappStoreDebug
   ```
2. Install on Android 12 device:
   ```bash
   adb install app/build/outputs/apk/dappStore/debug/app-dappStore-debug.apk
   ```
3. Verify:
   - [ ] App launches and shows setup screen
   - [ ] Notification permission dialog is skipped on API <33
   - [ ] Service starts and shows foreground notification
   - [ ] Node.js runtime starts successfully
   - [ ] Telegram bot responds to messages
   - [ ] All device bridge endpoints work
   - [ ] Boot receiver auto-starts service after reboot
4. Build release APK:
   ```bash
   ./gradlew assembleDappStoreRelease
   ```

---

## Summary of Code Changes Made

| File | Change | Reason |
|------|--------|--------|
| `app/build.gradle.kts` | `minSdk = 34` → `minSdk = 31` | Enable Android 12+ support |
| `AndroidManifest.xml` | Added `tools:targetApi="34"` to `FOREGROUND_SERVICE_SPECIAL_USE` | Suppress lint warning on API <34 |
| `OpenClawService.kt` | Version-gated `startForeground` with `ServiceCompat` | Use typed foreground service on API 34+, standard on API <34 |
| `SetupScreen.kt` | Guard `POST_NOTIFICATIONS` check/request behind `TIRAMISU` | Permission doesn't exist on API <33 |
| `ConfigManager.kt` | Guard `POST_NOTIFICATIONS` status behind `TIRAMISU` | Report "granted" on API <33 |
