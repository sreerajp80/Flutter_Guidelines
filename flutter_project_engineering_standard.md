# Flutter Project Engineering Standard

This document is a reusable engineering standard for future Flutter projects.

It is intentionally layered. Small apps should inherit the core baseline without being forced into
release-process or high-security requirements that do not fit the product.

---

## 1. How To Use This Standard

### 1.1 Conformance Language

Use these terms consistently:

- `MUST`: mandatory for the stated scope.
- `SHOULD`: expected default; deviations require a reason.
- `MAY`: optional.

### 1.2 Applicability Profiles

Every Flutter repository MUST declare which profile applies.

| Profile | Applies To | Purpose |
|---------|------------|---------|
| `Core Baseline` | All Flutter application repositories | Universal maintainability and code-quality rules |
| `Production App Extension` | Apps shipped to real users, external QA, or store review | Release, UX, and environment discipline |
| `Sensitive Data Extension` | Apps handling auth secrets, financial data, health data, PII, or locally encrypted content | Stronger security, storage, logging, and backup rules |

A simple internal tool may use only `Core Baseline`.
A public consumer app will usually use `Core Baseline` plus `Production App Extension`.
An authenticator, password manager, finance, or health app will usually use all three.

#### 1.2.1 Project Profile (Mandatory)

Every app repository MUST have a `docs/PROJECT_PROFILE.md`, filled in from
`PROJECT_PROFILE_TEMPLATE.md`. It is the one place that records the choices this standard leaves
open, so neither people nor AI agents have to guess them:

- app identity — name, organization, reverse-DNS application / bundle id;
- applicability profiles in force (this section);
- **target platforms** — any of Android, iOS, Windows, macOS, Linux, Web;
- **distribution channels** per platform — e.g. Google Play, Apple App Store, Microsoft Store,
  Mac App Store, Developer ID download, Snap Store, Flathub, direct download;
- **declared languages** (section 8) and any language packs that apply;
- About-screen options, including the optional signature badge (`guideline.md` §1.7);
- minimum OS versions per platform.

Rules elsewhere in this standard that depend on a platform, store, or language apply **only when
the profile declares it**. For every declared distribution channel, the matching gate in
`platform_store_readiness.md` MUST pass before the first release to that channel.

### 1.3 Repository Types

This document primarily targets Flutter application repositories.

If the repository is a Flutter package or plugin:

- `android/`, `ios/`, and build-flavor rules MAY be omitted if not applicable.
- `pubspec.lock` SHOULD follow package conventions rather than application conventions.
- UI, release, and integration-test requirements apply only if the package ships runnable example apps.

---

## 2. Core Principles

1. Structure by responsibility first, then by implementation detail.
2. Keep business logic outside widgets.
3. Prefer explicit code over clever abstractions.
4. Enforce standards through tooling where practical.
5. One repository SHOULD have one clear way to do state, navigation, theming, errors, and testing.
6. Optimize for current complexity, not hypothetical future complexity.
7. Every app is secure by default. Security and logging policy are product requirements, not
   cleanup work: never traded away for convenience, and designed so they do not burden the user
   or the app (section 15).
8. Performance is a feature; it is designed in, not bolted on.
9. Accessibility is a correctness requirement, not a nice-to-have.

---

## 3. Project Structure

### 3.1 Choose The Simplest Layout That Fits

#### Tier 1: Layer-First

Use for smaller apps, single-domain apps, or early products.

```text
lib/
|-- core/
|   `-- config/       # AppConfig + ConfigService (About). Fixed path — see guideline.md §1.
|-- models/
|-- providers/        # Or controllers/blocs/cubits
|-- screens/
|-- services/
|-- widgets/
`-- main.dart
```

The `core/config/` path is fixed across every tier: the About-screen `AppConfig` model and
`ConfigService` loader always live at `lib/core/config/`, as required by `guideline.md §1`.
A small Tier 1 app keeps the rest of the layout flat.

#### Tier 2: Feature-First

Use when multiple product areas evolve independently or when several developers routinely touch
unrelated features.

```text
lib/
|-- app/
|   |-- config/            # App-level config (routing, theme wiring, DI roots)
|   |-- routing/
|   `-- theme/
|-- core/
|   |-- config/            # AppConfig + ConfigService (About). Fixed path — see guideline.md §1.
|   |-- errors/
|   |-- lifecycle/
|   |-- logging/
|   |-- network/
|   |-- security/
|   |-- storage/
|   `-- widgets/
|-- features/
|   `-- <feature_name>/
|       |-- data/
|       |-- domain/
|       `-- presentation/
`-- main.dart
```

`app/config/` holds app-level wiring; the About-screen `AppConfig` + `ConfigService` stay at the
fixed `core/config/` path in every tier (`guideline.md §1`).

Promote from Tier 1 to Tier 2 only when the current shape creates actual boundary confusion,
naming collisions, or merge friction.

### 3.2 Structure Rules

These rules apply to all app repositories.

- `main.dart` MUST stay thin: framework initialization, config loading, provider or DI setup,
  then `runApp`. Heavy initialization MUST be moved to a startup service.
- A broad catch-all `utils/` directory SHOULD be avoided. If a `utils/` folder starts collecting
  unrelated concerns, split it into named locations.
- `test/` SHOULD mirror `lib/` closely enough that ownership is obvious.
- Platform directories MUST NOT contain business logic that belongs in Dart unless platform
  constraints require it.

### 3.3 Recommended Root Layout For App Repositories

```text
project/
|-- android/                  # Required when Android is a declared platform
|-- ios/                      # Required when iOS is a declared platform
|-- windows/                  # Required when Windows is a declared platform
|-- macos/                    # Required when macOS is a declared platform
|-- linux/                    # Required when Linux is a declared platform
|-- web/                      # Required when Web is a declared platform
|-- assets/
|   |-- config/               # app_config.json (About screen source of truth)
|   |-- fonts/
|   |-- icons/
|   `-- images/
|       |-- 2.0x/
|       `-- 3.0x/
|-- docs/
|   |-- GUIDELINES_MANIFEST.md
|   `-- PROJECT_PROFILE.md    # Mandatory — platforms, stores, languages, identity (1.2.1)
|-- plans/                    # change planning logs
|-- change_log/               # implemented change logs
|-- lib/
|-- test/
|-- integration_test/         # Required only when end-to-end coverage applies
|-- .github/                  # workflows/ci.yml (checks only, §19)
|-- CLAUDE.md                 # Mandatory project-root AI instructions (MUST)
|-- AGENTS.md                 # Mandatory project-root instructions for other AI agents (MUST)
|-- analysis_options.yaml
|-- pubspec.yaml
|-- README.md
`-- .gitignore
```

Application repositories SHOULD commit `pubspec.lock`.
Packages and plugins SHOULD follow normal package conventions.

Create the platform folders with the Flutter tool, never by hand, and only for declared platforms:

```bash
# New app — org is your reverse-DNS prefix; list only the declared platforms.
# Run it in the project root (the folder that holds docs/guidelines/).
# The final "." means "this folder"; without it Flutter makes a new subfolder.
flutter create --org com.example --project-name my_app --platforms=<declared platforms> .

# Add a platform to an existing app later.
flutter create --platforms=macos .
```

The application / bundle id MUST be the same reverse-DNS id on every platform where the store
allows it (Android `applicationId`, iOS/macOS `PRODUCT_BUNDLE_IDENTIFIER`, Linux `APPLICATION_ID`,
MSIX `identity_name` is assigned by Partner Center for the Microsoft Store).

---

## 4. Architecture Baseline

### 4.1 State Management

- SHOULD use one primary state-management approach per repository. Deviations are permissible
  when documented: name the second pattern, name the boundary where it applies, and commit to
  not crossing that boundary. A common legitimate case is using `ValueNotifier` or `setState`
  for purely local widget state while Riverpod or Bloc handles cross-widget and persistent state.
- Do not mix multiple state systems for the same problem. The restriction is on competing
  solutions to the same concern, not on the total count of patterns in the repository.
- Providers, controllers, or blocs MUST expose UI-facing state and transitions, not raw storage
  primitives.
- State layers MUST NOT import widget classes.

### 4.2 Data Flow

Preferred flow:

```text
Widget -> State Layer -> Service or Use Case -> Repository -> Datasource
```

Not every Tier 1 app needs an explicit repository layer. Introduce `Repository` and `Datasource`
boundaries when they reduce complexity or isolate external systems cleanly.

Rules:

- Widgets MUST NOT know SQL, encryption, HTTP, or storage implementation details.
- Services SHOULD be stateless where practical.
- Singletons SHOULD be limited to infrastructure concerns such as database access, app config, or
  logging.
- Services MUST NOT decide UI copy or navigation policy.

### 4.3 Models And Entities

- Prefer immutable models.
- In Tier 1, a single model MAY serve both storage and UI if the shape is simple.
- In Tier 2, transport models and domain entities SHOULD diverge when the serialization shape and
  business shape differ.
- Constants that define protocols, storage keys, or cryptographic formats SHOULD live in one
  reviewed location.

### 4.4 Dependency Injection

- Use framework-native dependency wiring first.
- Introduce a dedicated DI solution only when it clearly reduces complexity.
- Anything that tests need to replace MUST be injectable.

### 4.5 App Initialization Sequence

The startup order matters. Failing to initialize infrastructure in the right order causes silent
crashes in release builds that never appear in debug.

Recommended sequence in `main()`:

1. `WidgetsFlutterBinding.ensureInitialized()`
2. Logging infrastructure initialization (`AppLogger.init()`, section 14.2) and the global error
   handlers (section 11.1). Flavor values are compile-time constants (`AppFlavorConfig`,
   section 5.2), so nothing has to load first. Every later step can now log its failures.
3. Platform-specific FFI or native bindings (e.g. `sqfliteFfiInit()` for Windows/Linux desktop)
4. Secure storage or key material bootstrap
5. Database initialization and schema migration
6. App config loading (`ConfigService`, `guideline.md` §1) and the saved app language (8.4)
7. App lifecycle observer registration
8. `runApp(...)`

Document the actual sequence in `docs/architecture.md` §5 for the project.

Each initialization step MUST handle its own failure gracefully and surface a safe error state
rather than crashing silently.

---

## 5. Environment And Build Configuration

Which parts apply:

- **Every app**, for its declared platforms: 5.3 Toolchain Requirements (including 5.3.1),
  5.4 iOS deployment target and UIScene, and 5.5 Desktop Build Setup.
- **When the app uses flavors**: 5.1, 5.2, and the flavor parts of 5.4 and 5.5.
- **`Production App Extension`**: 5.6 Artifact Selection.

### 5.1 When Flavors Are Required

Build flavors are REQUIRED when any of the following is true:

- The app has distinct `dev`, `staging`, or `prod` environments.
- QA needs production-like builds against non-production config.
- Multiple variants must be installed side by side.
- Release behavior differs materially by environment.

If the app has only one environment and no parallel install need, flavors MAY be omitted.

### 5.2 Recommended Flavor Model

A common baseline is `dev` and `prod`.

The `AppFlavorConfig` MUST read the flavor value from two environment variables in priority
order. This shape is required because a build passes the flavor in one of two ways, depending
on what the platform and the toolchain support:

1. `APP_FLAVOR` — a custom, non-reserved name, passed with `--dart-define=APP_FLAVOR=<value>`.
   Use it only on a platform where `--flavor` is not available (see "Which way to pass the
   flavor" below). `FLUTTER_APP_FLAVOR` cannot be set via `--dart-define` (see below).
2. `FLUTTER_APP_FLAVOR` — the framework-owned name. The Flutter tool injects this
   automatically as a compile-time define whenever `--flavor` is passed to `flutter run` or
   `flutter build`.

> **Framework reservation.** The build fails with
> `Target kernel_snapshot_program failed: Error: FLUTTER_APP_FLAVOR is used by the framework
> and cannot be set using --dart-define or --dart-define-from-file` if any build command
> tries to pass `FLUTTER_APP_FLAVOR` explicitly. The two-variable pattern below is what lets
> a single `AppFlavorConfig` work on every platform, whichever way the flavor arrives.

```dart
enum AppFlavor { dev, prod }

class AppFlavorConfig {
  AppFlavorConfig._(this.flavor);

  // Passed with --dart-define=APP_FLAVOR=<value> on a platform without --flavor
  // support (see "Which Way To Pass The Flavor"). Empty when not provided.
  static const _appFlavorValue = String.fromEnvironment('APP_FLAVOR');

  // Injected by the Flutter tool on any platform whenever --flavor is passed.
  // Falls back to 'prod' so an unflavored debug build still has a deterministic value.
  static const _frameworkFlavorValue = String.fromEnvironment(
    'FLUTTER_APP_FLAVOR',
    defaultValue: 'prod',
  );

  static String _resolved() =>
      _appFlavorValue.isNotEmpty ? _appFlavorValue : _frameworkFlavorValue;

  static final AppFlavorConfig instance = AppFlavorConfig._(_parse(_resolved()));

  final AppFlavor flavor;

  static AppFlavor _parse(String value) {
    switch (value.trim().toLowerCase()) {
      case 'dev':
        return AppFlavor.dev;
      case 'prod':
      default:
        return AppFlavor.prod;
    }
  }

  bool get isDev => flavor == AppFlavor.dev;
  bool get isProd => flavor == AppFlavor.prod;

  /// Added to the localized app name (`appName` in app_config.json,
  /// guideline.md §1.8). Never hard-code the app name itself here.
  String get nameSuffix => isDev ? ' Dev' : '';
  bool get showEnvironmentBanner => isDev;
  bool get enableVerboseLogging => isDev;
}
```

The modern equivalent of `String.fromEnvironment('FLUTTER_APP_FLAVOR', ...)` is the global
`appFlavor` constant from `package:flutter/services.dart`. Either accessor is acceptable;
the two-variable resolution above MUST be preserved on top of whichever accessor is chosen,
otherwise a platform that cannot use `--flavor` has no way to communicate the flavor.

#### Which Way To Pass The Flavor

Use **one rule** for every platform, and check it against the project's own toolchain:

1. **Use `--flavor <name>`** when `flutter build <platform> -h` lists `--flavor` **and** the
   platform has the native setup it needs:
   - Android: product flavors in Gradle (below).
   - iOS and macOS: a matching Xcode scheme and build configurations (5.4, 5.5.2).
   - Windows and Linux: no native setup is needed when the tool supports the flag. Optional
     per-flavor native changes (window title, `APPLICATION_ID`) are in the flavors guide.
2. **Otherwise use `--dart-define=APP_FLAVOR=<name>`.** This covers a desktop platform on a
   toolchain without `--flavor` support, and macOS without per-flavor Xcode schemes.
3. **Never pass both** for the same build, and never pass `FLUTTER_APP_FLAVOR` yourself.

Record the chosen way per platform in `docs/architecture.md` §15 and in the command blocks of
`CLAUDE.md` / `AGENTS.md`.

```bash
# Platform and toolchain support --flavor (Android, iOS, and any desktop that lists it)
flutter run --flavor dev
flutter run -d windows --flavor dev
flutter build apk --flavor prod --release

