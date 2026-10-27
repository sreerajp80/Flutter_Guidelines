# Flutter Build Flavors Guide

Use this as a reusable reference for Flutter projects that define build flavors such as `dev`
and `prod` across Android, iOS, and Windows, macOS and Linux desktop.

This guide documents one well-tested approach for each platform. It is not the only valid
approach. Teams with different signing strategies or flavor matrices should treat
the examples here as a starting point and document their deviations in `docs/architecture.md`.

> **Version-independent.** This guide does not pin tool versions and does not tie a rule to one
> Flutter release. Where tools differ between versions, it says how to check your own
> toolchain (`flutter --version`, `flutter <command> -h`, the files `flutter create` writes)
> and covers both cases.

---

## Toolchain Prerequisites

The toolchain rules (latest stable Flutter, the AGP / KGP / Gradle versions it generates, each
project pins its own versions, never type a version from memory) are in the engineering standard
§5.3. This guide does not repeat them.

These minimums come from your toolchain and the stores — a build below them fails:

| Concern | Minimum | Where to find the current value |
|---------|---------|---------------------------------|
| JDK (Android builds) | What the project's AGP needs (current AGP: Java 17 or later) | `flutter doctor -v`; the AGP release notes |
| iOS deployment target | At least the Flutter minimum | `ios/Podfile` from a fresh `flutter create`, or the build error |
| macOS deployment target | At least the Flutter minimum (only for apps that ship on macOS) | `macos/Podfile` from a fresh `flutter create` |
| Linux build packages | The packages in Flutter's Linux setup page | `docs.flutter.dev/platform-integration/linux/setup`; build on the oldest supported distro |
| Xcode / Apple SDK (App Store uploads) | What App Store Connect currently accepts | `developer.apple.com/news/upcoming-requirements`. Adopt UIScene before building with an SDK that requires it (engineering standard §5.4) |
| Android target SDK (Play Store) | Play's current target API level, plus **16 KB page support** for API 35+ | Play Console policy pages; engineering standard §5.3.1 |

**Reference snapshot — not a rule.** When this guide was last checked (2026-10-15), the latest
stable Flutter generated the versions below. Use them only to spot a project that is clearly
behind; always use what your own Flutter generates.

| Tool | Last checked (Flutter 3.47.6, 2026-10-15) |
|------|-------------------------------------------|
| Flutter / Dart | 3.47.6 / 3.13.5 |
| AGP (Android Gradle Plugin) | 9.1.0 |
| Kotlin Gradle Plugin (KGP) | 2.4.0 |
| Gradle (wrapper) | 9.3.1 |
| JDK | 17 |
| iOS / macOS minimum | 15 / 12 |
| Android `compileSdk` / `targetSdk` / `minSdk` | 36 / 36 / 24 |

If the project is below any minimum in the first table, fix that before adopting any flavor
configuration below — the flavor mechanics work, but the resulting build will fail at
submission time.

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

Flutter owns the `FLUTTER_APP_FLAVOR` compile-time environment variable. Whenever `--flavor`
is passed to `flutter run` or `flutter build`, the tool injects `FLUTTER_APP_FLAVOR` with the
matching value before compilation. This means:

- **Never set `FLUTTER_APP_FLAVOR` yourself.** Passing
  `--dart-define=FLUTTER_APP_FLAVOR=<name>` (or putting it in a `--dart-define-from-file`
  file, Xcode or an xcconfig) fails the build with:

  ```
  Target kernel_snapshot_program failed: Error: FLUTTER_APP_FLAVOR is used by the
  framework and cannot be set using --dart-define or --dart-define-from-file
  ```

- **Use `--flavor <name>` wherever the platform supports it.** Check with
  `flutter build <platform> -h`: if `--flavor` is listed, the tool supports it for that
  platform. Each platform also needs its native setup:

  | Platform | Native setup needed for `--flavor` |
  |---|---|
  | Android | Gradle product flavors ("Android Flavor Setup" below) |
  | iOS | Xcode schemes + build configurations ("iOS Flavor Setup" below) |
  | macOS | Xcode schemes + build configurations ("macOS Desktop Flavor Setup" below) |
  | Windows, Linux | None, when the tool lists `--flavor` for the platform; optional per-flavor native changes are below |

