# Flutter Build Flavors Guide

Use this as a reusable reference for Flutter projects that define build flavors such as `dev`
and `prod` across Android, iOS, and Windows desktop.

This guide documents one well-tested approach for each platform. It is not the only valid
approach. Teams with different CI pipelines, signing strategies, or flavor matrices should treat
the examples here as a starting point and document their deviations in `docs/architecture.md`.

> **Reflects Flutter 3.41 / Dart 3.11 (early 2026).** Where a constraint comes from a
> specific Flutter version (UIScene, 16 KB pages, Java 17, etc.), the version is named so
> you can verify against your toolchain.

---

## Toolchain Prerequisites

Before any flavored build runs, the host toolchain MUST satisfy these minimums:

| Concern | Minimum | Notes |
|---------|---------|-------|
| Flutter | 3.38 (3.41+ recommended) | Below 3.38 misses UIScene auto-migration and Java 17 alignment |
| Dart | 3.11 (ships with Flutter 3.41) | `flutter pub run` is deprecated; use `dart run` |
| JDK (Android builds) | **Java 17** (Flutter 3.38+) | Java 11 builds fail; Gradle 8.14+ needs JDK 17 |
| AGP (Android Gradle Plugin) | 8.x — **NOT 9.x** | AGP 9 audit is paused as of Flutter 3.41 |
| iOS deployment target | **iOS 13** (Flutter 3.41 raised from iOS 12) | Set in `ios/Podfile` and Xcode Build Settings |
| Xcode (iOS prod builds) | Xcode 26 (iOS 26 SDK) | Required for App Store submissions; deadline (April 2026) is now in force |
| Android target SDK (Play Store) | API 35 (Android 15) with **16 KB page support** | Mandatory from Nov 1 2025 / May 31 2026 |

If any of these is missing, fix it before adopting any flavor configuration below — the
flavor mechanics work, but the resulting build will fail at submission time.

---

## Flavor Basics

A flavor usually represents an environment or release lane.

| Flavor | Typical Purpose | Typical Mode |
|--------|-----------------|--------------|
| `dev` | Local development, QA, internal testing | `debug` |
| `prod` | Production builds, store submissions, public release | `release` |

Common combinations:

| Flavor | Mode | Signing | Notes |
|--------|------|---------|-------|
| `dev` | `debug` | Automatic debug keystore | Daily development — no setup required |
| `dev` | `release` | Configurable — see signing strategy notes | QA release-like build |
| `prod` | `debug` | Automatic debug keystore | Rare; production config with debug tooling |
| `prod` | `release` | Release keystore required | Store submission or public distribution |

---

## How The Flavor Reaches Dart Code

Flutter ≥ 3.19 owns the `FLUTTER_APP_FLAVOR` compile-time environment variable. Whenever
`--flavor` is passed to `flutter run` or `flutter build`, the tool automatically injects
`FLUTTER_APP_FLAVOR` with the matching value before compilation. This means:

- On platforms that support `--flavor` (Android, iOS), pass only `--flavor <name>`.
  Do **not** also pass `--dart-define=FLUTTER_APP_FLAVOR=<name>`. The build will fail with:

  ```
  Target kernel_snapshot_program failed: Error: FLUTTER_APP_FLAVOR is used by the
  framework and cannot be set using --dart-define or --dart-define-from-file
  ```

  The same restriction applies to `--dart-define-from-file`. The name is reserved by the
  framework regardless of how the value is supplied.