# Platform without --flavor support (or macOS without per-flavor Xcode schemes)
flutter run -d macos --dart-define=APP_FLAVOR=dev
flutter build macos --release --dart-define=APP_FLAVOR=prod
```

Native flavor names (Gradle product flavors, Xcode schemes) SHOULD match the Dart flavor value.

#### Android Product Flavors

When Android flavors are used, the project SHOULD define product flavors in
`android/app/build.gradle.kts` (Kotlin DSL is what `flutter create` writes for new projects;
Groovy DSL `build.gradle` still works in older projects but uses
`flavorDimensions "environment"` without the `+=` operator).

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

Full signing and resource wiring: `flutter_build_flavors_guide.md`, "Android Flavor Setup".

### 5.3 Toolchain Requirements (Every App)

This standard does not pin tool versions. The rules are:

- **Use the latest stable Flutter** (and the Dart SDK that ships with it).
- **Android: use the AGP, Kotlin Gradle Plugin (KGP) and Gradle versions that Flutter
  generates.** `flutter create` writes them for new projects; `flutter upgrade` plus the
  Flutter migrator update existing ones. Do NOT hand-pick or hand-downgrade them.
- **Each project pins its own versions** in its own files: `android/settings.gradle.kts`
  (AGP, KGP), `android/gradle/wrapper/gradle-wrapper.properties` (Gradle), `pubspec.lock`,
  the CI image, and the identity table in `CLAUDE.md` / `AGENTS.md`.
- **Look up versions, never guess them.** AI agents and contributors MUST read the current
  versions from `flutter --version` and the project files above before writing or changing
  build files. Never write a version from memory.
- **Use the JDK the build needs.** Current AGP versions need Java 17 or later; an older JDK
  fails the build. `flutter doctor -v` shows which JDK Flutter uses; check it with
  `java -version`.

> **Reference snapshot — not a rule.** When this standard was last checked (2026-10-15), the
> latest stable Flutter (3.47.6 / Dart 3.13.5) generated AGP 9.1.0, KGP 2.4.0, Gradle 9.3.1,
> `compileSdk` / `targetSdk` 36 and `minSdk` 24, and needed Java 17. Use this only to spot a
> project that is clearly behind.

#### Android Build Files — Rules For Any AGP Version

- **Set the Kotlin JVM target with the top-level `kotlin { compilerOptions { } }` block** in
  `android/app/build.gradle.kts`. Do not use the legacy `kotlinOptions { }` block inside
  `android { }`: it is deprecated, and it fails once AGP's built-in Kotlin is on (AGP 9 and
  later). `compilerOptions` works in every mode:

  ```kotlin
  import org.jetbrains.kotlin.gradle.dsl.JvmTarget

  kotlin {
      compilerOptions {
          jvmTarget.set(JvmTarget.JVM_17) // match compileOptions in android { }
      }
  }
  ```

- **Keep the `gradle.properties` flags that Flutter writes.** On AGP 9 and later,
  `flutter create` and the Flutter migrator write `android.builtInKotlin` and `android.newDsl`
  to `android/gradle.properties`. Keep the values Flutter wrote. Turning built-in Kotlin on
  yourself (`android.builtInKotlin=true`, and removing `kotlin-android` from the app module) is
  an optional, separate migration. Do it only after checking that every Android plugin
  supports it: a plugin that always applies the Kotlin Gradle Plugin breaks the build. Steps
  and rollback: `docs/guidelines/flutter_build_flavors_guide.md` ("Android Toolchain
  Baseline").
- **Gradle 9 and later have no `project.exec { }`.** Custom Gradle tasks that run a process
  MUST use an injected `ExecOperations` instead (this also works on older Gradle).
- **R8 full mode is the default** on all current AGP versions. It removes seemingly-unused
  classes more aggressively, so keep rules must be deliberate (see
  `docs/guidelines/flutter_build_flavors_guide.md` ProGuard section).

Full `settings.gradle.kts` and `build.gradle.kts` examples are in
`docs/guidelines/flutter_build_flavors_guide.md` ("Android Toolchain Baseline").

#### Upgrading An Existing App To A New Flutter Release

Do the upgrade as its own change, with no feature work mixed in:

1. Read the official "What's new" post and the breaking-changes list
   (`docs.flutter.dev/release/breaking-changes`) for every release you are crossing.
2. `flutter upgrade`, then `flutter pub upgrade` and `flutter pub outdated` to see which
   plugins need a new major.
3. Run `dart fix --apply`, plus any `dart fix --apply --code=<name>` migration named in the
   release notes. Named migrations do not run with plain `dart fix --apply`.
4. Let the Flutter migrator update the Android files (AGP, KGP, Gradle wrapper,
   `gradle.properties`). Do NOT type versions from memory; check what Flutter wrote and keep
   it. Compare `ios/Podfile` / `macos/Podfile` and the Xcode deployment targets with what a
   fresh `flutter create` writes, and raise them if the new release raised the minimum.
5. Run `flutter analyze`, the full test suite, and a release build of every flavor on every
   declared platform. Update the pinned versions in `CLAUDE.md` / `AGENTS.md` and the CI
   setup, and record the upgrade in the change log (`change_log/`), and in `CHANGELOG.md` if
   the project keeps one.

**Known migrations — apply each one when its trigger applies to the project:**

| Trigger | Migration |
|---|---|
| The `material_ui` / `cupertino_ui` packages exist for your Flutter, and the project still imports `package:flutter/material.dart` or `cupertino.dart` | `dart fix --apply --code=migrate_design_widgets`, then follow 6.1. Replace any `material_ui: any` / `cupertino_ui: any` the tool wrote with a caret (`^`) constraint. |
| The project moves to AGP 9 or later / Gradle 9 or later | Replace `kotlinOptions { }` with `kotlin { compilerOptions { } }` and `project.exec { }` with `ExecOperations`; keep the `gradle.properties` flags Flutter writes. |
| The new Flutter raises the minimum iOS or macOS version | Raise `platform :ios` / `platform :osx` in the Podfiles and the Xcode deployment targets. |
| You will build with an Apple SDK that requires the UIScene lifecycle | Finish the UIScene migration (5.4) first. |
| `flutter build <desktop platform> -h` now lists `--flavor`, and the project passes `APP_FLAVOR` there | Optional: switch that platform to `--flavor` (5.2). |

#### 5.3.1 Android 16 KB Page Size Compliance

Google Play requires apps that target Android 15 (API 35) or higher to support 16 KB memory
pages on 64-bit devices. Check the current enforcement date on Android's page-size guide
(`developer.android.com/guide/practices/page-sizes`) before each release. When last checked
(2026-10), it said that from February 1, 2027 an app update without 16 KB support cannot be
released.

An app whose code is only Dart, Java or Kotlin is not affected: the Flutter engine and current
Android build tools produce 16 KB-aligned output. The risk is **precompiled native libraries
(`.so` files) inside dependencies** that were built with 4 KB alignment — for example older
media, image-processing or database packages that bundle their own `.so` files.

Required actions under `Production App Extension`, before every Play Store submission:

- Check the native libraries in the release artifact with one of:
  - Android Studio → **Build → Analyze APK…** → `lib/` folder → the *Alignment* column;
  - `zipalign -c -P 16 -v 4 <app>.apk` (Android build-tools 35 or later; the last line must
    say `Verification successful`);
  - `check_elf_alignment.sh <app>.apk` (script from Android's page-size guide).

  For an `.aab`, run the check on an APK built from it with `bundletool`.
- Run the release build on a 16 KB emulator: Android Studio → SDK Manager → SDK Platforms →
  Show Package Details → install a **"Google APIs Experimental 16 KB Page Size"** system
  image, create a device with it, and confirm with `adb shell getconf PAGE_SIZE` (prints
  `16384`).
- Document any dependency that is not yet 16 KB-aligned as a release blocker in
  `architecture.md §21 Known Risks`.

### 5.4 iOS Setup (Deployment Target, UIScene, Flavors)

#### iOS Deployment Target And UIScene Lifecycle

- **Minimum iOS deployment target: at least the minimum your Flutter version supports.** Read
  it from the `ios/Podfile` that a fresh `flutter create` writes, or from the build error a
  too-low target produces. Set `platform :ios, '<min>'` in `ios/Podfile` and the same value as
  the iOS Deployment Target in Xcode (Build Settings). Do the same for macOS
  (`platform :osx, '<min>'` in `macos/Podfile`; see 5.5.2). Record the chosen minimum in
  `docs/PROJECT_PROFILE.md`.
- **Two separate Apple rules — check both before each release:**
  - **Upload SDK minimum.** App Store Connect accepts only builds made with a recent Xcode /
    iOS SDK, and Apple raises it every year (`developer.apple.com/news/upcoming-requirements`).
  - **UIScene lifecycle.** Apple announced that UIKit apps built with the SDK released after
    iOS 26 must use the UIScene lifecycle, or they will not launch. Adopt UIScene before you
    build with such an SDK.
- **Flutter support for UIScene.** Current Flutter templates use UIScene, and the Flutter CLI
  migrates apps with an unmodified `AppDelegate` automatically on `flutter run` /
  `flutter build ios`. Watch the build log for `Finished migration to UIScene lifecycle`
  (success) or migration warnings (manual work required). If your Flutter has no UIScene
  support, upgrade Flutter first.
- **For projects with a customized `AppDelegate`** (analytics initialization, deep link
  handlers, custom plugin registration, method channels) the migration is manual:
  - `AppDelegate` must adopt `FlutterImplicitEngineDelegate`.
  - Plugin registration, method channels and platform views move from
    `application:didFinishLaunchingWithOptions:` to
    `didInitializeImplicitFlutterEngine(_ engineBridge: FlutterImplicitEngineBridge)`
    (example in the flavors guide, "iOS Flavor Setup").
  - `Info.plist` must include a `UIApplicationSceneManifest` (Application Scene Manifest)
    entry. The Flutter migrator usually adds this automatically.
  - Plugin developers using lifecycle events must implement `FlutterSceneLifeCycleDelegate`
    and register via `registrar.addSceneDelegate(self)`.
- See the official guide at `docs.flutter.dev/release/breaking-changes/uiscenedelegate` for
  the full migration steps. Test on the newest iOS simulator and device before the next App
  Store submission.

#### iOS Flavors (Xcode Build Configurations And Schemes)

Rules when the app uses iOS flavors (step-by-step setup: `flutter_build_flavors_guide.md`,
"iOS Flavor Setup"):

- Flutter maps `--flavor <name>` to an Xcode **scheme** named `<name>` and to **build
  configurations** named `Debug-<name>`, `Profile-<name>` and `Release-<name>`. All of them MUST
  exist; schemes on the plain `Debug` / `Release` configurations do not work.
- Set `PRODUCT_BUNDLE_IDENTIFIER` and the display name per build configuration (Build Settings).
  `Info.plist` reads them through `$(PRODUCT_BUNDLE_IDENTIFIER)` and a build variable such as
  `$(FLAVOR_APP_NAME)`; it never hard-codes an id.
- Map every new configuration in `ios/Podfile` (`project 'Runner', { 'Debug-dev' => :debug, … }`)
  and run `pod install`.
- Never set `FLUTTER_APP_FLAVOR` in Xcode or an xcconfig file (see 5.2).
- Provisioning profiles MUST be set per configuration. Do not share production profiles with dev
  builds.

### 5.5 Desktop Build Setup (Windows, macOS, Linux)

This section applies to each desktop platform the project declares. The shared rules come first,
then one sub-section per platform (5.5.1 Windows, 5.5.2 macOS, 5.5.3 Linux). Store and
direct-download release gates for each are in `platform_store_readiness.md`.

Desktop targets do not use Android product flavors. Pass the flavor as described in 5.2,
"Which Way To Pass The Flavor": `--flavor <name>` when `flutter build <platform> -h` lists it
(and, on macOS, matching Xcode schemes exist), otherwise `--dart-define=APP_FLAVOR=<name>`.
The `AppFlavorConfig` pattern reads either. A dart-define name MUST NOT be
`FLUTTER_APP_FLAVOR` — that name is owned by the framework and any attempt to set it via
`--dart-define` fails the build.

Additional desktop setup required before any DB or FFI work can run (`sqflite` needs the FFI
factory on Windows and Linux; macOS uses the normal plugin):

```dart
// In main() before runApp, for Windows and Linux desktop:
import 'dart:io';
import 'package:flutter/foundation.dart' show kIsWeb;
import 'package:flutter/widgets.dart';
import 'package:sqflite_common_ffi/sqflite_ffi.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  // Check kIsWeb first: Platform.* throws on Web.
  if (!kIsWeb && (Platform.isWindows || Platform.isLinux)) {
    sqfliteFfiInit();
    databaseFactory = databaseFactoryFfi;
  }
  runApp(const MyApp());
}
```

On Windows and Linux, `sqflite_common_ffi` also needs the SQLite native library at run time.
Follow the package's setup (for example bundling it with a helper package such as
`sqlite3_flutter_libs`, or installing `libsqlite3` as a Linux package dependency), and test the
release build on a clean machine.

**Window constraints** — set a minimum window size to prevent layouts breaking at small sizes.
This section is the single reference for desktop setup; the flavors guide links here. If you also
want an initial size and a centred window, use `WindowOptions` with
`windowManager.waitUntilReadyToShow(...)` instead of the calls below.

```dart
// In main() after WidgetsFlutterBinding.ensureInitialized() and before runApp.
import 'dart:io';
import 'package:flutter/foundation.dart' show kIsWeb;
import 'package:flutter/widgets.dart';
import 'package:window_manager/window_manager.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  // ... logging, error handlers, storage, database (section 4.5) ...
  final config = await ConfigService().loadAndVerify();
  final lang = savedOrSystemLanguageCode(); // the resolved app language (8.4)

  if (!kIsWeb &&
      (Platform.isWindows || Platform.isLinux || Platform.isMacOS)) {
    await windowManager.ensureInitialized();
    await windowManager.setMinimumSize(const Size(800, 600));
    // The title is the localized app name (guideline.md §1.8), never a literal.
    await windowManager.setTitle(
      config.appName.resolve(lang) + AppFlavorConfig.instance.nameSuffix,
    );
  }

  runApp(const MyApp());
}
```

When the user changes the language (8.4), set the window title again from the new language.

**Keyboard shortcuts** — register app-wide shortcuts using `Shortcuts` and `Actions` widgets at
the root. Use the platform's modifier key (Cmd on macOS, Ctrl on Windows and Linux). Document all
registered shortcuts in `docs/architecture.md`.

**Context menus** — for right-click menus on desktop, use `MenuAnchor` (opened from
`GestureDetector.onSecondaryTapDown` at the pointer position) or `showMenu`. For text fields,
use the built-in `contextMenuBuilder`. Do not assume touch-only interaction patterns.

**Window resizing** — every screen MUST stay usable from the minimum window size up to a full
4K screen. Use adaptive layouts (`LayoutBuilder`, breakpoints) instead of phone-only layouts.

#### 5.5.1 Windows

**App metadata.** Set the company, product and copyright strings in `windows/runner/Runner.rc`
(`CompanyName`, `FileDescription`, `ProductName`, `LegalCopyright`) and replace
`windows/runner/resources/app_icon.ico` with the real icon (17.5). The file version comes from
`pubspec.yaml`.

**MSIX packaging** (the `msix` package) is the default for Windows distribution:

```yaml
# pubspec.yaml
msix_config:
  display_name: MyApp
  publisher_display_name: <Publisher display name>
  identity_name: <from Partner Center, or your reverse-DNS id for sideloading>
  publisher: <CN=... from Partner Center, or from your code-signing certificate>
  msix_version: 1.0.0.0          # four parts; see "MSIX version" below
  logo_path: assets/icons/app_icon.png
  capabilities: internetClient   # least privilege — list only what the app uses
  languages: en-us               # one entry per declared language
  # Microsoft Store: set store: true — the Store signs the package, do not sign it yourself.
  # store: true
  # Direct download: sign with a trusted code-signing certificate kept OUTSIDE the repo,
  # passed in at local release build time (certificate_path / certificate_password), never committed.
