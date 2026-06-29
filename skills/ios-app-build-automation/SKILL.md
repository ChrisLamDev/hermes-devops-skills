---
name: ios-app-build-automation
description: Complete iOS App from Source Code to Build fully automated pipeline. Uses XcodeGen + xcodebuild CLI, no Xcode GUI required.
version: 2.0.0
author: Hermes Agent (BuddhaTranslator case study — real-world battle-tested)
license: MIT
tags: [ios, xcode, automation, build, xcodegen, swift, whisper, cicd]
---

# 📱 iOS App Build Automation Pipeline

## Overview

Automated pipeline using XcodeGen + xcodebuild CLI to build iOS apps from source code. **No Xcode GUI required.** Supports macOS fallback build (when iOS Simulator is not installed).

## Prerequisites

```bash
brew install xcodegen       # Xcode project generator (v2.45+)
xcode-select --install      # Xcode command line tools
```

## Directory Structure

```
Project_Root/
├── project.yml              # XcodeGen spec (only needs manual maintenance)
├── autobuild.sh             # Automation pipeline script
├── YourApp/                 # App source code directory
│   ├── YourAppApp.swift
│   ├── ContentView.swift
│   ├── Info.plist
│   ├── Models/
│   ├── Services/
│   ├── Views/
│   └── Assets.xcassets/
├── WhisperLocal/            # Embed whisper.cpp (without SPM)
│   ├── whisper.cpp
│   ├── ggml.c
│   ├── include/
│   └── ggml-metal.metal (excluded on macOS to avoid Metal toolchain dep)
└── Resources/
    ├── common_phrases/
    └── tts_cache/
```

## Pipeline Flow

```
Source code changes (single edit point: Project_Root/YourApp/)
    │
    ▼
Step 1: sync_source() — Ensure directory structure + copy source files
    │
    ▼
Step 2: xcodegen generate — Generate complete .xcodeproj from project.yml
    │   (Auto-generates all PBXBuildFile, PBXFileReference, PBXGroup)
    ▼
Step 3: xcodebuild build — CLI build (Simulator / device / macOS)
    │
    ▼
Build success → .app bundle at $DERIVED_DATA/Build/Products/Debug/
```

## project.yml Template

```yaml
name: YourAppName
options:
  bundleIdPrefix: com.yourdomain
  deploymentTarget:
    iOS: "18.0"
    macOS: "15.0"
  xcodeVersion: "26.5"
  createIntermediateGroups: true

settings:
  base:
    INFOPLIST_FILE: YourApp/Info.plist
    DEVELOPMENT_TEAM: ""
    MARKETING_VERSION: "1.0"
    CURRENT_PROJECT_VERSION: "1"
    SWIFT_VERSION: "6.0"

targets:
  YourApp:
    type: application
    platform: iOS
    sources:
      - path: YourApp/Models
      - path: YourApp/Services
      - path: YourApp/Views
      - path: YourApp/Resources
        type: folder
      - path: YourApp/YourAppApp.swift
      - path: YourApp/ContentView.swift
      - path: YourApp/Assets.xcassets
      # Embed whisper.cpp directly into app target sources
      - path: WhisperLocal/
        excludes:
          - "coreml/"
          - "ggml-metal.metal"
    settings:
      base:
        PRODUCT_BUNDLE_IDENTIFIER: com.yourdomain.YourApp
        INFOPLIST_FILE: YourApp/Info.plist
        TARGETED_DEVICE_FAMILY: "1"
        HEADER_SEARCH_PATHS: "$(SRCROOT)/WhisperLocal/include"
        OTHER_LDFLAGS: "-lz -lstdc++ -framework Accelerate -framework CoreML"

  # macOS build target (for testing when iOS Simulator not installed)
  YourAppMac:
    type: application
    platform: macOS
    sources:
      # Same sources as iOS target
    settings:
      base:
        PRODUCT_BUNDLE_IDENTIFIER: com.yourdomain.YourApp.mac
        SWIFT_ACTIVE_COMPILATION_CONDITIONS: $(inherited) MACOS_BUILD
        HEADER_SEARCH_PATHS: "$(SRCROOT)/WhisperLocal/include"
        OTHER_LDFLAGS: "-lz -lstdc++ -framework Accelerate -framework CoreML"
```