- **Fallback: `--dart-define=APP_FLAVOR=<name>`.** Use it on a platform where `--flavor` is not
  available — a desktop platform on a toolchain that does not list `--flavor`, or macOS
  without per-flavor Xcode schemes. `AppFlavorConfig` reads `APP_FLAVOR` first, then
  `FLUTTER_APP_FLAVOR` (engineering standard §5.2), so the same Dart code works either way.
- **Never pass both** for the same build.

### Reading The Flavor At Runtime

There are two equivalent ways to read the flavor that `--flavor` set:

```dart
// Option 1 — the global constant.
import 'package:flutter/services.dart';

final String? flavor = appFlavor; // matches the --flavor value, or null if none.
```

```dart
// Option 2 — String.fromEnvironment; works because the tool injects
// FLUTTER_APP_FLAVOR as a compile-time define when --flavor is present.
const flavor = String.fromEnvironment('FLUTTER_APP_FLAVOR', defaultValue: 'prod');
```

Either works for `--flavor` builds. Neither sees an `APP_FLAVOR` dart-define, so a project that
uses the fallback on any platform MUST use the two-variable `AppFlavorConfig` from
`docs/guidelines/flutter_project_engineering_standard.md §5.2`.

---

## Android Flavor Setup

> **Build script DSL.** Kotlin DSL (`build.gradle.kts`) is what `flutter create` writes for new
> Android projects. Older Groovy DSL projects (`build.gradle`) still work but use
> `flavorDimensions "environment"` (no `+=` operator) and string literals where the Kotlin
> examples below use named-argument syntax. The flavor mechanics are identical; only the
> surface syntax differs.

### Android Toolchain Baseline

Every flavored project MUST use the AGP, KGP and Gradle versions that its Flutter version
generates (see "Toolchain Prerequisites"). They are set in these two files:

```kotlin
// android/settings.gradle.kts — EXAMPLE versions (last checked 2026-10-15).
// Use the versions your Flutter generates, not these.
plugins {
    id("dev.flutter.flutter-plugin-loader") version "1.0.0"
    id("com.android.application") version "9.1.0" apply false
    id("org.jetbrains.kotlin.android") version "2.4.0" apply false
}
```

```properties
# android/gradle/wrapper/gradle-wrapper.properties — EXAMPLE version (last checked 2026-10-15)
distributionUrl=https\://services.gradle.org/distributions/gradle-9.3.1-all.zip
```

Set the Kotlin JVM target with the top-level `kotlin { compilerOptions { } }` block in
`android/app/build.gradle.kts`. It works on every AGP version and in both Kotlin modes below.
Do **not** use the legacy `kotlinOptions { }` block inside `android { }`: it is deprecated, and
it fails once AGP's built-in Kotlin is on.

```kotlin
import org.jetbrains.kotlin.gradle.dsl.JvmTarget

android {
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }
    // No kotlinOptions { jvmTarget = "17" } here.
}

kotlin {
    compilerOptions {
        jvmTarget.set(JvmTarget.JVM_17) // match compileOptions above
    }
}
```

Notes for AGP 9 and later:

- **Keep the `gradle.properties` flags Flutter writes.** `flutter create` and the Flutter
  migrator write `android.builtInKotlin` and `android.newDsl` to `android/gradle.properties`.
  Keep the values they wrote. `android.newDsl=false` stays until a separate migration to the
  new AGP DSL.
