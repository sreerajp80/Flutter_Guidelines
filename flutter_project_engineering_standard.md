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
| `Production App Extension` | Apps shipped to real users, external QA, or store review | Release, CI, UX, and environment discipline |
| `Sensitive Data Extension` | Apps handling auth secrets, financial data, health data, PII, or locally encrypted content | Stronger security, storage, logging, and backup rules |

A simple internal tool may use only `Core Baseline`.
A public consumer app will usually use `Core Baseline` plus `Production App Extension`.
An authenticator, password manager, finance, or health app will usually use all three.

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
7. Security and logging policy are product requirements, not cleanup work.
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
|-- android/                  # Optional for non-Android targets or packages
|-- ios/                      # Optional for non-iOS targets or packages
|-- windows/                  # Optional for non-Windows targets
|-- linux/                    # Optional
|-- macos/                    # Optional
|-- assets/
|   |-- config/               # app_config.json (About screen source of truth)
|   |-- fonts/
|   |-- icons/
|   `-- images/
|       |-- 2.0x/
|       `-- 3.0x/
|-- docs/
|   `-- GUIDELINES_MANIFEST.md
|-- plans/                    # change planning logs
|-- change_log/               # implemented change logs
|-- lib/
|-- test/
|-- integration_test/         # Required only when end-to-end coverage applies
|-- .github/workflows/
|-- CLAUDE.md                 # Mandatory project-root AI instructions (MUST)
|-- analysis_options.yaml
|-- pubspec.yaml
|-- README.md
`-- .gitignore
```

Application repositories SHOULD commit `pubspec.lock`.
Packages and plugins SHOULD follow normal package conventions.

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
2. Platform-specific FFI or native bindings (e.g. `sqfliteFfiInit()` for Windows/Linux desktop)
3. Secure storage or key material bootstrap
4. Database initialization and schema migration
5. App config / flavor loading
6. Logging infrastructure initialization
7. App lifecycle observer registration
8. `runApp(...)`

Document the actual sequence in `docs/architecture.md` for the project.

Each initialization step MUST handle its own failure gracefully and surface a safe error state
rather than crashing silently.

---

## 5. Environment And Build Configuration

This section is optional for `Core Baseline` projects and applies fully under
`Production App Extension`.

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
order. This shape is required because Flutter handles the flavor signal differently on each
platform target:

1. `APP_FLAVOR` — a custom, non-reserved name. This is what desktop builds (Windows, Linux,
   macOS) pass via `--dart-define=APP_FLAVOR=<value>`, because Flutter does not currently
   accept `--flavor` for those targets and `FLUTTER_APP_FLAVOR` cannot be set via
   `--dart-define` (see below).
2. `FLUTTER_APP_FLAVOR` — the framework-owned name. The Flutter tool injects this
   automatically as a compile-time define whenever `--flavor` is passed to `flutter run` or
   `flutter build`. This is the path used on Android and iOS.

> **Framework reservation (Flutter ≥ 3.19).** The build fails with
> `Target kernel_snapshot_program failed: Error: FLUTTER_APP_FLAVOR is used by the framework
> and cannot be set using --dart-define or --dart-define-from-file` if any build command
> tries to pass `FLUTTER_APP_FLAVOR` explicitly. The two-variable pattern below is what lets
> a single `AppFlavorConfig` work uniformly on Android, iOS, and desktop.

```dart
enum AppFlavor { dev, prod }

class AppFlavorConfig {
  AppFlavorConfig._(this.flavor);

  // Explicit value passed by desktop builds via --dart-define=APP_FLAVOR=<value>.
  // Empty when not provided.
  static const _appFlavorValue = String.fromEnvironment('APP_FLAVOR');

  // Auto-injected by Flutter on Android/iOS when --flavor is passed.
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

  String get appName => isDev ? 'MyApp Dev' : 'MyApp';
  bool get showEnvironmentBanner => isDev;
  bool get enableVerboseLogging => isDev;
}
```

The modern equivalent of `String.fromEnvironment('FLUTTER_APP_FLAVOR', ...)` is the global
`appFlavor` constant from `package:flutter/services.dart`. Either accessor is acceptable;
the two-variable resolution above MUST be preserved on top of whichever accessor is chosen,
otherwise desktop builds have no way to communicate the flavor.

#### Build And Run Command Conventions

| Platform | Pattern | Why |
|----------|---------|-----|
| Android, iOS | `--flavor <name>` only | Flutter auto-injects `FLUTTER_APP_FLAVOR`; passing `--dart-define=FLUTTER_APP_FLAVOR=...` fails the build. |
| Windows, Linux, macOS desktop | `--dart-define=APP_FLAVOR=<name>` only | `--flavor` is not accepted on these targets; the reserved `FLUTTER_APP_FLAVOR` name cannot be used via dart-define. |

```bash
# Android / iOS — --flavor only
flutter run --flavor dev
flutter run --flavor prod
flutter build apk --flavor prod --release

# Windows / Linux / macOS desktop — APP_FLAVOR dart-define only
flutter run -d windows --dart-define=APP_FLAVOR=dev
flutter build windows --release --dart-define=APP_FLAVOR=prod
```

Native Android and iOS flavor names SHOULD stay aligned with the Dart flavor value.

### 5.3 Android Flavor Setup

When Android flavors are used, the project SHOULD define product flavors in
`android/app/build.gradle.kts` (Kotlin DSL is the default for new projects on Flutter ≥ 3.41;
Groovy DSL `build.gradle` is still supported in inherited projects but uses
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

#### Toolchain Requirements (Flutter ≥ 3.38)

- **Java 17 minimum.** Flutter 3.38 raised the minimum required JDK to 17, matching the
  Gradle 8.14 minimum. Java 11 builds will fail. Verify with `java -version`.
- **AGP 9 is currently NOT supported.** As of Flutter 3.41, the team has paused AGP 9
  migration while auditing backward compatibility. Do not upgrade `android/build.gradle.kts`
  to AGP 9 even if Android Studio prompts to. Stay on AGP 8.x until guidance changes.
- **R8 full mode is the default** since AGP 8.0. This is more aggressive about removing
  seemingly-unused classes — make ProGuard keep rules deliberate (see
  `docs/flutter_build_flavors_guide.md` ProGuard section).

#### 5.3.1 Android 16 KB Page Size Compliance

Apps targeting Android 15+ MUST support 16 KB memory pages. Google Play enforces this for
new and updated apps from **November 1, 2025**, with a broader cutoff of **May 31, 2026**.
Non-compliant apps will be blocked from publication.

Build tooling on Flutter ≥ 3.24 is already 16 KB-aligned. The risk lies in **precompiled
native libraries (`.so` files) inside dependencies** that were compiled with 4 KB alignment.
Common offenders include older builds of `ffmpeg_kit_flutter`, image processing packages,
and any package that bundles its own `.so` files.

Required actions under `Production App Extension`:

- Run `flutter build appbundle --analyze-size` and inspect the `lib/` ABI directory listing
  for any third-party `.so` files. Verify the publisher's release notes confirm 16 KB
  alignment.
- Test the release build on an emulator configured with 16 KB pages
  (Android Studio → Device Manager → Edit → Advanced settings → "Page size: 16 KB") before
  every Play Store submission.
- Document any dependency that has not yet shipped a 16 KB-aligned release as a release-blocker
  in `architecture.md §21 Known Risks`.

### 5.4 iOS Flavor Setup

When iOS flavors are used, each flavor requires a separate Xcode scheme and xcconfig file pair.

#### iOS Deployment Target And UIScene Migration (Flutter ≥ 3.38)

- **Minimum iOS deployment target: iOS 13.** Flutter 3.41 raised the minimum from iOS 12
  to iOS 13. Set `platform :ios, '13.0'` in `ios/Podfile` and the iOS Deployment Target
  in Xcode under Build Settings.
- **UIScene lifecycle is mandatory.** Apple requires UIScene adoption for any UIKit app
  built with the iOS 26 SDK; the App Store requires iOS 26 SDK builds, and that deadline
  (April 2026) is now in force. As of Flutter 3.41, UIScene is enabled by default and the
  Flutter CLI automatically migrates apps with an unmodified `AppDelegate`. Watch the build
  log for `Finished migration to UIScene lifecycle` (success) or migration warnings (manual
  work required).
- **For projects with a customized `AppDelegate`** (analytics initialization, deep link
  handlers, custom plugin registration) the migration is manual:
  - `AppDelegate` must adopt `FlutterImplicitEngineDelegate`.
  - Plugin registration must move from `application:didFinishLaunchingWithOptions:` to a
    new `didInitializeImplicitFlutterEngine` callback.
  - `Info.plist` must include a `UIApplicationSceneManifest` (Application Scene Manifest)
    entry. The Flutter migrator usually adds this automatically.
  - Plugin developers using lifecycle events must implement `FlutterSceneLifeCycleDelegate`
    and register via `registrar.addSceneDelegate(self)`.
- See the official guide at `docs.flutter.dev/release/breaking-changes/uiscenedelegate` for
  the full migration steps. Test on iOS 26 simulator/device before the next App Store
  submission.

#### Xcode Scheme And xcconfig

Directory structure:

```text
ios/
|-- Flutter/
|   |-- dev/
|   |   |-- Debug.xcconfig
|   |   `-- Release.xcconfig
|   `-- prod/
|       |-- Debug.xcconfig
|       `-- Release.xcconfig
```

Each xcconfig file should set the bundle identifier and display name override:

```
// ios/Flutter/dev/Debug.xcconfig
#include "Generated.xcconfig"
FLUTTER_TARGET=lib/main.dart
BUNDLE_ID_SUFFIX=.dev
DISPLAY_NAME=MyApp Dev
```

In Xcode, create one scheme per flavor:
- `dev` scheme: uses `Debug.xcconfig` for run, `Release.xcconfig` for archive.
- `prod` scheme: uses `prod/Release.xcconfig` for archive and store submission.

Each flavor SHOULD have its own `Info.plist` overrides for `CFBundleIdentifier` and
`CFBundleDisplayName` using `$(BUNDLE_ID_SUFFIX)` and `$(DISPLAY_NAME)` variables.

Provisioning profiles MUST be set per scheme. Do not share production profiles with dev builds.

### 5.5 Windows Desktop Build Setup