## Whisper.cpp Embed Strategy

### ❌ Why Not SPM
whisper.spm uses `unsafeFlags` for optimization.
Xcode policy: **SPM packages with unsafeFlags cannot be used as dependencies**.
```error
The package product 'whisper' cannot be used as a dependency
because it uses unsafe build flags.
```

### ✅ Solution: Direct Embed
```bash
# First time setup (copy from cached SPM checkout into project)
WHISPER_SRC="$HOME/Library/Developer/Xcode/DerivedData/YourApp/SourcePackages/checkouts/whisper.spm/Sources/whisper"
mkdir -p ProjectRoot/WhisperLocal/include
cp -R "$WHISPER_SRC/"* ProjectRoot/WhisperLocal/
```

**Don't add XcodeGen target dependency** — XcodeGen 2.45.4 has a bug in validation of local static library targets (`invalid dependency` error). Embed source directly in the app target.

## Info.plist — Common Traps

**Apple plist parser does NOT accept `//` comments!**
```xml
<!-- ❌ This will cause build failure -->
// Info.plist comment

<!-- ✅ Correct format -->
<?xml version="1.0" encoding="UTF-8"?>
```

Use `plutil -lint Info.plist` to validate first.

## Swift 6 Concurrency Compatibility

### 1. `non-final class cannot conform to 'Sendable'`
```swift
// ✅ Fix: mark as final or @preconcurrency import
final class TTSEngine: NSObject, AVSpeechSynthesizerDelegate { ... }
```

### 2. `sending 'self' risks causing data races`
```swift
// ✅ Fix: @MainActor on ObservableObject
@MainActor class TranslationPipeline: ObservableObject { ... }
```

### 3. `missing CommonCrypto symbols`
```swift
import CommonCrypto

extension String {
    var md5: String { ... }
}
// cacheKey 可能係 NSString，用時 cast：(cacheKey as String).md5
```

### 4. macOS unavailable APIs
```swift
// AVAudioSession — macOS unavailable
#if !os(macOS)
let audioSession = AVAudioSession.sharedInstance()
#endif

// NavigationView + .navigationViewStyle(.stack) — iOS only
// Use NavigationStack (iOS 16+)
NavigationStack { ... }
```

## autobuild.sh Main Features

```bash
./autobuild.sh              # Sync → Gen → Build (macOS fallback)
./autobuild.sh device       # Sync → Gen → Build (iPhone)
./autobuild.sh clean        # Clean → Sync → Gen → Build
./autobuild.sh open         # Sync → Gen → Open Xcode
```

## Known Pitfalls

### 1. XcodeGen Target Dependency Bug
XcodeGen 2.45.4 validation: `invalid dependency` error.
**Solution**: Add C/ObjC/C++ source files directly to the app target sources, don't use a separate target.

### 2. iOS Simulator Runtime Not Installed
**Symptom**: `Unable to find a device matching the provided destination specifier`
**Diagnosis**: `xcrun simctl list runtimes` shows nothing
**Solution**:
- Xcode → Settings → Components → Download iOS Simulator
- Or use macOS target build first (`-scheme YourAppMac -destination 'platform=macOS'`)

### 3. Missing Metal Toolchain
**Symptom**: `cannot execute tool 'metal' due to missing Metal Toolchain`
**Solution**: `xcodebuild -downloadComponent MetalToolchain` (~688MB)

### 4. Whisper SPM Unsafe Flags
**Solution**: Embed source code directly (detailed above)

### 5. Info.plist Comment Problem
**Solution**: Validate with `plutil -lint`. Info.plist cannot have `//` comments.

### 6. DerivedData Stale Cache
If code changes produce stale build results:
- Clean DerivedData
- Or use `Product → Clean Build Folder`

## Common Code Fixes (macOS Cross-Compile)

When using macOS target to build iOS app (for quick testing), these fixes are needed:

### TTSEngine Sendable Error
```swift
// Problem: non-final class cannot conform to Sendable
// Fix: mark final + @preconcurrency import
import Foundation
@preconcurrency import AVFoundation
import CommonCrypto

final class TTSEngine: NSObject, AVSpeechSynthesizerDelegate {
```

### VADService AVAudioSession macOS
```swift
// Problem: AVAudioSession macOS unavailable
// Fix: #if !os(macOS)
#if !os(macOS)
private let audioSession = AVAudioSession.sharedInstance()
#endif
```

### TranslationPipeline Data Races
```swift
// Problem: sending 'self' risks causing data races (Swift 6)
// Fix: @MainActor
@MainActor
class TranslationPipeline: ObservableObject {
```

### NavigationView macOS
```swift
// 問題：.navigationViewStyle(.stack) macOS unavailable
// Fix: 用 NavigationStack (iOS 16+ / macOS 13+)
NavigationStack {
    ConversationView()
}
```

### UIColor / UITraitCollection
```swift
// 問題：Color(.systemBackground) 唔 cross-platform
// Fix：用 explicit RGB
Color(.sRGB, red: 1, green: 1, blue: 1, opacity: 1)
```

### 6. Simulator Runtime Missing
如果 `xcodebuild` 出 `Unable to find a device` 但 SDK 已存在（`ls /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/` check 到 `.sdk`），即係 Simulator runtime 未裝。

**解決**：
```bash
# Check runtime status
xcrun simctl list runtimes

# Download only iOS Simulator runtime
xcodebuild -downloadPlatform iOS
# Or download all (will do watchOS/tvOS too - slower):
xcodebuild -downloadAllPlatforms
```

Download 完之後 check：
```bash
xcrun simctl list devices  # Should show iPhone 17 Pro etc.
```

### 7. Whisper ObjC ARC Error
whisper.cpp 嘅 `ggml-metal.m` 用咗 `[release]` message，但 Xcode project default 係 ARC。

**症狀**：
```
error: 'release' is unavailable: not available in automatic reference counting mode
```

**解決**：喺 project.yml **用獨立 path entry**（XcodeGen map syntax `compilerFlags: file: flag` 唔 work）：
```yaml
sources:
  # whisper.cpp source files (without ggml-metal.m — 獨立處理)
  - path: WhisperLocal/
    excludes:
      - "coreml/"
      - "ggml-metal.metal"
    includes:
      - "*.c"
      - "*.cpp"
      - "*.h"
  # ggml-metal.m 獨立出嚟加 -fno-objc-arc（❗ map syntax 唔得，要用 array）
  - path: WhisperLocal/ggml-metal.m
    compilerFlags: ["-fno-objc-arc"]
```

### 8. Header Files 錯誤 Copy 入 .app Bundle
WhisperLocal/include 如果加做 `type: folder` source，build 時會 copy 入 .app bundle，導致 simctl install fail。

**症狀**：.app bundle 入面有 `include/ggml.h`、`include/whisper.h`
**解決**：用 `excludes: ["include/"]` 排除，headers 只透過 `HEADER_SEARCH_PATHS` 使用：
```yaml
sources:
  - path: WhisperLocal/
    excludes:
      - "coreml/"
      - "ggml-metal.metal"
      - "include/"       # ❗ 唔好 copy headers 入 .app
    includes:
      - "*.c"
      - "*.cpp"
      - "*.h"
```
```yaml
settings:
  base:
    HEADER_SEARCH_PATHS: "$(SRCROOT)/WhisperLocal/include"  # headers 只喺呢度用
```

### 9. Xcode 26.5 Simulator Install Bug (cryptex volume 已知 bug, 2026-06-25 confirmed)
**症狀**：`simctl install` 出 `Missing bundle ID` error，雖然 `plutil -lint Info.plist` 係 OK，`defaults read Info CFBundleIdentifier` 都正常傳回 bundle ID。

**原因**：Xcode 26.5 嘅 iOS Simulator runtime 用 cryptex volume 掛載（`/Library/Developer/CoreSimulator/Volumes/iOS_23F77/`），simctl 讀 Info.plist 時用到 runtime 嘅 plist parser 但 cryptex volume 有 bug。