- **Built-in Kotlin is an optional, separate migration.** With built-in Kotlin, AGP compiles
  Kotlin itself. To turn it on: check each Android plugin's `android/build.gradle(.kts)` first
  — a plugin that always applies KGP (the Kotlin Gradle Plugin) breaks the build; plugins that
  skip KGP when `android.builtInKotlin` is on, or never apply it, are fine. Then remove
  `kotlin-android` (or `org.jetbrains.kotlin.android`) from the app module's `plugins { }`
  block if it is there, and set `android.builtInKotlin=true`. Keep the
  `id("org.jetbrains.kotlin.android") version "..." apply false` line in
  `android/settings.gradle.kts`: it puts that Kotlin version on the build classpath, and AGP
  uses it instead of its own bundled Kotlin. Rollback: set `android.builtInKotlin=false` (and
  apply `kotlin-android` again if the app used it). Official steps:
  `docs.flutter.dev/release/breaking-changes/migrate-to-built-in-kotlin`.
- Gradle 9 and later have no `project.exec { }`. Custom tasks (for example a task that runs a Dart
  script to generate build metadata) MUST use an injected `ExecOperations` instead.
- After any toolchain upgrade, run a clean `flutter build apk --flavor dev` and
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
        }
        create("prod") {
            dimension = "environment"
        }
    }
}
```

Set the app name per flavor in `strings.xml` files (see "Flavor-Specific Resource Files"), not
with `resValue(...)`. A `resValue` string has one value for every language, so the launcher name
could not be translated (`guideline.md` §1.8, `platform_store_readiness.md` §2.1).

---

### Android Signing Configuration

#### Signing Strategy Options

> **Keystore location, `key.properties` naming, and the `.gitignore` rules follow
> `guideline.md §2` (the source of truth): the keystore lives in `android/` and
> `storeFile` is a relative path. This section only adds the flavor-specific Gradle wiring.**

Android signing is context-dependent. There is no single correct strategy for all teams.
Choose the approach that fits your team policy, then document it in
`docs/architecture.md §15`.

**Strategy A — Local file-based signing (single developer or small team)**

A `key.properties` file supplies keystore credentials on the release owner's machine. This is
the simplest approach for small projects.

- `* --debug` — Android provides the SDK debug keystore automatically. No setup required.
- `dev --release` — Configured by the team. Options: use the release keystore (same as prod),
  use a separate dev keystore, or allow the debug keystore as a fallback for internal QA builds.
  Document which policy your team uses; do not leave it implicit.
- `prod --release` — Release keystore required. The build MUST fail clearly if credentials are
  absent rather than silently signing with the wrong key.

**Strategy B — Separate keystores per flavor**

`dev` and `prod` flavors use completely separate keystores with different aliases. This
guarantees a dev-signed APK can never be mistaken for or substitute a prod APK.

Signing is never done in GitHub CI. CI runs only the analyze/test checks and
holds no signing secrets (engineering standard §19.3). Release builds are signed locally.

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
- Back it up and store its passwords as `guideline.md §2.3` says (at least two separate secure
  backups, at least one of them offline). Losing it permanently prevents you from publishing
  updates to Google Play for this app.

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
            // No proguardFiles(...) line needed: the Flutter Gradle plugin turns on R8 and adds
            // android/app/proguard-rules.pro by itself when that file exists ("ProGuard / R8
            // Rules" below). A proguardFiles line that names a missing file fails the build.
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
            // app_name comes from src/dev/res/values*/strings.xml (translatable).
        }
        create("prod") {
            dimension = "environment"
            // app_name comes from src/prod/res/values*/strings.xml (or src/main/res).
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
|       |-- values/
|       |   `-- strings.xml    # app_name in the template language, plus " Dev"
|       `-- values-<code>/
|           `-- strings.xml    # app_name in each other declared language, plus the dev suffix
`-- prod/
    `-- res/
        |-- mipmap-hdpi/       # Production app icon
        |-- values/
        |   `-- strings.xml    # app_name in the template language
        `-- values-<code>/
            `-- strings.xml    # app_name in each other declared language
```

```xml
<!-- android/app/src/prod/res/values/strings.xml -->
<resources>
    <string name="app_name">MyApp</string>
</resources>
```