- On Windows desktop, `--flavor` is **not** supported (tracked at flutter/flutter#98994).
  This leaves Windows with no built-in flavor mechanism. Use a different, non-reserved
  `--dart-define` name such as `APP_FLAVOR` and read it explicitly in your `AppFlavorConfig`.
  Do not use `FLUTTER_APP_FLAVOR` as the dart-define name on Windows — the same framework
  reservation applies even when `--flavor` is absent.

- Linux and macOS desktop currently behave like Windows for the purposes of this guide:
  prefer `APP_FLAVOR` for any desktop builds where `--flavor` is unavailable.

### Reading The Flavor At Runtime

There are two equivalent ways to read the flavor inside Dart code:

```dart
// Option 1 — the modern global constant (Flutter ≥ 3.19, recommended).
import 'package:flutter/services.dart';

final String? flavor = appFlavor; // matches the --flavor value, or null if none.
```

```dart
// Option 2 — String.fromEnvironment, which continues to work because the framework
// injects FLUTTER_APP_FLAVOR as a compile-time define when --flavor is present.
const flavor = String.fromEnvironment('FLUTTER_APP_FLAVOR', defaultValue: 'prod');
```

Option 1 is the canonical accessor in current Flutter. Option 2 is still valid and is
useful when the same `AppFlavorConfig` must also support desktop builds that pass a
custom `APP_FLAVOR` dart-define — see the desktop fallback pattern in
`docs/flutter_project_engineering_standard.md §5.2`.

---

## Android Flavor Setup

> **Build script DSL.** Flutter 3.41 makes Kotlin DSL (`build.gradle.kts`) the default for
> new Android projects. Inherited Groovy DSL projects (`build.gradle`) still work but use
> `flavorDimensions "environment"` (no `+=` operator) and string literals where the
> Kotlin examples below use named-argument syntax. The flavor mechanics are identical;
> only the surface syntax differs.

### Product Flavors In Gradle

Define product flavors in `android/app/build.gradle.kts`:

```kotlin
android {
    flavorDimensions += "environment"
    productFlavors {
        create("dev") {
            dimension = "environment"
            applicationIdSuffix = ".dev"
            versionNameSuffix = "-dev"
            resValue("string", "app_name", "MyApp Dev")
        }
        create("prod") {
            dimension = "environment"
            resValue("string", "app_name", "MyApp")
        }
    }
}
```

---

### Android Signing Configuration

#### Signing Strategy Options

> **Keystore location, `key.properties` naming, and the `.gitignore` rules follow
> `guideline.md §2` (the source of truth): the keystore lives in `android/` and
> `storeFile` is a relative path. This section only adds the flavor-specific Gradle wiring.**

Android signing is context-dependent. There is no single correct strategy for all teams.
Choose the approach that fits your CI environment and team policy, then document it in
`docs/architecture.md §15`.

**Strategy A — Local file-based signing (single developer or small team)**

A `key.properties` file supplies keystore credentials on the developer's machine. CI writes
the same file from environment variables. This is the simplest approach for small projects.

- `* --debug` — Android provides the SDK debug keystore automatically. No setup required.
- `dev --release` — Configured by the team. Options: use the release keystore (same as prod),
  use a separate dev keystore, or allow the debug keystore as a fallback for internal QA builds.
  Document which policy your team uses; do not leave it implicit.
- `prod --release` — Release keystore required. The build MUST fail clearly if credentials are
  absent rather than silently signing with the wrong key.

**Strategy B — CI-managed signing (recommended for multi-developer teams)**

Signing credentials are never stored on developer machines. CI injects the keystore and
credentials as secrets at build time. Local builds of `prod --release` are intentionally
blocked or produce an unsigned artifact. This is the lower-risk approach for team projects.

**Strategy C — Separate keystores per flavor**

`dev` and `prod` flavors use completely separate keystores with different aliases. This
guarantees a dev-signed APK can never be mistaken for or substitute a prod APK.

Whichever strategy is chosen: **document it explicitly** and commit that documentation. The
most common signing incidents happen when the strategy is assumed rather than written down.

---

#### Step 1 — Create The Keystore

If you do not yet have a release keystore, generate one with `keytool`. Run this once and
place the output `.jks` file directly in the app's `android/` directory, as required by
`guideline.md §2.1`. The filename is your choice per app (here `myapp-prod.jks`).

```bash
keytool -genkey -v \
  -keystore android/myapp-prod.jks \
  -alias myapp \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000
```

Keystore rules:
- Keep the `.jks` file in `android/` and never commit it to source control — it is
  protected by `.gitignore` (see Step 3).
- Back it up in at least two separate secure locations (cloud storage + physical).
  Losing it permanently prevents you from publishing updates to Google Play for this app.
- Store the passwords in a password manager. They cannot be recovered from the keystore file.

#### Step 2 — Create `android/key.properties` (Strategy A)

Create the file at `android/key.properties`. This file is gitignored (see Step 3).

```properties
storeFile=myapp-prod.jks
storePassword=your-store-password
keyAlias=myapp
keyPassword=your-key-password
```

`storeFile` is the keystore filename you chose in Step 1, as a path relative to the
directory Gradle resolves it from (see the `build.gradle.kts` example in Step 4, which
loads it via `rootProject.file(...)`). Keep it consistent with `guideline.md §2.2`.

For CI environments, set these values as environment variables and write the file from a
pre-build step rather than committing it.

#### Step 3 — Gitignore Signing Artefacts

Add to `.gitignore` at the project root:

```gitignore
# Android signing — never commit
android/key.properties
android/*.jks
android/*.keystore
```

Verify the file is not tracked:

```bash
git status android/key.properties
# Expected: nothing (the file should not appear)
```

If the file was previously committed, remove it from history before it reaches a remote
repository. A committed keystore or key.properties is a security incident.

#### Step 4 — Configure `android/app/build.gradle.kts`

The example below implements Strategy A (local file-based signing) with a Gradle guard that
blocks `prod --release` builds if credentials are absent. Read the caveats below the example
before adopting this pattern.

```kotlin
// ─── Signing ─────────────────────────────────────────────────────────────────
val keystorePropertiesFile = rootProject.file("android/key.properties")

android {
    // ... namespace, compileSdk, defaultConfig, etc. ...

    signingConfigs {
        create("release") {
            if (keystorePropertiesFile.exists()) {
                val props = java.util.Properties()
                props.load(keystorePropertiesFile.inputStream())
                keyAlias      = props.getProperty("keyAlias")
                keyPassword   = props.getProperty("keyPassword")
                storeFile     = file(props.getProperty("storeFile"))
                storePassword = props.getProperty("storePassword")
            }
        }
    }

    buildTypes {
        release {
            isMinifyEnabled = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
            if (keystorePropertiesFile.exists()) {
                signingConfig = signingConfigs.getByName("release")
            }
        }
        debug {
            // Android applies the SDK debug keystore automatically.
        }
    }

    flavorDimensions += "environment"
    productFlavors {
        create("dev") {
            dimension = "environment"
            applicationIdSuffix = ".dev"
            versionNameSuffix = "-dev"
            resValue("string", "app_name", "MyApp Dev")
        }
        create("prod") {
            dimension = "environment"
            resValue("string", "app_name", "MyApp")
        }
    }
}

// ─── Signing enforcement ──────────────────────────────────────────────────────
// Block prod --release tasks at execution time when key.properties is absent.
// See caveats below before adopting this pattern.
afterEvaluate {
    listOf("assembleProdRelease", "bundleProdRelease").forEach { taskName ->
        tasks.findByName(taskName)?.doFirst {
            if (!keystorePropertiesFile.exists()) {
                throw GradleException(
                    "\n" +
                    "══════════════════════════════════════════════════════════\n" +
                    "  SIGNING REQUIRED — prod --release build blocked         \n" +
                    "══════════════════════════════════════════════════════════\n" +
                    "  android/key.properties not found.                       \n" +
                    "  Create the file with your release keystore credentials. \n" +
                    "  See docs/flutter_build_flavors_guide.md                 \n" +
                    "  Section: Android Signing Configuration                  \n" +
                    "══════════════════════════════════════════════════════════\n"
                )
            }
        }
    }
}
```

**Caveats for this Gradle enforcement pattern:**

- The task names `assembleProdRelease` and `bundleProdRelease` are derived from the
  `prod` flavor name and the `release` build type. If your project uses different flavor
  names, a custom build type name, or more than one flavor dimension, the task names will
  differ and this guard will silently do nothing. Verify the actual task names with
  `./gradlew tasks --all | grep -i release` before relying on this check.
- This pattern assumes signing credentials come from a local file. Teams using CI-managed
  signing (Strategy B) should replace this guard with a CI pipeline check rather than a
  Gradle-level file existence test.
- `afterEvaluate` with `tasks.findByName` is sensitive to the Gradle configuration phase.
  In some project configurations, tasks are registered lazily and `findByName` may return
  null even for tasks that will exist at execution time. Use `tasks.matching { ... }` or
  a configuration-time check if you encounter this issue.

---

### Flavor-Specific Resource Files

Place flavor-specific files in the corresponding source set:

```text
android/app/src/
|-- dev/
|   `-- res/
|       |-- mipmap-hdpi/       # Dev app icon (with badge)
|       `-- values/
|           `-- strings.xml
`-- prod/
    `-- res/
        |-- mipmap-hdpi/       # Production app icon
        `-- values/
            `-- strings.xml
```

---

### Android Run And Build Commands

`--flavor` is sufficient on Android. The Flutter tool injects `FLUTTER_APP_FLAVOR`
automatically — see "How The Flavor Reaches Dart Code" above.

Run the development flavor (no signing setup needed):

```bash
flutter run --flavor dev
```

Run the production flavor for debug inspection (no signing setup needed):

```bash
flutter run --flavor prod
```

Build a development debug APK (no signing setup needed):

```bash
flutter build apk --flavor dev --debug
```

Build production split APKs for direct distribution:

```bash
flutter build apk --flavor prod --release \
  --obfuscate \
  --split-debug-info=build/symbols/android-prod-<version>/ \
  --split-per-abi
```

Build a Play Store bundle:

```bash
flutter build appbundle --flavor prod --release \
  --obfuscate \
  --split-debug-info=build/symbols/android-prod-<version>/
```

---

### ProGuard / R8 Rules

R8 shrinks and optimizes the Java/Kotlin bytecode in Android release builds. It can strip
classes that are loaded dynamically or accessed via JVM reflection, causing
`ClassNotFoundException` or `NoSuchMethodException` at runtime — errors that only appear
in release builds.

> **R8 full mode is the default** since AGP 8.0 (and therefore for any Flutter ≥ 3.38
> project on AGP 8.x). Full mode is more aggressive about removing seemingly-unused
> classes than the legacy compatibility mode, which makes correct keep rules more
> important — not less. If a release build crashes with `ClassNotFoundException` for a
> class that exists in source, suspect missing keep rules first.

**Which Flutter packages actually require R8 keep rules:**

Packages that contain Java or Kotlin plugin code accessed via method channels are the primary
risk. The Dart layer of a Flutter app compiles to native AOT machine code and is not subject
to R8. Only the native Android plugin side is affected.

Common cases:
- **Native Android plugins** — any package that registers a `FlutterPlugin` implementation
  in Java or Kotlin. The Flutter engine classes themselves need keeping.
- **sqflite** — uses a Java plugin (`com.tekartik.sqflite`) that can be affected.
- **Packages using JVM reflection internally** — check the package's own README or
  ProGuard documentation for any required keep rules it publishes.

`freezed` and `json_serializable` generate Dart source files at build time. Their output
is compiled Dart, not JVM bytecode, and does not require R8 keep rules.

Create or edit `android/app/proguard-rules.pro`:

```proguard
# Flutter engine — always required
-keep class io.flutter.** { *; }
-keep class io.flutter.plugins.** { *; }

# sqflite native plugin
-keep class com.tekartik.sqflite.** { *; }

# Add keep rules for any other native Android plugin packages
# that document reflection-based class loading in their README.
# Check each package's documentation rather than adding blanket rules.
```

Reference the file in the `buildTypes.release` block in `android/app/build.gradle.kts`:

```kotlin
buildTypes {
    release {
        isMinifyEnabled = true
        proguardFiles(
            getDefaultProguardFile("proguard-android-optimize.txt"),
            "proguard-rules.pro"
        )
    }
}
```

Always test the production release build after adding a new dependency. R8 issues only surface
in release mode and can be hard to trace if not caught immediately after the dependency is added.

---

### Which Android Artifact To Use

Use split APKs when you distribute the app yourself.

Output files from `--split-per-abi`:

- `app-armeabi-v7a-prod-release.apk`
- `app-arm64-v8a-prod-release.apk`
- `app-x86_64-prod-release.apk`

Use an App Bundle (`.aab`) when publishing to Google Play. Google Play serves optimized
device-specific downloads from the `.aab`.

### Important Flag Distinction

`--target-platform` controls compilation targets but does NOT automatically guarantee ABI-specific
APKs. Use `--split-per-abi` for separate per-ABI APKs.

> **ABI deprecation note.** Google Play deprecated 32-bit-only `armeabi-v7a` submissions
> for new apps in 2024 and Flutter dropped `x86` (32-bit) support in 3.27. The minimum
> modern mobile ABI is `arm64-v8a` plus `x86_64` for emulators. Existing apps may keep
> `armeabi-v7a` for legacy device coverage; new apps SHOULD ship 64-bit only unless a
> specific market requires otherwise.

---

### 16 KB Page Size Compliance (Mandatory for Play Store)

Google Play requires apps targeting Android 15+ to support 16 KB memory pages:
- **November 1, 2025** — apps targeting Android 15+ must be 16 KB-aligned for new submissions
  and updates.
- **May 31, 2026** — broader cutoff; non-compliant apps cannot be updated on Play Store.

The Flutter build tooling (≥ 3.24) is already 16 KB-aligned. The risk lies in **precompiled
native libraries (`.so` files) bundled inside dependencies**. Common offenders include older
releases of `ffmpeg_kit_flutter`, image processing libraries, and any package that ships its
own native code.

Verification workflow before each Play Store release:

```bash
# 1. Build the appbundle and inspect bundled native libraries.
flutter build appbundle --flavor prod --release \
  --obfuscate --split-debug-info=build/symbols/android-prod-<version>/

# 2. List .so files and check their alignment.
unzip -l build/app/outputs/bundle/prodRelease/app-prod-release.aab | grep '\.so$'

# 3. For any third-party .so found, verify the package's release notes confirm
#    16 KB-page alignment. If not, file a release-blocking issue and either upgrade
#    to a compliant version or remove the dependency.
```

Test the release build on an emulator configured with 16 KB pages
(Android Studio → Device Manager → Advanced settings → "Page size: 16 KB") before tagging.
Document any non-compliant dependency as a release-blocking risk in
`docs/architecture.md §21`.

---

## iOS Flavor Setup

### iOS Deployment Target And UIScene Migration (Mandatory, Flutter ≥ 3.38)

Two pre-flight requirements precede any iOS flavor work:

1. **Deployment target: iOS 13.** Flutter 3.41 raised the minimum from iOS 12 to iOS 13.
   Set `platform :ios, '13.0'` in `ios/Podfile` and update the iOS Deployment Target in
   Xcode (Build Settings → Deployment → iOS Deployment Target).

2. **UIScene lifecycle adoption.** Apple requires UIScene for any UIKit app built with the
   iOS 26 SDK; the iOS release after iOS 26 will refuse to launch non-UIScene apps. The App
   Store requires iOS 26 SDK builds, and that deadline (**April 2026**) is now in force.
   Flutter 3.41 enables UIScene by default and its CLI auto-migrates apps with an unmodified
   `AppDelegate`.
   - **Auto-migration success log:** `Finished migration to UIScene lifecycle` after a build.
   - **Manual migration required when:** `AppDelegate` has custom code (analytics SDK init,
     deep-link handlers, custom plugin registration).

Flavor-specific custom `AppDelegate` code that previously ran in
`application:didFinishLaunchingWithOptions:` (for example, a flavor-conditional Firebase or
analytics key) should now move to `didInitializeImplicitFlutterEngine`. Plugin registration
in particular MUST move there:

```swift
import UIKit
import Flutter

@main
@objc class AppDelegate: FlutterAppDelegate, FlutterImplicitEngineDelegate {

  // OLD (pre-UIScene): plugin registration + flavor-specific init lived here.
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }

  // NEW: runs after the implicit Flutter engine is created. Use this for
  // GeneratedPluginRegistrant.register, FirebaseApp.configure, etc.
  func didInitializeImplicitFlutterEngine(_ engine: FlutterEngine) {
    GeneratedPluginRegistrant.register(with: engine)
    // Flavor-conditional initialization (analytics keys, etc.) goes here.
  }
}
```

Plugins themselves migrate by adopting `FlutterSceneLifeCycleDelegate` and registering with
`registrar.addSceneDelegate(self)` in addition to (or instead of) the legacy
`addApplicationDelegate(self)`. See
`docs.flutter.dev/release/breaking-changes/uiscenedelegate` for the canonical migration steps.

`Info.plist` must include a `UIApplicationSceneManifest` (Application Scene Manifest) entry.
The Flutter migrator usually adds it automatically; verify it exists before building for
release.

---

### Xcode Scheme And xcconfig Setup

Each flavor requires a separate Xcode scheme and a pair of xcconfig files (Debug and Release).

**Directory structure:**

```text
ios/
|-- Flutter/
|   |-- dev/
|   |   |-- Debug.xcconfig
|   |   `-- Release.xcconfig
|   `-- prod/
|       |-- Debug.xcconfig
|       `-- Release.xcconfig
|-- Runner/
|   |-- Info.plist
|   `-- ... (Xcode project files)
```

**`ios/Flutter/dev/Debug.xcconfig`:**

```
#include "Generated.xcconfig"
#include "../../Flutter/Flutter.xcconfig"
FLUTTER_TARGET=lib/main.dart
BUNDLE_ID_SUFFIX=.dev
DISPLAY_NAME=MyApp Dev
```

**`ios/Flutter/dev/Release.xcconfig`:**

```
#include "Generated.xcconfig"
#include "../../Flutter/Flutter.xcconfig"
FLUTTER_TARGET=lib/main.dart
BUNDLE_ID_SUFFIX=.dev
DISPLAY_NAME=MyApp Dev
```

**`ios/Flutter/prod/Release.xcconfig`:**

```
#include "Generated.xcconfig"
#include "../../Flutter/Flutter.xcconfig"
FLUTTER_TARGET=lib/main.dart
BUNDLE_ID_SUFFIX=
DISPLAY_NAME=MyApp
```

The xcconfig files MUST NOT set `DART_DEFINES=FLUTTER_APP_FLAVOR%3D...`. The Flutter tool
injects `FLUTTER_APP_FLAVOR` automatically from the `--flavor` argument that the Xcode
scheme passes to `flutter run`/`flutter build`. Setting it again from xcconfig produces the
same `kernel_snapshot_program` build failure described above.

**In `ios/Runner/Info.plist`**, use the variable references:

```xml
<key>CFBundleIdentifier</key>
<string>com.yourcompany.myapp$(BUNDLE_ID_SUFFIX)</string>
<key>CFBundleDisplayName</key>
<string>$(DISPLAY_NAME)</string>
```

### Creating Xcode Schemes

In Xcode:

1. Product → Scheme → New Scheme → name it `dev`.
2. Product → Scheme → New Scheme → name it `prod`.
3. For the `dev` scheme:
   - Edit Scheme → Build: set configuration to `Debug`.
   - Edit Scheme → Run: set configuration to `Debug`.
   - Edit Scheme → Archive: set configuration to `Release`.
4. In each scheme's Build Configuration, select the matching xcconfig via
   Project → Info → Configurations → Expand each configuration → set xcconfig for Runner.

### iOS Run And Build Commands

Run the dev flavor (automatic development signing):

```bash
flutter run --flavor dev
```

Run the prod flavor (automatic development signing for device testing):

```bash
flutter run --flavor prod
```

Build a release IPA (**requires App Store distribution provisioning profile**):

```bash
flutter build ipa --flavor prod --release \
  --obfuscate \
  --split-debug-info=build/symbols/ios-prod-<version>/