Debug builds 多咗 `__preview.dylib` + `BuddhaTranslator.debug.dylib` split binary 都加劇問題。

**解決方法 (by priority)**：
1. **用 Xcode GUI Run (⌘R)** — Xcode IDE 嘅 install 流程繞過 simctl bug，直接 work
2. **Release build** — 雖然都係 fail，但 .app bundle 係正常結構（冇 split dylib）
3. **Workaround** — 如果一定要 CLI，試 `xcodebuild build-for-testing`（build 成功但 launch 仍可能 fail）

**預防**：project.yml 加 `configs` block 跳過 codesign（simulator builds 唔需要）：
```yaml
settings:
  configs:
    debug:
      CODE_SIGNING_ALLOWED: "NO"
    release:
      CODE_SIGNING_ALLOWED: "NO"
```

## Case Study: BuddhaTranslator

### 踩過嘅坑（時間線）

| Phase | Method | Result | Problem |
|-------|--------|--------|---------|
| 1 | 手寫 .pbxproj | ❌ Build failed | productTypeIdentifier missing |
| 2 | Xcode GUI Add Files | ⚠️ Lost sync | 人為操作不可重複 |
| 3 | ContentView.swift overwrite | ❌ Cache 舊 code | Xcode cache issue |
| 4 | SPM whisper.spm | ❌ unsafeFlags | Xcode policy blocks it |
| 5 | XcodeGen + SPM | ❌ 同上 | 同一問題 |
| 6 | XcodeGen + embed source | ✅ XcodeGen ✅ Build compile | Metal toolchain missing |
| 7 | Full pipeline (final) | ✅ XcodeGen ✅ Code compile | 剩 iOS Simulator runtime 未裝 |

### Final 檔案
- `project.yml` — ~92 lines
- `autobuild.sh` — ~419 lines bash
- `WhisperLocal/` — 17 files (whisper.cpp + ggml source)
- `Info.plist` — 37 lines XML

### Key Lesson
**Source code 修改點只有一個**。唔好分散去多個目錄。所有改動喺 `BuddhaTranslator/` folder 做，`autobuild.sh` 負責 sync + generate + build。

## Real Device Deployment (CLI-Only)

### devicectl 正確語法（經 trial & error 驗證）

```bash
# ✅ Correct
xcrun devicectl device install app \
    --device "iPhone Name" \
    "/path/to/Release-iphoneos/App.app"

# ❌ Wrong: --device is positional arg syntax
xcrun devicectl device install app --device "name" --package path  # Unknown option '--package'
xcrun devicectl install app --device name path  # Unknown option '--device'
```

**devicectl 限制：**
- 冇 `launch` subcommand — user 要自己㩒 app icon
- 冇 log streaming — 只能用 `device info apps` 確認已安裝
- `device process launch` 可以 launch app 但冇 stdout capture

### Keychain Lock 繞過

第一次 Debug build 成功簽名後，第二次 build 會 `errSecInternalComponent`：
- Debug build 有 `BuddhaTranslator.debug.dylib` + `__preview.dylib`
- 第二次 code sign 時 keychain 鎖咗
- **Fix**: 用 `-configuration Release` — Release build 冇 debug dylib

從 source code 到真機安裝嘅全自動流程，**唔使㩒 Xcode 任何掣**。

### Pre-flight: Check Device Connections

```bash
# List connected devices (offline + online)
xcrun xctrace list devices 2>&1

# Check via devicectl (shows "available (paired)" status)
xcrun devicectl list devices 2>&1

# Check if device shows in xcodebuild destinations
xcodebuild -project YourApp.xcodeproj \
    -scheme YourApp \
    -showdestinations 2>&1 | grep "platform:iOS, arch:arm64"
```

**Expected output:**
```
{ platform:iOS, arch:arm64, id:00008140-XXXXXXXX, name:My iPhone }
```

### Step 0: Find the Apple Developer Team ID

當 `security find-identity -v -p codesigning` 出 **"0 valid identities found"** 時，代表未有任何 signing certificate。即使用戶已經喺 Xcode → Settings → Accounts 加咗 Apple ID，仍然需要搵返個 Team ID：