`app_name` MUST match `appName` in `assets/config/app_config.json` for each language
(`guideline.md` §1.8).

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

> **R8 full mode is the default** on all current AGP versions. Full mode removes
> seemingly-unused classes more aggressively than the legacy compatibility mode, which makes
> correct keep rules more important — not less. If a release build crashes with
> `ClassNotFoundException` for a class that exists in source, suspect missing keep rules first.

**What Flutter already does for you.** For release builds, the Flutter Gradle plugin turns on
R8 (code shrinking) and adds three rule files by itself: Android's
`proguard-android-optimize.txt`, Flutter's own rules for the engine and embedding, and the
app's `android/app/proguard-rules.pro` **if that file exists**. Each Android plugin also ships
its own consumer rules. So:

- Do **not** add blanket rules such as `-keep class io.flutter.** { *; }`. They are not needed,
  and they stop R8 from shrinking.
- Do not add rules "just in case" for a plugin (for example sqflite). Add a rule only when a
  package's documentation asks for it, or when a release build shows a missing class.
- The Dart code (including `freezed` / `json_serializable` output) is compiled AOT to native
  code and is never processed by R8. Only Java/Kotlin code is.

Create `android/app/proguard-rules.pro` only when you need a rule:

```proguard
# Keep rules required by a specific package, copied from its documentation.
# Example shape (replace with the package's real rule):
# -keep class com.example.somepackage.model.** { *; }
```

No change to `buildTypes.release` is needed for this file: the Flutter Gradle plugin picks it
up. If you do add a `proguardFiles(...)` line in `build.gradle.kts`, the file it names MUST
exist (it may be empty): a missing file fails the release build with "Supplied proguard
configuration does not exist".

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

The rule, the current deadline and the checks are in the engineering standard §5.3.1. In short:
Google Play requires apps that target Android 15 (API 35) or higher to support 16 KB memory
pages; the risk is precompiled `.so` files inside dependencies.

Verification workflow before each Play Store release (run it on the exact build you will
upload):

```bash
# 1. Build the hardened artifact (as in "Android Run And Build Commands").

# 2. Check native-library alignment of an APK (Android build-tools 35 or later).
#    The last line must say "Verification successful".
zipalign -c -P 16 -v 4 build/app/outputs/flutter-apk/app-arm64-v8a-prod-release.apk

# 3. For an .aab: build an APK set from it with bundletool, extract the APK for
#    an arm64 device, and run the same zipalign check on it.
```

Android Studio's **Build → Analyze APK…** shows the same result in the *Alignment* column of
the `lib/` folder. Then run the release build on an emulator that uses a **"Google APIs
Experimental 16 KB Page Size"** system image (`adb shell getconf PAGE_SIZE` prints `16384`).
For any `.so` that is not aligned, upgrade or remove the dependency, and record it as a
release-blocking risk in `docs/architecture.md §21` until it is fixed.

---

## iOS Flavor Setup

### iOS Deployment Target And UIScene Lifecycle

Two pre-flight requirements precede any iOS flavor work:

1. **Deployment target: at least the Flutter minimum.** Read it from the `ios/Podfile` a fresh
   `flutter create` writes (or from the build error a too-low target produces). Set
   `platform :ios, '<min>'` in `ios/Podfile` and the same value in Xcode (Build Settings →
   Deployment → iOS Deployment Target). Details: engineering standard §5.4.

2. **UIScene lifecycle adoption.** Apple announced that UIKit apps built with the SDK released
   after iOS 26 must use the UIScene lifecycle, or they will not launch. Adopt it before you
   build with such an SDK. (This is a separate rule from the App Store's minimum upload SDK —
   check both; engineering standard §5.4.) Current Flutter templates use UIScene, and the
   Flutter CLI auto-migrates apps with an unmodified `AppDelegate`.
   - **Auto-migration success log:** `Finished migration to UIScene lifecycle` after a build.
   - **Manual migration required when:** `AppDelegate` has custom code (analytics SDK init,
     deep-link handlers, custom plugin registration, method channels).