```

Build command:

```bash
# `flutter pub run` is deprecated; use `dart run`.
dart run msix:create
```

> **`msix:create` builds the app again unless told not to.** By default (`build_windows: true`)
> it runs its own `flutter build windows`, **without** `--obfuscate`, `--split-debug-info` or
> the flavor. A release package MUST therefore either:
>
> - set `build_windows: false` in `msix_config` and run `msix:create` right after the hardened
>   `flutter build windows` command (it then packages that build), **or**
> - pass the full hardened arguments in `windows_build_args`.
>
> `release_process.md` §11 uses the first way.

- **MSIX version**: map `pubspec.yaml` `X.Y.Z+N` to `msix_version: X.Y.Z.0`. The Microsoft
  Store needs the last (revision) part to be `0`, so every Store release MUST raise `X.Y.Z` —
  a change of the build number `+N` alone cannot be uploaded. A direct-download MSIX MAY use
  `X.Y.Z.N` when `N` is 65535 or less. Check the current Partner Center rule before each release.
- **Architecture**: one MSIX per architecture (`architecture: x64` or `arm64`). Flutter builds
  Windows arm64 only on an arm64 Windows machine. Ship arm64 only if it is built and tested
  there.
- **Microsoft Store**: build with `store: true`, with `identity_name`, `publisher` and
  `publisher_display_name` copied exactly from Partner Center → Product identity.
- **Direct download**: the MSIX (or an installer such as Inno Setup / WiX wrapping the release
  folder) MUST be signed with a code-signing certificate trusted on the target machines.
  Unsigned packages are blocked or warned about by SmartScreen and cannot be installed as MSIX.
- A raw unsigned `build/windows/x64/runner/Release/` folder is acceptable for internal tools only.

#### 5.5.2 macOS

**Identity.** Set `PRODUCT_NAME`, `PRODUCT_BUNDLE_IDENTIFIER` and `PRODUCT_COPYRIGHT` in
`macos/Runner/Configs/AppInfo.xcconfig`. The bundle id SHOULD match the iOS one when both
platforms are declared. Set the deployment target in `macos/Podfile` (`platform :osx, '<min>'`)
and in Xcode, at or above the Flutter minimum (see 5.4).

**Entitlements and App Sandbox.** macOS apps run in the **App Sandbox** (required for the Mac App
Store, strongly recommended for Developer ID). Flutter creates two files:

| File | Used for |
|---|---|
| `macos/Runner/DebugProfile.entitlements` | debug and profile builds |
| `macos/Runner/Release.entitlements` | release builds — this is what ships |

Add **only** the entitlements the app needs, to **both** files, for example:

```xml
<key>com.apple.security.app-sandbox</key>
<true/>
<!-- Outgoing network requests (HTTP, APIs). Missing this = every request fails in release. -->
<key>com.apple.security.network.client</key>
<true/>
<!-- Files the user picks in an open/save panel (file_picker, file_selector). -->
<key>com.apple.security.files.user-selected.read-write</key>
<true/>
```

A missing entitlement often works in debug and fails only in release. Test the release build.

**Signing.** In Xcode → Runner → Signing & Capabilities, select the team and enable
**Hardened Runtime** for Release. Distribution needs either:

- **Mac App Store** — an "Apple Distribution" certificate and a Mac App Store provisioning profile;
  uploaded to App Store Connect (see `platform_store_readiness.md`).
- **Direct download (Developer ID)** — a "Developer ID Application" certificate, then
  **notarization** with `xcrun notarytool` and **stapling** with `xcrun stapler`. An un-notarized
  app is blocked by Gatekeeper on users' Macs.

**Flavors.** If a flavor needs its own bundle id or app name, add an Xcode scheme and xcconfig
per flavor, the same way as iOS (5.4), and pass `--flavor <name>`. If dev and prod differ only
in Dart behavior and there are no per-flavor schemes, pass `--dart-define=APP_FLAVOR=<name>`
instead (5.2). Either way, the two-variable `AppFlavorConfig` keeps working.

**CPU architectures.** A universal binary (arm64 + x86_64) still runs on Intel Macs. Flutter is
reducing Intel Mac support: check `flutter build macos` for warnings about Intel hosts or
universal builds, and the release notes for the current state. Shipping arm64 only is a
documented product decision (record it in `docs/PROJECT_PROFILE.md`); when the toolchain
supports it, `flutter config --enable-macos-arm64-only` builds arm64 only. Recent Xcode
versions run only on Apple silicon build machines.

**Secure storage** uses the Keychain (e.g. `flutter_secure_storage`); follow the plugin's macOS
setup, which may require the Keychain Sharing capability.

#### 5.5.3 Linux

**Build machine.** Linux builds need the GTK toolchain. Install the packages listed in the
current Flutter Linux setup page (`docs.flutter.dev/platform-integration/linux/setup`). When
last checked, on Debian/Ubuntu that was:

```bash
sudo apt-get install clang cmake ninja-build pkg-config libgtk-3-dev liblzma-dev libstdc++-12-dev
```

Build release binaries on the **oldest distribution you support** (or inside its container), so
the binary does not require a newer `glibc` than users have.

**Identity.** Set `BINARY_NAME` and `APPLICATION_ID` in `linux/CMakeLists.txt`. `APPLICATION_ID`
MUST be the reverse-DNS app id; desktop files, icons, Snap and Flatpak metadata all use it.

```cmake
set(BINARY_NAME "my_app")
set(APPLICATION_ID "com.example.my_app")
```

**Desktop integration.** Ship a `<APPLICATION_ID>.desktop` file and icons (at least 256×256 PNG,
plus SVG if available) so the app appears in the launcher with the right name and icon. Flatpak
and Flathub also require an AppStream `<APPLICATION_ID>.metainfo.xml` file.

**Packaging** — pick per declared channel (details in `platform_store_readiness.md`):

| Channel | Format |
|---|---|
| Snap Store | `.snap`, built from `snap/snapcraft.yaml` (Flutter plugin) |
| Flathub | Flatpak, built from a manifest in the Flathub repo |
| Direct download | AppImage, `.deb`, and/or `.rpm` |

**Secure storage** uses the Secret Service (`libsecret`, e.g. `flutter_secure_storage`). Add
`libsecret-1-dev` to the build machine and declare the runtime dependency in the package.

### 5.6 Artifact Selection

Under `Production App Extension`, for each declared platform:

| Platform | Store artifact | Direct-download artifact |
|---|---|---|
| Android | `.aab` (Google Play) | split APKs (`--split-per-abi`) |
| iOS | `.ipa` via `flutter build ipa` → App Store Connect | — (Ad Hoc / Enterprise only) |
| Windows | `.msix` with `store: true`, one per shipped architecture (Microsoft Store) | signed `.msix` or signed installer `.exe` |
| macOS | signed `.app` → `.pkg` uploaded to App Store Connect (Mac App Store) | Developer ID signed, notarized and stapled `.dmg` (or `.zip`) |
| Linux | `.snap` (Snap Store), Flatpak (Flathub) | AppImage, `.deb`, `.rpm` |

- Avoid universal release APKs unless there is a specific distribution reason.
- `--target-platform` does not replace `--split-per-abi`.
- Raw, unsigned build folders are acceptable for internal tools only.

---

## 6. UI And UX Baseline

This section applies to user-facing applications. It is advisory for infrastructure packages.

### 6.1 Theme And Design Tokens

- Use one source of truth for theme configuration.
- Centralize colors, typography, spacing, radius, and motion values.
- Prefer semantic names over raw literals in widgets.
- If the product supports both light and dark themes, both MUST be tested.
- Design token constants SHOULD live in a single reviewed file such as `lib/theme/tokens.dart`
  (Tier 1) or `lib/app/theme/tokens.dart` (Tier 2).
- Never hardcode `Color(0xFF...)` literals inside widget `build` methods; always reference a token.

#### Material 3 Is The Default

Material 3 is the default `ThemeData`; the explicit `useMaterial3: true` flag is not needed.
`useMaterial3: false` and Material 2 are on Flutter's deprecation path — do not introduce new
code that depends on M2 visuals.

#### Material And Cupertino Packages (`material_ui` / `cupertino_ui`)

Material and Cupertino ship as standalone `pub.dev` packages, `material_ui` and
`cupertino_ui`. The copies inside the SDK (`package:flutter/material.dart`,
`package:flutter/cupertino.dart`) are frozen and on Flutter's deprecation path. Upstream,
moving to the packages is opt-in for now; **this standard makes it a MUST** so every app is on
the maintained code. A project on a Flutter version that has no `material_ui` package keeps the
SDK imports until it upgrades (5.3, "Known migrations").

- Every app MUST depend on `material_ui` (and `cupertino_ui` if it uses any Cupertino
  widget, or installs the fallback localization delegates of 8.3.1), with a caret (`^`)
  constraint on the current version from pub.dev:

  ```yaml
  dependencies:
    material_ui: ^<current>     # current version from pub.dev at project start
    cupertino_ui: ^<current>    # only if Cupertino widgets or delegates are used
  ```

- New code MUST import `package:material_ui/material_ui.dart` /
  `package:cupertino_ui/cupertino_ui.dart`, never `package:flutter/material.dart` or
  `package:flutter/cupertino.dart`. `package:flutter/widgets.dart`,
  `package:flutter/services.dart` and `package:flutter/foundation.dart` are unchanged.
- Migrate existing code with `dart fix --apply --code=migrate_design_widgets`. The `--code`
  part is required; plain `dart fix --apply` does not run this migration. If the tool adds
  `material_ui: any`, replace it with a caret (`^`) constraint.
- **Dependencies that still use the SDK libraries.** While a package the app uses still
  imports `package:flutter/material.dart`, wrap the app with `MaterialUiCompatibilityBridge`
  in `MaterialApp.builder` (from `material_ui`), so those widgets still find the theme and
  localizations. The bridge cannot fix API type mismatches: an SDK `ColorScheme`, `TextTheme`
  and similar types are different types from the `material_ui` ones. A package whose public
  API takes such types must be updated before the app can pass `material_ui` values to it.
  Record such packages in `docs/architecture.md` §21.
- Code samples in these guidelines that omit imports assume the `material_ui` import.

### 6.2 Widget Structure

- Screens compose flows and sections.
- Reusable widgets belong in shared widget locations only when they are actually shared.
- Widgets MUST NOT own persistence, cryptography, or network behavior.
- Large `build` methods SHOULD be split when readability drops.

### 6.3 Screen-State Guidance

Async and task-oriented screens SHOULD define the states they genuinely need:

1. Loading
2. Empty
3. Success
4. Error

Not every static screen needs all four states. Apply this rule where asynchronous data or user
actions make those states meaningful.

**Loading state pattern:**

- Use a skeleton / shimmer loader for screens that display a list or content-heavy layout. This
  reduces perceived wait time and avoids layout shift.
- Use a centered `CircularProgressIndicator` only for short-lived action feedback (form submission,
  save, delete).
- Never block the entire screen with a spinner for initial data loads that have skeleton
  alternatives available.

### 6.4 User Feedback Components

Use the correct feedback component for each context. Do not substitute freely between them.

| Context | Component | Rationale |
|---------|-----------|-----------|
| Non-blocking operation result (save, copy, undo) | `SnackBar` | Dismissable, low interruption |
| Destructive action confirmation | `AlertDialog` | Requires explicit user decision |
| Contextual detail or secondary flow | `BottomSheet` (modal) | Preserves navigation context |
| Persistent form field error | Inline validation text | Closest to the cause |
| Critical system error requiring action | `AlertDialog` | Cannot be dismissed accidentally |
| Transient ambient status | `Banner` or custom overlay | Does not interrupt flow |

Rules:

- `SnackBar` MUST provide an action when the operation is undoable.
- `AlertDialog` for destructive actions MUST use a clearly destructive label on the confirm button
  (e.g. "Delete", not "OK").
- Do not stack multiple modals. Dismiss the current one before presenting another.

### 6.5 Animation Guidelines

Use the Material motion system as the default baseline.

> **Note on token names.** The five-bucket scheme below (extra small / small / medium / large
> / extra large) is a deliberate simplification of Material 3's official motion-duration tokens
> (`short1…short4`, `medium1…medium4`, `long1…long4`, `extra-long1…extra-long4`). The
> simpler scheme is easier to keep in mind across a small team and maps cleanly into the
> richer M3 set when needed. Document which scheme your project uses in
> `architecture.md §16` and stay consistent.

**Duration tokens:**

| Category | Duration | Use |
|----------|----------|-----|
| Extra small | 50 ms | Micro-interactions, checkbox toggle |
| Small | 100 ms | Icon swap, fab expand |
| Medium | 200 ms | Card expand, bottom sheet partial |
| Large | 300 ms | Screen transition, modal open |
| Extra large | 500 ms | Hero or shared-element transition |

**Easing curves:**

- Use `Curves.easeInOut` for elements that stay within the screen bounds.
- Use `Curves.easeOut` for elements entering the screen.
- Use `Curves.easeIn` for elements leaving the screen.
- Use `Curves.fastOutSlowIn` (Material standard) as the default transition curve.

Rules:

- Animations MUST respect `MediaQuery.disableAnimationsOf(context)`. If `true`, skip or
  complete animations instantly.
- Looping animations MUST be paused when the app is in the background (`AppLifecycleState.paused`).
- Prefer animating transform and opacity over properties that trigger layout recalculation
  (such as `Padding`, `SizedBox` dimensions, or `Align` factors). Use the transition widgets
  driven by an `Animation` — `FadeTransition`, `ScaleTransition`, `SlideTransition`,
  `RotationTransition` — or `AnimatedOpacity`. They update the compositing layer without a
  rebuild or relayout. Do **not** animate by rebuilding a plain `Opacity` or `Transform`
  widget with `setState` on every frame: that rebuilds and repaints the child each frame.
  Animating layout-affecting properties via `AnimatedPadding`, `AnimatedContainer`, or
  `TweenAnimationBuilder` is a legitimate Flutter pattern and is not prohibited — it requires
  profiling evidence before shipping to confirm the frame budget is met on a mid-range device.
- Use `AnimationController.dispose()` — always dispose controllers in `State.dispose()`.

### 6.6 Haptic Feedback

- Use `HapticFeedback.lightImpact()` for non-destructive confirmations (item selection, toggle).
- Use `HapticFeedback.mediumImpact()` for significant actions (task complete, bookmark added).
- Use `HapticFeedback.heavyImpact()` for destructive or irreversible actions (delete confirmed).
- Use `HapticFeedback.selectionClick()` for navigating through discrete options (picker scroll).
- MUST NOT use haptics for every tap. Reserve for meaningful state changes only.
- Haptic calls SHOULD be wrapped in a platform check; they are no-ops on platforms that do not
  support them but still good practice to guard explicitly.

### 6.7 Keyboard And Scroll Behavior

- All screens with text input MUST be wrapped in a `SingleChildScrollView` or equivalent
  scrollable if the content can overflow when the keyboard is raised.
- Set `resizeToAvoidBottomInset: true` on `Scaffold` (this is the default; do not set it to
  `false` unless there is a documented layout reason).
- Use `keyboardDismissBehavior: ScrollViewKeyboardDismissBehavior.onDrag` on scrollable lists
  to allow keyboard dismissal by dragging.
- The focused field MUST remain visible when the keyboard is raised. Use
  `ScrollController.animateTo` or `Scrollable.ensureVisible` if automatic scroll is insufficient.
- Prefer `TextInputAction.next` for multi-field forms and advance focus programmatically via
  `FocusScope.of(context).nextFocus()`.
- `TextInputAction.done` MUST close the keyboard and trigger form submission or save.

### 6.8 Safe Area And Display Cutout Handling

- Every top-level `Scaffold` MUST be aware of safe areas. The `Scaffold` widget handles this for
  `appBar` and `bottomNavigationBar`. For custom full-screen layouts, wrap content with `SafeArea`.
- Do not hardcode top or bottom padding. Read insets from `MediaQuery.paddingOf(context)` /
  `MediaQuery.viewPaddingOf(context)`, or use `SafeArea`.
- Test layouts on a device or emulator with a display notch, punch-hole camera, and gesture
  navigation bar. These three configurations expose the most common safe-area bugs.
- **Android edge-to-edge.** Apps that target API 35 or higher draw edge-to-edge on Android 15
  and later. The temporary opt-out attribute stops working once the app targets API 36 on
  Android 16 and later, so treat edge-to-edge as permanent: never rely on the system bars
  padding the app. Flutter reports the system-bar and cutout insets through `MediaQuery`, so
  `SafeArea` and the `Scaffold` handle them. Test with three-button navigation, gesture
  navigation, and a tablet with a display cutout.

### 6.9 UX Rules For Production Apps

Under `Production App Extension`:

- Forms MUST validate before submission and show actionable errors.
- Destructive actions MUST require confirmation or provide undo.
- Layouts SHOULD work on common phone sizes (360 dp to 430 dp width) before release.
- Layouts SHOULD also be tested at 600 dp width (small tablet) if the app targets tablets.
- Route definitions SHOULD be centralized.
- Deep links, if supported, SHOULD have integration coverage.
- Tap-target sizes: see 7.1 (they apply to every app, not only production apps).

---

## 7. Accessibility Standard

Accessibility is a correctness requirement. These rules apply under `Core Baseline` for all
user-facing features.

### 7.1 Touch Target Sizes

- Minimum interactive target size: **48 × 48 dp** on mobile (Material and platform requirement).
- Minimum interactive target size: **32 × 32 dp** on desktop.
- If the visual widget is smaller than the minimum target, wrap it in a `SizedBox` or use
  `Padding` to expand the hit area without changing the visual appearance.
- Use `debugPaintPointersEnabled = true` in development to verify actual hit areas.

### 7.2 Color Contrast

- Normal text: minimum contrast ratio **4.5 : 1** (WCAG AA).
- Large text: minimum contrast ratio **3.0 : 1** (WCAG AA). WCAG defines large text in points:
  18 pt, or 14 pt bold. In Flutter logical pixels (sp / dp) that is about **24 sp**, or about
  **18.7 sp bold**. Text smaller than that is normal text.
- Interactive component boundaries and focus indicators: minimum **3.0 : 1** against adjacent colors.
- Never communicate information using color alone. Always pair color with a label, icon, or
  pattern.
- Both light and dark themes MUST independently pass contrast requirements.

### 7.3 Semantics

- Use `Semantics` widgets to provide labels for any custom widget that assistive technology cannot
  infer from its visual content.
- Hide decorative images and icons that carry no meaning from assistive technology: use
  `Image(excludeFromSemantics: true)` or wrap them in `ExcludeSemantics`. An `Icon` with no
  `semanticLabel` is already silent.
- Custom interactive widgets (gestures, custom painters, canvas) MUST provide `onTap`, `label`,
  and `hint` semantics.
- A `Tooltip`'s message is always part of the semantics tree (screen readers announce it), not
  only on long-press. Every icon-only control MUST have one — see 7.8, which is a hard
  requirement, not a suggestion.
- Use `MergeSemantics` when multiple widgets form a single logical unit (e.g. a list tile with an
  icon and a label).
- Do not suppress semantics on content that communicates state (loading spinners, error badges).

### 7.4 Font And Text Scaling

> **API note.** `textScaleFactor` is deprecated (it may be removed in your Flutter version
> already). Use `TextScaler` instead. The new API supports Android 14's
> non-linear font scaling, which the old single-double API cannot represent.
>
> | Old (deprecated) | New |
> |------------------|-----|
> | `MediaQuery.of(context).textScaleFactor` | `MediaQuery.textScalerOf(context)` |
> | `MediaQuery(data: data.copyWith(textScaleFactor: 1.5), ...)` | `MediaQuery(data: data.copyWith(textScaler: TextScaler.linear(1.5)), ...)` |
> | `Text('x', textScaleFactor: 1.2)` | `Text('x', textScaler: TextScaler.linear(1.2))` |
> | manual clamping logic | `MediaQuery.withClampedTextScaling(minScaleFactor: 1.0, maxScaleFactor: 2.0, child: ...)` |
> | disable scaling | `MediaQuery.withNoTextScaling(child: ...)` |

- All text layouts MUST remain functional and readable at scaler values of `1.0`, `1.5`, and
  `2.0`. Test these values in the Flutter inspector before release. Use
  `TextScaler.linear(1.5)` etc. when writing tests.
- Do not hardcode pixel heights for containers that hold text. Use `IntrinsicHeight`,
  `FittedBox`, or `Flexible` to let text expand.
- `maxLines` clipping SHOULD be accompanied by `overflow: TextOverflow.ellipsis` and the full
  text available via a tap action or tooltip.
- Globally disabling text scaling MUST NOT be done without a documented, user-controlled reason
  (e.g. a font-size setting in the app itself). If a subtree must be exempt, wrap it with
  `MediaQuery.withNoTextScaling` rather than passing `textScaler: TextScaler.noScaling`
  to every `Text`.

### 7.5 Focus And Keyboard Navigation

- All interactive widgets MUST be reachable and activatable via keyboard Tab and Enter on
  platforms that support a physical keyboard (desktop, tablets with keyboard).
- Use `FocusTraversalGroup` to define logical traversal boundaries within complex screens.
- Focus order SHOULD match the visual reading order (top-left to bottom-right for LTR).
- Modal dialogs and bottom sheets MUST trap focus inside themselves until dismissed.
- Use `FocusNode.requestFocus()` to move focus programmatically when a screen or dialog opens.

### 7.6 Screen Reader Testing

Under `Production App Extension`:

- Before each release, test critical flows with the screen reader of **each declared platform**:
  **TalkBack** (Android), **VoiceOver** (iOS, macOS), **Narrator** (Windows), **Orca** (Linux).
- At minimum: app navigation, primary data entry flow, and error states must be fully operable
  with each of them.

### 7.7 Compliance Verification Methods

The following named methods are the expected ways to prove accessibility compliance before
shipping. "We checked" is not sufficient — the method used should be recorded in the release
checklist.

**Touch target verification:**
Enable `debugPaintPointersEnabled = true` in a debug build, tap the interactive elements, and
confirm their hit areas meet the 48 × 48 dp minimum on mobile (`debugPaintSizeEnabled` shows
layout bounds, not hit areas). Alternatively, enable Flutter DevTools → Widget Details
and inspect `Size` values for interactive widgets.

**Contrast verification:**
Use a contrast checker such as the
[WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/), or compute the ratio in
a test, to verify foreground/background pairs. (Material's
[color system overview](https://m3.material.io/styles/color/system/how-the-system-works) is
background reading on how its color roles are built; it is not a checker.) Both light
and dark themes must be checked independently. Record the checked pairs and their ratios.

**Semantics verification:**
Enable `SemanticsDebugger` by wrapping the root widget temporarily:
```dart
// In main() for a verification build:
runApp(SemanticsDebugger(child: MyApp()));
```
This renders the semantics tree visually over the UI. Confirm every interactive element has a
readable label and that decorative elements are excluded. Remove before committing.

**Font scaling verification:**
In the Flutter inspector, use the Text Scale slider to verify layouts at 1.0×, 1.5×, and 2.0×.
For widget tests, wrap the widget under test with
`MediaQuery(data: const MediaQueryData(textScaler: TextScaler.linear(1.5)), child: ...)`
(or `2.0`) and assert no overflow or clipping. No text should be clipped or overflow its
container at any of these values.

### 7.8 Tooltips On Icon-Only Controls (Mandatory)

**Every button whose only visible content is an icon MUST have a tooltip.** An icon without a
label is a guess for a sighted user and silence for a screen reader; the tooltip fixes both at
once, because Flutter feeds tooltip text into the semantics tree.

This applies to:

| Widget | How the tooltip is supplied |
|---|---|
| `IconButton` | `tooltip:` parameter |
| `FloatingActionButton` / `.small` / `.large` | `tooltip:` parameter |
| `PopupMenuButton` | `tooltip:` parameter (and each item carries its own visible label) |
| `DropdownButton` with an icon-only child | wrap in `Tooltip` |
| Icon-only `InkWell`, `GestureDetector`, a `Slider` with only icons around it, custom painters | wrap in `Tooltip(message: ...)` **and** provide `Semantics(label:, hint:)` |
| `BottomNavigationBarItem`, `NavigationDestination` shown **without** a persistent visible label | `tooltip:` on the destination |
| `NavigationRailDestination` shown **without** a visible label (`labelType: none`) | It has no `tooltip:` parameter. Its required `label` is the accessible name; also wrap the destination's `icon` in a `Tooltip` with the same localized text. |
| `AppBar` actions, `SliverAppBar` actions, toolbar overflow buttons | `tooltip:` on each action |
| `Chip` / `ListTile` trailing icon buttons | `tooltip:` on the button |

Rules:

- The tooltip text MUST come from ARB via `AppLocalizations` — never a raw literal — and is
  therefore rendered in the user's chosen language like every other string (section 8).
- The tooltip names **the action, not the icon**: "Delete note", not "Trash icon".
- Tooltip text follows the short-label budget in 8.6 (it is UI chrome, not prose).
- A control that already shows a persistent text label next to its icon does not need a tooltip;
  adding one is allowed but MUST NOT repeat the label verbatim.
- Never use a tooltip as the only way to convey information required to operate the app — it is
  a hint, not content.
- Destructive actions SHOULD still confirm; a tooltip is not a confirmation.

```dart
IconButton(
  icon: const Icon(Icons.delete_outline),
  tooltip: AppLocalizations.of(context).tooltipDelete, // short, localized
  onPressed: _delete,
)

// Custom icon-only control: tooltip + explicit semantics.
Tooltip(
  message: l10n.tooltipShare,
  child: Semantics(
    button: true,
    label: l10n.tooltipShare,
    child: InkWell(onTap: _share, child: const Icon(Icons.share)),
  ),
)
```

**Verification.** Add this widget test to any screen with icon buttons; it fails the build when a
tooltip is missing:

```dart
void expectAllIconButtonsHaveTooltips(WidgetTester tester) {
  final buttons = tester.widgetList<IconButton>(find.byType(IconButton));
  for (final button in buttons) {
    expect(
      button.tooltip != null && button.tooltip!.trim().isNotEmpty,
      isTrue,
      reason: 'IconButton with icon ${button.icon} has no tooltip',
    );
  }
  final fabs = tester.widgetList<FloatingActionButton>(
    find.byType(FloatingActionButton),
  );
  for (final fab in fabs) {
    expect(fab.tooltip?.trim().isNotEmpty ?? false, isTrue,
        reason: 'FloatingActionButton has no tooltip');
  }
}
```

Run it for every screen under test, in every declared locale, as part of the screen's widget test.

---

## 8. Localization And Internationalization

This section is `Core Baseline` and applies to every user-facing app repository.

**Every app externalizes its strings and ships the languages it declares.** Each project lists
its languages in `docs/PROJECT_PROFILE.md` (the *declared languages*): one **template language**
(usually English, `en`) plus zero or more other languages. A single-language app is allowed, but
its strings still live in ARB files from day one so adding a language later is a small job.

When an app declares two or more languages, the app starts in the system language when that is
one of the declared languages and in the template language otherwise, and the user can change the
language inside the app at any time. Every feature, every screen, and every string — labels,
menus, buttons, tooltips, dialogs, notifications, errors, empty states, About content — renders in
the language the user selected.

**Language packs.** Some languages need extra rules (script, grammar, glossary, framework gaps).
These live in `language_packs/<name>.md` and apply only when the project declares that language.
Available packs:

| Pack | Languages |
|---|---|
| `language_packs/sanskrit_malayalam.md` | Sanskrit (`sa`), Malayalam (`ml`) |

| Sub-section | Rule |
|---|---|
| 8.1 | Minimum `MaterialApp` / `l10n.yaml` setup |
| 8.2 | String externalization into ARB — no user-visible literals |
| 8.3 | Declared languages, framework-translation gaps, fonts |
| 8.4 | In-app language selection, persistence, resolution order |
| 8.5 | Translation quality and language packs |
| 8.6 | Short UI labels vs. descriptive text |
| 8.7 | Per-feature language completeness and the parity test |
| 8.8 | RTL layout support |
| 8.9 | Locale-sensitive formatting |

### 8.1 Minimum Setup (All Apps)

Every Flutter app MUST declare the framework localization delegates and `supportedLocales` in the
root `MaterialApp` or `CupertinoApp`. Without this, some Material widgets (date pickers, number
inputs, dialog buttons) render incorrectly on devices with non-English system locales.

```dart
import 'package:material_ui/material_ui.dart';