Windows desktop does not use Android product flavors and does not currently accept the
Flutter `--flavor` argument. Environment separation is achieved through
`--dart-define=APP_FLAVOR=<value>` at build time, read by the `AppFlavorConfig` pattern from
section 5.2. The dart-define name MUST be `APP_FLAVOR` (or any other non-reserved name) and
MUST NOT be `FLUTTER_APP_FLAVOR` — that name is owned by the framework and any attempt to
set it via `--dart-define` fails the build.

Additional Windows-specific setup required before any DB or FFI work can run:

```dart
// In main() before runApp, for Windows and Linux desktop:
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

**Window constraints** — set a minimum window size to prevent layouts breaking at small sizes.
This is the concise `setMinimumSize` form; `docs/flutter_build_flavors_guide.md` shows the
equivalent `WindowOptions` + `waitUntilReadyToShow` form (which also sets the initial size and
centres the window). Both configure the same `window_manager`; pick one per project. The flavors
guide is the canonical desktop-setup reference.

```dart
// In main() after WidgetsFlutterBinding.ensureInitialized() and before runApp.
import 'dart:io';
import 'package:flutter/widgets.dart';
import 'package:window_manager/window_manager.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();

  if (Platform.isWindows || Platform.isLinux || Platform.isMacOS) {
    await windowManager.ensureInitialized();
    await windowManager.setMinimumSize(const Size(800, 600));
    await windowManager.setTitle('MyApp');
  }

  runApp(const MyApp());
}
```

**MSIX packaging** for Windows distribution:

```yaml
# pubspec.yaml
msix_config:
  display_name: MyApp
  publisher_display_name: YourName
  identity_name: com.yourcompany.myapp
  msix_version: 1.0.0.0
  logo_path: assets/icons/icon.png
  capabilities: 'runFullTrust'
  # Microsoft Store submission: produce a multi-architecture .msixbundle.
  # Sideloading: a single architecture is sufficient — drop arm64.
  architecture: x64, arm64
```

Build command:

```bash
# `flutter pub run` was deprecated; the toolchain now requires `dart run`.
dart run msix:create
```

**Keyboard shortcuts** — register app-wide shortcuts using `Shortcuts` and `Actions` widgets at
the root. Document all registered shortcuts in `docs/architecture.md`.

**Context menus** — use `ContextMenuRegion` or `GestureDetector` with `onSecondaryTap` for
right-click context menus on desktop. Do not assume touch-only interaction patterns.

### 5.6 Artifact Selection

Under `Production App Extension`:

- Android: Use split APKs when distributing directly; use `.aab` for Google Play submission.
- iOS: Use `.ipa` exported via Xcode organizer or `flutter build ipa`.
- Windows: Use MSIX for distribution. Raw `.exe` is acceptable for internal tools only.
- Avoid universal release APKs unless there is a specific distribution reason.

`--target-platform` does not replace `--split-per-abi`.

---

## 6. UI And UX Baseline

This section applies to user-facing applications. It is advisory for infrastructure packages.

### 6.1 Theme And Design Tokens

- Use one source of truth for theme configuration.
- Centralize colors, typography, spacing, radius, and motion values.
- Prefer semantic names over raw literals in widgets.
- If the product supports both light and dark themes, both MUST be tested.
- Design token constants SHOULD live in a single reviewed file such as `lib/app/theme/tokens.dart`.
- Never hardcode `Color(0xFF...)` literals inside widget `build` methods; always reference a token.

#### Material 3 Is The Default

`useMaterial3: true` has been the default since Flutter 3.16 and the explicit flag is no
longer needed in `ThemeData`. New projects automatically receive M3 styling. `useMaterial3:
false` and Material 2 support are slated for deprecation per Flutter's deprecation policy —
do not introduce new code that depends on M2 visuals.

#### Forward-Looking: Material And Cupertino Decoupling

Google has signaled that the Material and Cupertino libraries will move to versioned
`pub.dev` packages over 2026 as part of Flutter's decoupling effort. Imports may change
from `package:flutter/material.dart` to a dedicated package; migration tooling is expected.
No action is required today, but be prepared for an import-rewrite pass when the change
ships.

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

- Animations MUST respect `MediaQuery.of(context).disableAnimations`. If `true`, skip or
  complete animations instantly.
- Looping animations MUST be paused when the app is in the background (`AppLifecycleState.paused`).
- Prefer animating `Transform` and `Opacity` over properties that trigger layout recalculation
  (such as `Padding`, `SizedBox` dimensions, or `Align` factors). `Transform` and `Opacity` are
  composited on the GPU and do not trigger layout or paint passes, which makes them cheaper by
  default. Animating layout-affecting properties via `AnimatedPadding`, `AnimatedContainer`, or
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
- Do not hardcode top or bottom padding. Always read from `MediaQuery.of(context).padding`.
- Test layouts on a device or emulator with a display notch, punch-hole camera, and gesture
  navigation bar. These three configurations expose the most common safe-area bugs.
- For edge-to-edge designs on Android 15+: edge-to-edge is **enforced by default** for apps
  targeting SDK 35+ on Android 15+. You cannot opt out without a documented compatibility
  justification. Set `SystemChrome.setEnabledSystemUIMode` appropriately and ensure
  `WindowInsetsController` padding is applied to your content via `MediaQuery`. Test on a
  device with three-button navigation, gesture navigation, and tablet display cutout.

### 6.9 UX Rules For Production Apps

Under `Production App Extension`:

- Forms MUST validate before submission and show actionable errors.
- Destructive actions MUST require confirmation or provide undo.
- Layouts SHOULD work on common phone sizes (360 dp to 430 dp width) before release.
- Layouts SHOULD also be tested at 600 dp width (small tablet) if the app targets tablets.
- Route definitions SHOULD be centralized.
- Deep links, if supported, SHOULD have integration coverage.
- Every tap target MUST be at least 48 × 48 dp on mobile.
- Every tap target on desktop MUST be at least 32 × 32 dp.

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

- Normal text (below 18 sp or 14 sp bold): minimum contrast ratio **4.5 : 1** (WCAG AA).
- Large text (18 sp or above, or 14 sp bold): minimum contrast ratio **3.0 : 1** (WCAG AA).
- Interactive component boundaries and focus indicators: minimum **3.0 : 1** against adjacent colors.
- Never communicate information using color alone. Always pair color with a label, icon, or
  pattern.
- Both light and dark themes MUST independently pass contrast requirements.

### 7.3 Semantics

- Use `Semantics` widgets to provide labels for any custom widget that assistive technology cannot
  infer from its visual content.
- Use `excludeSemantics: true` on decorative images and icons that carry no meaning.
- Custom interactive widgets (gestures, custom painters, canvas) MUST provide `onTap`, `label`,
  and `hint` semantics.
- `Tooltip` widgets automatically contribute to semantics on long-press. Every icon-only control
  MUST have one — see 7.8, which is a hard requirement, not a suggestion.
- Use `MergeSemantics` when multiple widgets form a single logical unit (e.g. a list tile with an
  icon and a label).
- Do not suppress semantics on content that communicates state (loading spinners, error badges).

### 7.4 Font And Text Scaling

> **API note (Flutter ≥ 3.12).** `textScaleFactor` is deprecated and will be removed in a
> future Flutter version. Use `TextScaler` instead. The new API supports Android 14's
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

- Test critical flows with **TalkBack** on Android before each release.
- Test critical flows with **Narrator** on Windows desktop before each release.
- Test with **VoiceOver** on iOS and macOS if those platforms are supported.
- At minimum: app navigation, primary data entry flow, and error states must be fully operable
  under TalkBack/Narrator.

### 7.7 Compliance Verification Methods

The following named methods are the expected ways to prove accessibility compliance before
shipping. "We checked" is not sufficient — the method used should be recorded in the release
checklist.

**Touch target verification:**
Enable `debugPaintSizeEnabled = true` in a debug build and visually confirm interactive elements
meet the 48 × 48 dp minimum on mobile. Alternatively, enable Flutter DevTools → Widget Details
and inspect `Size` values for interactive widgets.

**Contrast verification:**
Use the [Material Design Color System contrast tool](https://m3.material.io/styles/color/system/how-the-system-works),
the [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/), or a verified
pub package such as `color_contrast_checker` to verify foreground/background pairs. Both light
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
| Icon-only `InkWell`, `GestureDetector`, `IconSlider`, custom painters | wrap in `Tooltip(message: ...)` **and** provide `Semantics(label:, hint:)` |
| `BottomNavigationBarItem`, `NavigationDestination`, `NavigationRailDestination` shown **without** a persistent visible label | `tooltip:` on the destination |
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
  tooltip: AppLocalizations.of(context).actionDelete, // short, localized
  onPressed: _delete,
)

// Custom icon-only control: tooltip + explicit semantics.
Tooltip(
  message: l10n.actionShare,
  child: Semantics(
    button: true,
    label: l10n.actionShare,
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

Run it for every screen under test, in all three locales, as part of the screen's widget test.

---

## 8. Localization And Internationalization

This section is `Core Baseline` and applies to every user-facing app repository.

**Every app ships three languages: English (`en`), Malayalam (`ml`) and Sanskrit (`sa`).** There is
no single-language app. The app starts in the system language when that is one of the three and in
English otherwise, and the user can change the language inside the app at any time. Every feature,
every screen, and every string — labels, menus, buttons, tooltips, dialogs, notifications, errors,
empty states, About content — renders in the language the user selected.

| Sub-section | Rule |
|---|---|
| 8.1 | Minimum `MaterialApp` / `l10n.yaml` setup |
| 8.2 | String externalization into ARB — no user-visible literals |
| 8.3 | The three mandatory languages, the Sanskrit delegate gap, fonts |
| 8.4 | In-app language selection, persistence, resolution order |
| 8.5 | Sanskrit & Malayalam quality — rules, orthography, standard glossary |
| 8.6 | Short UI labels vs. descriptive text |
| 8.7 | Per-feature language completeness and the parity test |
| 8.8 | RTL layout support |
| 8.9 | Locale-sensitive formatting |

### 8.1 Minimum Setup (All Apps)

Every Flutter app MUST declare `flutter_localizations` delegates and `supportedLocales` in the
root `MaterialApp` or `CupertinoApp`. Without this, some Material widgets (date pickers, number
inputs, dialog buttons) render incorrectly on devices with non-English system locales.

```dart
import 'package:flutter_localizations/flutter_localizations.dart';