Flavor-specific custom `AppDelegate` code that ran in
`application:didFinishLaunchingWithOptions:` (for example, a flavor-conditional Firebase or
analytics key) moves to `didInitializeImplicitFlutterEngine`. Plugin registration, method
channels and platform views MUST move there:

```swift
import UIKit
import Flutter

@main
@objc class AppDelegate: FlutterAppDelegate, FlutterImplicitEngineDelegate {

  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    // No plugin registration here any more.
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }

  // Runs after the implicit Flutter engine is created.
  func didInitializeImplicitFlutterEngine(_ engineBridge: FlutterImplicitEngineBridge) {
    GeneratedPluginRegistrant.register(with: engineBridge.pluginRegistry)
    // Method channels: use engineBridge.applicationRegistrar.messenger().
    // Flavor-conditional initialization (analytics keys, etc.) goes here.
  }
}
```

Plugins themselves migrate by adopting `FlutterSceneLifeCycleDelegate` and registering with
`registrar.addSceneDelegate(self)` in addition to (or instead of) the legacy
`addApplicationDelegate(self)`. See
`docs.flutter.dev/release/breaking-changes/uiscenedelegate` for the canonical migration steps;
if the API names there differ from this example, the official page wins.

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

> **Translated app names and flavors.** A `CFBundleDisplayName` entry in a language's
> `InfoPlist.strings` replaces `$(FLAVOR_APP_NAME)`, so the dev suffix would disappear in that
> language. Pick one way and record it in `docs/architecture.md` §15:
>
> - **Default:** accept it. In a translated language the dev build shows the translated name
>   without " Dev"; the dev icon badge (engineering standard §17.5) still tells the builds
>   apart. In the template language the suffix still shows.
> - Or, if testers must see the suffix in every language, generate each language's
>   `InfoPlist.strings` per flavor in a build script before the build (for example from a
>   per-flavor folder), so the dev values already contain the suffix.
>
> Prod builds MUST show the translated names (`guideline.md` §1.8).

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

## Desktop Flavors — Which Flag To Use

For each desktop platform the project declares, check once and record the result in
`docs/architecture.md` §15:

```bash
flutter build windows -h | grep -- --flavor   # PowerShell: | Select-String -- '--flavor'
flutter build linux -h   | grep -- --flavor
flutter build macos -h   | grep -- --flavor
```

- **Listed** → pass `--flavor <name>` (on macOS only when the per-flavor Xcode schemes exist —
  see "macOS Desktop Flavor Setup"). The tool sets `FLUTTER_APP_FLAVOR` / `appFlavor`, keeps a
  separate build folder per flavor, and filters flavor-specific assets
  (`flavors:` on an asset entry in `pubspec.yaml`).
- **Not listed** (older toolchain), or macOS without per-flavor schemes → pass
  `--dart-define=APP_FLAVOR=<name>`. `AppFlavorConfig` reads it (engineering standard §5.2).

The commands below show both forms. Use the one your check selected — never both at once.

---

## Windows Desktop Flavor Setup

When `--flavor` is available for Windows, no CMake changes are needed: the tool writes the
flavor name into `windows/flutter/ephemeral/generated_config.cmake` as `FLUTTER_APP_FLAVOR`.
Per-flavor native changes (for example the window title in `windows/runner/main.cpp`) are
optional. For most behavioral differences — feature flags, environment URLs, logging
verbosity — the Dart-level `AppFlavorConfig` is enough.

**Important:** a dart-define name MUST NOT be `FLUTTER_APP_FLAVOR`. That name is reserved by
the Flutter framework and any attempt to set it via `--dart-define` or `--dart-define-from-file`
fails the build at `kernel_snapshot_program`. The fallback name is `APP_FLAVOR`. The reference
`AppFlavorConfig` is in `docs/guidelines/flutter_project_engineering_standard.md §5.2`.

Neither flag handles all Windows packaging needs:

- **MSIX package identity** — if dev and prod builds need to be installed side by side on the
  same machine, they require distinct `identity_name` values in `msix_config`. This requires
  either separate config per flavor, or a build script that substitutes the correct value
  before calling `msix:create`.
- **Distribution certificates** — sideloaded MSIX packages require a code-signing certificate
  trusted by the target machine. Store-distributed packages are signed by the Microsoft Store.
  These are different processes; a single `msix_config` block cannot serve both without
  adjustment.
- **App display name and icon** — these are set in `msix_config` statically. If dev and prod
  builds need distinct display names or icons in the installed app list, the config must differ
  per flavor.

Document which of these cases apply to your project before settling on a Windows build strategy.

### Windows Run And Build Commands

```bash
# Run (use the form your "Which Flag To Use" check selected)
flutter run -d windows --flavor dev
flutter run -d windows --dart-define=APP_FLAVOR=dev      # fallback

# Hardened release build
flutter build windows --release --flavor prod \
  --obfuscate \
  --split-debug-info=build/symbols/windows-prod-<version>/
# fallback: replace "--flavor prod" with "--dart-define=APP_FLAVOR=prod"
```

### MSIX Packaging

For distributing Windows builds outside a direct EXE copy, package as MSIX.

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
  languages: en-us               # one entry per declared language, e.g. en-us,ml-in
  architecture: x64              # one architecture per package: x64 or arm64
  build_windows: false           # package the hardened build made above — see the warning below
```

> **Warning — `msix:create` rebuilds the app by default.** With the default
> `build_windows: true`, `msix:create` runs its own `flutter build windows` **without**
> `--obfuscate`, `--split-debug-info` or the flavor, and packages that unhardened build. Either
> set `build_windows: false` (as above) and always run the hardened `flutter build windows`
> command right before `msix:create`, or put the full hardened arguments in
> `windows_build_args`. Check the current option names on the `msix` package page.

Build the MSIX after the hardened build:

```bash
# `flutter pub run` is deprecated; use `dart run`.
dart run msix:create
```

**Architectures.** The `msix` package builds one package per architecture. Flutter builds
Windows arm64 only on an arm64 Windows machine. For the Microsoft Store, upload x64 and — if you
build and test it — a separate arm64 package in the same submission. For sideloading, one
x64 package is usually enough. Record the choice in `docs/architecture.md §15`.

If dev and prod must be installed side by side, use a distinct `identity_name` for the dev
flavor (e.g. `com.yourcompany.myapp.dev`). A simple approach is a dev build script that passes
the dev values on the command line (`dart run msix:create --identity-name ... --display-name ...`)
or swaps the `msix_config` block before packaging. Document the chosen approach in
`docs/architecture.md §15`.

### Desktop Setup (sqflite, Window Size, Shortcuts)

`sqflite` FFI initialization, minimum window size and other desktop setup are not flavor-specific.
They are described once, in the engineering standard §5.5.

---

## macOS Desktop Flavor Setup

When dev and prod only differ in Dart behavior and you do not want per-flavor Xcode schemes,
pass `--dart-define=APP_FLAVOR=<name>`, never `FLUTTER_APP_FLAVOR`.

When dev and prod must be **installed side by side** or show **different names**, they need
different bundle ids. Use the iOS pattern (see "Xcode Build Configurations And Schemes" above)
inside `macos/`, and then pass `--flavor <name>` (check that `flutter build macos -h` lists it):

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

Create the `dev` / `prod` schemes and the `Debug-<flavor>`, `Profile-<flavor>`,
`Release-<flavor>` build configurations in `macos/Runner.xcworkspace` that use these files, and
map them in `macos/Podfile` the same way as iOS.

Entitlements are per build configuration: if a dev build needs extra entitlements (for example a
local debug server), add them to `DebugProfile.entitlements` only — never to
`Release.entitlements`.

### macOS Run And Build Commands

```bash
# With per-flavor Xcode schemes
flutter run -d macos --flavor dev
flutter build macos --release --flavor prod \
  --obfuscate \
  --split-debug-info=build/symbols/macos-prod-<version>/