MaterialApp(
  // GlobalMaterialLocalizations comes from material_ui; its `.delegates`
  // list already includes the Cupertino and Widgets delegates.
  // Do NOT use the generated AppLocalizations.localizationsDelegates list: it
  // names the SDK's flutter_localizations Material/Cupertino delegates.
  localizationsDelegates: [
    AppLocalizations.delegate,
    ...GlobalMaterialLocalizations.delegates,
  ],
  // Exactly the declared languages from docs/PROJECT_PROFILE.md, template first.
  // The generated AppLocalizations.supportedLocales list is the easiest source.
  supportedLocales: AppLocalizations.supportedLocales,
);
```

`supportedLocales` MUST equal the declared languages — no more, no fewer. See 8.3.1 for languages
that Flutter has no framework translation for, and 8.4 for the in-app switcher.

Add to `pubspec.yaml`:

```yaml
dependencies:
  flutter_localizations:
    sdk: flutter
  material_ui: ^<current>   # Material widgets and GlobalMaterialLocalizations (6.1)
  cupertino_ui: ^<current>  # only if Cupertino widgets are used or 8.3.1 fallback delegates exist
  intl: ^<current>          # take each <current> from pub.dev; never type a version from memory
```

`flutter_localizations` is still required: the generated `AppLocalizations` file imports it,
and the fallback delegate in 8.3.1 uses its `GlobalWidgetsLocalizations`. Only the Material
and Cupertino delegates moved into `material_ui` / `cupertino_ui`.

And in `pubspec.yaml` under `flutter:`:

```yaml
flutter:
  generate: true
```

Create `l10n.yaml` at the project root. `flutter gen-l10n` reads this to find your ARB files
and produce typed accessors. Without it the generator falls back to defaults that may not
match the directory layout above:

```yaml
# l10n.yaml — at project root, alongside pubspec.yaml
arb-dir: lib/l10n
template-arb-file: app_en.arb   # app_<template language>.arb
output-localization-file: app_localizations.dart
output-class: AppLocalizations
nullable-getter: false
```

The generated files go into the source tree (next to the ARB files, or into `output-dir` if you
set one). Import `AppLocalizations` from `lib/l10n/` (for example
`package:<your_app>/l10n/app_localizations.dart`). Do not use the old synthetic
`package:flutter_gen` import and do not add a `synthetic-package` line; current Flutter no
longer supports them.

**Android App Bundle language splitting MUST be disabled** when the app has an in-app language
picker (two or more declared languages), in `android/app/build.gradle.kts` (or `build.gradle`):

```kotlin
// android/app/build.gradle.kts
android {
    bundle {
        language {
            enableSplit = false
        }
    }
}
```

Google Play defaults to language splitting when delivering Android App Bundles (.aab). If a user
downloads the app on an English phone, Play only installs English resources. If the user later
switches to another language inside the app, the platform strings for it are missing. Setting
`enableSplit = false` keeps all language resources on the device.

**iOS and macOS** list the declared languages in `CFBundleLocalizations` in `ios/Runner/Info.plist`
and `macos/Runner/Info.plist`, so the App Store shows the right languages and the system language
reaches Flutter:

```xml
<key>CFBundleLocalizations</key>
<array>
  <string>en</string>
  <!-- one <string> per declared language -->
</array>
```

### 8.2 String Externalization (Mandatory, All Apps)

Every app MUST externalize its user-visible strings into ARB files. This is not optional and does
not wait for a translation request.

ARB files are the Flutter equivalent of Android's `res/values/strings.xml`. On Android we create
`strings.xml` from day one, so a new language is just a new `values-xx/strings.xml`. We follow the
same habit in Flutter.

Required for every app:

- `l10n.yaml` MUST exist at the project root (see 8.1).
- One ARB file MUST exist per declared language: `lib/l10n/app_<code>.arb`, with the template
  language's file as the template.
- Every user-visible string MUST be defined in the ARB files and read through
  `AppLocalizations.of(context)`. A raw string literal in a widget is not allowed.
- Every key MUST exist in **every** declared language's file with a real translation. A
  template-language value copied into another file as a placeholder is an unfinished feature, not
  a translation (8.7).
- Every ARB entry MUST have an `@key` description in the template file, so a translator has context.
  Where a key is UI chrome rather than prose, say so in the description (it drives the length budget
  in 8.6), e.g. `"description": "Toolbar button label. Keep to one or two words."`.

**Narrow exceptions** — these MAY stay as plain Dart literals, because a user never reads them:

| Allowed as a literal | Example |
|---|---|
| Log and debug messages | `AppLogger.debug('cache miss for $id')` |
| Exception messages not shown in the UI | `throw StateError('db not initialized')` |
| Technical identifiers | asset paths, route names, map/JSON keys, `Semantics` test tags |
| Developer-only screens | a debug menu that never ships to users |

**One second source of user-visible text.** The About-screen values (`appName`, `description`
and the `details` rows) come from the locale maps in `assets/config/app_config.json`
(`guideline.md` §1), not from ARB. They still carry every declared language (8.7), and their
row labels still come from ARB (`aboutDetail<Key>`). No other screen text may live outside ARB.

Anything a real user reads — screen titles, buttons, labels, hints, error text shown on screen,
empty states, snackbars, dialogs, notification text — goes in the ARB file.

Directory structure (example: English template plus two more declared languages):

```text
lib/
`-- l10n/
    |-- app_en.arb   # REQUIRED — template language
    |-- app_es.arb   # one file per declared language
    `-- app_hi.arb
```

Example ARB file:

```json
{
  "@@locale": "en",
  "titleHome": "Home",
  "@titleHome": { "description": "Home screen app-bar title. Keep to one or two words." },
  "welcomeMessage": "Welcome, {name}",
  "@welcomeMessage": {
    "description": "Greeting shown on the home screen",
    "placeholders": {
      "name": { "type": "String" }
    }
  }
}
```

Generate typed accessors:

```bash
flutter gen-l10n
```

Use in code via `AppLocalizations.of(context).titleHome` (no `!`, because `l10n.yaml` sets
`nullable-getter: false`). Never use raw string literals for user-visible text.

**Adding a language later.** Because the strings are already externalized, this is a small,
mechanical job:

1. Add the language to the languages row of `docs/PROJECT_PROFILE.md`.
2. Add `lib/l10n/app_<code>.arb` with the same keys and translated values.
3. Add it to the in-app language picker (8.4), `CFBundleLocalizations` (8.1), and the parity test
   list (8.7). If a language pack exists for it, apply the pack.
4. Run `flutter gen-l10n`.

No screen or widget code changes.

### 8.3 Declared Languages

The project's languages are recorded once, in `docs/PROJECT_PROFILE.md`, as a table like:

| Locale | Language | Script | Role |
|---|---|---|---|
| `en` | English | Latin | Template ARB, ultimate fallback |
| `<code>` | `<language>` | `<script>` | Full UI translation |

All declared languages MUST be listed in `supportedLocales`, all their ARB files MUST be complete,
and — when there are two or more — the in-app picker (8.4) MUST offer all of them plus
"System default".

#### 8.3.1 Languages without a Flutter framework translation — install a fallback delegate

The framework localizations (`GlobalMaterialLocalizations` from `material_ui`,
`GlobalCupertinoLocalizations` from `cupertino_ui`, `GlobalWidgetsLocalizations` from
`flutter_localizations`) cover a long list of locales, **but not every language** (for example,
Sanskrit `sa` is missing). Adding such a locale to `supportedLocales` without handling this throws
at runtime the first time a Material widget needs framework strings (date picker, dialog buttons,
text-selection menu, `Scaffold` semantics labels).

For every declared language where `GlobalMaterialLocalizations.delegate.isSupported(locale)` is
false, the app MUST install a delegate that serves framework strings from the **template
language**, while the app's own strings (via `AppLocalizations`) stay in the selected language.
Never fall back to a different "related" language: users would see text in a language they did not
choose.

```dart
// lib/l10n/fallback_localizations.dart
// Keep the Cupertino fallback too: Material widgets on iOS / macOS can also read
// CupertinoLocalizations (for example the adaptive text-selection menu).
import 'package:cupertino_ui/cupertino_ui.dart';
import 'package:material_ui/material_ui.dart';
// Material and Cupertino delegates come from material_ui / cupertino_ui.
// Only the widgets-level delegate still lives in flutter_localizations.
import 'package:flutter_localizations/flutter_localizations.dart'
    show GlobalWidgetsLocalizations;

/// Declared languages that Flutter has no framework translation for.
/// Keep in sync with docs/PROJECT_PROFILE.md. Empty set = no fallback needed.
const Set<String> kFrameworkFallbackLanguages = {/* e.g. 'sa' */};

/// Language that framework strings fall back to (the template language).
const Locale kFrameworkFallbackLocale = Locale('en');

class FallbackMaterialLocalizationsDelegate
    extends LocalizationsDelegate<MaterialLocalizations> {
  const FallbackMaterialLocalizationsDelegate();

  @override
  bool isSupported(Locale locale) =>
      kFrameworkFallbackLanguages.contains(locale.languageCode);

  @override
  Future<MaterialLocalizations> load(Locale locale) =>
      GlobalMaterialLocalizations.delegate.load(kFrameworkFallbackLocale);

  @override
  bool shouldReload(covariant LocalizationsDelegate old) => false;
}

class FallbackCupertinoLocalizationsDelegate
    extends LocalizationsDelegate<CupertinoLocalizations> {
  const FallbackCupertinoLocalizationsDelegate();

  @override
  bool isSupported(Locale locale) =>
      kFrameworkFallbackLanguages.contains(locale.languageCode);

  @override
  Future<CupertinoLocalizations> load(Locale locale) =>
      GlobalCupertinoLocalizations.delegate.load(kFrameworkFallbackLocale);

  @override
  bool shouldReload(covariant LocalizationsDelegate old) => false;
}

class FallbackWidgetsLocalizationsDelegate
    extends LocalizationsDelegate<WidgetsLocalizations> {
  const FallbackWidgetsLocalizationsDelegate();

  @override
  bool isSupported(Locale locale) =>
      kFrameworkFallbackLanguages.contains(locale.languageCode);

  @override
  Future<WidgetsLocalizations> load(Locale locale) =>
      GlobalWidgetsLocalizations.delegate.load(kFrameworkFallbackLocale);

  @override
  bool shouldReload(covariant LocalizationsDelegate old) => false;
}
```

Register the fallback delegates **before** the global ones, so they win for those languages:

```dart
MaterialApp(
  localizationsDelegates: const [
    AppLocalizations.delegate,
    FallbackMaterialLocalizationsDelegate(),
    FallbackCupertinoLocalizationsDelegate(),
    FallbackWidgetsLocalizationsDelegate(),
    ...GlobalMaterialLocalizations.delegates, // Material + Cupertino + Widgets
  ],
  supportedLocales: AppLocalizations.supportedLocales,
);
```

A widget test MUST cover every fallback language: pump the app with that locale, open a date
picker and a dialog, and assert no exception.

#### 8.3.2 `intl` formatting for languages without CLDR data

`intl` has no date or number symbols for some languages, so for example `DateFormat.yMMMMd('sa')`
throws. Format with a supported locale while the surrounding UI text stays in the selected
language:

```dart
/// Locale to hand to `intl`. Languages with no CLDR data are formatted with
/// the template language's patterns while the UI text stays translated.
String formattingLocale(Locale locale) {
  // intl keys use underscores ('en_US'), not language tags ('en-US').
  final full = Intl.canonicalizedLocale(locale.toString());
  if (DateFormat.localeExists(full)) return full;
  if (DateFormat.localeExists(locale.languageCode)) return locale.languageCode;
  return 'en'; // the template language
}
```

`DateFormat.localeExists` only knows the locales whose date symbols are loaded. The Material
localization delegate loads them for the locales it supports; call `initializeDateFormatting()`
(`package:intl/date_symbol_data_local.dart`) at startup if you format dates before that.

Use `formattingLocale(...)` everywhere 8.9 calls for a locale argument.

#### 8.3.3 Fonts and script coverage

Non-Latin scripts (for example Devanagari, Malayalam, Tamil, Thai, Arabic, CJK) are not
guaranteed on every Android device, on Windows, or on Linux, and a missing glyph renders as a
blank box — a silent, ship-blocking bug.

- For every declared language whose script is not Latin, the app MUST either bundle fonts covering
  that script (e.g. the matching Noto Sans family) or declare an explicit `fontFamilyFallback`
  chain and verify rendering on a clean device image for each declared platform.
- Bundled fonts MUST be subset where possible and recorded under the font-licensing rules (17.4).
- Verification is per release: open every screen in every declared language on a clean device and
  confirm no boxes, no clipped ascenders/descenders (many scripts are taller than Latin), and no
  overflow. Do not hard-code text container heights (7.4).

### 8.4 In-App Language Selection (Mandatory When Two Or More Languages)

When an app declares two or more languages, the language is the user's choice, not the device's
alone. A single-language app skips this sub-section.

**Resolution order** — the app resolves its locale as:

1. the language the user saved inside the app, if any;
2. otherwise the system locale, when its language code is a declared language;
3. otherwise the template language.

Rules:

- The choice MUST persist across restarts (`SharedPreferences` key `app_language`, values
  `system` | `<declared code>`) and MUST be read **before** the first frame, so the app never
  flashes the wrong language at startup (see 4.5).
- Changing the language MUST apply **immediately and app-wide**, without restarting the app and
  without popping the user back to the home screen.
- The picker MUST live in Settings, MUST offer **System default** as an explicit first option, and
  MUST list each language in its own script (its endonym), not translated — e.g. `English`,
  `Español`, `हिन्दी`, `日本語`. Language packs list the exact endonyms for their languages.
- The current selection MUST be visibly marked (radio / check), and the setting row MUST be
  reachable by screen reader with a label describing the current value.
- Locale state MUST live in one place (`LocaleController` / a provider), and `MaterialApp.locale`
  MUST be driven by it. Screens MUST NOT read the language from anywhere else.

Reference controller:

```dart
/// Single source of truth for the app language. `null` locale means
/// "follow the system", resolved by MaterialApp against supportedLocales.
class LocaleController extends ChangeNotifier {
  static const String prefKey = 'app_language';
  static const String systemValue = 'system';

  /// The declared language codes (docs/PROJECT_PROFILE.md), template first.
  static final List<String> supported = AppLocalizations.supportedLocales
      .map((l) => l.languageCode)
      .toList(growable: false);

  final SharedPreferences _prefs;
  Locale? _locale;

  LocaleController(this._prefs) {
    final saved = _prefs.getString(prefKey) ?? systemValue;
    _locale = supported.contains(saved) ? Locale(saved) : null;
  }

  /// null => follow the system locale.
  Locale? get locale => _locale;

  bool get isSystem => _locale == null;

  Future<void> setLanguage(String value) async {
    _locale = value == systemValue ? null : Locale(value);
    await _prefs.setString(prefKey, value);
    notifyListeners();
  }
}
```

```dart
MaterialApp(
  locale: localeController.locale, // null => system locale
  supportedLocales: AppLocalizations.supportedLocales,
  localeResolutionCallback: (deviceLocale, supported) {
    for (final l in supported) {
      if (l.languageCode == deviceLocale?.languageCode) return l;
    }
    return supported.first; // the template language
  },
);
```

### 8.5 Translation Quality And Language Packs

Translations are product text, not a checkbox.

- Every non-template language MUST be reviewed by a fluent reader before its first release, and
  new or changed strings SHOULD be reviewed before each release. AI or machine translation is a
  draft, not a final translation.
- When an AI agent writes a translation it is not confident about, it MUST list that string in the
  change log as "needs native-reader review".
- Never substitute a related language or script for a declared language (for example Hindi for
  Sanskrit, Simplified for Traditional Chinese, Brazilian for European Portuguese when the project
  declares the other one).
- Keep terminology consistent: the same English term maps to the same translated term across the
  app. A project with more than a few screens SHOULD keep a small glossary in
  `docs/glossary.md`, or use the glossary in a language pack.
- **Language packs** (`language_packs/*.md`) hold the strict, language-specific rules — grammar,
  forbidden words, CI gates, glossaries, picker endonyms, store-listing notes. When a project
  declares a language that has a pack, that pack is **mandatory** for that project.

To add a new language pack, copy the structure of `language_packs/sanskrit_malayalam.md`
(framework gaps → fonts → picker labels → quality rules and glossary → label budget → store
listings → checklist) and list it in the table at the top of section 8.

### 8.6 Label Conciseness (Short UI Text vs. Descriptive Text)

UI chrome MUST be short in **every** declared language. A long translated word wrapping onto two
lines in a toolbar, tab, or bottom-navigation item is a layout bug.

**Budget for short text** — menu items, buttons, tabs, chips, navigation destinations, tooltips,
app-bar titles, list-row labels, form-field labels, switch/checkbox labels, dialog action buttons:

| Language | Target | Hard limit |
|---|---|---|
| Latin script (English, Spanish, …) | 1–2 words | 20 characters |
| CJK (Chinese, Japanese, Korean) | 1 word | 10 characters |
| Any other script (Devanagari, Arabic, Malayalam, …) | 1–2 words | 22 characters |

A language pack MAY set a different limit for its languages; the pack wins.

**How characters are counted.** A character is a visible character (a grapheme cluster), not a
code unit: a vowel sign or virama belongs to the letter before it. Count using Dart's
`characters.length` (`package:characters`, which is bundled with Flutter): `string.characters.length`.

Rules:

- Prefer a single word. Drop articles and filler: "Delete" not "Delete this item".
- Do not solve a long translation by shrinking the font, truncating, or adding an ellipsis —
  choose a shorter word.
- Sentence case in English (`Add note`), not Title Case. Follow each language's own casing rules.

**Descriptive text is exempt** from the budget — and MUST still be complete, natural prose in every
declared language: onboarding copy, empty-state explanations, help text, About `description`,
error explanations, confirmation dialog bodies, notification bodies, tutorial content. About-screen
row labels (`aboutDetail<Key>`) are also exempt from this budget so they can wrap to two lines.

**ARB key naming makes the category checkable.** Prefix every key so the budget can be enforced
mechanically. The only exceptions are the fixed keys named in `guideline.md` (`madeWithLove`,
`madeWithLoveA11y`, and the `aboutDetail<Key>` labels):

| Prefix | Category | Budget |
|---|---|---|
| `action…` | buttons, menu items, dialog actions | short |
| `label…` | field labels, row labels, chips | short |
| `title…` | screen / app-bar / dialog titles | short |
| `tab…`, `nav…` | tabs and navigation destinations | short |
| `tooltip…` | tooltips on icon-only controls (7.8) | short |
| `desc…`, `help…`, `empty…`, `error…`, `body…`, `aboutDetail…` | descriptive prose & About row labels | exempt |

```dart
// test/l10n/label_length_test.dart — fails when a short key exceeds its budget.
const shortPrefixes = ['action', 'label', 'title', 'tab', 'nav', 'tooltip'];
// One entry per declared language; values from the table above or the language pack.
const limits = {'en': 20 /*, '<code>': 22 */};
// For each ARB file: for each key starting with a short prefix,
// expect(value.characters.length, lessThanOrEqualTo(limits[locale]!));
```

### 8.7 Per-Feature Language Completeness

A feature is **not done** until it works fully in every declared language.

- No feature may ship with strings in the template ARB only. Adding a key to the template without
  adding it to every other declared language's ARB MUST fail CI.
- No feature may render template-language text under another declared language — including
  snackbars, validation messages, notification text, share sheets, exported file headers a user
  sees, and the About screen.
- Feature-level content shipped as an asset (JSON, Markdown help pages, seed data a user reads)
  MUST also carry every declared language, or the screen that shows it MUST resolve a per-language
  asset (`assets/content/help_<lang>.md`). `app_config.json` prose fields (`appName`,
  `description`, `details`) MUST provide an entry for every declared language.
- Screenshots for a release are taken in every declared language when the feature changes layout.
- Widget tests for a screen MUST run in every declared locale, asserting no overflow and no
  untranslated template text leaking through.

**Translation parity test** — required in every app with two or more declared languages:

```dart
// test/l10n/translation_parity_test.dart
import 'dart:convert';
import 'dart:io';
import 'package:flutter_test/flutter_test.dart';