```

### iOS Provisioning

- Each flavor MUST use a separate provisioning profile matching its bundle ID.
  - Dev: `com.yourcompany.myapp.dev` — development or ad-hoc profile.
  - Prod: `com.yourcompany.myapp` — App Store distribution profile.
- Never share a production distribution certificate with dev builds.
- Configure signing in Xcode under Signing & Capabilities, per scheme.
- `flutter run` uses automatic signing by default; `flutter build ipa` requires an explicit
  distribution profile configured in Xcode for the prod scheme.

### iOS Flavor-Specific Assets

Place flavor-specific app icons in `ios/Runner/Assets.xcassets` using an `AppIcon-dev` asset
catalog set for the dev flavor and `AppIcon` for prod. Reference the correct set in each scheme's
`Info.plist` via `CFBundleIcons`.

---

## Windows Desktop Flavor Setup

Windows does not have a native flavor system equivalent to Android product flavors or Xcode
schemes, and the Flutter tool does not currently accept `--flavor` for the Windows desktop
target (see flutter/flutter#98994). For most behavioral differences — feature flags,
environment URLs, logging verbosity — `--dart-define` combined with `AppFlavorConfig` is
sufficient at the Dart layer.

**Important:** the dart-define name on Windows MUST NOT be `FLUTTER_APP_FLAVOR`. That name
is reserved by the Flutter framework and any attempt to set it via `--dart-define` or
`--dart-define-from-file` fails the build at `kernel_snapshot_program`. Use a non-reserved
name such as `APP_FLAVOR` and have your `AppFlavorConfig` read both names — `APP_FLAVOR`
first (used on desktop) with a fallback to `FLUTTER_APP_FLAVOR` (auto-injected on
Android/iOS when `--flavor` is passed). The reference implementation is in
`docs/flutter_project_engineering_standard.md §5.2`.

`--dart-define` does not handle all flavor-differentiation needs:

- **MSIX package identity** — if dev and prod builds need to be installed side by side on the
  same machine, they require distinct `identity_name` values in `msix_config`. This requires
  either separate `pubspec.yaml` sections per flavor, or a build script that substitutes the
  correct value before calling `msix:create`.
- **Distribution certificates** — sideloaded MSIX packages require a code-signing certificate
  trusted by the target machine. Store-distributed packages go through Microsoft Partner Center
  signing. These are fundamentally different processes; a single `msix_config` block cannot
  serve both without adjustment.
- **App display name and icon** — these are set in `msix_config` statically. If dev and prod
  builds need distinct display names or icons in the installed app list, the config must differ
  per flavor.

Document which of these cases apply to your project before settling on a Windows build strategy.

### Windows Run And Build Commands

Run the dev flavor:

```bash
flutter run -d windows --dart-define=APP_FLAVOR=dev
```

Run the prod flavor:

```bash
flutter run -d windows --dart-define=APP_FLAVOR=prod
```

Build a production Windows release:

```bash
flutter build windows --release \
  --dart-define=APP_FLAVOR=prod \
  --obfuscate \
  --split-debug-info=build/symbols/windows-prod-<version>/