# Without per-flavor schemes (fallback)
flutter run -d macos --dart-define=APP_FLAVOR=dev
flutter build macos --release --dart-define=APP_FLAVOR=prod \
  --obfuscate \
  --split-debug-info=build/symbols/macos-prod-<version>/
```

Signing, notarization and store upload: `docs/guidelines/platform_store_readiness.md` §5.

---

## Linux Desktop Flavor Setup

Build machine setup and `APPLICATION_ID` are described in engineering standard §5.5.3. Pass the
flavor with `--flavor <name>` when `flutter build linux -h` lists it, otherwise with
`--dart-define=APP_FLAVOR=<name>`.

When dev and prod must be installed side by side, they need different `APPLICATION_ID` values
(and usually different `BINARY_NAME`s). Override them in `linux/CMakeLists.txt` **after** the
`add_subdirectory(${FLUTTER_MANAGED_DIR})` line and **before** `add_subdirectory("runner")`:

```cmake
# ... set(BINARY_NAME "my_app") and set(APPLICATION_ID "com.example.my_app") near the top ...

add_subdirectory(${FLUTTER_MANAGED_DIR})

# Side-by-side dev installs.
# With --flavor: the Flutter tool defines FLUTTER_APP_FLAVOR, but only from the
# add_subdirectory(${FLUTTER_MANAGED_DIR}) line on — a check placed earlier is skipped.
# Without --flavor (APP_FLAVOR fallback): pass APP_ID_SUFFIX=.dev in the environment.
if(DEFINED FLUTTER_APP_FLAVOR AND FLUTTER_APP_FLAVOR STREQUAL "dev")
  set(APPLICATION_ID "${APPLICATION_ID}.dev")
  set(BINARY_NAME "${BINARY_NAME}_dev")
elseif(DEFINED ENV{APP_ID_SUFFIX})
  set(APPLICATION_ID "${APPLICATION_ID}$ENV{APP_ID_SUFFIX}")
  set(BINARY_NAME "${BINARY_NAME}_dev")
endif()

add_subdirectory("runner")
```

Keep separate `.desktop`, icon and packaging files (`snapcraft.yaml`, Flatpak manifest) per
flavor only if a dev flavor is actually distributed; most projects package `prod` only.

### Linux Run And Build Commands

```bash
flutter run -d linux --flavor dev
flutter run -d linux --dart-define=APP_FLAVOR=dev        # fallback

flutter build linux --release --flavor prod \
  --obfuscate \
  --split-debug-info=build/symbols/linux-prod-<version>/