/// The template language and the other declared languages (docs/PROJECT_PROFILE.md).
const template = 'en';
const locales = <String>[/* e.g. 'es', 'hi' */];

void main() {
  /// Strings allowed to match the template: brand names and symbols. Keep this list short.
  const sameAsTemplateAllowed = <String>{};

  group('ARB parity tests', () {
    test('every ARB file has the same keys as the template', () {
      final base = _keys('lib/l10n/app_$template.arb');
      for (final locale in locales) {
        final other = _keys('lib/l10n/app_$locale.arb');
        expect(other.difference(base), isEmpty, reason: 'extra keys in $locale');
        expect(base.difference(other), isEmpty, reason: 'missing keys in $locale');
      }
    });

    test('no translation is a copy of the template value', () {
      final baseStrings = _strings('lib/l10n/app_$template.arb');
      final problems = <String>[];
      for (final locale in locales) {
        final other = _strings('lib/l10n/app_$locale.arb');
        for (final entry in other.entries) {
          final key = entry.key;
          final value = entry.value;
          if (sameAsTemplateAllowed.contains(key)) continue;
          if (value == baseStrings[key] && value.trim().isNotEmpty) {
            problems.add('lib/l10n/app_$locale.arb: $key is untranslated (matches template)');
          }
        }
      }
      expect(problems, isEmpty, reason: problems.join('\n'));
    });

    test('the optional About badge is complete in every language', () {
      // The badge is on when the template ARB has `madeWithLove` (guideline.md §1.7).
      if (!_strings('lib/l10n/app_$template.arb').containsKey('madeWithLove')) return;
      final heartEmoji = RegExp('\u2764|\u2665|\u{1F496}|\u{1F497}|\u{1F49C}');
      final problems = <String>[];
      for (final locale in [template, ...locales]) {
        final strings = _strings('lib/l10n/app_$locale.arb');
        final text = strings['madeWithLove'];
        final a11y = strings['madeWithLoveA11y'];
        if (text == null) problems.add('app_$locale.arb: madeWithLove is missing');
        if (a11y == null) problems.add('app_$locale.arb: madeWithLoveA11y is missing');
        if (text != null && '{heart}'.allMatches(text).length != 1) {
          problems.add('app_$locale.arb: madeWithLove must contain exactly one {heart}');
        }
        for (final entry in {'madeWithLove': text, 'madeWithLoveA11y': a11y}.entries) {
          if (entry.value != null && heartEmoji.hasMatch(entry.value!)) {
            problems.add('app_$locale.arb: ${entry.key} has a heart emoji; use {heart}');
          }
        }
      }
      expect(problems, isEmpty, reason: problems.join('\n'));
    });
  });

  group('About JSON config parity tests', () {
    test('app_config.json has every declared language and valid detail labels', () {
      final file = File('assets/config/app_config.json');
      if (!file.existsSync()) return;

      final json = jsonDecode(file.readAsStringSync()) as Map<String, dynamic>;
      final templateKeys = _keys('lib/l10n/app_$template.arb');
      final problems = <String>[];

      void checkLanguages(String path, dynamic value) {
        if (value is Map<String, dynamic>) {
          for (final lang in [template, ...locales]) {
            final text = value[lang]?.toString().trim() ?? '';
            if (text.isEmpty) {
              problems.add('$path.$lang is missing or empty');
            }
          }
        }
      }

      checkLanguages('appName', json['appName']);
      checkLanguages('description', json['description']);

      // Rows that hold technical values (never translated) may be plain strings.
      // Every other row is display text and needs a locale map (guideline.md §1.2).
      const technicalKeys = {'email', 'website', 'privacyPolicy'};

      final details = json['details'];
      if (details is Map<String, dynamic>) {
        for (final entry in details.entries) {
          final id = entry.key;
          if (locales.isNotEmpty &&
              entry.value is String &&
              !technicalKeys.contains(id)) {
            problems.add('details.$id is a plain string; use a locale map');
          }
          checkLanguages('details.$id', entry.value);

          // Details key is lowerCamelCase; label in ARB is aboutDetail<Key>
          final labelKey = 'aboutDetail${id[0].toUpperCase()}${id.substring(1)}';
          if (!templateKeys.contains(labelKey)) {
            problems.add('details.$id has no corresponding ARB key "$labelKey"');
          }
        }
      }

      expect(problems, isEmpty, reason: problems.join('\n'));
    });
  });

  group('Content asset parity tests', () {
    test('content and help assets exist in every declared language', () {
      final assetsDir = Directory('assets');
      if (!assetsDir.existsSync()) return;

      final problems = <String>[];
      final templateFiles = assetsDir
          .listSync(recursive: true)
          .whereType<File>()
          .where((f) => RegExp('_$template\\.[^.]+\$').hasMatch(f.path));

      for (final baseFile in templateFiles) {
        for (final lang in locales) {
          final twinPath = baseFile.path.replaceAllMapped(
            RegExp('_$template(\\.[^.]+)\$'),
            (match) => '_$lang${match[1]}',
          );
          if (!File(twinPath).existsSync()) {
            problems.add('Missing localized asset: $twinPath (matching ${baseFile.path})');
          }
        }
      }

      expect(problems, isEmpty, reason: problems.join('\n'));
    });
  });
}

Set<String> _keys(String path) =>
    (jsonDecode(File(path).readAsStringSync()) as Map<String, dynamic>)
        .keys
        .where((k) => !k.startsWith('@'))
        .toSet();

Map<String, String> _strings(String path) {
  final map = jsonDecode(File(path).readAsStringSync()) as Map<String, dynamic>;
  return {
    for (final e in map.entries)
      if (!e.key.startsWith('@')) e.key: e.value.toString(),
  };
}
```

### 8.8 RTL Layout Support

- Never use `left` and `right` for padding, alignment, or positioning of UI elements. Use `start`
  and `end` equivalents: `EdgeInsetsDirectional`, `AlignmentDirectional`, `MainAxisAlignment.start`.
- Icons that carry directional meaning (back arrow, forward arrow) MUST be mirrored in RTL.
  Many directional `Icons.*` (for example `Icons.arrow_back`) already mirror themselves. For
  any other directional icon, use `Icon(..., matchTextDirection: true)`. `BackButton` and
  `Icons.adaptive.*` handle this for you.
- **If any declared language is RTL** (Arabic `ar`, Hebrew `he`, Persian `fa`, Urdu `ur`, …), every
  screen MUST be checked in that language, and widget tests MUST run under it.
- **If no declared language is RTL**, these rules are about staying ready: test RTL by wrapping a
  screen in `Directionality(textDirection: TextDirection.rtl)` in a widget test. Do **not** add an
  RTL language to `supportedLocales` just to test; `supportedLocales` equals the declared languages
  (8.3).

### 8.9 Locale-Sensitive Formatting

Use the `intl` package for all locale-sensitive formatting. Never use `toString()` on dates,
numbers, or currencies in user-visible strings. Pass `formattingLocale(...)` from 8.3.2 as the
`locale` argument, so languages without CLDR data fall back to the template language instead of
throwing.

```dart
import 'package:intl/intl.dart';

// Dates
DateFormat.yMMMMd(locale).format(date);       // "April 5, 2025" (en_US)
// Time — output depends on locale: 24-hour for de_DE, 12-hour for en_US, etc.
DateFormat.Hm(locale).format(time);           // "14:30" (24-h locales)
DateFormat.jm(locale).format(time);           // "2:30 PM" (12-h locales)

// Numbers
NumberFormat.decimalPattern(locale).format(value);

// Currency
NumberFormat.currency(locale: locale, symbol: '€').format(amount);
```

---

## 9. App Lifecycle Management

### 9.1 WidgetsBindingObserver

Register a `WidgetsBindingObserver` at a high level in the widget tree (typically at the root
provider or app widget level) to respond to system lifecycle events.

> **Modern alternative.** `AppLifecycleListener` is the recommended
> replacement for `WidgetsBindingObserver` when the only thing being observed is lifecycle
> state (no metrics, locale, accessibility, or memory-pressure callbacks). It is a regular
> Dart object — no mixin, no widget tree mounting — and exposes typed callbacks
> (`onPause`, `onResume`, `onDetach`, `onHide`, `onShow`, `onRestart`, `onExitRequested`).
> Prefer `AppLifecycleListener` for new code; keep `WidgetsBindingObserver` when you also
> need `didChangeMetrics`, `didChangeLocales`, or `didHaveMemoryPressure`.

```dart
class AppLifecycleService with WidgetsBindingObserver {
  void init() {
    WidgetsBinding.instance.addObserver(this);
  }

  void dispose() {
    WidgetsBinding.instance.removeObserver(this);
  }

  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    switch (state) {
      case AppLifecycleState.paused:
        // App moved to background.
        // Trigger app lock if security policy requires it.
        // Pause looping animations.
        // Flush pending write buffers to disk.
        break;
      case AppLifecycleState.resumed:
        // App returned to foreground.
        // Re-check app lock state.
        // Re-subscribe to data sources if needed.
        break;
      case AppLifecycleState.inactive:
        // App partially obscured (incoming call, notification shade).
        // For sensitive apps: obscure screen content.
        break;
      case AppLifecycleState.detached:
        // App is being terminated.
        // Flush any remaining writes.
        break;
      case AppLifecycleState.hidden:
        // App window hidden (desktop).
        break;
    }
  }

  @override
  void didHaveMemoryPressure() {
    // Clear non-critical caches (image cache, computed results).
    PaintingBinding.instance.imageCache.clear();
    PaintingBinding.instance.imageCache.clearLiveImages();
  }
}
```

### 9.2 Required Lifecycle Behaviors

| State | Required Behavior |
|-------|-------------------|
| `paused` | Flush unsaved data to DB; trigger app lock if `Sensitive Data Extension` |
| `paused` | Pause looping animations and background timers |
| `inactive` | Obscure screen content for sensitive apps (apply `IgnorePointer` + blur overlay) |
| `resumed` | Re-validate app lock; refresh time-sensitive UI state |
| `detached` | Finalize any in-progress DB writes; close open file handles |
| Memory pressure | Clear image cache; release non-critical in-memory buffers |

### 9.3 Database And File Handle Safety

- Open database connections MUST be kept open for the app lifetime; do not open and close per
  operation (this is expensive on mobile).
- On `detached`, call the database's close method if the platform supports a clean shutdown.
- File handles opened for writing MUST be flushed and closed before the app moves to `paused`.
- Temporary files SHOULD be cleaned up on `resumed` if the previous session ended abnormally.

---

## 10. Performance And Rendering Optimization

### 10.0 Rendering Engine: Impeller

Impeller is Flutter's rendering engine. Which platforms use it by default, and which still
allow a fallback, depends on the Flutter version — check the release notes of the version the
project uses. When last checked (2026-10), Impeller was the default on iOS (no opt-out), on
Android devices with Vulkan (others fall back to OpenGL automatically), and on macOS (Metal),
Windows and Linux (Vulkan). No app code changes are needed to use it.

Rules:

- **Test every declared platform on Impeller** after each Flutter upgrade that changes the
  renderer for that platform, especially desktop and custom drawing.
- **Shader compilation jank is eliminated** for the standard widget set. Custom fragment
  shaders MUST be tested under Impeller on every declared platform's backend (Metal, Vulkan,
  OpenGL fallback), as the shader pipeline differs from Skia. File any rendering regressions
  under Impeller against `flutter/flutter` rather than working around them.
- **Disabling Impeller** is a deliberate, documented decision — it is not a default tuning
  knob — and is only possible where your Flutter version still has the switch. Known
  switches (check they still exist before relying on one):
  - Android: `<meta-data android:name="io.flutter.embedding.android.EnableImpeller"
    android:value="false" />` in `AndroidManifest.xml`.
  - macOS: `FLTEnableImpeller` = `false` in `macos/Runner/Info.plist`.
  - Windows: `project.set_impeller_switch(flutter::ImpellerSwitch::Disabled)` in
    `windows/runner/main.cpp`.
  - Linux: `fl_dart_project_set_enable_impeller(project, FALSE)` in
    `linux/runner/my_application.cc`.
  - iOS: no opt-out.

  Every opt-out MUST be justified in `architecture.md §20 Decisions And Tradeoffs` and have a
  removal plan in `architecture.md §21 Known Risks`: Flutter is removing these fallbacks, so
  treat them as short-lived workarounds.
- **Binary size note.** Pre-compiled shaders make the binary slightly larger than a Skia
  build. This is accounted for in the size budget (§10.7) but worth flagging when comparing
  pre/post-Impeller builds.

### 10.1 Frame Budget

The target rendering budget is:

- **60 Hz displays**: 16 ms per frame.
- **90 Hz / 120 Hz displays**: 11 ms / 8 ms per frame.

A frame that exceeds its budget is called a jank frame. Sustained jank above 5% of frames is a
release-blocking regression.

Use `flutter run --profile` and the Flutter DevTools Performance tab to measure jank. Never
profile in debug mode.

### 10.2 Widget Rebuild Optimization

- Prefer `const` constructors on widgets that do not depend on runtime state. The `const`
  constructor rule in `analysis_options.yaml` (`prefer_const_constructors`) enforces this as a
  lint but understanding why matters: `const` widgets are never rebuilt.
- Use `RepaintBoundary` around widgets that update frequently and independently from their
  siblings (animated elements, real-time counters, video frames). This isolates their repaint to
  their own compositing layer.
- Split large `build` methods into focused sub-widgets. Flutter rebuilds the smallest widget
  subtree that calls `setState`. A monolithic build method forces the entire screen to rebuild on
  any state change.
- Use `ValueListenableBuilder`, `StreamBuilder`, or state management selectors (e.g.
  `ref.watch(provider.select(...))` in Riverpod) to scope rebuilds to the specific part of the
  tree that cares about a value change.

### 10.3 List And Grid Performance

- A list that is unbounded, or that can grow past **20 items**, MUST use `ListView.builder`,
  `GridView.builder`, or `SliverList` with a `SliverChildBuilderDelegate`. The eager-children
  variant (`ListView(children: [...])`) builds every child at once, whether visible or not.
- A list with a fixed, known size of 20 items or fewer MAY use `ListView(children: [...])`.
- Consider `itemExtent` on `ListView.builder` when all items have the same height. This eliminates
  per-item layout measurement and can significantly improve scrolling on long lists. Benchmark
  before committing to a fixed extent, as it precludes variable-height items.
- Use `const` constructors inside list item widgets wherever possible.
- Avoid loading all data into memory for very long lists. Implement pagination or cursor-based
  loading at the repository layer.
- `ListView.separated` builds its items and separators lazily, like `ListView.builder`, so it
  is fine for long lists too.

### 10.4 Image Handling And Memory

- Never load full-resolution images when a thumbnail or reduced size is sufficient. Use
  `ResizeImage` or specify `cacheWidth` / `cacheHeight` on `Image.asset` and `Image.file`:

  ```dart
  Image.file(
    file,
    cacheWidth: 400,  // Decoded at 400px wide; reduces GPU memory usage.
  )
  ```

- Use WebP format for photographic images. Lossless WebP is typically 25–30% smaller than PNG
  at equal quality; lossy WebP is typically 25–35% smaller than JPEG.
- For icons and simple illustrations, prefer SVG via `flutter_svg` over raster assets. SVGs
  scale without quality loss and add zero resolution variants to the asset bundle.
- Provide `2.0x` and `3.0x` resolution variants for all raster assets used in the UI. Missing
  variants cause blurry rendering on high-density screens.
- Do not load large images in `initState`. Use `FutureBuilder` or an async provider to load
  images off the frame budget.

### 10.5 Isolates And Background Computation

Any operation that risks holding the main isolate for long enough to drop a frame should be
moved off the UI thread. The 16 ms frame budget on a 60 Hz display is the ceiling; operations
approaching or exceeding that budget are candidates for offloading. The following are common
trigger categories — treat them as decision prompts, not automatic thresholds:

- JSON or CSV parsing of large record sets (a rough starting point is a few hundred records on
  older hardware; profile before assuming).
- Encryption or decryption of large payloads.
- Image compression or resizing in Dart.
- Complex data aggregation or transformation at the service layer.

Use `compute()` for simple single-call operations:

```dart
final result = await compute(_parseJsonInBackground, rawJsonString);