MaterialApp(
  localizationsDelegates: const [
    GlobalMaterialLocalizations.delegate,
    GlobalWidgetsLocalizations.delegate,
    GlobalCupertinoLocalizations.delegate,
  ],
  supportedLocales: const [
    Locale('en'), // English — template locale
    Locale('ml'), // Malayalam
    Locale('sa'), // Sanskrit (Devanagari)
  ],
);
```

The three locales above are **fixed**: every app declares exactly these, in this order. See 8.3 for
the extra delegate Sanskrit requires and 8.4 for the in-app switcher.

Add to `pubspec.yaml`:

```yaml
dependencies:
  flutter_localizations:
    sdk: flutter
  intl: ^0.20.2   # Example — pin the current line at project start and update deliberately.
```

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
template-arb-file: app_en.arb
output-localization-file: app_localizations.dart
output-class: AppLocalizations
nullable-getter: false
synthetic-package: false
```

`synthetic-package: false` writes the generated file into your `lib/` tree (preferred for
import clarity); leaving the default `true` writes it into the synthetic `flutter_gen` package.
Pick one and document the choice.

**Android App Bundle language splitting MUST be disabled** in `android/app/build.gradle.kts` (or `build.gradle`):

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

Google Play defaults to language splitting when delivering Android App Bundles (.aab). If a user downloads the app on an English phone, Play only installs English resources. If the user later switches to Malayalam or Sanskrit inside the app, the strings will be missing. Setting `enableSplit = false` ensures that the bundle retains all language resources on the device.

### 8.2 String Externalization (Mandatory, All Apps)

Every app MUST externalize its user-visible strings into ARB files. This is not optional and does
not wait for a translation request.

ARB files are the Flutter equivalent of Android's `res/values/strings.xml`. On Android we create
`strings.xml` from day one, so a new language is just a new `values-xx/strings.xml`. We follow the
same habit in Flutter — with the difference that all three of our languages exist from day one.

Required for every app:

- `l10n.yaml` MUST exist at the project root (see 8.1).
- All three ARB files MUST exist: `lib/l10n/app_en.arb` (template), `lib/l10n/app_ml.arb`,
  `lib/l10n/app_sa.arb`.
- Every user-visible string MUST be defined in the ARB files and read through
  `AppLocalizations.of(context)`. A raw string literal in a widget is not allowed.
- Every key MUST exist in **all three** files with a real translation. An English value copied into
  `app_ml.arb` or `app_sa.arb` as a placeholder is an unfinished feature, not a translation (8.7).
- Every ARB entry MUST have an `@key` description in the template file, so a translator has context.
  Where a key is UI chrome rather than prose, say so in the description (it drives the length budget
  in 8.6), e.g. `"description": "Toolbar button label. Keep to one or two words."`.

**Narrow exceptions** — these MAY stay as plain Dart literals, because a user never reads them:

| Allowed as a literal | Example |
|---|---|
| Log and debug messages | `AppLogger.d('cache miss for $id')` |
| Exception messages not shown in the UI | `throw StateError('db not initialized')` |
| Technical identifiers | asset paths, route names, map/JSON keys, `Semantics` test tags |
| Developer-only screens | a debug menu that never ships to users |

Anything a real user reads — screen titles, buttons, labels, hints, error text shown on screen,
empty states, snackbars, dialogs, notification text — goes in the ARB file.

Directory structure:

```text
lib/
`-- l10n/
    |-- app_en.arb   # REQUIRED — English, the template locale
    |-- app_ml.arb   # REQUIRED — Malayalam
    `-- app_sa.arb   # REQUIRED — Sanskrit (Devanagari)
```

Example ARB file:

```json
{
  "@@locale": "en",
  "appTitle": "My App",
  "@appTitle": { "description": "The application title" },
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

Use in code via `AppLocalizations.of(context)!.appTitle` (or `AppLocalizations.of(context).appTitle`
when `nullable-getter: false` is set). Never use raw string literals for user-visible text.

**Adding a fourth language later.** Because the strings are already externalized, this is a small,
mechanical job:

1. Add `lib/l10n/app_<code>.arb` with the same keys and translated values.
2. Add `Locale('<code>')` to `supportedLocales` and to the in-app language picker (8.4).
3. Run `flutter gen-l10n`.

No screen or widget code changes.

### 8.3 The Three Mandatory Languages

| Locale | Language | Script | Role |
|---|---|---|---|
| `en` | English | Latin | Template ARB, ultimate fallback |
| `ml` | Malayalam | Malayalam | Full UI translation |
| `sa` | Sanskrit | Devanagari | Full UI translation (see 8.5) |

All three MUST be listed in `supportedLocales`, all three ARB files MUST be complete, and the
in-app picker (8.4) MUST offer all three plus "System default".

#### 8.3.1 Sanskrit has no Flutter framework translation — install a fallback delegate

`flutter_localizations` ships `GlobalMaterialLocalizations` and `GlobalCupertinoLocalizations` for a
long list of locales, **but not for `sa`**. Adding `Locale('sa')` to `supportedLocales` without
handling this throws at runtime the first time a Material widget needs framework strings (date
picker, dialog buttons, text-selection menu, `Scaffold` semantics labels).

Every app MUST therefore install a delegate that serves framework strings for `sa` from a supported
locale, while the app's own strings (via `AppLocalizations`) stay Sanskrit. Use English as the
framework fallback — not Hindi — so no Hindi text can ever leak into a Sanskrit UI.

```dart
// lib/l10n/sa_material_localizations.dart
//
// flutter_localizations has no Sanskrit ('sa') translation. This delegate
// answers for Locale('sa') by loading the English framework strings, so the
// app's own Sanskrit strings render while Material widgets still work.
class SaMaterialLocalizationsDelegate
    extends LocalizationsDelegate<MaterialLocalizations> {
  const SaMaterialLocalizationsDelegate();

  @override
  bool isSupported(Locale locale) => locale.languageCode == 'sa';

  @override
  Future<MaterialLocalizations> load(Locale locale) =>
      GlobalMaterialLocalizations.delegate.load(const Locale('en'));

  @override
  bool shouldReload(covariant LocalizationsDelegate old) => false;
}

class SaCupertinoLocalizationsDelegate
    extends LocalizationsDelegate<CupertinoLocalizations> {
  const SaCupertinoLocalizationsDelegate();

  @override
  bool isSupported(Locale locale) => locale.languageCode == 'sa';

  @override
  Future<CupertinoLocalizations> load(Locale locale) =>
      GlobalCupertinoLocalizations.delegate.load(const Locale('en'));

  @override
  bool shouldReload(covariant LocalizationsDelegate old) => false;
}

class SaWidgetsLocalizationsDelegate
    extends LocalizationsDelegate<WidgetsLocalizations> {
  const SaWidgetsLocalizationsDelegate();

  @override
  bool isSupported(Locale locale) => locale.languageCode == 'sa';

  @override
  Future<WidgetsLocalizations> load(Locale locale) =>
      GlobalWidgetsLocalizations.delegate.load(const Locale('en'));

  @override
  bool shouldReload(covariant LocalizationsDelegate old) => false;
}
```

Register the Sanskrit delegates **before** the global ones, so they win for `sa`:

```dart
MaterialApp(
  localizationsDelegates: const [
    AppLocalizations.delegate,
    SaMaterialLocalizationsDelegate(),
    SaCupertinoLocalizationsDelegate(),
    SaWidgetsLocalizationsDelegate(),
    GlobalMaterialLocalizations.delegate,
    GlobalWidgetsLocalizations.delegate,
    GlobalCupertinoLocalizations.delegate,
  ],
  supportedLocales: const [Locale('en'), Locale('ml'), Locale('sa')],
);
```

A widget test MUST cover this: pump the app with `locale: Locale('sa')`, open a date picker and a
dialog, and assert no exception. This is the single most likely Sanskrit runtime failure.

#### 8.3.2 `intl` formatting under Sanskrit

`intl` has no `sa` date or number symbols, so `DateFormat.yMMMMd('sa')` throws. Format with a
supported locale while the surrounding UI text stays Sanskrit:

```dart
/// Locale to hand to `intl`. Sanskrit has no CLDR data, so dates and numbers
/// are formatted with English patterns while the UI text stays Sanskrit.
String formattingLocale(Locale locale) =>
    DateFormat.localeExists(locale.toLanguageTag())
        ? locale.toLanguageTag()
        : 'en';
```

Use `formattingLocale(...)` everywhere 8.9 calls for a locale argument. Do not fall back to `hi`.

#### 8.3.3 Fonts and script coverage

Malayalam and Devanagari glyphs are not guaranteed on every Android device or on Windows, and a
missing glyph renders as a blank box — a silent, ship-blocking bug for two of our three languages.

- The app MUST either bundle fonts covering both scripts (e.g. Noto Sans Malayalam and Noto Sans
  Devanagari) or declare an explicit `fontFamilyFallback` chain and verify rendering on a clean
  device image for each supported platform.
- Bundled fonts MUST be subset where possible and recorded under the font-licensing rules (17.4).
- Verification is per release: open every screen in `ml` and in `sa` on a clean device and confirm
  no boxes, no clipped ascenders/descenders (Malayalam and Devanagari are taller than Latin), and
  no overflow. Do not hard-code text container heights (7.4).

### 8.4 In-App Language Selection (Mandatory)

The language is the user's choice, not the device's alone.

**Resolution order** — the app resolves its locale as:

1. the language the user saved inside the app, if any;
2. otherwise the system locale, when its language code is `en`, `ml` or `sa`;
3. otherwise English.

Rules:

- The choice MUST persist across restarts (`SharedPreferences` key `app_language`, values
  `system` | `en` | `ml` | `sa`) and MUST be read **before** the first frame, so the app never
  flashes the wrong language at startup (see 4.5).
- Changing the language MUST apply **immediately and app-wide**, without restarting the app and
  without popping the user back to the home screen.
- The picker MUST live in Settings, MUST offer **System default** as an explicit first option, and
  MUST list each language in its own script (its endonym), not translated:

  | Option | Shown as |
  |---|---|
  | System default | localized label, e.g. "System default" / "സിസ്റ്റം സ്വതവേ" / "तन्त्रसिद्धम्" |
  | English | `English` |
  | Malayalam | `മലയാളം` |
  | Sanskrit | `संस्कृतम्` |

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
  static const List<String> supported = ['en', 'ml', 'sa'];

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
  supportedLocales: const [Locale('en'), Locale('ml'), Locale('sa')],
  localeResolutionCallback: (deviceLocale, supported) {
    for (final l in supported) {
      if (l.languageCode == deviceLocale?.languageCode) return l;
    }
    return const Locale('en'); // system language is none of the three
  },
);
```