```bash
# Extract Team ID from Xcode preferences (no signing cert required)
defaults read com.apple.dt.Xcode 2>&1 | grep -A 5 "IDEProvisioningTeamByIdentifier"
```

**Expected output:**
```
{
    isFreeProvisioningTeam = 1;
    teamID = ABC123XYZ;
    teamName = "User Name (Personal Team)";
    teamType = "Personal Team";
```

Copy 個 `teamID`（如 `ABC123XYZ`）落 project.yml。

### Step 1: Update project.yml with Team ID + Automatic Signing

```yaml
settings:
  base:
    DEVELOPMENT_TEAM: "ABC123XYZ"  # 從 Xcode preferences 抽出
    # 唔好留空，否則 xcodebuild 會出 "requires a development team" error

targets:
  YourApp:
    settings:
      base:
        CODE_SIGN_STYLE: Automatic  # or leave out, set via CLI
```

**唔使**加 `CODE_SIGN_STYLE=Automatic` 落 `project.yml` — 可以由 CLI 參數傳入。

### Step 2: Build for Real Device with Automatic Provisioning

```bash
cd /path/to/project

# Generate xcodeproj (always do this first)
rm -rf YourApp.xcodeproj
xcodegen generate --project . --spec project.yml

# Build + sign for real device
xcodebuild build \
    -project YourApp.xcodeproj \
    -scheme YourApp \
    -destination 'platform=iOS,id=DEVICE_ID' \
    -allowProvisioningUpdates \
    CODE_SIGN_STYLE=Automatic \
    CODE_SIGNING_REQUIRED=YES \
    2>&1 | tail -20
```

**Key flags:**
| Flag | Purpose |
|------|---------|
| `-allowProvisioningUpdates` | Allows xcodebuild to **auto-create** the development certificate + provisioning profile on the Apple Developer portal. Without this, no cert will be generated even with `DEVELOPMENT_TEAM` set. |
| `CODE_SIGN_STYLE=Automatic` | Use automatic signing (matches the Personal Team setup) |
| `CODE_SIGNING_REQUIRED=YES` | Enforce signing (default when building for real device) |

**What happens during build:**
```
Signing Identity:     "Apple Development: user@email.com (XXXXXX)"
Provisioning Profile: "iOS Team Provisioning Profile: com.chrislam.MyApp" (UUID)
```

If this is the first time, Xcode will:
1. Register the bundle ID on the Apple Developer portal
2. Generate a development certificate in Keychain
3. Create a team provisioning profile
4. Sign all binaries (`.app`, `__preview.dylib`, `.debug.dylib`)

### Step 3: Install on Real Device via devicectl

```bash
BUILD_DIR="$HOME/Library/Developer/Xcode/DerivedData/YourApp-XXXXX/Build/Products/Debug-iphoneos"

# Find the device name from Step 0, then:
xcrun devicectl device install app \
    --device "iPhone Name" \
    "$BUILD_DIR/YourApp.app"
```

**Correct syntax learned from trial-and-error:**
```
devicectl device install app --device <name|uuid|hostname> <path-to-.app>
```

⚠️ **NOT** `--package` flag — the path is a **positional argument**, not a named option.
⚠️ **NOT** `--device E0D1FB3E...` with `--package` — `app` subcommand takes `<path>` as positional arg.
⚠️ The `device` subcommand structure is: `devicectl device install app --device NAME APP_PATH`

**Expected output:**
```
Acquired tunnel connection to device.
Enabling developer disk image services.
Acquired usage assertion.
App installed:
• bundleID: com.chrislam.MyApp
• installationURL: file:///private/var/containers/Bundle/Application/UUID/MyApp.app/
```

### Step 4: Launch the App

**⚠️ devicectl has NO `launch` subcommand** — unlike `simctl launch`, there is no CLI way to launch an app on a real device via devicectl (as of Xcode 26.5).

User must manually tap the app icon on their iPhone:
- App appears on home screen with the name from `CFBundleDisplayName` in Info.plist
- First launch triggers permission dialogs (microphone, speech recognition)
- Free Apple ID signing: app expires after **7 days** → must rebuild and reinstall