List<Todo> _parseJsonInBackground(String json) {
  // Runs in a separate isolate.
  return (jsonDecode(json) as List).map(Todo.fromJson).toList();
}
```

Use `Isolate.spawn` or an `IsolateNameServer` for long-lived background workers.

Profile in `--profile` mode before adding isolate complexity. `compute()` has measurable
spawn overhead for very small payloads; do not add it preemptively to operations that already
complete well within the frame budget.

### 10.6 Startup Performance

- The cold startup time target for release builds is under **2 seconds** to first meaningful
  frame on a mid-range device.
- Track binary size with a size-analysis build after each significant dependency addition
  (10.7).
- Defer non-critical initialization. Services that are not needed on the first screen SHOULD be
  initialized lazily (on first use), not eagerly in `main()`.
- Avoid synchronous disk reads in `main()`. Database migrations and file reads MUST be async.
- Use `flutter run --trace-startup` to measure startup phases during development.

### 10.7 App Size Budget

Under `Production App Extension`:

| Platform | Target | Hard Limit |
|----------|--------|------------|
| Android APK (arm64) | Under 30 MB | 50 MB |
| Android AAB download size | Under 20 MB | 40 MB |
| iOS App Store download size | Under 40 MB | 80 MB |
| Windows MSIX | Under 80 MB | 150 MB |
| macOS `.app` / DMG | Under 80 MB | 150 MB |
| Linux bundle / package | Under 80 MB | 150 MB |

These are starting budgets only for the platforms the project declares; a project MAY set its own
in `docs/PROJECT_PROFILE.md`. Exceeding the hard limit requires a documented justification in the release checklist.

Track size locally at each release (CI does not build releases, §19.3). Record the baseline at
project start and diff on each release. Two ways, both valid:

- **Measure the real artifact** you will ship (APK/AAB/IPA/MSIX/DMG file size; Play Console and
  App Store Connect also report download size).
- **A size-analysis build** for a per-package breakdown. `--analyze-size` is a **separate,
  throw-away build**: `flutter build -h` says it cannot be combined with `--split-debug-info`
  (so not with `--obfuscate` either), and on Android it needs a single ABI. It writes into the
  same output folders as the release build, so run it **before** the hardened release build,
  or copy the hardened artifacts out first. Never ship the analysis build. Example:

  ```bash
  flutter build apk --release --target-platform android-arm64 --analyze-size \
    --code-size-directory=build/size-analysis/
  ```

  Open the result with `dart devtools` → App Size. `release_process.md` §6.3 has the command
  for each platform.

---

## 11. Error Handling Architecture

### 11.1 Global Error Boundaries

Every Flutter app MUST configure global error handlers in `main()` before `runApp`. Without these,
unhandled errors in release builds crash silently with no user feedback.

```dart
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await AppLogger.init(); // section 4.5, step 2 — logging first, then the handlers

  // 1. Framework errors (widget build, layout, and paint errors).
  FlutterError.onError = (FlutterErrorDetails details) {
    AppLogger.error(
      'Flutter framework error',
      error: details.exception,
      stackTrace: details.stack,
    );
    // In debug, keep Flutter's console dump (and its default red error box).
    // Do NOT navigate to an error screen from here — the release-safe UI is
    // handled by ErrorWidget.builder below (see the note after this block).
    if (!kReleaseMode) {
      FlutterError.dumpErrorToConsole(details);
    }
  };

  // 2. Uncaught async errors that escape the widget tree (futures, timers,
  //    platform channels). Supersedes the older runZonedGuarded pattern.
  PlatformDispatcher.instance.onError = (Object error, StackTrace stack) {
    AppLogger.error('Uncaught async error', error: error, stackTrace: stack);
    return true; // Returning true suppresses the default crash.
  };

  // 3. In release, replace the raw red/grey error box with a safe, neutral
  //    fallback when a widget fails to build. Keep the default in debug so the
  //    red box and stack trace stay visible to developers. SafeErrorFallback is
  //    a minimal, context-free widget the app provides (no exception text, no
  //    controls that could fail the same way).
  if (kReleaseMode) {
    ErrorWidget.builder =
        (FlutterErrorDetails details) => const SafeErrorFallback();
  }

  // The remaining init steps from section 4.5 run here.
  // They are async (DB migration, secure storage bootstrap, etc.),
  // so main() must be Future<void> and use await.

  runApp(const MyApp());
}
```

> **Show the release-safe UI through `ErrorWidget.builder`, not by navigating from
> `FlutterError.onError`.** It is tempting to "open an error screen" directly inside
> `FlutterError.onError`. Do not — it is unreliable for three reasons:
>
> - `FlutterError.onError` runs outside the widget tree and has no `BuildContext`, so it
>   cannot `Navigator.push` or `showDialog`.
> - It fires during the build/layout/paint phase; navigating or mutating the tree mid-frame
>   throws further errors.
> - A widget that keeps failing re-fires the handler on every frame, which would stack
>   duplicate error routes many times per second.
>
> `ErrorWidget.builder` is the framework's purpose-built hook: Flutter invokes it at the exact
> point a widget fails to build and substitutes the returned widget in place — no context, no
> navigation, no repeated pushes. Keep `FlutterError.onError` for logging only. This satisfies
> the "show a safe screen in release" requirement more robustly than a route push would.
>
> **`PlatformDispatcher.instance.onError` vs `runZonedGuarded`.** In current Flutter,
> `PlatformDispatcher.instance.onError` is the recommended way to catch top-level async errors
> and replaces wrapping `runApp` in `runZonedGuarded`. Use one, not both — running both can
> double-report errors and cause zone conflicts.

### 11.2 Error Classification

Classify errors at the point of catch so that the correct response is taken.

| Class | Definition | Response |
|-------|-----------|----------|
| **Recoverable** | Operation failed but app state is intact | Show inline message, offer retry |
| **Degraded** | A feature is unavailable but the app is usable | Show banner, disable affected section |
| **Session** | The current session must be reset (lock triggered, corruption detected) | Navigate to safe state (lock screen or home), log detail |
| **Fatal** | App cannot continue safely | Show fatal error screen with restart action, log full detail |

Rules:
- Never escalate a recoverable error to a fatal error screen.
- Never silently swallow an error that changes application state.
- Error messages shown to the user MUST be human-readable and actionable. They MUST NOT contain
  stack traces, internal exception class names, or database error codes.
- Internal error detail (exception type, stack trace, operation context) MUST be logged at the
  service or repository layer regardless of what is shown in the UI.

### 11.3 Repository And Service Layer Error Handling

- Repositories MUST catch datasource-layer exceptions (`SqliteException`, `FileSystemException`,
  etc.) and re-throw typed domain exceptions.
- Define a sealed domain exception hierarchy:

  ```dart
  sealed class AppException implements Exception {}

  final class StorageException extends AppException {
    StorageException(this.message, {this.cause});
    final String message;
    final Object? cause;
  }

  final class ValidationException extends AppException {
    ValidationException(this.field, this.message);
    final String field;
    final String message;
  }
  ```

- Services MUST NOT throw raw `Exception` or `Error`. They MUST throw or return typed domain
  exceptions.
- State layers MUST catch domain exceptions and translate them into UI state (e.g.
  `AsyncError`, a sealed state variant, or an error field on the state object).

### 11.4 UI Error Presentation

- Use the standard four screen states (section 6.3) for asynchronous data loading errors.
- Inline field errors MUST appear below the relevant field, not as a toast.
- Operation errors (save failed, delete failed) MUST use a `SnackBar` with a retry action where
  possible.
- Fatal error screens MUST provide: a human-readable description, a primary action (Restart or
  Go Home), and a secondary action to copy diagnostic info to the clipboard (for support).

---

## 12. Code Generation

Many core packages in the recommended Flutter stack require `build_runner`. This section defines
how code generation is managed.

### 12.1 Required Commands

Run once after cloning or after modifying annotated source files:

```bash
dart run build_runner build --delete-conflicting-outputs
```

Watch mode during development (regenerates on file save):

```bash
dart run build_runner watch --delete-conflicting-outputs
```

Always use `--delete-conflicting-outputs`. Without it, stale generated files from a previous run
cause confusing type errors.

Add to `pubspec.yaml` under `dev_dependencies` (replace each `<current>` with the current version
from pub.dev at project start, and update deliberately):

```yaml
dev_dependencies:
  build_runner: ^<current>
  freezed: ^<current>             # if using freezed models (also freezed_annotation)
  json_serializable: ^<current>   # if using json annotation
  riverpod_generator: ^<current>  # if using riverpod code gen
```

### 12.2 Generated File Policy

The repository MUST declare one of the following policies for generated files and document it in
`README.md`.

**Option A: Commit generated files** (recommended for most apps)

- `*.freezed.dart`, `*.g.dart`, and `*.gr.dart` files are committed to source control.
- Benefit: The repository is always buildable without running `build_runner`. Useful for CI that
  does not run `build_runner` as a separate step.
- Requirement: Generated files MUST be regenerated and committed whenever the source file changes.
  A CI check SHOULD verify generated files are not stale.

**Option B: Exclude generated files** (acceptable for packages or large codebases)

- `*.freezed.dart`, `*.g.dart` and `*.gr.dart` are added to `.gitignore`.
- CI MUST run `dart run build_runner build --delete-conflicting-outputs` before `flutter analyze`
  and `flutter test`.

Whichever option is chosen, it applies to the entire repository. Mixed policies (some files
committed, some ignored) MUST NOT be used.

### 12.3 What NOT To Add To `.gitignore` Unconditionally

The following are sometimes incorrectly excluded. Clarify intent:

| File pattern | Default policy |
|-------------|----------------|
| `*.freezed.dart` | Commit (Option A) or exclude (Option B), not mixed |
| `*.g.dart` | Same as above |
| `.dart_tool/` | EXCLUDE — always. This is machine-local build state. |
| `build/` | EXCLUDE — always. This is build output. |

### 12.4 Freezed Model Pattern

```dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'todo.freezed.dart';
part 'todo.g.dart';

@freezed
abstract class Todo with _$Todo {
  const factory Todo({
    required int id,
    required String title,
    required bool isCompleted,
    required DateTime createdAt,
    DateTime? completedAt,
  }) = _Todo;

  factory Todo.fromJson(Map<String, dynamic> json) => _$TodoFromJson(json);
}
```

Rules:
- Freezed models MUST be immutable. Use `copyWith` for all mutations.
- Do not add mutable fields or `late` properties to freezed classes.
- Freezed classes are declared `abstract class` (single constructor) or `sealed class` (union).
- Handle union types with an exhaustive Dart `switch` (pattern matching), not `when` / `maybeWhen`.

---

## 13. Database And Persistence Standard

### 13.1 SQLite Migration Strategy

Schema changes MUST go through versioned migrations. Never modify the table structure in the
`onCreate` callback after the initial version is shipped to users — this only runs for new
installs.

```dart
final database = await openDatabase(
  path,
  version: 3,
  onCreate: (db, version) async {
    await _runMigrations(db, fromVersion: 0, toVersion: version);
  },
  onUpgrade: (db, oldVersion, newVersion) async {
    await _runMigrations(db, fromVersion: oldVersion, toVersion: newVersion);
  },
);

Future<void> _runMigrations(Database db, {
  required int fromVersion,
  required int toVersion,
}) async {
  for (var v = fromVersion + 1; v <= toVersion; v++) {
    await _migrations[v]!(db);
  }
}

final Map<int, Future<void> Function(Database)> _migrations = {
  1: _v1CreateTables,
  2: _v2AddIndexes,
  3: _v3AddNewColumn,
};
```

Rules:
- Migrations are append-only. Never modify a migration that has already been shipped.
- Each migration MUST be atomic. sqflite already runs `onCreate`, `onUpgrade` and
  `onDowngrade` inside one transaction, so all migration steps run in that transaction — use
  the `db` passed to the callback and do **not** open a second `db.transaction(...)` inside
  it. Outside sqflite (another database package), wrap each multi-statement migration in a
  transaction yourself.
- Migrations MUST be covered by integration tests that exercise the upgrade path from the minimum
  supported version to the current version.
- The current schema version MUST be documented in `docs/architecture.md`.

### 13.2 WAL Mode

Enable WAL (Write-Ahead Logging) mode on SQLite databases used in apps. WAL allows concurrent
readers and a single writer without locking, which significantly improves performance under
typical app workloads.

```dart
await openDatabase(
  path,
  version: schemaVersion,
  onConfigure: (db) async {
    await db.execute('PRAGMA foreign_keys = ON;');
    // journal_mode returns a row, so use rawQuery: execute() fails on Android.
    await db.rawQuery('PRAGMA journal_mode=WAL;');
  },
  onCreate: ...,
  onUpgrade: ...,
);
```

The `journal_mode` setting persists in the database file header, so once set it survives
across sessions. However, in sqflite specifically, setting it inside `onConfigure` (the same
hook recommended for `foreign_keys` in §13.4) is the safest pattern: `onConfigure` runs on
every connection sqflite opens, so any per-connection re-application happens automatically.
Issuing it once after open works in many cases but is fragile under sqflite's connection
lifecycle.

### 13.3 Index Strategy

- Add an index on every column used in a `WHERE` clause that filters a table with more than
  approximately 1,000 rows.
- Add a composite index when queries filter on two or more columns together.
- Never index columns that are mutated on every row update (e.g. `updated_at` on a high-frequency
  write table) unless read queries genuinely require it.
- Document indexes in the schema section of `docs/architecture.md`.

```sql
CREATE INDEX IF NOT EXISTS idx_todos_created_at ON todos(created_at);
CREATE INDEX IF NOT EXISTS idx_time_segments_todo_id ON time_segments(todo_id);
```

### 13.4 Data Integrity Rules

- Use foreign keys and enable enforcement: `PRAGMA foreign_keys = ON;`.
  In sqflite, set this inside the `onConfigure` callback of `openDatabase` rather than as a
  one-shot `execute` after open — `foreign_keys` is a per-connection setting, and depending
  on connection lifecycle a manually-issued PRAGMA can silently revert. `onConfigure` runs on
  every connection sqflite opens, which is the only reliable hook.

  ```dart
  await openDatabase(
    path,
    version: schemaVersion,
    onConfigure: (db) async {
      await db.execute('PRAGMA foreign_keys = ON;');
    },
    onCreate: ...,
    onUpgrade: ...,
  );
  ```

- Define `ON DELETE CASCADE` or `ON DELETE SET NULL` explicitly; never rely on application code
  to clean up orphan records.
- Use `NOT NULL` constraints on all columns that should never be null at the schema level.
- Use `CHECK` constraints for enumeration columns (e.g. `CHECK(status IN ('open', 'done'))`).

---

## 14. Logging Infrastructure

### 14.1 Logging Levels

Use a consistent level taxonomy across the codebase. All logging calls MUST use one of these
levels:

| Level | When To Use |
|-------|-------------|
| `trace` | Extremely detailed: individual DB rows, loop iterations. Dev-only. |
| `debug` | Useful dev context: function entry/exit, query parameters. Dev-only. |
| `info` | Normal significant events: app start, screen load, user action completed. |
| `warning` | Unexpected but recoverable: retry attempted, deprecated path used. |
| `error` | Operation failed: DB write failed, parse error, expected flow broke. |
| `fatal` | App cannot continue: unrecoverable state, data corruption detected. |

### 14.2 Recommended Logger Setup

The standard requires a named logger abstraction with a consistent level taxonomy (section 14.1)
and a defined sensitive-data policy (section 14.3). The implementation details — package choice,
output targets, rotation mechanism — are project decisions. The following is a reference
implementation using the `logger` package. Adapt it to your project's requirements.

```yaml
dependencies:
  logger: ^<current>  # current version from pub.dev
```

```dart
// lib/core/logging/app_logger.dart
import 'dart:io';
import 'package:flutter/foundation.dart' show kIsWeb;
import 'package:logger/logger.dart';
import 'package:path/path.dart' as p;
import 'package:path_provider/path_provider.dart';

class AppLogger {
  /// Console-only logger used until [init] finishes, so a log call (for example
  /// from the global error handlers) never fails during startup.
  static Logger _logger = Logger(
    level: AppFlavorConfig.instance.isDev ? Level.trace : Level.info,
    printer: SimplePrinter(),
  );

  /// Call once at startup (section 4.5, step 2). It is async because
  /// `getApplicationCacheDirectory()` is async; it swaps in the full logger.
  /// On Web there is no log file (no `dart:io` files): it keeps the console logger.
  static Future<void> init() async {
    if (kIsWeb) return;
    final cacheDir = await getApplicationCacheDirectory();
    final logFile = File(p.join(cacheDir.path, 'app.log'));

    _logger = Logger(
      level: AppFlavorConfig.instance.isDev ? Level.trace : Level.info,
      printer: PrettyPrinter(
        methodCount: 2,
        errorMethodCount: 8,
        lineLength: 120,
        colors: AppFlavorConfig.instance.isDev,
        printEmojis: false,
        dateTimeFormat: DateTimeFormat.onlyTimeAndSinceStart,
      ),
      output: AppFlavorConfig.instance.isDev
          ? ConsoleOutput()
          : MultiOutput([ConsoleOutput(), FileOutput(file: logFile)]),
    );
  }