### 8.5 Sanskrit & Malayalam Quality — Standard UI Glossary

Every app ships three languages: English, Malayalam, and Sanskrit (8.3). Both Malayalam and Sanskrit
demand deliberate linguistic care to avoid common pitfalls: Hindi leakage in Sanskrit due to the
shared Devanagari script, and awkward English transliterations or calques in Malayalam.

#### 8.5.1 Sanskrit Quality Rules (Pure Sanskrit, Never Hindi)

Sanskrit's derivational system — verbal roots (`धातु`), prefixes (`उपसर्ग`), suffixes
(`कृत्` / `तद्धित प्रत्यय`), and compounds (`समास`) — can derive a term for any UI concept.

Because Sanskrit and Hindi share the Devanagari script, Hindi text *looks* like Sanskrit to anyone
who does not read it. Never use Hindi anywhere as a substitute, crutch, or fallback for Sanskrit.
`app_sa.arb` MUST be authentic, uncompromised Sanskrit.

- **Classical vocabulary and grammar**: Use authentic Sanskrit nominal stems, proper case endings,
  and correct verbal forms (e.g. polite passive imperative `परिवर्त्यताम्`, not Hindi `बदलें`).
- **No transliterated English loans**: Never transliterate English words into Devanagari when a
  standard Sanskrit word exists (`सेटिंग्स` is Hindi/English in Devanagari; use `विन्यासः`).
- **No Hindi function words or syntax**: Do not use Hindi postpositions (`का`, `की`, `के`, `को`,
  `में`, `से`, `पर`), copulas (`है`, `हैं`, `था`, `थे`, `थी`, `हूं`), or verb endings (`करें`,
  `करना`, `रहा`, `गया`, `चाहिए`).
- **No nukta consonants**: The Perso-Arabic consonants with nukta (`क़`, `ख़`, `ग़`, `ज़`, `ड़`, `ढ़`,
  `फ़`) do not occur in Sanskrit.
- **Strict grammatical agreement**: Participles and adjectives must agree with their subject in
  gender and case. In "No data found", `दत्तांशः` is masculine nominative, so the participle must be
  `प्राप्तः` and the indefinite pronoun `कोऽपि`: `न कोऽपि दत्तांशः प्राप्तः` (never neuter `न किमपि दत्तांशं प्राप्तम्`).
- **Valid morphological derivation**:
  - Do not invent verbs by slapping verbal endings onto nouns. "Copy" is `प्रतिलिख्यताम्` (from verb
    root `लिख्` with `प्रति`) or `प्रतिलिपिः क्रियताम्`, never pseudo-verb `प्रतिलिप्यताम्`.
  - The past passive participle for "Copied" is `प्रतिलिखितम्` (or `प्रतिलिपीकृतम्`), never `प्रतिलिपितम्`.
  - Causative passive of `या` (go) is `निर्याप्यते` / `निर्याप्यताम्` (Export), never `निर्यात्यताम्`.
  - "Confirm" is `स्थिरीक्रियताम्` or `दृढीक्रियताम्` (let it be made firm), never `संपुष्यताम्` (which means "let it be nourished").
  - Do not use Hindi loanwords for concepts that have native Sanskrit terms (use `उपयोक्तृविवरणम्` for Account, never Hindi `खाता`; `लेखा` strictly means a line/furrow).
  - Use `ध्वनिः` for audio/sound to avoid confusion with `शब्दः` (Word).
- **Form conventions**:
  - A button or menu item (action commanding the app): polite `-ताम्` imperative (`लोट्`). For a verb
    that takes an object it is passive (`कर्मणि`), e.g. `रक्ष्यताम्` (Save), `अन्विष्यताम्` (Search); for a
    verb that takes no object it is impersonal (`भावे`), e.g. `निष्क्रम्यताम्` (Exit).
  - A title, tab, label, heading, or status: nominal / abstract noun, e.g. `अन्वेषणम्` (Search), `विन्यासः` (Settings).
  - A confirmation or boolean response: indeclinable, e.g. `आम्` (Yes), `न` (No), `अस्तु` (OK).
  - Direction words (Back, Next, Previous, More) are nominal or adverbial labels and MAY keep that
    form on a button, like Yes / No. Close and Exit are actions: on a button they MUST use the
    imperative (`पिधीयताम्`, `निष्क्रम्यताम्`); the nominal form (`निष्क्रमणम्`) is for titles and labels.
- **Punctuation**: Use the **daṇḍa** `।` to end a sentence in descriptive prose; UI labels take no terminator.
- **Locale marker**: Always set `"@@locale": "sa"` at the top of `app_sa.arb`.
- **Pre-release review**: Machine translation tools commonly output Hindi for Sanskrit requests. All
  Sanskrit ARB entries MUST be reviewed before release.

**Forbidden markers.** None of these tokens may appear anywhere in `app_sa.arb`, `assets/config/app_config.json`,
or any `*_sa.*` asset file. They are reliable Hindi giveaways and make a good grep-based gate:

```text
है  हैं  था  थे  थी  हूं  हो  करें  करना  करके  रहा  रही  रहे  गया  गयी  चाहिए
नहीं  और  लेकिन  क्या  आपका  आपकी  आपके  हमारा  मेरा  कृपया  सेटिंग्स  ऐप
◌़ (nukta U+093C, and the precomposed nukta letters U+0958–U+095F)
```