### Step 5: Verify Installation

```bash
# Check app is installed
xcrun devicectl device info apps --device "iPhone Name" 2>&1 | grep "YourApp"

# Output: "AppName   com.chrislam.MyApp   1.0.0     1"
```

### Full Automation Script (autobuild.sh with device support)

```bash
#!/bin/bash
set -e

PROJECT_DIR="$(cd "$(dirname "$0")" && pwd)"
SCHEME="YourApp"
DEVICE_ID="00008140-XXXXXXXX"  # From xcodebuild -showdestinations
DEVICE_NAME="My iPhone"
BUILD_DIR="$HOME/Library/Developer/Xcode/DerivedData"
TEAM_ID="ABC123XYZ"  # From defaults read com.apple.dt.Xcode

# Regenerate xcodeproj
rm -rf "$PROJECT_DIR/YourApp.xcodeproj"
cd "$PROJECT_DIR"
xcodegen generate --project . --spec project.yml

# Check if device is connected
if xcrun devicectl list devices 2>&1 | grep -q "$DEVICE_NAME"; then
    echo "📱 Device connected: $DEVICE_NAME"
else
    echo "❌ Device not found. Run: xcrun devicectl list devices"
    exit 1
fi

# Build for real device
xcodebuild build \
    -project YourApp.xcodeproj \
    -scheme "$SCHEME" \
    -destination "platform=iOS,id=$DEVICE_ID" \
    -allowProvisioningUpdates \
    CODE_SIGN_STYLE=Automatic \
    CODE_SIGNING_REQUIRED=YES \
    -derivedDataPath "$BUILD_DIR" \
    2>&1 | grep -E "BUILD SUCCEEDED|BUILD FAILED|error:|Signing Identity"

# Find the .app
APP_PATH=$(find "$BUILD_DIR" -name "*.app" -path "*/Debug-iphoneos/*" -type d 2>/dev/null | head -1)
if [ -z "$APP_PATH" ]; then
    echo "❌ Can't find .app in DerivedData"
    exit 1
fi

# Install on device
echo "📲 Installing..."
xcrun devicectl device install app \
    --device "$DEVICE_NAME" \
    "$APP_PATH" 2>&1

echo ""
echo "✅ App installed! Tap it on your iPhone to launch."
echo "   Bundle: $(defaults read "$APP_PATH/Info.plist" CFBundleIdentifier 2>/dev/null)"
```

### Known Pitfalls: Real Device

| Pitfall | Symptom | Solution |
|---------|---------|----------|
| `security find-identity -v -p codesigning` = 0 valid identities | Signing identity not created yet | Use `defaults read com.apple.dt.Xcode` to find `IDEProvisioningTeamByIdentifier` → extract teamID → use `-allowProvisioningUpdates` to auto-create cert during build |
| `requires a development team` | BUILD FAILED | Set `DEVELOPMENT_TEAM` in project.yml or pass via CLI |
| `devicectl device install app --device ... --package ...` | `Unknown option '--package'` | `<path>` is positional, not `--package`. Correct: `devicectl device install app --device NAME PATH` |
| `devicectl device launch app ...` | `Unknown subcommand` | devicectl has no `launch`. User must tap manually |
| Free Apple ID signing | App expires after 7 days | Rebuild + reinstall weekly, or get $99/yr Apple Developer Program |
| `AllowProvisioningUpdates` requires Xcode accounts setup | Error about no accounts | User must add Apple ID in Xcode → Settings → Accounts first (at least once)
| `errSecInternalComponent` signing failure | Debug build CodeSign fails with this error | Keychain is locked (no password = can't unlock via CLI). **Fix:** Use `-configuration Release` — Release builds skip `__preview.dylib` + `BuddhaTranslator.debug.dylib` split binary, avoiding the multi-binary signing chain that triggers the error in locked-keychain scenario |
| 第一次安裝後彈「不受信任的開發者」 | iOS blocks launch for free Apple ID signed apps | Go to **Settings → General → VPN & Device Management** → tap the developer profile → tap **Trust** → confirm |