```

### MSIX Packaging

For distributing Windows builds outside direct EXE copy, package as MSIX.

Add to `pubspec.yaml`:

```yaml
dev_dependencies:
  msix: ^3.16.0

msix_config:
  display_name: MyApp
  publisher_display_name: Your Name Or Company
  identity_name: com.yourcompany.myapp
  publisher: CN=YourPublisherCN
  msix_version: 1.0.0.0
  logo_path: assets/icons/icon.png
  capabilities: runFullTrust
  languages: en-us
  # For Microsoft Store submissions, build a multi-architecture .msixbundle.
  # For sideloading, a single-architecture .msix is sufficient — drop arm64.
  architecture: x64, arm64
```

Build the MSIX:

```bash
# `flutter pub run` is deprecated; the toolchain now requires `dart run`.
dart run msix:create
```

For Microsoft Store submission, generate a multi-architecture `.msixbundle` using the
`architecture` config option (typically `x64` plus `arm64`); for direct sideloading, a
single-architecture `.msix` is sufficient. The exact `msix_config` shape depends on the
target distribution channel — record the choice in `docs/architecture.md §15`.

If dev and prod must be installed side by side, use a distinct `identity_name` for the dev
flavor (e.g. `com.yourcompany.myapp.dev`). The simplest approach is a separate
`pubspec_dev.yaml` that overrides only the `msix_config` block, invoked explicitly in your
dev build script. Document the chosen approach in `docs/architecture.md §15`.

### Windows-Specific sqflite Initialization

This is required before any database operation on Windows or Linux desktop. Add it to `main()`
before `runApp`:

```dart
import 'dart:io';
import 'package:sqflite_common_ffi/sqflite_ffi.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  if (Platform.isWindows || Platform.isLinux) {
    sqfliteFfiInit();
    databaseFactory = databaseFactoryFfi;
  }

  runApp(const MyApp());
}
```

### Window Size Constraints

Prevent the window from being resized to dimensions that break the UI:

```yaml
# pubspec.yaml
dependencies:
  window_manager: ^0.4.0
