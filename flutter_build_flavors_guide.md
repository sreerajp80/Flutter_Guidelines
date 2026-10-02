# Flutter Build Flavors Guide

Use this as a reusable reference for Flutter projects that define build flavors such as `dev`
and `prod` across Android, iOS, and Windows, macOS and Linux desktop.

This guide documents one well-tested approach for each platform. It is not the only valid
approach. Teams with different CI pipelines, signing strategies, or flavor matrices should treat
the examples here as a starting point and document their deviations in `docs/architecture.md`.

> **Last checked against Flutter 3.47 / Dart 3.13 (2026-10).** This guide does not pin tool
> versions. Use the latest stable Flutter. Where a rule only applies from a certain version
> on (UIScene, 16 KB pages, AGP 9, iOS 15, etc.), the version is named as a condition so you
> can check it against your own toolchain.

---

## Toolchain Prerequisites

The toolchain rules (latest stable Flutter, the AGP / KGP / Gradle versions it generates, each
project pins its own versions, never type a version from memory) are in the engineering standard
§5.3. This guide does not repeat them.

These minimums are hard limits — a build below them fails:

| Concern | Minimum | Notes |
|---------|---------|-------|
| JDK (Android builds) | **Java 17** | Java 11 builds fail; Gradle 9 needs JDK 17 |
| iOS deployment target | **iOS 15** (since Flutter 3.47) | Set in `ios/Podfile` and Xcode Build Settings |
| macOS deployment target | **macOS 12** (since Flutter 3.47) | Only for apps that ship on macOS |
| Linux build packages | clang, cmake, ninja-build, pkg-config, libgtk-3-dev, liblzma-dev, libstdc++-12-dev | Only for apps that ship on Linux; build on the oldest supported distro |
| Xcode (iOS prod builds) | Xcode 26 (iOS 26 SDK) or later | App Store requires the iOS 26 SDK (since April 2026). With Xcode 27, apps without UIScene fail to launch |
| Android target SDK (Play Store) | API 35 (Android 15) with **16 KB page support** | Mandatory from Nov 1 2025 / May 31 2026 |

**Reference snapshot — not a rule.** When this guide was last checked (2026-10), the latest
stable Flutter generated the versions below. Use them only to spot a project that is clearly
behind; always use what your own Flutter generates.

| Tool | Last checked (Flutter 3.47.6, 2026-10) |
|------|----------------------------------------|
| Flutter / Dart | 3.47 / 3.13 |
| AGP (Android Gradle Plugin) | 9.1.0 |
| Kotlin Gradle Plugin (KGP) | 2.4.0 |
| Gradle (wrapper) | 9.3.1 |

If the project is below any minimum in the first table, fix that before adopting any flavor
configuration below — the
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

- Linux and macOS desktop use the same `APP_FLAVOR` pattern as Windows. (macOS also accepts
  `--flavor` when matching Xcode schemes exist, but `APP_FLAVOR` stays the rule.) See "macOS
  Desktop Flavor Setup" and "Linux Desktop Flavor Setup" below for side-by-side installs.

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
`docs/guidelines/flutter_project_engineering_standard.md §5.2`.

---

## Android Flavor Setup

> **Build script DSL.** Kotlin DSL (`build.gradle.kts`) is the default for new Android
> projects (since Flutter 3.41). Inherited Groovy DSL projects (`build.gradle`) still work but
> use `flavorDimensions "environment"` (no `+=` operator) and string literals where the
> Kotlin examples below use named-argument syntax. The flavor mechanics are identical;
> only the surface syntax differs.

### Android Toolchain Baseline (AGP 9+ / Gradle 9+)

Every flavored project MUST use the AGP, KGP and Gradle versions that its Flutter version
generates (see "Toolchain Prerequisites"). Since Flutter 3.47 that means AGP 9 and Gradle 9,
so the rules below apply. The versions are set in these two files:

```kotlin
// android/settings.gradle.kts — EXAMPLE versions (last checked 2026-10).
// Use the versions your Flutter generates, not these.
plugins {
    id("dev.flutter.flutter-plugin-loader") version "1.0.0"
    id("com.android.application") version "9.1.0" apply false
    id("org.jetbrains.kotlin.android") version "2.4.0" apply false
}
```

```properties
# android/gradle/wrapper/gradle-wrapper.properties — EXAMPLE version (last checked 2026-10)
distributionUrl=https\://services.gradle.org/distributions/gradle-9.3.1-all.zip
```

On AGP 9 and later, AGP **rejects the `kotlinOptions { }` block**. Set the JVM target with the top-level
`kotlin { compilerOptions { } }` block in `android/app/build.gradle.kts` instead:

```kotlin
import org.jetbrains.kotlin.gradle.dsl.JvmTarget

android {
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }
    // NO kotlinOptions { jvmTarget = "17" } here — AGP 9 fails the build.
}

kotlin {
    compilerOptions {
        jvmTarget.set(JvmTarget.JVM_17)
    }
}
```

Notes:

- The Flutter migrator adds `android.builtInKotlin=false` and `android.newDsl=false` to
  `android/gradle.properties`. Keep them until the Flutter app template drops them. With
  `builtInKotlin=false` the Flutter Gradle plugin applies Kotlin, so the app module's
  `plugins { }` block does not need `org.jetbrains.kotlin.android`.
- Gradle 9 and later have no `project.exec { }`. Custom tasks (for example a task that runs a Dart
  script to generate build metadata) MUST use an injected `ExecOperations` instead.
- After the upgrade, run a clean `flutter build apk --flavor dev` and
  `flutter build appbundle --flavor prod --release` before any other change, so toolchain
  failures are not mixed with feature changes.

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

`storeFile` is the keystore filename you chose in Step 1, relative to `android/`. Step 4 resolves
it with `rootProject.file(...)`, and the root project of the Android build is the `android/`
folder (`guideline.md` §2.2).

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
// rootProject is the android/ folder, so both paths below are relative to android/.
val keystorePropertiesFile = rootProject.file("key.properties")