```bash
# CI gate: fail the build if any Hindi marker appears in the Sanskrit ARB,
# app_config.json, or Sanskrit asset files.
# Uses PCRE (-P) with lookarounds so standalone copulas/words are not confused with
# legitimate Sanskrit roots (e.g. स्थाप्यताम्, स्थानम्) or indeclinables (यथा, तथा, कथा).
# Word edges: whitespace, quotes, brackets, punctuation, daṇḍa, XML/HTML tag edges (< >),
# and Markdown marks (* _ ` # | : ; ~ -) so help files in Markdown are checked too.
PATTERN='(?<=[\s"'\''([{<>।,*_`#|:;~-]|^)(?:था|थे|थी|हो|है|हैं|हूं|और)(?=[\s"'\''\)\]}<>।,.\?!*_`#|:;~-]|$)|करें|करना|करके|रहा|रही|रहे|गया|गयी|चाहिए|नहीं|लेकिन|क्या|कृपया|सेटिंग्स|ऐप|\x{093C}|[\x{0958}-\x{095F}]'

mapfile -t FILES < <(find . -path '*/build' -prune -o -type f \( \
    -name 'app_sa.arb' -o \
    -path '*/assets/*_sa.*' -o \
    -path '*/assets/config/app_config.json' \) -print)

if [ "${#FILES[@]}" -eq 0 ]; then
  echo 'No Sanskrit files found — every app ships app_sa.arb.'; exit 1
fi

# Self-test: the pattern must still catch a Hindi copula in ARB/JSON, XML and Markdown text.
for sample in '"greeting": "है"' '<b>है</b>' 'यह **है**' 'यह `है`' 'वह *था*'; do
  if ! printf '%s\n' "$sample" | LC_ALL=C.UTF-8 grep -qP "$PATTERN"; then
    echo "Sanskrit check self-test failed on: $sample"; exit 1
  fi
done

if LC_ALL=C.UTF-8 grep -nP "$PATTERN" "${FILES[@]}"; then
  echo 'Hindi markers found in Sanskrit text (standard 8.5).'; exit 1
fi
exit 0
```

> The gate is a smoke test, not a proof of correctness: passing it means no obvious Hindi marker is
> present, not that the Sanskrit is good. Human review still applies.
> Standalone words (था, थे, थी, हो, है, हैं, हूं, और) match only between word edges: whitespace,
> quotes, brackets, punctuation (including the daṇḍa `।`), the tag edges `<` and `>`, and the
> Markdown marks `*`, `_`, `` ` ``, `#`, `|`, `:`, `;`, `~`, `-`. So `यह **है**` in a help file fails
> the build, while legitimate Sanskrit such as `स्थाप्यताम्`, `स्थानम्`, `पुनःस्थाप्यताम्`, `यथा`,
> `तथा` and `कथा` never does.


#### 8.5.2 Malayalam Quality Rules (Natural Malayalam, Not English Transliterations)

Malayalam UI strings must sound natural and idiomatic to native Malayalam speakers.

- **Avoid lazy English transliterations; established loanwords allowed**: Do not phonetically
  transliterate English UI jargon into Malayalam script when standard, authentic Malayalam words exist.
  - Save: `സൂക്ഷിക്കുക` (never bare `സേവ്`).
  - Print: `അച്ചടിക്കുക` (never `പ്രിന്റ്`).
  - Vibration: `കമ്പനം` (never `വൈബ്രേഷൻ`).
  - Optional: `ഐച്ഛികം` (never `ഓപ്ഷണൽ`).
  - Number: `സംഖ്യ` (never `നമ്പർ`).
  - Page: `താൾ` (never `പേജ്`).
  - Widely established digital loanwords (such as `ഹോം`, `മെനു`, `പ്രൊഫൈൽ`, `അക്കൗണ്ട്`, `ഡൗൺലോഡ്`,
    `ഓഫ്‌ലൈൻ`, `തീം`, `ഫയൽ`, `ഫോൾഡർ`, `ലിങ്ക്`, `ലൈസൻസ്`) are accepted where no single native term
    carries universal recognition.
- **Action buttons use verb forms**: Action buttons commanding an operation MUST use the verbal
  form ending in `-ക്കുക` / `-ക` (`തിരുത്തുക`, `സൂക്ഷിക്കുക`, `നീക്കുക`, `തുറക്കുക`, `പുറത്തുകടക്കുക`,
  `ലോഗൗട്ട് ചെയ്യുക`), never a bare English noun or uninflected loan.
- **Accurate negation (`ഇല്ല` vs `അല്ല`)**:
  - `ഇല്ല` denotes non-existence, absence, or refusal to perform an action. For confirmation dialog
    action buttons (Yes / No), use **`അതെ` / `ഇല്ല`**.
  - `അല്ല` denotes negation of identity or qualification ("is not", e.g. `ശരിയല്ല`). Do not put
    `അല്ല` on a confirmation prompt's "No" button when the dialog asks if an action should be done.
- **Avoid ungrammatical standalone postpositions**: Postpositions like `കുറിച്ച്` govern an accusative
  noun (e.g. `ആപ്പിനെക്കുറിച്ച്`); standing alone as a screen title or heading, `കുറിച്ച്` is
  ungrammatical. Use `ആപ്പിനെക്കുറിച്ച്` for "About"; `വിവരണം` is reserved for Description.
- **Contextual accuracy over literal calques**:
  - Preferences: `താൽപ്പര്യങ്ങൾ` or `ഇഷ്ടങ്ങൾ` (matches Sanskrit `रुचयः`). `മുൻഗണനകൾ` strictly means
    **Priorities** (precedence/rank) and is a misleading false friend.
  - Apply (theme/filters): `പ്രയോഗിക്കുക` or `നടപ്പിലാക്കുക`. `ബാധകമാക്കുക` means legal liability/enforcement.
  - Sort: `ക്രമീകരിക്കുക` (arrange in order / sort sequence). `അടുക്കുക` means to stack or draw near.
- **Modern Unicode orthography**: Always use standard Unicode Malayalam atomic chillu characters (`ൺ`, `ൻ`, `ർ`, `ൽ`, `ൾ`). Avoid legacy ZWJ sequences or non-standard glyphs.

#### 8.5.3 Bad → Good Translations

| English | Bad (Hindi / English loan / Calque) | Good (Sanskrit) | Good (Malayalam) | Linguistic Rationale |
|---|---|---|---|---|
| Settings | सेटिंग्स / സെറ്റിംഗ്സ് | विन्यासः | ക്രമീകരണങ്ങൾ | Standard native terminology |
| Save | सेव करें / സേവ് | रक्ष्यताम् | സൂക്ഷിക്കുക | Polite imperative in SA; `-ക്കുക` verb in ML |
| Delete | डिलीट करें / ഡിലീറ്റ് | लुप्यताम् / विलुप्यताम् | ഇല്ലാതാക്കുക | Authentic verbal action |
| Cancel | कैंसिल / ക്യാൻസൽ | निरस्यताम् | റദ്ദാക്കുക | Native rejection/dismissal term |
| Copy | कॉपी करें / കോപ്പി | प्रतिलिख्यताम् | പകർത്തുക | `प्रति + लिख्` verb in SA; NOT `प्रतिलिप्यताम्` |
| Export | निर्यात करें / എക്സ്പോർട്ട് | निर्याप्यताम् | കയറ്റുമതി ചെയ്യുക | Correct causative passive of `या` in SA |
| Search | खोजें / സെർച്ച് | अन्वेषणम् (title) / अन्विष्यताम् (action) | തിരയുക | Distinct noun title vs. action button |
| No data found | कोई डेटा नहीं मिला / ഡാറ്റ ഇല്ല | न कोऽपि दत्तांशः प्राप्तः | വിവരങ്ങളൊന്നും കണ്ടെത്തിയില്ല | Gender agreement in SA (`दत्तांशः` masculine nom.) |
| Preferences | प्रेफरेंसेस / മുൻഗണനകൾ | रुचयः | താൽപ്പര്യങ്ങൾ / ഇഷ്ടങ്ങൾ | `മുൻഗണനകൾ` means priorities, not preferences |
| Confirm | संपुष्यताम् / കൺഫേം | स्थिरीक्रियताम् / दृढीक्रियताम् | സ്ഥിരീകരിക്കുക | `पुष्` means nourish; `स्थिरी` means confirm |
| Print | प्रिंट करें / പ്രിന്റ് | मुद्र्यताम् | അച്ചടിക്കുക | Standard Malayalam verb |
| About | ऐप के बारे में / കുറിച്ച് / परिचयः | विषयपरिचयः | ആപ്പിനെക്കുറിച്ച് | `കുറിച്ച്` is a bound postposition, not a title; `विषये` is locative ("regarding"). One term per meaning: `परिचयः` is Profile and `വിവരണം` is Description (8.5.4). |

#### 8.5.4 Standard UI Glossary

Use these exact terms across all apps, in all three languages. When a term you need is missing, add it
**here**, in this standard, rather than inventing inconsistent per-app variants. All short UI terms
MUST fit within the 22-character limit defined in Section 8.6.

**Review rule.** A new or changed Malayalam or Sanskrit glossary term MUST be reviewed by a fluent
reader before any app uses it. The change that adds the term lists it in its change log as
"needs native-reader review" until that review is done.

##### Navigation and structure

| English | Malayalam | Sanskrit |
|---|---|---|
| Home | ഹോം | गृहम् |
| Back | പിന്നോട്ട് | प्रत्यागमनम् |
| Next | അടുത്തത് | अग्रिमम् |
| Previous | മുമ്പത്തേത് | पूर्वम् |
| Menu | മെനു | सूची |
| More | കൂടുതൽ | अधिकम् |
| Close | അടയ്ക്കുക | पिधीयताम् |
| Exit (button) | പുറത്തുകടക്കുക | निष्क्रम्यताम् |
| Exit (title, label) | പുറത്തുകടക്കൽ | निष्क्रमणम् |
| Profile | പ്രൊഫൈൽ | परिचयः |
| Notifications | അറിയിപ്പുകൾ | सूचनाः |
| Favorites | പ്രിയപ്പെട്ടവ | प्रियाणि |
| History | നാൾവഴി | इतिवृत्तम् |
| Details | വിശദാംശങ്ങൾ | विवरणम् |
| List | പട്ടിക | आवली |
| Category | വിഭാഗം | वर्गः |
| Page | താൾ | पृष्ठम् |
| Section | ഖണ്ഡം | खण्डः |

##### Actions (buttons, menu items)

| English | Malayalam | Sanskrit |
|---|---|---|
| Save | സൂക്ഷിക്കുക | रक्ष्यताम् |
| Cancel | റദ്ദാക്കുക | निरस्यताम् |
| Delete | ഇല്ലാതാക്കുക | लुप्यताम् |
| Edit | തിരുത്തുക | सम्पाद्यताम् |
| Add | ചേർക്കുക | योज्यताम् |
| Remove | നീക്കുക | अपनीयताम् |
| Create | സൃഷ്ടിക്കുക | सृज्यताम् |
| Update | നവീകരിക്കുക | अद्यतनीक्रियताम् |
| Copy | പകർത്തുക | प्रतिलिख्यताम् |
| Paste | ഒട്ടിക്കുക | स्थाप्यताम् |
| Undo | പഴയപടിയാക്കുക | प्रत्यावर्त्यताम् |
| Redo | വീണ്ടും ചെയ്യുക | पुनःक्रियताम् |
| Search | തിരയുക | अन्विष्यताम् |
| Filter | അരിക്കുക | परिशोध्यताम् |
| Sort | ക്രമീകരിക്കുക | क्रमीक्रियताम् |
| Refresh | പുതുക്കുക | नवीक्रियताम् |
| Share | പങ്കിടുക | वितीर्यताम् |
| Send | അയയ്ക്കുക | प्रेष्यताम् |
| Download | ഡൗൺലോഡ് ചെയ്യുക | अवतार्यताम् |
| Upload | അപ്‌ലോഡ് ചെയ്യുക | आरोप्यताम् |
| Import | ഇറക്കുമതി ചെയ്യുക | आनीयताम् |
| Export | കയറ്റുമതി ചെയ്യുക | निर्याप्यताम् |
| Print | അച്ചടിക്കുക | मुद्र्यताम् |
| Select | തിരഞ്ഞെടുക്കുക | चीयताम् |
| Select all | എല്ലാം തിരഞ്ഞെടുക്കുക | सर्वं चीयताम् |
| Clear | മായ്ക്കുക | रिक्तीक्रियताम् |
| Reset | പുനഃസജ്ജമാക്കുക | पुनःसज्जीक्रियताम् |
| Confirm | സ്ഥിരീകരിക്കുക | स्थिरीक्रियताम् |
| Apply | പ്രയോഗിക്കുക | प्रयुज्यताम् |
| Open | തുറക്കുക | उद्घाट्यताम् |
| Start | ആരംഭിക്കുക | आरभ्यताम् |
| Stop | നിർത്തുക | विरम्यताम् |
| Pause | നിർത്തിവയ്ക്കുക | स्थग्यताम् |
| Resume | പുനരാരംഭിക്കുക | पुनरारभ्यताम् |
| Continue | തുടരുക | अनुवर्त्यताम् |
| Skip | ഒഴിവാക്കുക | त्यज्यताम् |
| Retry | വീണ്ടും ശ്രമിക്കുക | पुनः प्रयत्यताम् |
| Login | പ്രവേശിക്കുക | प्रविश्यताम् |
| Logout | ലോഗൗട്ട് ചെയ്യുക | निर्गम्यताम् |

##### Settings and preferences

| English | Malayalam | Sanskrit |
|---|---|---|
| Settings | ക്രമീകരണങ്ങൾ | विन्यासः |
| Preferences | താൽപ്പര്യങ്ങൾ | रुचयः |
| Language | ഭാഷ | भाषा |
| Theme | തീം | रूपविन्यासः |
| Dark mode | ഇരുണ്ട രൂപം | श्यामरूपम् |
| Light mode | തെളിഞ്ഞ രൂപം | दीप्तरूपम् |
| System default | സിസ്റ്റം സ്വതവേ | तन्त्रसिद्धम् |
| Font size | അക്ഷരവലുപ്പം | अक्षरपरिमाणम् |
| Sound | ശബ്ദം | ध्वनिः |
| Vibration | കമ്പനം | कम्पनम् |
| Backup | കരുതൽശേഖരം | प्रतिलिपिरक्षणम् |
| Restore | പുനഃസ്ഥാപിക്കുക | पुनःस्थाप्यताम् |
| Permissions | അനുമതികൾ | अनुमतयः |
| Account | അക്കൗണ്ട് | उपयोक्तृविवरणम् |
| Privacy | സ്വകാര്യത | गोपनीयता |
| Security | സുരക്ഷ | सुरक्षा |
| Storage | സംഭരണം | सङ्ग्रहः |
| Data | വിവരങ്ങൾ | दत्तांशः |

##### Status, feedback, and empty states

| English | Malayalam | Sanskrit |
|---|---|---|
| Loading | ലോഡുചെയ്യുന്നു | आपूर्यते |
| Please wait | കാത്തിരിക്കുക | प्रतीक्ष्यताम् |
| Success | വിജയം | सफलम् |
| Failed | പരാജയപ്പെട്ടു | असफलम् |
| Error | പിശക് | दोषः |
| Warning | മുന്നറിയിപ്പ് | पूर्वसूचना |
| Information | വിവരം | सूचना |
| Done | പൂർത്തിയായി | समाप्तम् |
| Empty | ശൂന്യം | रिक्तम् |
| No results | ഫലങ്ങളില്ല | न किमपि प्राप्तम् |
| Offline | ഓഫ്‌ലൈൻ | असंयुक्तम् |
| Online | ഓൺലൈൻ | संयुक्तम् |
| Saved | സൂക്ഷിച്ചു | रक्षितम् |
| Deleted | ഇല്ലാതാക്കി | लुप्तम् |
| Copied | പകർത്തി | प्रतिलिखितम् |
| Updated | നവീകരിച്ചു | अद्यतनीकृतम् |
| Required | ആവശ്യം | आवश्यकम् |
| Optional | ഐച്ഛികം | वैकल्पिकम् |
| Invalid | അസാധു | अमान्यम् |

##### Time and date

| English | Malayalam | Sanskrit |
|---|---|---|
| Date | തീയതി | दिनाङ्कः |
| Time | സമയം | समयः |
| Today | ഇന്ന് | अद्य |
| Yesterday | ഇന്നലെ | ह्यः |
| Tomorrow | നാളെ | श्वः |
| Now | ഇപ്പോൾ | इदानीम् |
| Day | ദിവസം | दिनम् |
| Week | ആഴ്ച | सप्ताहः |
| Month | മാസം | मासः |
| Year | വർഷം | वर्षम् |
| Duration | ദൈർഘ്യം | कालावधिः |

##### Content and fields

| English | Malayalam | Sanskrit |
|---|---|---|
| Title | ശീർഷകം | शीर्षकम् |
| Name | പേര് | नाम |
| Description | വിവരണം | वर्णनम् |
| Note | കുറിപ്പ് | टिप्पणी |
| Text | പാഠം | पाठः |
| Image | ചിത്രം | चित्रम् |
| Audio | ഓഡിയോ | श्रव्यम् |
| Video | വീഡിയോ | दृश्यम् |
| File | ഫയൽ | सञ्चिका |
| Folder | ഫോൾഡർ | संपुटम् |
| Document | രേഖ | लेखः |
| Link | ലിങ്ക് | अनुबन्धः |
| Word | വാക്ക് | शब्दः |
| Line | വരി | पङ्क्तिः |
| Number | സംഖ്യ | सङ्ख्या |
| Total | ആകെ | योगः |
| Count | എണ്ണം | गणना |
| Size | വലുപ്പം | परिमाणम् |
| Type | തരം | प्रकारः |
| Status | നില | स्थितिः |

##### Confirmation words

| English | Malayalam | Sanskrit |
|---|---|---|
| Yes | അതെ | आम् |
| No | ഇല്ല | न |
| OK | ശരി | अस्तु |
| Are you sure? | ഉറപ്പാണോ? | निश्चयेन वा? |

##### About screen (matches the `aboutDetail<Key>` ARB keys in `guideline.md` §1.6)

| English | Malayalam | Sanskrit |
|---|---|---|
| About | ആപ്പിനെക്കുറിച്ച് | विषयपरिचयः |
| Version | പതിപ്പ് | संस्करणम् |
| Build | നിർമ്മിതി | निर्मितिसङ्ख्या |
| Author | രചയിതാവ് | लेखकः |
| Email | ഇമെയിൽ | विद्युत्पत्रम् |
| License | ലൈസൻസ് | अनुज्ञापत्रम् |
| AI used | ഉപയോഗിച്ച AI | प्रयुक्ता कृत्रिमबुद्धिः |
| IDE used | ഉപയോഗിച്ച IDE | प्रयुक्तं विकाससाधनम् |
| Help | സഹായം | साहाय्यम् |
| Feedback | പ്രതികരണം | प्रतिक्रिया |
| Contact | ബന്ധപ്പെടുക | सम्पर्कः |
| Terms | നിബന്ധനകൾ | नियमाः |
| Privacy policy | സ്വകാര്യതാ നയം | गोपनीयतानीतिः |

> About-screen row labels use the `aboutDetail<Key>` pattern (`guideline.md` §1.6), which is exempt
> from the 8.6 budget, so a long row label may wrap to two lines. Do not copy that liberty into a
> toolbar or a tab.

### 8.6 Label Conciseness (Short UI Text vs. Descriptive Text)

UI chrome MUST be short in **all three** languages. A long Malayalam or Sanskrit word wrapping onto
two lines in a toolbar, tab, or bottom-navigation item is a layout bug, and Malayalam and Sanskrit
compounds grow fast if written carelessly.

**Budget for short text** — menu items, buttons, tabs, chips, navigation destinations, tooltips,
app-bar titles, list-row labels, form-field labels, switch/checkbox labels, dialog action buttons:

| Language | Target | Hard limit |
|---|---|---|
| English | 1–2 words | 20 characters |
| Malayalam | 1–2 words | 22 characters |
| Sanskrit | 1 word (nominal form preferred) | 22 characters |

**How characters are counted.** A character is a visible character (a grapheme cluster), not a
code unit: a vowel sign or virama belongs to the letter before it. Count using Dart's
`characters.length` (`package:characters`, which is bundled with Flutter): `string.characters.length`.
Every term in the 8.5.4 glossary fits within the budget.

Rules:

- Prefer a single word. Drop articles and filler: "Delete" not "Delete this item".
- In Sanskrit follow the form convention in 8.5: a nominal form for titles, tabs and labels
  (`अन्वेषणम्`), a single-word polite imperative for buttons (`अन्विष्यताम्`). Never a multi-word
  verb phrase.
- In Malayalam prefer the common everyday word over a Sanskritized formal one, unless the app's
  subject matter calls for the formal register.
- Do not solve a long translation by shrinking the font, truncating, or adding an ellipsis —
  choose a shorter word.
- Sentence case in English (`Add note`), not Title Case, and never ALL CAPS in Malayalam or
  Sanskrit.

**Descriptive text is exempt** from the budget — and MUST still be complete, natural prose in all
three languages: onboarding copy, empty-state explanations, help text, About `description`, error
explanations, confirmation dialog bodies, notification bodies, tutorial content. About-screen row labels
(`aboutDetail<Key>`) are also exempt from this budget so they can wrap to two lines.

**ARB key naming makes the category checkable.** Prefix every key so the budget can be enforced
mechanically:

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
const limits = {'en': 20, 'ml': 22, 'sa': 22};
// For each ARB file: for each key starting with a short prefix,
// expect(value.characters.length, lessThanOrEqualTo(limits[locale]!));
```

### 8.7 Per-Feature Language Completeness

A feature is **not done** until it works fully in English, Malayalam and Sanskrit.

- No feature may ship with strings in `app_en.arb` only. Adding a key to the template without
  adding it to `app_ml.arb` and `app_sa.arb` MUST fail CI.
- No feature may render English text under `ml` or `sa` — including snackbars, validation messages,
  notification text, share sheets, exported file headers a user sees, and the About screen.
- Feature-level content shipped as an asset (JSON, Markdown help pages, seed data a user reads)
  MUST also carry all three languages, or the screen that shows it MUST resolve a per-language
  asset (`assets/content/help_<lang>.md`). `app_config.json` prose fields (`appName`, `description`,
  `details`) must provide all three `{"en","ml","sa"}` entries.
- Screenshots for a release are taken in all three languages when the feature changes layout.
- Widget tests for a screen MUST run in all three locales (pump with `locale: Locale('ml')` and
  `Locale('sa')`), asserting no overflow and no untranslated English leaking through.

**Translation parity test** — required in every app:

```dart
// test/l10n/translation_parity_test.dart
import 'dart:convert';
import 'dart:io';
import 'package:flutter_test/flutter_test.dart';

void main() {
  const locales = ['ml', 'sa'];

  /// Strings allowed to match English: brand names and symbols. Keep this list short.
  const sameAsEnglishAllowed = <String>{};

  group('ARB parity tests', () {
    test('every ARB file has the same keys as the template', () {
      final en = _keys('lib/l10n/app_en.arb');
      for (final locale in locales) {
        final other = _keys('lib/l10n/app_$locale.arb');
        expect(other.difference(en), isEmpty, reason: 'extra keys in $locale');
        expect(en.difference(other), isEmpty, reason: 'missing keys in $locale');
      }
    });

    test('no translation is a copy of the English value', () {
      final enStrings = _strings('lib/l10n/app_en.arb');
      final problems = <String>[];
      for (final locale in locales) {
        final other = _strings('lib/l10n/app_$locale.arb');
        for (final entry in other.entries) {
          final key = entry.key;
          final value = entry.value;
          if (sameAsEnglishAllowed.contains(key)) continue;
          if (value == enStrings[key] && value.trim().isNotEmpty) {
            problems.add('lib/l10n/app_$locale.arb: $key is untranslated (matches English)');
          }
        }
      }
      expect(problems, isEmpty, reason: problems.join('\n'));
    });

    test('aboutMadeWithLove keeps the {heart} marker in all three languages', () {
      for (final locale in ['en', ...locales]) {
        final strings = _strings('lib/l10n/app_$locale.arb');
        final text = strings['madeWithLove'] ?? strings['aboutMadeWithLove'];
        if (text != null) {
          expect(text.contains('{heart}'), isTrue,
              reason: 'app_$locale.arb madeWithLove is missing the {heart} marker');
        }
      }
    });
  });

  group('About JSON config parity tests', () {
    test('app_config.json has all three languages and valid detail labels', () {
      final file = File('assets/config/app_config.json');
      if (!file.existsSync()) return; // Pattern B app (no assets config)

      final json = jsonDecode(file.readAsStringSync()) as Map<String, dynamic>;
      final enKeys = _keys('lib/l10n/app_en.arb');
      final problems = <String>[];

      void checkLanguages(String path, dynamic value) {
        if (value is Map<String, dynamic>) {
          for (final lang in ['en', 'ml', 'sa']) {
            final text = value[lang]?.toString().trim() ?? '';
            if (text.isEmpty) {
              problems.add('$path.$lang is missing or empty');
            }
          }
        }
      }

      checkLanguages('appName', json['appName']);
      checkLanguages('description', json['description']);

      final details = json['details'];
      if (details is Map<String, dynamic>) {
        for (final entry in details.entries) {
          final id = entry.key;
          checkLanguages('details.$id', entry.value);

          // Details key is lowerCamelCase; label in ARB is aboutDetail<Key>
          final labelKey = 'aboutDetail${id[0].toUpperCase()}${id.substring(1)}';
          if (!enKeys.contains(labelKey)) {
            problems.add('details.$id has no corresponding ARB key "$labelKey"');
          }
        }
      }

      expect(problems, isEmpty, reason: problems.join('\n'));
    });
  });

  group('Content asset parity tests', () {
    test('content and help assets exist in all three languages', () {
      final assetsDir = Directory('assets');
      if (!assetsDir.existsSync()) return;

      final problems = <String>[];
      final enFiles = assetsDir
          .listSync(recursive: true)
          .whereType<File>()
          .where((f) => RegExp(r'_en\.[^.]+$').hasMatch(f.path));

      for (final enFile in enFiles) {
        for (final lang in locales) {
          final twinPath = enFile.path.replaceAllMapped(
            RegExp(r'_en(\.[^.]+)$'),
            (match) => '_$lang${match[1]}',
          );
          if (!File(twinPath).existsSync()) {
            problems.add('Missing localized asset: $twinPath (matching ${enFile.path})');
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
- Use `Directionality` widget tests to verify layouts do not break in RTL mode.
- Icons that carry directional meaning (back arrow, forward arrow) MUST be mirrored in RTL.
  Use `Directionality.of(context)` or set `textDirection` in `Icon` semantics.
- None of our three shipped languages is RTL, so these rules are about staying ready rather than a
  current feature. Test RTL by wrapping a screen in `Directionality(textDirection: TextDirection.rtl)`
  in a widget test — do **not** add Arabic or Hebrew to `supportedLocales`, which is fixed at
  `en`, `ml`, `sa` (8.3).

### 8.9 Locale-Sensitive Formatting

Use the `intl` package for all locale-sensitive formatting. Never use `toString()` on dates,
numbers, or currencies in user-visible strings. Pass `formattingLocale(...)` from 8.3.2 as the
`locale` argument, so Sanskrit falls back to English CLDR data instead of throwing.

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

> **Modern alternative (Flutter ≥ 3.13).** `AppLifecycleListener` is the recommended
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

Impeller is the default Flutter rendering engine on **iOS** (no Skia opt-out as of Flutter
3.38) and on **Android API 29+** (since Flutter 3.27). On older Android devices and devices
without Vulkan, Flutter falls back to the legacy OpenGL renderer automatically; no app code
changes are required.

What this means in practice for an app team:

- **Shader compilation jank is eliminated** for the standard widget set. Custom fragment
  shaders MUST be tested under Impeller on both Metal (iOS) and Vulkan (Android), as the
  shader pipeline differs from Skia. File any rendering regressions under Impeller against
  `flutter/flutter` rather than working around them.
- **Disabling Impeller** is a deliberate, documented decision — it is not a default tuning
  knob. On Android, `<meta-data android:name="io.flutter.embedding.android.EnableImpeller"
  android:value="false" />` in `AndroidManifest.xml` opts out. iOS no longer supports
  opt-out. Disabling Impeller anywhere else MUST be justified in `architecture.md §20
  Decisions And Tradeoffs`.

  > **Shelf-life note.** The Android opt-out was deprecated in Flutter 3.38 and the official
  > 2026 roadmap commits to removing the Skia backend on Android 10+ during the year. Treat
  > this escape hatch as a short-lived workaround, not as a stable architectural option;
  > any code or test infrastructure that depends on it MUST have a migration plan recorded
  > in `architecture.md §21 Known Risks`.
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

- MUST use `ListView.builder`, `GridView.builder`, or `SliverList` with a
  `SliverChildBuilderDelegate` for any list that is unbounded or can grow without a known upper
  limit. The eager-children variants (`ListView(children: [...])`) build every child at layout
  time regardless of visibility; on an unbounded list this is a correctness failure, not just a
  performance concern.
- For lists with a known, fixed upper bound of roughly 20 items or fewer, `ListView(children:
  [...])` is acceptable. Beyond that count, consider `ListView.builder` — the decision point is
  whether all children being built simultaneously causes a measurable frame-time impact on a
  mid-range device.
- Consider `itemExtent` on `ListView.builder` when all items have the same height. This eliminates
  per-item layout measurement and can significantly improve scrolling on long lists. Benchmark
  before committing to a fixed extent, as it precludes variable-height items.
- Use `const` constructors inside list item widgets wherever possible.
- Avoid loading all data into memory for very long lists. Implement pagination or cursor-based
  loading at the repository layer.
- `ListView.separated` is acceptable for short lists with separators but use
  `ListView.builder` with conditional separator rendering for long lists.

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
- Use `flutter build apk --analyze-size` or `flutter build appbundle --analyze-size` to track
  binary size after each significant dependency addition.
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
| Windows MSIX | Under 80 MB | 150 MB |

Exceeding the hard limit requires a documented justification in the release checklist.

Track size in CI using `--analyze-size` output. Record the baseline at project start and
diff on each release.

---

## 11. Error Handling Architecture

### 11.1 Global Error Boundaries

Every Flutter app MUST configure global error handlers in `main()` before `runApp`. Without these,
unhandled errors in release builds crash silently with no user feedback.

```dart
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();

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
> **`PlatformDispatcher.instance.onError` vs `runZonedGuarded`.** Since Flutter 3.3,
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

Add to `pubspec.yaml` under `dev_dependencies` (versions as of Flutter 3.41 / early 2026 —
pin to the current major at project start and update deliberately):

```yaml
dev_dependencies:
  build_runner: ^2.5.0
  freezed: ^2.5.0          # If using freezed models
  json_serializable: ^6.9.0 # If using json annotation
  riverpod_generator: ^2.6.0 # If using riverpod code gen
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

- `*.freezed.dart` and `*.g.dart` are added to `.gitignore`.
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
class Todo with _$Todo {
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
- Freezed union types (sealed classes) MUST use `when` or `maybeWhen` exhaustively at call sites.

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
- Each migration MUST be atomic. Wrap multi-statement migrations in a transaction.
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
    await db.execute('PRAGMA journal_mode=WAL;');
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
  logger: ^2.4.0
```

```dart
// lib/core/logging/app_logger.dart
import 'dart:io';
import 'package:logger/logger.dart';
import 'package:path/path.dart' as p;
import 'package:path_provider/path_provider.dart';

class AppLogger {
  static late Logger _logger;

  /// Call once during app startup, before any log call.
  /// `getApplicationCacheDirectory()` is async, so logger initialization
  /// cannot happen at static-field initialization time.
  static Future<void> init() async {
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

`AppLogger.init()` MUST run during the `main()` initialization sequence (section 4.5, step 6
"Logging init") and before any other code calls `AppLogger`. Calling a logging method before
`init` returns will throw `LateInitializationError`.

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

### 15.1 Core Security Rules

These rules apply to all Flutter apps.

- Never log secrets, tokens, private payloads, or decrypted sensitive data.
- Request only the permissions the app actually uses.
- Ask for permissions at point of use where the platform allows it.
- Production logs SHOULD avoid personal data unless operationally necessary.
- Production builds MUST be compiled with `--obfuscate --split-debug-info=<symbols_path>`.
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
- Screenshot and screen-recording protection MUST be enabled:
  - Android: `FlutterWindowManager` or `FLAG_SECURE` via method channel.
  - iOS: Overlay an opaque view on the appropriate "will resign active" hook —
    `sceneWillResignActive(_:)` after the UIScene migration (mandatory for iOS 26 SDK builds),
    or `applicationWillResignActive` for legacy `AppDelegate`-only projects.
- App lock, background lock, and session-expiry behavior MUST be explicit in app state.
- Export of sensitive data SHOULD be encrypted by default; plaintext export, if allowed, MUST be
  explicit and user-confirmed.
- Backup, recovery, import, and migration flows MUST be tested as critical flows.

### 15.3 OWASP Mobile Top 10 Compliance Checklist

Before each production release, verify the following OWASP Mobile Top 10 controls:

| ID | Risk | Control |
|----|------|---------|
| M1 | Improper Credential Usage | No hardcoded secrets; use secure storage |
| M2 | Inadequate Supply Chain Security | Dependency audit; pin versions in `pubspec.lock` |
| M3 | Insecure Authentication | App lock with proper background/foreground enforcement |
| M4 | Insufficient Input/Output Validation | Validate all user input; sanitize before DB write |
| M5 | Insecure Communication | TLS only for any network traffic; no HTTP |
| M6 | Inadequate Privacy Controls | Data inventory reviewed; no PII in logs |
| M7 | Insufficient Binary Protections | `--obfuscate` applied to release builds |
| M8 | Security Misconfiguration | `android:debuggable=false` verified; permissions minimal |
| M9 | Insecure Data Storage | No sensitive data in `SharedPreferences` or unencrypted files |
| M10 | Insufficient Cryptography | Versioned encrypted formats; secure key derivation |

Mark each item as verified, not applicable, or risk-accepted (with documented justification)
before release sign-off.

### 15.4 Data Retention And Purge Policy

Every app that stores user-generated data MUST define and implement a retention policy.

- Document in `docs/security.md`: what data is stored, how long it is retained, and what triggers
  deletion.
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

---

## 16. Coding Standards

### 16.1 Formatting And Analysis

- Run `dart format .` before commit.
- New work MUST NOT introduce analyzer issues.
- Repositories SHOULD aim for zero analyzer warnings overall.
- Start from `package:flutter_lints/flutter.yaml` (pin `flutter_lints: ^5.0.0` or the
  current major as of project start in `dev_dependencies`) and add stricter rules
  deliberately. Pinning the major prevents the lint set silently shifting under your CI
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
- **Build hooks (Dart ≥ 3.10).** Dart packages can integrate native build steps directly
  via stable build hooks. Any new dependency that uses build hooks (`hook/build.dart`,
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
- Run `flutter pub licenses` before the first public release and on any release that adds new
  dependencies. Verify all transitive licenses are compatible with your distribution model.
- Pin critical security dependencies (encryption, secure storage) to exact versions in
  `pubspec.yaml` and update them deliberately after reviewing changelogs.

---

## 17. Asset Management

### 17.1 Image Format Policy

| Content Type | Required Format | Rationale |
|-------------|-----------------|-----------|
| Photographs, complex gradients | WebP (lossy) | 25–35% smaller than JPEG at equal quality |
| Logos, UI illustrations with transparency | WebP (lossless) or SVG | Smaller than PNG; SVG preferred if vector |
| Icons (monochrome or multi-color) | SVG via `flutter_svg` | Resolution-independent, tree-shakeable |
| Raster fallback (when SVG not viable) | PNG with 2x/3x variants | Only when SVG cannot achieve the result |

Never use JPEG for UI assets that require transparency.

#### Per-Platform Asset Bundling (Flutter ≥ 3.41)

`pubspec.yaml` supports a `platforms:` filter on individual asset entries, letting heavy
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
budget (§10.7) thanks you.

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

Register in `pubspec.yaml`:

```yaml
flutter:
  assets:
    - assets/images/
    - assets/images/2.0x/
    - assets/images/3.0x/
```

### 17.3 Icon Font Tree Shaking

Flutter's `--tree-shake-icons` build flag removes unused Material icons from the binary. It
activates automatically in release builds when icons are referenced via `const` constructors.

- Always use `const Icon(Icons.add)`, never `Icon(Icons.add)` with a variable.
- Never reference icon code points via integer literals. The tree shaker cannot analyze integer
  references.
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
- **Script coverage is part of licensing work.** Because every app ships Malayalam and Sanskrit
  (section 8.3.3), the chosen font stack MUST cover the Malayalam and Devanagari blocks, or the
  app MUST bundle fonts that do (Noto Sans Malayalam and Noto Sans Devanagari are OFL). Runtime
  fetching is not acceptable for these — a device offline on first launch would render boxes for
  two of the three languages.

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
- Database migration tests MUST cover the full upgrade path from version 1 to the current version,
  not just the latest increment.

### 18.3 Test Quality

- Tests MUST be independent.
- Use descriptive test names.
- Mock external systems, not the logic under test.
- Important test files SHOULD be runnable in isolation.
- Coverage trends are useful, but arbitrary percentage gates SHOULD NOT replace judgment.

### 18.4 Performance Testing

Under `Production App Extension`:

- Use `flutter test --profile` with `WidgetTester.runAsync` for widget-level performance checks.
- For frame-rate regression testing, use Flutter's integration test `traceAction` API to collect
  frame timing:

  ```dart
  final timeline = await driver.traceAction(() async {
    // Scroll through a long list.
    await driver.scroll(listFinder, 0, -5000, const Duration(seconds: 2));
  });
  final summary = TimelineSummary.summarize(timeline);
  expect(summary.computePercentileFrameBuildTimeMillis(90), lessThan(16.0));
  ```

- Capture and store performance baseline results. Treat regressions beyond 20% as blocking.

### 18.5 Widget Previewer (Flutter ≥ 3.35, Experimental)

The Flutter Widget Previewer renders annotated widgets in a dedicated VS Code or Android
Studio panel without launching the full app. It is faster feedback than running widget
tests for visual iteration, but it is **experimental** as of Flutter 3.41 and is not a
substitute for widget tests, golden tests, or integration tests.

```dart
import 'package:flutter/widgets.dart';

@Preview(name: 'Empty state — light')
Widget previewEmptyStateLight() => const TodoEmptyState(theme: AppTheme.light);

@Preview(name: 'Empty state — dark')
Widget previewEmptyStateDark() => const TodoEmptyState(theme: AppTheme.dark);
```

Rules:
- Treat previews as a **development convenience**, not as a checked-in test artifact. Do
  not gate CI on previewer behavior while the feature is experimental.
- Cover the same widget with widget tests for behavior assertions and (optionally) golden
  tests for pixel-level regressions.
- Remove orphaned `@Preview` annotations during refactors so the previewer panel stays
  meaningful.

---

## 19. CI Standard

### 19.1 Minimum CI

All active app repositories SHOULD have CI on pull requests or on the merge path to the protected
branch.

Minimum checks:

```yaml
steps:
  - run: flutter pub get
  - run: dart run build_runner build --delete-conflicting-outputs   # If using code gen
  - run: dart format --output=none --set-exit-if-changed .
  - run: flutter analyze
  - run: flutter test
```

### 19.2 Production App Extension

For shipped apps:

```yaml
  - run: flutter test --coverage
  - run: flutter build apk --flavor dev --debug
  - run: flutter build apk --flavor prod --release
      --obfuscate
      --split-debug-info=build/symbols/android-prod-${{ env.APP_VERSION }}/
  - run: flutter build appbundle --flavor prod --release
      --obfuscate
      --split-debug-info=build/symbols/android-prod-${{ env.APP_VERSION }}/
```

The `<platform>-<version>/` subdirectory is required so that symbols from different releases
do not collide; `release_process.md §6.1` and `security.md §8.1` both depend on this layout.
`${{ env.APP_VERSION }}` is the GitHub Actions form — substitute the equivalent for your CI.

Additional recommended steps:
- Dependency license check: `flutter pub licenses > licenses.txt && <verify script>`.
- Generated file staleness check (if Option A from section 12.2 is chosen):
  verify that `*.g.dart` and `*.freezed.dart` files match what `build_runner` would produce.
- Artifact size check: compare `--analyze-size` output against the project's size budget from
  section 10.7.

### 19.3 Pre-Commit

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
- Debug symbol archives (`*.symbols/` from `--split-debug-info`).

### 20.3 Usually Commit

- `analysis_options.yaml`
- `.gitignore`
- `pubspec.lock` for application repositories
- `*.freezed.dart` and `*.g.dart` if Option A (commit generated files) is chosen — see section 12.2.

### 20.4 Generated File Policy In .gitignore

Add to `.gitignore` regardless of whether generated Dart source is committed:

```gitignore
# Build output
build/
*.apk
*.aab
*.ipa
*.msix

# Debug symbols
*.symbols/

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
| `docs/architecture.md` | Module boundaries, initialization sequence, schema version, major decisions |
| `docs/release_process.md` | Required for shipped apps |
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
- `docs/security.md` for sensitive-data apps.
- `docs/adr/` for architecture decision records that are likely to be revisited.

### 21.3 README Must Include

- Prerequisites (Flutter version, Dart version, platform SDK versions).
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
- Identify which applicability profile is in force for the repository.
- Check the current database schema version before writing any migration.
- Check whether the repository commits or excludes generated files before creating new models.
- Write a plan to `plans/` and obtain explicit user approval before modifying project files.

### 22.2 While Writing Code

- Respect the current project structure unless the task explicitly includes restructuring.
- Do not introduce a second state-management system without a documented reason.
- Do not add boilerplate comments or type annotations to unchanged code.
- Do not invent abstractions for one-time operations.
- Apply the security profile in force; never log secrets or weaken cryptographic behavior.
- Ensure all `plans/` and `change_log/` entries follow the privacy rule in 21.1.1: **relative repository paths only**, **no local system details** (OS user name, computer/host name, home or drive-letter paths, network shares, LAN/internal IPs, local server URLs with ports, device serial numbers, personal email addresses), and **no secrets** (API keys, tokens, passwords, keystore passphrases, credentials, PII).
- Put all user-visible strings in `lib/l10n/*.arb` and read them through `AppLocalizations` (section 8.2) — never a raw string literal in a widget.
- Add every new key to **all three** ARB files — `app_en.arb`, `app_ml.arb`, `app_sa.arb` — with a real translation in each (sections 8.2, 8.7). Never leave the English value as a placeholder in the Malayalam or Sanskrit file.
- Never use Hindi anywhere as a substitute or crutch for Sanskrit: Sanskrit is fully generative and possesses all roots, affixes, and compounding formulas to create any technical or future terminology. Follow the rules, the forbidden-marker gate, and the glossary in 8.5. Flag any Sanskrit string you are not confident about in the change log so a Sanskrit reader can review it.
- Keep `action…`, `label…`, `title…`, `tab…`, `nav…` and `tooltip…` strings within the length budget in 8.6; only `desc…`/`help…`/`empty…`/`error…`/`body…` keys may be long.
- Give every icon-only control a localized `tooltip:` (section 7.8).
- Keep the About screen data-driven and localized, and never remove the "Made with ❤️ from India" badge (`guideline.md` §1.6–§1.7).
- Do not use `kDebugMode` or `kReleaseMode` as a substitute for application flavor when the
  project has explicit environments.
- Always add `const` to constructors and widget instantiations where possible.
- Never use `ListView(children: [...])` for lists that can have more than 20 items.
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
- All `plans/` and `change_log/` files use relative repository paths only and contain zero local system details and zero sensitive data — safe to publish on the internet (section 21.1.1).
- `l10n.yaml` and all three ARB files (`app_en.arb`, `app_ml.arb`, `app_sa.arb`) exist, and every
  user-visible string added or changed by this task comes from `AppLocalizations` (section 8.2).
- Every ARB key added or changed by this task exists and is genuinely translated in all three
  files; the parity test passes (section 8.7).
- Sanskrit strings pass the Hindi-marker gate and follow the glossary (section 8.5).
- Short-label keys are within the length budget for all three languages (section 8.6).
- Every icon-only control added or changed has a localized tooltip (section 7.8).
- The screen was checked in all three languages — no English leaking through, no overflow, no
  missing glyphs (sections 8.3.3, 8.7).
- The About screen still ends with the "Made with ❤️ from India" badge if this task touched About
  (`guideline.md` §1.7).
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
- The Google Play readiness gate in `docs/release_process.md` still holds for any change touching
  the manifest, permissions, target SDK, signing, data collection, or store-listed behavior.

### 23.3 Sensitive Data Extension

- Sensitive data handling was reviewed against the security section.
- Logging was reviewed for protected data exposure.
- Backup, import, export, migration, or recovery paths were tested if touched.
- OWASP checklist items affected by the change were re-verified.

---

## 24. Practical Guidance

- A thin `main.dart` scales better than a smart one.
- Mirrored tests reduce search time and ownership confusion.
- `utils/` is acceptable only when its scope stays clear and small.
- Flavors solve real problems, but not every app needs them on day one.
- Security requirements should be attached to product risk, not copied blindly.
- CI should enforce the boring rules so review can focus on behavior and design.
- `const` is free performance; use it everywhere it compiles.
- Performance regressions are easiest to catch immediately after the change that caused them.
  Profile before merging, not six months later.
- Accessibility failures discovered late in a project are expensive to fix. Add semantics labels
  as you build each widget, not in a post-hoc pass.
- Every unhandled exception that reaches a user is a trust failure. Design error boundaries first.

Treat this document as a baseline plus extensions. Tighten it for higher-risk apps, and relax
optional guidance only with a deliberate reason.