  static void trace(String message) => _logger.t(message);
  static void debug(String message) => _logger.d(message);
  static void info(String message) => _logger.i(message);
  static void warning(String message, {Object? error}) =>
      _logger.w(message, error: error);
  static void error(String message, {Object? error, StackTrace? stackTrace}) =>
      _logger.e(message, error: error, stackTrace: stackTrace);
  static void fatal(String message, {Object? error, StackTrace? stackTrace}) =>
      _logger.f(message, error: error, stackTrace: stackTrace);
}
```

`AppLogger.init()` MUST run at step 2 of the `main()` initialization sequence (section 4.5), right
after `WidgetsFlutterBinding.ensureInitialized()`. Until it finishes, log calls go to the console
only; they never throw.

### 14.3 Logging Rules

- NEVER log: secrets, tokens, passwords, recovery codes, decrypted content, or full database rows
  that may contain PII.
- Log the operation name and error category, not raw exception messages that may contain user data.
- `debugPrint` MAY be used for quick investigative logging but MUST NOT be committed. Use
  `AppLogger.debug()` for committed debug logs. The `avoid_print` lint catches `print` but NOT
  `debugPrint`; treat both as banned in committed code.
- `AppLogger.trace` and `AppLogger.debug` MUST NOT produce output in production builds. Gate them
  behind the flavor config.
- All `error` and `fatal` logs MUST include an `error` object and a `stackTrace` when available.

### 14.4 Log Rotation And File Output

For apps that write logs to disk over extended periods:

- Limit the log file to a maximum of **5 MB**. Rotate to a new file when the limit is reached.
- Retain a maximum of **3 rotated log files** before deleting the oldest.
- Store log files in the app's cache directory (`getApplicationCacheDirectory()`), not the
  documents directory. Cache files may be cleared by the OS under storage pressure.
- Provide a diagnostic log export action in developer or settings UI so logs can be retrieved for
  support without requiring a device connection.

**Important:** The `logger` package's built-in `FileOutput` does not implement log rotation.
It writes to a single file indefinitely. Projects that need rotation must either wrap
`FileOutput` with size-check logic before each write, use a separate log file management
package, or implement rotation as a startup task that checks file size and renames the current
file before opening a new one. Document whichever approach is chosen in `docs/architecture.md §17`.

### 14.5 What To Log At Each Layer

| Layer | What To Log |
|-------|------------|
| `main()` startup | Initialization steps, flavor, platform, app version |
| Repository | Operation name, record count, duration for slow queries (> 50 ms) |
| Service | Significant state transitions, unexpected branch taken |
| State layer | Screen/feature entered, key user action completed |
| Error boundary | Full error class, message, and stack at `error` or `fatal` level |

---

## 15. Security Standard

Every app MUST be secure. The Security Baseline in 15.1 is a `MUST` for **every** app, in every
profile, including small internal tools. The other profiles only **add** controls; they never
remove a baseline one. If you are not sure whether some data is sensitive, treat it as sensitive.

Security must also stay easy to live with. Controls are matched to the risk, use platform
features where they exist, and must not slow the app or add steps for the user without a clear
reason (15.6).

### 15.1 Security Baseline (Every App)

These rules apply to all Flutter apps. Record how the app meets them in `docs/security.md` §1.1.

**Logging and permissions**

- Never log secrets, tokens, private payloads, or decrypted sensitive data.
- Request only the permissions the app actually uses.
- Ask for permissions at point of use where the platform allows it.
- Production logs SHOULD avoid personal data unless operationally necessary.

**Secrets in the app**

- Anything shipped inside the app — Dart code, assets, `--dart-define` values, `.env` files,
  native config files — can be read by anyone who has the app. Treat it as **public**.
- Real secrets (server or admin keys, signing keys, private API keys) MUST NOT ship in the app.
  They stay on a server the project controls.
- Keys that must ship in the app (for example a maps or Firebase key) MUST be restricted on the
  provider's side: by app id / signing certificate, by API scope, and by quota.

**Network**

- HTTPS (TLS) only. No cleartext HTTP in release builds on any platform.
- Never disable certificate checks: no `badCertificateCallback` that returns `true`, no global
  `HttpOverrides` that trusts every certificate, no "allow insecure" switch in release code.
- Keep the platform defaults on: Android cleartext traffic off (`release_process.md` §6.5), iOS
  and macOS App Transport Security on with no `NSAllowsArbitraryLoads`.
- Set connect and receive timeouts on network clients.
- Certificate pinning is **not** the default. Use it only when the project controls certificate
  rotation and ships a backup pin, and record the decision in `docs/security.md` §5. A wrong pin
  locks every user out of the app.

**Tokens, credentials and sign-in**

- Every sign-in token, session cookie, refresh token, or user credential MUST be kept in
  platform secure storage (for example `flutter_secure_storage`), even when the Sensitive Data
  Extension does not apply. Never in `SharedPreferences` or a plain file.
- Never store a user's password. Sign-out removes all tokens from the device.
- If the app has user accounts, sign in with OAuth 2.0 / OpenID Connect using PKCE through the
  system browser, or the platform's own sign-in sheet or passkeys. Never build a login form
  inside an embedded WebView for a third-party identity provider.
- Access to server data is enforced **on the server**. The app never relies on its own checks
  (hidden buttons, client-side role flags, decoded tokens) to protect server data.

**Input from outside the app**

- Treat everything that comes from outside as untrusted and validate it before use: deep links
  and app links, Android intents, push notification payloads, files the user imports, clipboard
  content, QR codes, platform-channel arguments, and server responses.
- Imported files: check size and type before parsing. When extracting archives or building file
  paths from input, block `..` and absolute paths so nothing is written outside the target
  folder ("zip slip").
- Use parameterized SQL queries only; never build SQL by joining strings with input.
- A deep link MUST NOT perform a sensitive action (delete, pay, change account, share data) by
  itself. Open the screen and let the user confirm.
- Sign-in and other callback links use verified links (Android App Links, iOS / macOS Universal
  Links) where the platform supports them, not a custom URL scheme alone.

**WebView**

- Avoid WebViews when a native screen or the system browser can do the job.
- If a WebView is used: load only an allowlist of HTTPS origins, turn on JavaScript only when
  the page needs it, expose no JavaScript bridge (channel) to content the project does not
  control, and open every other link in the system browser.

**Platform hardening**

- Android: `android:exported="false"` on every component that does not need to be reached from
  outside; exported components validate their input (`release_process.md` §6.7).
  `PendingIntent` uses `FLAG_IMMUTABLE` unless the feature needs a mutable one. `FileProvider`
  paths are as narrow as possible. No world-readable or world-writable files.
- App data lives in the app's private folders (for example `getApplicationSupportDirectory()`),
  never in shared or public storage unless the user chose that location.
- Desktop platforms follow the controls in `docs/security.md` §10 (sandbox, hardened runtime,
  confinement, signing).

**Randomness and cryptography**

- Use `Random.secure()` for anything security-related (ids that must not be guessed, nonces,
  keys). Never `Random()`.
- Never invent your own cryptography. Use well-known, maintained libraries and standard
  algorithms.

**Release builds**

- No debug menus, test accounts, backdoors, or "skip login" switches in production builds.
  Developer tools exist only in non-production flavors (section 5).
- `android:debuggable` is `false` in release (`docs/security.md` §8.3).

**Third-party SDKs and privacy**

- Review every analytics, ads, crash-reporting, or other SDK for what data it collects and where
  it sends it. List each one in `docs/security.md` §4.
- Crash reports and analytics never carry secrets or personal data.
- Collect and keep only the data a feature needs (data minimization).

**Web (if declared)**

- No secrets in the web bundle; everything in it is public.
- Set a Content Security Policy where the hosting allows it.

**Binary protection**

- Production builds for every declared **native** platform (Android, iOS, Windows, macOS,
  Linux) MUST be compiled with `--obfuscate --split-debug-info=<symbols_path>`. Web builds do
  not produce the same symbol files: use the hardening options that `flutter build web -h` lists
  for your Flutter (`release_process.md` §11C).
  Obfuscation renames Dart class and method names in the compiled binary to meaningless
  identifiers. This raises the cost of casual inspection and automated analysis of the release
  binary — an attacker without a decompiler cannot read class or method names directly from the
  binary. It does not prevent a determined reverse engineer with a Dart decompiler from
  reconstructing application logic, but it removes the low-effort attack surface. It also
  marginally reduces binary size as a secondary effect.

### 15.2 Sensitive Data Extension

Apply this section when the app handles authentication factors, private documents, health data,
financial data, recovery codes, or local encrypted stores.

- Sensitive values MUST NOT be stored in `SharedPreferences`.
- Use platform-backed secure storage for keys, tokens, or secret material.
- Use authenticated encryption such as AES-GCM for stored sensitive payloads.
- Never hardcode keys, IVs, salts, recovery passwords, or backup passwords.
- Cryptographic formats SHOULD be versioned so migrations remain possible.
- Clipboard use for secrets SHOULD be time-bounded or explicitly communicated.
- Screen-capture protection MUST be enabled on every screen that shows sensitive data (record
  the screens in `docs/security.md` §10). What each platform allows differs:
  - Android (block): set `WindowManager.LayoutParams.FLAG_SECURE` on the window — in
    `MainActivity`, through a method channel, or with a maintained plugin. This blocks
    screenshots and screen recording. (The old `flutter_windowmanager` plugin is no longer
    maintained; do not add it.)
  - iOS (hide and detect): iOS does not let an app block screenshots. Instead:
    - overlay an opaque view on the "will resign active" hook so the app-switcher snapshot
      shows no data — `sceneWillResignActive(_:)` for apps on the UIScene lifecycle (5.4), or
      `applicationWillResignActive` for apps not yet migrated;
    - hide sensitive content while the screen is being recorded or mirrored. Read the capture
      state from `UITraitCollection.sceneCaptureState` on UIScene apps and react to its change
      (`UIScreen.isCaptured` and its change notification in older code). Detection is
      best-effort, not a guarantee;
    - the app MAY react to `UIApplication.userDidTakeScreenshotNotification` (for example a
      short warning), but cannot stop the screenshot.
- App lock, background lock, and session-expiry behavior MUST be explicit in app state.
- Export of sensitive data SHOULD be encrypted by default; plaintext export, if allowed, MUST be
  explicit and user-confirmed.
- Backup, recovery, import, and migration flows MUST be tested as critical flows.
- Keep these controls easy and fast to use — biometric unlock with fallback, a short lock grace
  period, key derivation off the main isolate — as described in 15.6.

### 15.3 OWASP Mobile Top 10 Compliance Checklist

This checklist is a MUST under the `Sensitive Data Extension` **and** the
`Production App Extension`: sign it off in `docs/security.md` §12 before every production
release. Items that do not apply to the app are marked `n/a`, so a small app is not burdened.
For deeper detail on any item, use the OWASP Mobile Application Security Verification Standard
(MASVS). Verify the following controls:

| ID | Risk | Control |
|----|------|---------|
| M1 | Improper Credential Usage | No secrets shipped in the app (code, assets, `--dart-define`); shipped keys restricted by the provider; tokens in secure storage |
| M2 | Inadequate Supply Chain Security | Dependency audit; pin versions in `pubspec.lock` |
| M3 | Insecure Authentication/Authorization | Server-side authorization; OAuth/OIDC with PKCE via the system browser if the app signs in; app lock with proper background/foreground enforcement where used |
| M4 | Insufficient Input/Output Validation | All outside input validated (deep links, intents, imports, payloads); parameterized SQL; no path traversal |
| M5 | Insecure Communication | TLS only for any network traffic; no HTTP; certificate checks never disabled |
| M6 | Inadequate Privacy Controls | Data inventory reviewed; no PII in logs |
| M7 | Insufficient Binary Protections | `--obfuscate` applied to release builds |
| M8 | Security Misconfiguration | `android:debuggable=false` verified; no accidental exported components; permissions minimal; no debug backdoors |
| M9 | Insecure Data Storage | No sensitive data in `SharedPreferences` or unencrypted files |
| M10 | Insufficient Cryptography | Versioned encrypted formats; secure key derivation |

Mark each item as verified, not applicable, or risk-accepted (with documented justification)
before release sign-off.

### 15.4 Data Retention And Purge Policy

Every app that stores user-generated data MUST define and implement a retention policy.

- Document in `docs/security.md` §13 (every app has this file — `DOCS_FOLDER_GUIDELINE.md` §6):
  what data is stored, how long it is retained, and what triggers deletion.
- Provide a user-accessible "Delete all data" action that removes all local app data including
  the database, log files, cached files, and secure storage entries.
- If the app supports account deletion or reset, verify the purge is complete: no residual files
  in the app's documents, cache, or database directories.
- Temporary files (export staging, image resize cache) MUST be deleted within the same session
  they are created.

### 15.5 Logging And Telemetry

Under `Sensitive Data Extension`:

- Use structured logging rather than scattered `print` calls. See section 14 for the full
  logging standard.
- Verbose logging MUST be gated by environment or flavor config.
- Error logs SHOULD contain operation and error context without exposing protected data.

### 15.6 Security With Good Usability And Performance

Strong security and an easy, fast app go together. These rules apply to every app.

- **Match the control to the risk.** Controls follow the data sensitivity recorded in
  `docs/security.md` §4. A low-risk app does not get high-friction controls such as a forced app
  lock or a new sign-in on every resume.
- **Prefer platform features.** They are usually both safer and easier: biometrics with a
  device-credential (PIN / passcode) fallback, passkeys, the system photo and file pickers
  (which need no broad storage permission), and platform secure storage.
- **Permissions at the moment of need.** Ask with a short, localized reason. If the user says
  no, the app keeps working in a reduced but safe way.
- **App lock grace period.** When an app lock is used, lock after a short, documented grace
  period in the background instead of on every quick app switch. The user may choose a stricter
  setting. Record the period in `docs/security.md` §7.1.
- **Rooted, jailbroken, or emulator devices.** By default warn the user or limit the most
  sensitive features. Block the app only when the threat model in `docs/security.md` §3
  justifies it.
- **No slow-down.** Encryption, key derivation, hashing, and large-file crypto run off the main
  isolate (`compute()` or `Isolate`, section 10.5). Tune the key-derivation cost so unlock stays
  quick on the slowest target device; derive the key once per unlock and keep it in memory only
  while unlocked. Initialize secure storage and crypto lazily so startup time is not hurt
  (section 10.6).
- **Clear messages.** Security messages are localized, plain, and say what the user can do next.
  Lock screens and biometric fallbacks are accessible (section 7).
- **Fail safe, but recoverable.** On a security error, deny the action and give the user a clear
  way forward. Never fail open — for example, never skip the lock because biometrics could not
  load.
- **Record friction.** Every control that adds a step for the user is listed in
  `docs/security.md` §7.1 with its reason.

---

## 16. Coding Standards

### 16.1 Formatting And Analysis

- Run `dart format .` before commit.
- New work MUST NOT introduce analyzer issues.
- Repositories SHOULD aim for zero analyzer warnings overall.
- Start from `package:flutter_lints/flutter.yaml` (add it to `dev_dependencies` with a caret
  constraint on the current version from pub.dev) and add stricter rules deliberately. The caret
  keeps the major version fixed, which prevents the lint set silently shifting under your CI
  when a contributor upgrades dependencies.

Recommended baseline additions:

```yaml
include: package:flutter_lints/flutter.yaml

linter:
  rules:
    avoid_print: true
    prefer_single_quotes: true
    prefer_const_constructors: true
    prefer_const_declarations: true
    prefer_final_fields: true
    prefer_final_locals: true
    avoid_unnecessary_containers: true
    sized_box_for_whitespace: true
    use_key_in_widget_constructors: true
    prefer_is_empty: true
    avoid_empty_else: true
    unnecessary_brace_in_string_interps: true
    unnecessary_this: true
    no_duplicate_case_values: true
    avoid_redundant_argument_values: true
    sort_child_properties_last: true
    use_full_hex_values_for_flutter_colors: true
    always_use_package_imports: true
    cancel_subscriptions: true
    close_sinks: true
    use_decorated_box: true
    avoid_bool_literals_in_conditional_expressions: true
    noop_primitive_operations: true
    use_enums: true
```

### 16.2 Size And Complexity Guidance

These are prompts to review, not automatic failures.

| Metric | Guideline |
|--------|-----------|
| File length | Around 300 lines: consider splitting |
| File length | Around 500 lines: split or justify |
| Function length | Around 50 lines: consider extracting |
| Widget build method | Around 120 lines: consider sub-widgets |
| Parameters per function | More than 5: consider a parameter object |

### 16.3 Naming

- Use `snake_case` for files, `PascalCase` for classes, and `camelCase` for variables and
  functions.
- Suffix state objects with their role where helpful, such as `AccountProvider` or
  `AuthController`.
- Prefer explicit names over abbreviations.

### 16.4 Comments And Error Handling

- Comments SHOULD explain why, not restate what code does.
- TODOs SHOULD include an owner, issue, or clear follow-up context.
- Do not swallow exceptions silently.
- Show user-safe error messages in the UI while preserving internal diagnostic context
  appropriately. See section 11 for the full error handling standard.

### 16.5 Dependencies

- Add dependencies only when they remove meaningful complexity.
- Prefer maintained packages with clear ownership and null-safety support.
- Review transitive risk for packages that handle auth, storage, files, camera, or encryption.
- Remove unused dependencies promptly.
- For offline-only apps: audit every new dependency to verify it does not introduce transitive
  HTTP or network activity. Run `dart pub deps` and inspect for unexpected network packages.
- **Build hooks.** Dart packages can run native build steps through build hooks (stable in
  current Dart; check `dart --version` and the Dart release notes). Any new dependency that uses build hooks (`hook/build.dart`,
  `hook/link.dart`) MUST be reviewed: native code in dependencies has the same supply-chain
  implications as native plugins, plus the additional surface of arbitrary code running at
  package resolution time. Run `dart pub deps --style=tree` and review every package whose
  hook scripts you do not personally maintain.

### 16.6 Dependency Audit Cadence

Under `Production App Extension`:

- Run `flutter pub outdated` monthly and before each release. Review major version upgrades
  individually.
- Run `dart pub deps --style=tree` at least quarterly to inspect the full transitive dependency
  tree for unexpected additions.
- Check dependency licenses before the first public release and on any release that adds new
  dependencies. There is no built-in `flutter pub` command for this: list the direct and
  transitive packages with `dart pub deps --style=list`, then read each package's license on
  pub.dev (or use a license-listing package and keep its output with the release evidence).
  Verify all licenses are compatible with your distribution model. The app's own
  `showLicensePage` / `LicensePage` shows the licenses Flutter collected at build time.
- Pin critical security dependencies (encryption, secure storage) to exact versions in
  `pubspec.yaml` and update them deliberately after reviewing changelogs.

---

## 17. Asset Management

### 17.1 Image Format Policy

| Content Type | Required Format | Rationale |
|-------------|-----------------|-----------|
| Photographs, complex gradients | WebP (lossy) | 25–35% smaller than JPEG at equal quality |
| Logos, UI illustrations with transparency | WebP (lossless) or SVG | Smaller than PNG; SVG preferred if vector |
| Icons (monochrome or multi-color) | Material / Cupertino icon fonts (`Icons.*`, 17.3); SVG via `flutter_svg` for custom icons | Resolution-independent |
| Raster fallback (when SVG not viable) | PNG with 2x/3x variants | Only when SVG cannot achieve the result |

Never use JPEG for UI assets that require transparency.

#### Per-Platform Asset Bundling

When your Flutter supports it, `pubspec.yaml` accepts a `platforms:` filter on individual asset entries, letting heavy
platform-specific assets be excluded from builds for other platforms. This is the preferred
way to ship desktop-only or web-only assets without bloating mobile binaries.

```yaml
flutter:
  assets:
    - path: assets/logo.webp                     # Bundled everywhere
    - path: assets/desktop_hero_4k.webp
      platforms: [windows, linux, macos]         # Excluded from mobile builds
    - path: assets/web_worker.js
      platforms: [web]                            # Web only
```

Use this whenever an asset is only meaningful on a subset of target platforms; the size
budget (§10.7) thanks you. If `flutter pub get` rejects the `platforms:` key, the project's
Flutter does not support it yet — keep a plain asset list until it is upgraded.

### 17.2 Resolution Variants

Provide `2.0x` and `3.0x` resolution variants for all raster assets used in the UI. Missing
variants cause blurry rendering on high-density screens (most modern phones are 2x–3x).

Directory structure:

```text
assets/images/
|-- hero_banner.webp           # 1x (baseline)
|-- 2.0x/
|   `-- hero_banner.webp       # 2x
`-- 3.0x/
    `-- hero_banner.webp       # 3x
```

Register only the main folder in `pubspec.yaml`. Flutter finds the `2.0x/` and `3.0x/`
variants by itself and picks the right one for each screen:

```yaml
flutter:
  assets:
    - assets/images/
```

### 17.3 Icon Font Tree Shaking

Icon tree shaking (`--tree-shake-icons`, on by default in release builds) removes unused icon
glyphs from icon fonts. It works only when every `IconData` the app uses is a compile-time
constant.

- Use the constant `IconData` values: `Icons.add`, `CupertinoIcons.share`, or your own
  `static const IconData` fields. `Icon(Icons.add)` is fine with or without `const`.
- Never build an `IconData` at run time (for example `IconData(codePointFromJson)`). The build
  then fails with a non-constant `IconData` error, or tree shaking must be turned off
  (`--no-tree-shake-icons`), which ships the whole font.
- Custom icon fonts MUST be subset to include only the glyphs actually used. Use a font subsetting
  tool (e.g. `fonttools`, `glyphhanger`) before committing font files.

### 17.4 Font Licensing

- Verify the license of every bundled font before first release.
- OFL (SIL Open Font License) fonts are generally safe for commercial use without modification.
- The `google_fonts` package fetches fonts at runtime by default. For fully offline apps,
  either disable runtime fetching and bundle the `.ttf` files locally:

  ```dart
  GoogleFonts.config.allowRuntimeFetching = false;
  // and add the font files under assets/google_fonts/ in pubspec.yaml
  ```

  or skip the package entirely and reference fonts directly from `pubspec.yaml`. The default
  configuration violates offline requirements; verify the configuration in your app's startup
  before relying on `google_fonts`.
- Document font sources and licenses in `docs/architecture.md` or a `LICENSES` file.
- **Script coverage is part of licensing work.** The chosen font stack MUST cover the script of
  every declared language (section 8.3.3), or the app MUST bundle fonts that do (the Noto
  families are OFL). Runtime fetching is not acceptable for these — a device offline on first
  launch would render boxes instead of text.

### 17.5 App Icons And Splash Screens

Every declared platform needs its own correctly sized app icon; a missing or default Flutter icon
is a common store rejection.

- Keep one high-resolution source icon in the repo (at least 1024×1024 PNG, no transparency for
  iOS) under `assets/icons/`.
- Generate platform icons with a tool such as `flutter_launcher_icons` rather than by hand, and
  commit the generated files. Configure every declared platform:

  ```yaml
  # pubspec.yaml (dev_dependencies: flutter_launcher_icons) — example, check current options
  flutter_launcher_icons:
    image_path: assets/icons/app_icon.png
    android: true
    adaptive_icon_background: "#FFFFFF"
    adaptive_icon_foreground: assets/icons/app_icon_foreground.png
    ios: true
    remove_alpha_ios: true
    windows: { generate: true, image_path: assets/icons/app_icon.png }
    macos: { generate: true, image_path: assets/icons/app_icon.png }
  ```

  ```bash
  dart run flutter_launcher_icons
  ```

- **Android** MUST ship an adaptive icon (foreground + background layers) and SHOULD ship a
  monochrome layer for themed icons.
- **Web** (if declared): generate the favicon and `web/icons/` with the same tool
  (`web: { generate: true, image_path: ... }` in `flutter_launcher_icons`, check current options)
  and set the app name and theme colour in `web/manifest.json`.
- **Linux** icons are installed by the package (Snap / Flatpak / `.deb`) under the
  `APPLICATION_ID` name (5.5.3); the tool above does not cover Linux.
- **Splash screen**: use the native launch screen (Android 12+ `SplashScreen` API, iOS
  `LaunchScreen.storyboard`), for example via `flutter_native_splash`. Keep it to the logo on a
  plain background — no text that would need translating.
- Flavor builds SHOULD use a visibly different icon (e.g. a "DEV" badge) so testers never confuse
  them with production.

---

## 18. Testing Standard

### 18.1 Test Levels

| Level | Core Baseline | Production App Extension |
|-------|---------------|--------------------------|
| Unit tests | Required for business logic, models, parsing, validation, and services | Required |
| Widget tests | Required for screens or widgets with meaningful UI logic | Required |
| Integration tests | Optional unless the app has critical end-to-end flows | Required for critical release paths |
| Golden tests | Optional | Optional but recommended for design systems |
| Performance tests | Optional | Required for screens with complex lists or animations |

### 18.2 Test Rules

- `test/` SHOULD mirror `lib/` closely.
- Services, state layers, and models with non-trivial logic SHOULD have corresponding tests.
- Critical math, parsing, migration, and security logic MUST use deterministic vectors where
  available.
- Bug fixes SHOULD add regression tests when feasible.
- Run `flutter test` after code changes that affect Dart behavior.
- Shared test scaffolding SHOULD live in `test/helpers/` or an equally obvious location.
- Database migration tests MUST cover the full upgrade path from the oldest schema version still
  installed by users (usually version 1) to the current version, not just the latest increment
  (13.1).

### 18.3 Test Quality

- Tests MUST be independent.
- Use descriptive test names.
- Mock external systems, not the logic under test.
- Important test files SHOULD be runnable in isolation.
- Coverage trends are useful, but arbitrary percentage gates SHOULD NOT replace judgment.

### 18.4 Performance Testing

Under `Production App Extension`:

- Measure performance on a real device or emulator in **profile mode**, never in widget tests
  (`flutter test` runs in debug mode on the host, so its timings mean nothing).