# fallback: replace "--flavor prod" with "--dart-define=APP_FLAVOR=prod"
```

Packaging and store upload: `docs/guidelines/platform_store_readiness.md` §6.

---

## Recommended Release Matrix

Keep the rows for the platforms your project declares. On a desktop platform, `<flavor flag>`
is `--flavor <name>` or `--dart-define=APP_FLAVOR=<name>`, as chosen in "Desktop Flavors —
Which Flag To Use".

> **Implicit flag — `--tree-shake-icons`.** Release builds tree-shake unused icon glyphs by
> default, as long as every `IconData` is a compile-time constant such as `Icons.add` (see
> engineering standard §17.3). The flag is not shown in the commands below because it is on by
> default in release mode.

| Platform | Flavor | Mode | Signing Required | Command |
|----------|--------|------|-----------------|---------|
| Android | `dev` | `debug` | No — automatic debug keystore | `flutter run --flavor dev` |
| Android | `dev` | `release` | Team policy — document your choice | `flutter build apk --flavor dev --release` |
| Android | `prod` | `debug` | No — automatic debug keystore | `flutter run --flavor prod` |
| Android | `prod` | `release` split APK | Yes — release keystore required | `flutter build apk --flavor prod --release --obfuscate --split-debug-info=... --split-per-abi` |
| Android | `prod` | `release` Play Store | Yes — release keystore required | `flutter build appbundle --flavor prod --release --obfuscate --split-debug-info=...` |
| iOS | `dev` | `debug` | No — automatic development signing | `flutter run --flavor dev` |
| iOS | `prod` | `release` | Yes — distribution profile required | `flutter build ipa --flavor prod --release --obfuscate --split-debug-info=...` |
| Windows | `dev` | `debug` | No | `flutter run -d windows <flavor flag dev>` |
| Windows | `prod` | `release` MSIX | Depends on distribution channel — see Windows section | `flutter build windows --release <flavor flag prod> --obfuscate --split-debug-info=...` then `dart run msix:create` with `build_windows: false` |
| macOS | `dev` | `debug` | No — development signing | `flutter run -d macos <flavor flag dev>` |
| macOS | `prod` | `release` | Yes — Apple Distribution (Mac App Store) or Developer ID + notarization | `flutter build macos --release <flavor flag prod> --obfuscate --split-debug-info=...` |
| Linux | `dev` | `debug` | No | `flutter run -d linux <flavor flag dev>` |
| Linux | `prod` | `release` | Store-side for Snap / Flathub; optional GPG for direct packages | `flutter build linux --release <flavor flag prod> --obfuscate --split-debug-info=...` then package |

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
- ProGuard / R8 keep rules only where a package documents them (Flutter adds the engine rules
  itself — see "ProGuard / R8 Rules").
- A documented and implemented signing strategy for each flavor × mode combination.
- `android/key.properties`, `android/*.jks`, and `android/*.keystore` added to `.gitignore`.
- The JDK the project's AGP needs, and the AGP / KGP / Gradle versions the latest stable
  Flutter generates — pinned in the project's own `settings.gradle.kts` and Gradle wrapper.
  Use `kotlin { compilerOptions { } }`, never `kotlinOptions { }` (see "Android Toolchain
  Baseline" above).
- **16 KB page-size compliance** for any release targeting Android 15+. Audit bundled
  `.so` files in dependencies before each Play Store submission (see "16 KB Page Size
  Compliance" above).

**iOS:**
- Xcode schemes aligned with flavor names.
- `Debug-<flavor>` / `Profile-<flavor>` / `Release-<flavor>` build configurations, with the
  bundle id and display name set per configuration in Build Settings (or xcconfig files).
- Provisioning profiles per flavor bundle ID.
- Flavor-specific app icons in asset catalogs.
- Distribution profile required for `prod --release`; automatic signing for debug and dev.
- At least the Flutter minimum iOS deployment target in `ios/Podfile` and Xcode Build
  Settings.
- **UIScene Application Scene Manifest** in `Info.plist`. `FlutterImplicitEngineDelegate`
  adoption in `AppDelegate` if any custom code (analytics, deep-link handlers, plugin
  registration) was added beyond the default. Move flavor-conditional setup to
  `didInitializeImplicitFlutterEngine`.
- Build with the Xcode / iOS SDK that App Store Connect currently requires. Before moving to
  an SDK that requires the UIScene lifecycle, confirm the UIScene migration is complete —
  non-UIScene apps built with such an SDK fail to launch.

**Windows:**
- `sqflite_common_ffi` initialization in `main()` for any desktop + sqflite usage.
- `window_manager` for size constraints (engineering standard §5.5).
- `msix` package for distribution packaging; use `dart run msix:create` (the deprecated
  `flutter pub run msix:create` will eventually stop working).
- A documented strategy for MSIX identity and display name differentiation between flavors
  if side-by-side installation is required.
- One MSIX per architecture (x64; arm64 only if built and tested on an arm64 machine), with
  `build_windows: false` so `msix:create` packages the hardened build (see "MSIX
  Packaging").

**macOS:**
- Bundle id, name and copyright in `macos/Runner/Configs/AppInfo.xcconfig` (plus a dev
  xcconfig and scheme if dev and prod install side by side).
- At least the Flutter minimum macOS deployment target in `macos/Podfile` and Xcode.
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