```

```dart
import 'package:window_manager/window_manager.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await windowManager.ensureInitialized();

  const windowOptions = WindowOptions(
    size: Size(1024, 768),
    minimumSize: Size(800, 600),
    title: 'MyApp',
    center: true,
  );
  await windowManager.waitUntilReadyToShow(windowOptions, () async {
    await windowManager.show();
    await windowManager.focus();
  });

  runApp(const MyApp());
}
```

---

## Recommended Release Matrix

For most Flutter projects targeting Android, iOS, and Windows:

> **Implicit flag — `--tree-shake-icons`.** Release builds tree-shake unused Material
> icons automatically when icons are referenced via `const Icon(Icons.x)` constructors
> (see engineering standard §17.3). The flag is not shown explicitly in the commands
> below because it is on by default in release mode; the rule is "always use `const Icon`
> with a literal icon reference" rather than "always pass the flag."

| Platform | Flavor | Mode | Signing Required | Command |
|----------|--------|------|-----------------|---------|
| Android | `dev` | `debug` | No — automatic debug keystore | `flutter run --flavor dev` |
| Android | `dev` | `release` | Team policy — document your choice | `flutter build apk --flavor dev --release` |
| Android | `prod` | `debug` | No — automatic debug keystore | `flutter run --flavor prod` |
| Android | `prod` | `release` split APK | Yes — release keystore required | `flutter build apk --flavor prod --release --obfuscate --split-debug-info=... --split-per-abi` |
| Android | `prod` | `release` Play Store | Yes — release keystore required | `flutter build appbundle --flavor prod --release --obfuscate --split-debug-info=...` |
| iOS | `dev` | `debug` | No — automatic development signing | `flutter run --flavor dev` |
| iOS | `prod` | `release` | Yes — distribution profile required | `flutter build ipa --flavor prod --release --obfuscate --split-debug-info=...` |
| Windows | n/a | `debug` | No | `flutter run -d windows --dart-define=APP_FLAVOR=dev` |
| Windows | n/a | `release` MSIX | Depends on distribution channel — see Windows section | `flutter build windows --release --dart-define=APP_FLAVOR=prod --obfuscate --split-debug-info=...` then `dart run msix:create` |

---

## Debug Symbol Management

Every production release build MUST be built with:

```bash
--obfuscate
--split-debug-info=build/symbols/<platform>-<version>/
```

**What `--obfuscate` does:** Dart compiles to native AOT machine code; it is not bytecode and
does not require decompilation in the way Java or C# do. The `--obfuscate` flag additionally
renames Dart class and method identifiers in the compiled binary's symbol table, making
class and method names meaningless strings rather than readable source names. This is a useful
hardening step that raises the cost of static analysis and makes crash symbolication without the
accompanying symbols file impossible.

It is not a strong security boundary on its own. A determined analyst with the binary and
sufficient time can still reconstruct logic from the machine code. Do not treat `--obfuscate`
as a substitute for sound data security, proper secret management, or server-side enforcement
of sensitive operations.

**Symbol archive policy:**

The symbols directory produced by `--split-debug-info` MUST be:

- Stored securely for the lifetime of the released version.
- Never committed to source control.
- Archived alongside the release artifact (e.g. in a release artifacts storage bucket or
  secure folder).

Without the symbols file, crash reports from that release version cannot be decoded. Losing
it permanently means those crashes are undiagnosable.

---

## Notes For New Projects

To support this workflow, the native projects typically need:

**Android:**
- Product flavors in `android/app/build.gradle.kts`.
- Distinct `applicationIdSuffix` for side-by-side installation.
- Flavor-specific icons and resource values.
- ProGuard rules for the Flutter engine and any native plugins that require them.
- A documented and implemented signing strategy for each flavor × mode combination.
- `android/key.properties`, `android/*.jks`, and `android/*.keystore` added to `.gitignore`.
- **Java 17** as the minimum JDK (Flutter ≥ 3.38). AGP must remain on 8.x; AGP 9 migration
  is paused.
- **16 KB page-size compliance** for any release targeting Android 15+. Audit bundled
  `.so` files in dependencies before each Play Store submission (see "16 KB Page Size
  Compliance" above).

**iOS:**
- Xcode schemes aligned with flavor names.
- xcconfig files per flavor per build configuration.
- Provisioning profiles per flavor bundle ID.
- Flavor-specific app icons in asset catalogs.
- Distribution profile required for `prod --release`; automatic signing for debug and dev.
- **iOS 13** as the minimum deployment target (Flutter ≥ 3.41); update `ios/Podfile` and
  Xcode Build Settings.
- **UIScene Application Scene Manifest** in `Info.plist`. `FlutterImplicitEngineDelegate`
  adoption in `AppDelegate` if any custom code (analytics, deep-link handlers, plugin
  registration) was added beyond the default. Move flavor-conditional setup to
  `didInitializeImplicitFlutterEngine`.
- Plan an iOS 26 SDK (Xcode 26) build before April 2026 to remain App-Store-submittable.
  The April 2026 deadline is now in force; new submissions without iOS 26 SDK are rejected.

**Windows:**
- `sqflite_common_ffi` initialization in `main()` for any desktop + sqflite usage.
- `window_manager` for size constraints.
- `msix` package for distribution packaging; use `dart run msix:create` (the deprecated
  `flutter pub run msix:create` will eventually stop working).
- A documented strategy for MSIX identity and display name differentiation between flavors
  if side-by-side installation is required.
- For Microsoft Store submission, build a multi-architecture `.msixbundle` (x64 + arm64);
  for sideloading, single-architecture `.msix` is sufficient.