- For frame-rate regression testing, use the `integration_test` package and record frame
  timings with `watchPerformance` (or `traceAction` for a full timeline):

  ```dart
  // integration_test/scroll_perf_test.dart
  final binding = IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  testWidgets('long list scrolls smoothly', (tester) async {
    await tester.pumpWidget(const MyApp());
    final list = find.byType(Scrollable).first;
    await binding.watchPerformance(() async {
      await tester.fling(list, const Offset(0, -5000), 3000);
      await tester.pumpAndSettle();
    }, reportKey: 'scroll_timing');
  });
  ```

  ```dart
  // test_driver/perf_driver.dart — writes the summary to build/
  import 'package:integration_test/integration_test_driver.dart';

  Future<void> main() => integrationDriver(
        responseDataCallback: (data) async {
          if (data != null) {
            await writeResponseData(data['scroll_timing'] as Map<String, dynamic>,
                testOutputFilename: 'scroll_timing');
          }
        },
      );
  ```

  ```bash
  flutter drive --profile \
    --driver=test_driver/perf_driver.dart \
    --target=integration_test/scroll_perf_test.dart
  ```

  Compare `90th_percentile_frame_build_time_millis` and
  `90th_percentile_frame_rasterizer_time_millis` in the report with the frame budget (10.1).
- Capture and store performance baseline results. Treat regressions beyond 20% as blocking.

### 18.5 Widget Previews

Flutter Widget Previews render annotated widgets in a dedicated IDE panel without launching
the full app. It is faster feedback than running widget tests for visual iteration. Check
that the project's Flutter and IDE plugin support it (the `@Preview` annotation exists in
`package:flutter/widget_previews.dart`). It is never a substitute for widget tests, golden
tests, or integration tests.

```dart
import 'package:flutter/widget_previews.dart'; // @Preview
import 'package:flutter/widgets.dart';         // Widget

@Preview(name: 'Empty state — light')
Widget previewEmptyStateLight() => const TodoEmptyState(theme: AppTheme.light);

@Preview(name: 'Empty state — dark')
Widget previewEmptyStateDark() => const TodoEmptyState(theme: AppTheme.dark);
```

Rules:
- Treat previews as a **development convenience**, not as a checked-in test artifact. Do
  not gate CI on previewer behavior.
- Cover the same widget with widget tests for behavior assertions and (optionally) golden
  tests for pixel-level regressions.
- Remove orphaned `@Preview` annotations during refactors so the previewer panel stays
  meaningful.

---

## 19. CI Standard (Checks Only)

GitHub CI in **every** repository (all profiles) does exactly one thing: it runs the
analyze/test checks (19.1). Dependencies are updated by hand, not by a bot (19.2). Nothing else
runs in CI (19.3).

### 19.1 Required CI Checks

Every repository hosted on GitHub MUST have a CI workflow that runs on pull requests and on
pushes to the protected main branch. It MUST be in place before the first merge to the main
branch. A repository on another git host runs the same checks in that host's CI, or locally
before every merge; the same "checks only" rules (19.2, 19.3) apply. Record which one the project
uses in `docs/PROJECT_PROFILE.md` §7.

Allowed steps — only these:

- Check out the code and set up Flutter (and Java only if a check needs it).
- `flutter pub get`.
- `dart run build_runner build --delete-conflicting-outputs` — only if the project uses code
  generation (needed before analyze and test).
- `dart format --output=none --set-exit-if-changed .`
- `flutter analyze`
- `flutter test` — this includes the localization parity test (§8) and any other unit, widget,
  or golden test.
- Other read-only check scripts that only pass or fail, such as the generated-file staleness
  check (Option A in §12.2) or a language-pack check (`language_packs/`). A secret-scanning
  check that only passes or fails and uses no secrets MAY be added here too.

Rules:

- The workflow uses no secrets and the least permissions (`permissions: contents: read`).
- One `ubuntu-latest` runner is enough; checks do not need a per-platform runner.
- Pin the exact Flutter version the project uses (the one recorded in `CLAUDE.md` /
  `AGENTS.md`), so CI and contributors agree. `channel: stable` alone follows every new
  release and can break CI without any change in the repository.

Example `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  pull_request:
  push:
    branches: [main]

permissions:
  contents: read

jobs:
  checks:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4          # Update action versions by hand (19.2)
      - uses: subosito/flutter-action@v2
        with:
          channel: stable
          flutter-version: <x.y.z>         # the exact version pinned in CLAUDE.md / AGENTS.md
      - run: flutter pub get
      - run: dart run build_runner build --delete-conflicting-outputs   # Only if using code gen
      - run: dart format --output=none --set-exit-if-changed .
      - run: flutter analyze
      - run: flutter test
```

### 19.2 Dependency Updates (Manual)

Projects do not use Dependabot or any other bot that opens update pull requests.

- Do not add `.github/dependabot.yml` (or a similar update-bot config) to the repository.
- Update packages by hand, following §16.6: run `flutter pub outdated`, review each major
  version update on its own, and follow the project's plan → approve → log workflow.
- When you update packages, also check the versions of the GitHub Actions used in
  `.github/workflows/` (for example `actions/checkout`) and update them by hand.
- GitHub's security alerts for dependencies (a repository setting) MAY stay on. They need no
  file in the repository.

### 19.3 What CI Must Not Do

GitHub CI MUST NOT:

- build release artifacts (APK, AAB, IPA, MSIX, DMG, Linux bundles) for any platform;
- sign anything, or hold signing material or store API keys as CI secrets;
- upload to any store or distribution channel, or deploy anything;
- create GitHub releases or tags, or bump versions;
- run any other workflow besides the 19.1 checks.

Release builds — with `--obfuscate` and
`--split-debug-info=build/symbols/<platform>-<flavor>-<version>/` — signing, size checks,
license checks, and uploads are done **locally** by the release owner, following each
platform's steps in `release_process.md`. Signing material stays outside the repository and
outside CI.

The symbols folder is always `build/symbols/<platform>-<flavor>-<version>/` (drop `-<flavor>` when
the app has no flavors; `<version>` is the full `pubspec.yaml` version, e.g. `1.4.0+27`), so symbols
from different releases never collide. `release_process.md` §6.1 and `security.md` §8.1 use the
same layout.

### 19.4 Pre-Commit

A pre-commit hook MAY run formatting and analysis locally, but CI remains the source of truth.

Recommended local pre-commit script:

```bash
#!/bin/bash
dart format --output=none --set-exit-if-changed . || exit 1
flutter analyze --no-pub || exit 1
echo "Pre-commit checks passed."
```

---

## 20. Git And Repository Hygiene

### 20.1 Branching And Commits

- Protect the main branch for team repositories.
- Use short-lived branches.
- Prefer conventional commit prefixes such as `feat:`, `fix:`, `refactor:`, `test:`, `docs:`,
  and `build:`.
- Keep commits cohesive.

### 20.2 Never Commit

- Build output such as `build/`, APKs, AABs, generated release artifacts.
- Secrets, keys, keystores, and signing material. (Keystore location, `key.properties`
  naming, and the `.gitignore` rules: see `guideline.md §2`.)
- Local machine configuration files containing credentials or machine-specific paths.
- `.dart_tool/` directory (machine-local build state).
- Debug symbols from `--split-debug-info` (they live under `build/symbols/`, covered by `build/`).

### 20.3 Usually Commit

- `analysis_options.yaml`
- `.gitignore`
- `pubspec.lock` for application repositories
- `*.freezed.dart` and `*.g.dart` if Option A (commit generated files) is chosen — see section 12.2.

### 20.4 Generated File Policy In .gitignore

Add to `.gitignore` regardless of whether generated Dart source is committed:

```gitignore
# Build output (also holds the debug symbols in build/symbols/)
build/
*.apk
*.aab
*.ipa
*.msix
*.msixbundle
*.dmg
*.pkg
*.snap
*.AppImage
*.deb
*.rpm
*.flatpak

# Signing material (see guideline.md §2 and platform_store_readiness.md)
*.p12
*.pfx
*.p8
*.mobileprovision
*.provisionprofile

# Local secrets and environment files — never ship or commit them
.env
.env.*

# Machine-local state
.dart_tool/
.flutter-plugins
.flutter-plugins-dependencies

# IDE files (keep .vscode/settings.json committed if it contains project-wide settings)
.idea/
*.iml

# Generated Dart source — REMOVE these lines if Option A (commit generated files) is chosen:
# *.freezed.dart
# *.g.dart
# *.gr.dart
```

---

## 21. Documentation Standard

### 21.1 Required Documents For App Repositories

| Document | Purpose |
|----------|---------|
| `CLAUDE.md` | Mandatory project-root AI instructions following `CLAUDE_MD_GUIDELINE.md` (MUST) |
| `AGENTS.md` | Mandatory project-root AI agent instructions following `AGENTS_MD_GUIDELINE.md` (MUST) |
| `README.md` | Setup, run, test, and build instructions |
| `docs/GUIDELINES_MANIFEST.md` | Portable pointer manifest indexing shared Flutter guidelines |
| `docs/PROJECT_PROFILE.md` | Platforms, stores, languages, identity and About options (1.2.1) |
| `docs/architecture.md` | Module boundaries, initialization sequence, schema version, major decisions |
| `docs/security.md` | Threat model, data inventory, retention policy; full detail under `Sensitive Data Extension` |
| `docs/release_process.md` | Release scope and runbook; full detail under `Production App Extension` |
| The rest of the baseline `docs/` set | `workflow_rules.md`, `dependencies.md`, `project_structure.md`, `implementation_plan.md`, `implementation_progress.md` (`DOCS_FOLDER_GUIDELINE.md` §6) |
| Privacy policy (text + public URL) | Required for any app on a public store. Stores need a public **URL**; a file in the repo counts only if it is reachable at a public URL. A common setup is `PRIVACY.md` in a public repo, linked by its URL (`platform_store_readiness.md` §1, "Where to host it") |
| `plans/` | One plan per change — MUST follow the privacy rule in 21.1.1 |
| `change_log/` | One log per change — MUST follow the privacy rule in 21.1.1 |

#### 21.1.1 Privacy Rule For `plans/` And `change_log/`

Files in `plans/` and `change_log/` are committed and may become public on the internet. They MUST
use relative repository paths only and MUST NOT contain any **local system details** — OS user
name, computer/host name, home or drive-letter paths (`C:\Users\...`, `l:\...`, `file:///...`),
network share names, LAN or internal IP addresses, local server URLs with ports, device serial
numbers, personal email addresses — or any secret (API key, token, password, keystore passphrase,
credential, PII).

Write them as if a stranger will read them. Nothing should reveal the machine they were written on.

| Do not write | Write instead |
|---|---|
| `l:\Android\MyApp\lib\main.dart` | `lib/main.dart` |
| `C:\Users\<name>\.gradle\gradle.properties` | "the local Gradle home" |
| `file:///l:/Android/MyApp/plans/x.md` | `../plans/x.md` |
| `\\OFFICE-PC\share\build` | "the shared build folder" |
| `192.168.1.42:8080` | "the local dev server" |
| `someone@example.com` | "the release owner" |
| `keystorePassword=hunter2` | "the keystore password (stored outside the repo)" |

### 21.2 Recommended Documents

- `CHANGELOG.md` for user-facing release history.
- `docs/adr/` for architecture decision records that are likely to be revisited.

### 21.3 README Must Include

- Prerequisites (Flutter version, Dart version, platform SDK versions, and the build machine each
  declared platform needs — e.g. macOS + Xcode for iOS/macOS, GTK packages for Linux).
- Setup steps from a clean clone to a running app.
- How to run tests.
- How to run code generation (`build_runner`).
- Build commands for each target platform.
- How to add a new database migration.
- Environment variable or `--dart-define` values needed.

---

## 22. AI Coding Assistant Instructions

When this standard is supplied to an AI coding assistant, the assistant MUST:

### 22.1 Before Writing Code

- Read and adhere strictly to the project's root `CLAUDE.md` / `AGENTS.md` instructions (following `CLAUDE_MD_GUIDELINE.md` and `AGENTS_MD_GUIDELINE.md`).
- Read the existing code before modifying it.
- Identify whether the repo is Tier 1 or Tier 2 and follow the existing structure.
- Identify the existing state-management pattern and follow it.
- Read `docs/PROJECT_PROFILE.md`: applicability profiles, target platforms, distribution
  channels, declared languages, language packs, About options. If it is missing, run the intake
  in `AI_AGENT_START_HERE.md` §3 and get the user's approval of the filled-in profile before
  creating it from `PROJECT_PROFILE_TEMPLATE.md`. Never guess platforms, stores, languages,
  author or package id.
- Read every language pack that applies to a declared language.
- Check the current database schema version before writing any migration.
- Check whether the repository commits or excludes generated files before creating new models.
- Write a plan to `plans/` and obtain explicit user approval before modifying project files.

### 22.2 While Writing Code

- Respect the current project structure unless the task explicitly includes restructuring.
- Do not introduce a second state-management system without a documented reason.
- Do not add boilerplate comments or type annotations to unchanged code.
- Do not invent abstractions for one-time operations.
- Apply the security profile in force; never log secrets or weaken cryptographic behavior.
- Follow the Security Baseline (section 15.1) in every app. Never weaken a security control to
  make something work — for example cleartext HTTP, trusting all certificates, loosening
  `allowBackup` or `android:exported`, adding broad permissions, turning on debug flags, or
  disabling a check. If a task seems to need it, stop and ask the user in the plan.
- Give every plan that touches network, storage, sign-in, permissions, deep links, WebViews,
  platform folders, or dependencies a short **"Security impact"** line.
- Ensure all `plans/` and `change_log/` entries follow the privacy rule in 21.1.1: **relative repository paths only**, **no local system details** (OS user name, computer/host name, home or drive-letter paths, network shares, LAN/internal IPs, local server URLs with ports, device serial numbers, personal email addresses), and **no secrets** (API keys, tokens, passwords, keystore passphrases, credentials, PII).
- Put all user-visible strings in `lib/l10n/*.arb` and read them through `AppLocalizations` (section 8.2) — never a raw string literal in a widget.
- Add every new key to the ARB file of **every declared language**, with a real translation in each (sections 8.2, 8.7). Never leave the template-language value as a placeholder in another language's file.
- Follow every language pack that applies (section 8.5). Never substitute a related language for a declared one. Flag any translation you are not confident about in the change log as "needs native-reader review".
- Keep `action…`, `label…`, `title…`, `tab…`, `nav…` and `tooltip…` strings within the length budget in 8.6; only `desc…`/`help…`/`empty…`/`error…`/`body…`/`aboutDetail…` keys may be long.
- Give every icon-only control a localized `tooltip:` (section 7.8).
- Keep the About screen data-driven and localized (`guideline.md` §1.6). If the project profile enables the signature badge, never remove or reword it (`guideline.md` §1.7).
- Only touch platform folders (`android/`, `ios/`, `windows/`, `macos/`, `linux/`, `web/`) for declared platforms. When adding a feature that needs a permission, entitlement, or capability, add it on **every** declared platform (Android manifest permission, iOS/macOS `Info.plist` usage string and entitlement, MSIX capability, Snap plug / Flatpak permission) — and nowhere it is not needed.
- Never hard-code personal data (author name, email, company) in Dart code; it comes from `docs/PROJECT_PROFILE.md` and `assets/config/app_config.json`.
- Do not use `kDebugMode` or `kReleaseMode` as a substitute for application flavor when the
  project has explicit environments.
- Always add `const` to constructors and widget instantiations where possible.
- Use a builder (`ListView.builder` etc.) for any list that is unbounded or can exceed 20 items (10.3).
- Never call heavy synchronous work on the main isolate; use `compute()` or `Isolate`.
- Always use `AppLogger` (or the project's logging service), never `print` or `debugPrint`.
- Always add a `Semantics` label to custom interactive widgets.

### 22.3 After Writing Code

- Run `flutter test` after code changes that affect behavior.
- Run `flutter analyze` before considering the task complete.
- Run `dart run build_runner build --delete-conflicting-outputs` after modifying annotated files.
- Add or update tests when logic changes.
- Write a change log to `change_log/` referencing the plan, using relative paths only and excluding all local system details and sensitive information (section 21.1.1).
- Re-read the new plan and change log once before finishing, purely to check for leaked local system details.
- Verify that no secrets, local machine files, or build artifacts are staged.
- Verify that any new database schema change is accompanied by a migration.

---

## 23. Definition Of Done

A task is complete only when all applicable items are true.

### 23.1 Core Baseline

- Architecture boundaries were respected.
- New code follows the repository's chosen state-management pattern.
- Tests were added or updated for changed logic where appropriate.
- `flutter analyze` is clean for the change.
- `flutter test` passes for behavior-affecting code changes.
- `dart format .` produces no required follow-up changes.
- No secrets, build output, or local machine files were added to git.
- The change follows the Security Baseline (section 15.1); no security control was weakened.
- New network calls use HTTPS with normal certificate checks.
- New input from outside the app (links, intents, files, payloads) is validated.
- `docs/security.md` was updated if the change touched permissions, storage, network, sign-in,
  third-party SDKs, or entry points.
- All `plans/` and `change_log/` files use relative repository paths only and contain zero local system details and zero sensitive data — safe to publish on the internet (section 21.1.1).
- `l10n.yaml` and one ARB file per declared language exist, and every user-visible string added or
  changed by this task comes from `AppLocalizations` (section 8.2).
- Every ARB key added or changed by this task exists and is genuinely translated in every declared
  language; the parity test passes (section 8.7).
- Every applicable language pack's rules and gates pass (section 8.5).
- Short-label keys are within the length budget for every declared language (section 8.6).
- Every icon-only control added or changed has a localized tooltip (section 7.8).
- The screen was checked in every declared language — no template text leaking through, no
  overflow, no missing glyphs (sections 8.3.3, 8.7).
- If the project profile enables the About signature badge, the About screen still ends with it
  when this task touched About (`guideline.md` §1.7).
- The app still builds for every declared platform if the change touched dependencies, plugins,
  native code, permissions or platform folders.
- Generated files were regenerated if any annotated source was changed.

### 23.2 Production App Extension

- Environment-specific behavior was verified if the change touched it.
- Required CI checks pass.
- User-facing documentation was updated if behavior changed.
- Release builds or flavor builds were verified when the change touched build, config, signing,
  or release behavior.
- No new jank frames introduced on the primary user flow (verified in profile mode if the change
  touched rendering, lists, or animations).
- App size budget was checked if a new dependency was added.
- OWASP checklist items affected by the change were re-verified (the checklist is a MUST under
  this extension — 15.3).
- The store readiness gate of every declared distribution channel (`platform_store_readiness.md`)
  still holds for any change touching manifests, `Info.plist`, entitlements, MSIX capabilities,
  Snap/Flatpak permissions, target SDKs, signing, data collection, or store-listed behavior.

### 23.3 Sensitive Data Extension

- Sensitive data handling was reviewed against the security section.
- Logging was reviewed for protected data exposure.
- Backup, import, export, migration, or recovery paths were tested if touched.
- OWASP checklist items affected by the change were re-verified (the checklist is a MUST under
  this extension — 15.3).

---

## 24. Practical Guidance

- A thin `main.dart` scales better than a smart one.
- Mirrored tests reduce search time and ownership confusion.
- `utils/` is acceptable only when its scope stays clear and small.
- Flavors solve real problems, but not every app needs them on day one.
- The security baseline is for every app; extra controls are matched to product risk, not
  copied blindly.
- CI should enforce the boring rules so review can focus on behavior and design.
- `const` is free performance; use it everywhere it compiles.
- Performance regressions are easiest to catch immediately after the change that caused them.
  Profile before merging, not six months later.
- Accessibility failures discovered late in a project are expensive to fix. Add semantics labels
  as you build each widget, not in a post-hoc pass.
- Every unhandled exception that reaches a user is a trust failure. Design error boundaries first.

Treat this document as a baseline plus extensions. Tighten it for higher-risk apps, and relax
optional guidance only with a deliberate reason.