android {
    // ... namespace, compileSdk, defaultConfig, etc. ...

    signingConfigs {
        create("release") {
            if (keystorePropertiesFile.exists()) {
                val props = java.util.Properties()
                props.load(keystorePropertiesFile.inputStream())
                keyAlias      = props.getProperty("keyAlias")
                keyPassword   = props.getProperty("keyPassword")
                storeFile     = rootProject.file(props.getProperty("storeFile"))
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

    bundle {
        language {
            // REQUIRED when the app has an in-app language picker (engineering standard
            // §8.1). Play splits App Bundles by language by default, so a phone set to
            // English would get no resources for the other declared languages, and the
            // in-app language picker could not switch to them.
            enableSplit = false
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
                    "  See docs/guidelines/flutter_build_flavors_guide.md                 \n" +
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

> **R8 full mode is the default** since AGP 8.0 (and therefore for every current Flutter
> project, which is on AGP 9 or later). Full mode is more aggressive about removing seemingly-unused
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

### iOS Deployment Target And UIScene Migration (Mandatory, Since Flutter 3.47)

Two pre-flight requirements precede any iOS flavor work:

1. **Deployment target: iOS 15.** Flutter 3.47 raised the minimum from iOS 13 to iOS 15.
   Set `platform :ios, '15.0'` in `ios/Podfile` and update the iOS Deployment Target in
   Xcode (Build Settings → Deployment → iOS Deployment Target).

2. **UIScene lifecycle adoption.** Apple requires UIScene for any UIKit app built with the
   iOS 26 SDK, and apps built with **Xcode 27** that do not adopt UIScene fail to launch. The
   App Store requires iOS 26 SDK builds, and that deadline (**April 2026**) is now in force.
   Flutter enables UIScene by default (since 3.41) and its CLI auto-migrates apps with an
   unmodified `AppDelegate`.
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

### Xcode Build Configurations And Schemes

Flutter maps `--flavor <name>` to an Xcode **scheme** called `<name>`, which must use **build
configurations** called `Debug-<name>`, `Profile-<name>` and `Release-<name>`. Plain `Debug` /
`Release` configurations are not enough. Official reference:
`docs.flutter.dev/deployment/flavors-ios`.

**1. Create the build configurations.** Open `ios/Runner.xcworkspace`. In Project → Runner →
Info → Configurations, duplicate each existing configuration once per flavor:

| Flavor | Build configurations |
|---|---|
| `dev` | `Debug-dev`, `Profile-dev`, `Release-dev` |
| `prod` | `Debug-prod`, `Profile-prod`, `Release-prod` |

**2. Create one scheme per flavor.** Product → Scheme → New Scheme, named exactly `dev` and
`prod`. Edit each scheme so Run uses `Debug-<flavor>`, Profile uses `Profile-<flavor>`, and
Archive uses `Release-<flavor>`. Tick **Shared** so the schemes are committed.

**3. Set identity per build configuration.** In Runner target → Build Settings:

| Setting | `*-dev` configurations | `*-prod` configurations |
|---|---|---|
| `PRODUCT_BUNDLE_IDENTIFIER` | `com.yourcompany.myapp.dev` | `com.yourcompany.myapp` |
| `FLAVOR_APP_NAME` (user-defined setting) | `MyApp Dev` | `MyApp` |

**4. Read them in `ios/Runner/Info.plist`** — never hard-code the id:

```xml
<key>CFBundleIdentifier</key>
<string>$(PRODUCT_BUNDLE_IDENTIFIER)</string>
<key>CFBundleDisplayName</key>
<string>$(FLAVOR_APP_NAME)</string>
```

**5. Map the configurations in `ios/Podfile`**, then run `pod install`:

```ruby
project 'Runner', {
  'Debug-dev' => :debug, 'Profile-dev' => :release, 'Release-dev' => :release,
  'Debug-prod' => :debug, 'Profile-prod' => :release, 'Release-prod' => :release,
}
```

Do **not** set `DART_DEFINES=FLUTTER_APP_FLAVOR%3D...` in Xcode or in any xcconfig file. The Flutter
tool injects `FLUTTER_APP_FLAVOR` from `--flavor`; setting it again produces the same
`kernel_snapshot_program` build failure described above.

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
- Configure signing in Xcode under Signing & Capabilities, per build configuration.
- `flutter run` uses automatic signing by default; `flutter build ipa` requires an explicit
  distribution profile configured in Xcode for `Release-prod`.

### iOS Flavor-Specific Assets

Place flavor-specific app icons in `ios/Runner/Assets.xcassets` using an `AppIcon-dev` asset
catalog set for the dev flavor and `AppIcon` for prod. Select the set per build configuration with
the `ASSETCATALOG_COMPILER_APPICON_NAME` build setting (`AppIcon-dev` for `*-dev`).

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
`docs/guidelines/flutter_project_engineering_standard.md §5.2`.

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
  msix: ^<current>   # current version from pub.dev

msix_config:
  display_name: MyApp
  publisher_display_name: Your Name Or Company
  identity_name: com.yourcompany.myapp
  publisher: CN=YourPublisherCN
  msix_version: 1.0.0.0
  logo_path: assets/icons/app_icon.png
  capabilities: internetClient   # only what the app uses; the msix package adds runFullTrust itself
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

### Desktop Setup (sqflite, Window Size, Shortcuts)

`sqflite` FFI initialization, minimum window size and other desktop setup are not flavor-specific.
They are described once, in the engineering standard §5.5.

---

## macOS Desktop Flavor Setup

macOS reads the flavor the same way as the other desktops: pass
`--dart-define=APP_FLAVOR=<name>`, never `FLUTTER_APP_FLAVOR`. This is enough when dev and prod
only differ in Dart behavior.

When dev and prod must be **installed side by side** or show **different names**, they need
different bundle ids. Use the iOS pattern (see "Xcode Build Configurations And Schemes" above) inside
`macos/`:

```text
macos/Runner/Configs/
|-- AppInfo.xcconfig        # prod: PRODUCT_NAME, PRODUCT_BUNDLE_IDENTIFIER, PRODUCT_COPYRIGHT
`-- AppInfo-dev.xcconfig    # #include "AppInfo.xcconfig", then override name and bundle id
```

```
// macos/Runner/Configs/AppInfo-dev.xcconfig
#include "AppInfo.xcconfig"
PRODUCT_NAME = MyApp Dev
PRODUCT_BUNDLE_IDENTIFIER = com.example.myapp.dev
```

Create a `dev` scheme / build configuration in `macos/Runner.xcworkspace` that uses it. Recent
Flutter releases accept `--flavor` for macOS when a matching Xcode scheme exists; confirm with
`flutter build macos -h` on your toolchain before relying on it, and keep passing `APP_FLAVOR`
so `AppFlavorConfig` works either way.

Entitlements are per build configuration: if a dev build needs extra entitlements (for example a
local debug server), add them to `DebugProfile.entitlements` only — never to
`Release.entitlements`.

### macOS Run And Build Commands

```bash
flutter run -d macos --dart-define=APP_FLAVOR=dev

flutter build macos --release \
  --dart-define=APP_FLAVOR=prod \
  --obfuscate \
  --split-debug-info=build/symbols/macos-prod-<version>/
```

Signing, notarization and store upload: `docs/guidelines/platform_store_readiness.md` §5.

---

## Linux Desktop Flavor Setup

Linux also uses `--dart-define=APP_FLAVOR=<name>`. Build machine setup and `APPLICATION_ID` are
described in engineering standard §5.5.3.

When dev and prod must be installed side by side, they need different `APPLICATION_ID` values
(and usually different `BINARY_NAME`s). `flutter build linux` runs CMake with the caller's
environment, so one simple approach is an environment-variable suffix in `linux/CMakeLists.txt`:

```cmake
set(BINARY_NAME "my_app")
set(APPLICATION_ID "com.example.my_app")

# Optional flavor suffix, e.g. APP_ID_SUFFIX=.dev for side-by-side dev installs.
if(DEFINED ENV{APP_ID_SUFFIX})
  set(APPLICATION_ID "${APPLICATION_ID}$ENV{APP_ID_SUFFIX}")
  set(BINARY_NAME "${BINARY_NAME}_dev")
endif()
```

Keep separate `.desktop`, icon and packaging files (`snapcraft.yaml`, Flatpak manifest) per
flavor only if a dev flavor is actually distributed; most projects package `prod` only.

### Linux Run And Build Commands

```bash
flutter run -d linux --dart-define=APP_FLAVOR=dev

flutter build linux --release \
  --dart-define=APP_FLAVOR=prod \
  --obfuscate \
  --split-debug-info=build/symbols/linux-prod-<version>/
```

Packaging and store upload: `docs/guidelines/platform_store_readiness.md` §6.

---

## Recommended Release Matrix

Keep the rows for the platforms your project declares:

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
| macOS | n/a | `debug` | No — development signing | `flutter run -d macos --dart-define=APP_FLAVOR=dev` |
| macOS | n/a | `release` | Yes — Apple Distribution (Mac App Store) or Developer ID + notarization | `flutter build macos --release --dart-define=APP_FLAVOR=prod --obfuscate --split-debug-info=...` |
| Linux | n/a | `debug` | No | `flutter run -d linux --dart-define=APP_FLAVOR=dev` |
| Linux | n/a | `release` | Store-side for Snap / Flathub; optional GPG for direct packages | `flutter build linux --release --dart-define=APP_FLAVOR=prod --obfuscate --split-debug-info=...` then package |

---

## Debug Symbol Management

Every production release build MUST be built with:

```bash
--obfuscate
--split-debug-info=build/symbols/<platform>-<flavor>-<version>/
```

`<version>` is the full `pubspec.yaml` version (e.g. `1.4.0+27`); drop `-<flavor>` when the app has
no flavors.

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
- Archived alongside the release artifact, **outside the repository** (e.g. in a release
  artifacts storage bucket or secure folder).

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
- **Java 17** minimum, and the AGP / KGP / Gradle versions the latest stable Flutter
  generates — pinned in the project's own `settings.gradle.kts` and Gradle wrapper. On
  AGP 9+, use `kotlin { compilerOptions { } }`, never `kotlinOptions { }` (see "Android
  Toolchain Baseline" above).
- **16 KB page-size compliance** for any release targeting Android 15+. Audit bundled
  `.so` files in dependencies before each Play Store submission (see "16 KB Page Size
  Compliance" above).

**iOS:**
- Xcode schemes aligned with flavor names.
- xcconfig files per flavor per build configuration.
- Provisioning profiles per flavor bundle ID.
- Flavor-specific app icons in asset catalogs.
- Distribution profile required for `prod --release`; automatic signing for debug and dev.
- **iOS 15** as the minimum deployment target (since Flutter 3.47); update `ios/Podfile` and
  Xcode Build Settings.
- **UIScene Application Scene Manifest** in `Info.plist`. `FlutterImplicitEngineDelegate`
  adoption in `AppDelegate` if any custom code (analytics, deep-link handlers, plugin
  registration) was added beyond the default. Move flavor-conditional setup to
  `didInitializeImplicitFlutterEngine`.
- Build with the iOS 26 SDK (Xcode 26) or later to remain App-Store-submittable; new
  submissions without the iOS 26 SDK are rejected. Before moving to Xcode 27, confirm the
  UIScene migration is complete — non-UIScene apps built with Xcode 27 fail to launch.

**Windows:**
- `sqflite_common_ffi` initialization in `main()` for any desktop + sqflite usage.
- `window_manager` for size constraints (engineering standard §5.5).
- `msix` package for distribution packaging; use `dart run msix:create` (the deprecated
  `flutter pub run msix:create` will eventually stop working).
- A documented strategy for MSIX identity and display name differentiation between flavors
  if side-by-side installation is required.
- For Microsoft Store submission, build a multi-architecture `.msixbundle` (x64 + arm64);
  for sideloading, single-architecture `.msix` is sufficient.

**macOS:**
- Bundle id, name and copyright in `macos/Runner/Configs/AppInfo.xcconfig` (plus a dev
  xcconfig and scheme if dev and prod install side by side).
- **macOS 12** minimum deployment target (since Flutter 3.47) in `macos/Podfile` and Xcode.
- Least-privilege entitlements in both `DebugProfile.entitlements` and
  `Release.entitlements` — add `com.apple.security.network.client` if the app uses the
  network.
- Hardened Runtime on; Developer ID signing + notarization for direct download, or Apple
  Distribution signing for the Mac App Store.

**Linux:**
- GTK build packages on the build machine; build on the oldest supported distro.
- `APPLICATION_ID` (reverse-DNS) and `BINARY_NAME` in `linux/CMakeLists.txt`.
- `sqflite_common_ffi` initialization in `main()` (as on Windows).
- `.desktop` file, icons and AppStream metainfo for packaging; Snap / Flatpak / AppImage /
  `.deb` packaging chosen per declared channel.
