## What does `flutter_build_flavors_guide.md` say?

> **Last checked against Flutter 3.47 / Dart 3.13 (2026-10).** The guide does not pin tool
> versions — it says "use the latest stable Flutter and the Android versions it generates".
> When the SDK moves on, refresh the "last checked" reference tables and the "since 3.xx"
> conditions (UIScene, 16 KB pages, Java 17, iOS 15 minimum) first.

It's a **platform-by-platform technical reference** for setting up Flutter build flavors. Unlike `architecture.md` and `security.md` (project-specific templates) or `flutter_project_engineering_standard.md` (a universal rulebook), this guide is **task-focused**: it tells you exactly how to wire flavors into Android, iOS, Windows, macOS and Linux builds, and explains the framework-level rules that constrain those choices.

It has the following major sections:

- **Toolchain Prerequisites** — the hard minimums (**Java 17**, iOS 15, macOS 12, Linux build packages, Xcode 26 for the App Store, 16 KB pages on Android) and a "last checked" snapshot of the versions Flutter generates (not a rule). The toolchain rules themselves live in engineering standard §5.3.
- **Flavor basics** — what a flavor represents, the typical `dev`/`prod` matrix, and the four flavor × mode combinations (debug/release × dev/prod) with their signing implications.
- **How the flavor reaches Dart code** — the critical Flutter ≥ 3.19 reservation: `FLUTTER_APP_FLAVOR` is owned by the framework. `--flavor` works on Android/iOS but not desktop. Two ways to read the flavor at runtime (`appFlavor` constant vs `String.fromEnvironment`).
- **Android flavor setup** — Kotlin DSL is the default for new projects (Groovy DSL note for inherited projects), the **Android toolchain baseline for AGP 9+ / Gradle 9+** (where `settings.gradle.kts` and the wrapper set the versions, the Gradle wrapper, `kotlin { compilerOptions { } }` instead of the rejected `kotlinOptions { }`, the migrator's `gradle.properties` flags, and `ExecOperations` instead of the removed `project.exec`), product flavors in Gradle, three signing strategies (local file, CI-managed, separate keystores per flavor), step-by-step keystore creation with `keytool`, the `key.properties` pattern, the `.gitignore` rules, the full `build.gradle.kts` example with a Gradle execution-time guard for prod releases, and the caveats of that guard.
- **ProGuard / R8 rules** — R8 full mode is the default since AGP 8.0; which Flutter packages actually require keep rules (the native plugin side, not the Dart side), why `freezed` and `json_serializable` do not need rules, and the minimum keep rules for the Flutter engine and `sqflite`.
- **Which Android artifact to use** — split APKs vs AAB, the important distinction between `--target-platform` and `--split-per-abi`, and the deprecation of 32-bit-only `armeabi-v7a` for new submissions.
- **16 KB page size compliance** — Play Store mandate from Nov 1 2025 (Android 15+) and May 31 2026 (broader), how to audit bundled `.so` files in dependencies, and the 16 KB-page emulator setup for verification.
- **iOS flavor setup** — iOS 15 minimum deployment target (raised in 3.47), **UIScene lifecycle migration (mandatory for iOS 26 SDK builds and for launching apps built with Xcode 27; default since 3.41; manual steps for customized `AppDelegate`)**, the `Debug-<flavor>` / `Release-<flavor>` build configurations and matching schemes Flutter needs, the bundle ID and display name per build configuration, the Podfile mapping, why Xcode settings MUST NOT set `DART_DEFINES=FLUTTER_APP_FLAVOR%3D...`, and provisioning profile rules per flavor.
- **Windows desktop flavor setup** — why `--flavor` is not supported (flutter/flutter#98994), the `--dart-define=APP_FLAVOR=<value>` pattern, MSIX packaging via `dart run msix:create` (the deprecated `flutter pub run msix:create` form is called out), `.msixbundle` for store vs `.msix` for sideloading. Desktop setup that is not flavor-specific (`sqflite_common_ffi`, window size) is in engineering standard §5.5.
- **macOS desktop flavor setup** — the same `APP_FLAVOR` pattern, a per-flavor `AppInfo` xcconfig and Xcode scheme when dev and prod must install side by side, and keeping debug-only entitlements out of `Release.entitlements`.
- **Linux desktop flavor setup** — the same `APP_FLAVOR` pattern, and an optional `APPLICATION_ID` suffix in `linux/CMakeLists.txt` for side-by-side dev installs.
- **Recommended release matrix** — a complete table of platform × flavor × mode × signing × command for the typical Flutter project, with `--tree-shake-icons` behavior explained.
- **Debug symbol management** — what `--obfuscate` actually does (and doesn't do), the symbol archive policy, why losing the symbols means crash reports become undecodable.
- **Notes for new projects** — a per-platform setup checklist to run when bootstrapping a new repository, including UIScene and 16 KB requirements.

Unlike the templates, this guide is **read-only reference** — you do not fill anything in. You consult it when wiring up build configuration, then record your actual decisions in `architecture.md §15` and `release_process.md §5`.

---

## How do you use it in a Flutter project?

Think of it as three things simultaneously.

**1. A pre-flight checklist before the first `flutter build` command.** Before running any flavored build, verify that your `AppFlavorConfig` reads both `APP_FLAVOR` and `FLUTTER_APP_FLAVOR` (the two-variable pattern), your Android `key.properties` is in `.gitignore`, your iOS xcconfig files don't try to set `DART_DEFINES=FLUTTER_APP_FLAVOR%3D...`, and your Windows builds use `--dart-define=APP_FLAVOR=<value>` (not `FLUTTER_APP_FLAVOR`). Getting any of these wrong produces the `kernel_snapshot_program failed` build error or — worse — a build that succeeds but reads the wrong flavor at runtime.

**2. A copy-paste reference during setup.** When you are configuring `android/app/build.gradle.kts`, the example block is ready to drop in. When you are creating Xcode build configurations and schemes, every step is spelled out. When you are packaging Windows with MSIX, the `pubspec.yaml` config is provided. You read the section, copy the relevant pattern, adapt it to your project's names.

**3. A decision recorder.** Every section has decision points: which Android signing strategy (A, B, or C), whether to commit `key.properties` to source control (no — Strategy A documented), whether dev and prod must install side by side on Windows (different `identity_name` if yes), whether to use split APKs or AAB (channel-dependent). The guide forces those decisions to be made consciously and points you to record them in `architecture.md §15`.

---

## What should you decide BEFORE running the first flavored build?

The build flavors guide is **not a fill-in-the-blanks template**. You read it, make decisions from it, then record those decisions in your project's `architecture.md §15` and `release_process.md §5`.

These decisions must be made before the first flavored build is attempted, because each one shapes file layout that is expensive to change later:

| Priority | Decision | Why Before First Build |
|----------|----------|-----------------------|
| 🔴 Must | **Flavor names** | Decide `dev` and `prod` (or your variant). The names propagate to Gradle product flavors, Xcode schemes, xcconfig folders, `applicationIdSuffix` values, and `AppFlavorConfig` enum cases. Renaming later means touching every native project. |
| 🔴 Must | **Two-variable `AppFlavorConfig`** | Implement the pattern from engineering standard §5.2: read `APP_FLAVOR` first, fall back to `FLUTTER_APP_FLAVOR`. Skipping this works on Android/iOS but breaks silently on Windows where the flavor reads as `prod` regardless of the build command. |
| 🔴 Must | **Android signing strategy** | Choose Strategy A (local `key.properties`), B (CI-managed), or C (separate keystores per flavor). Each implies different `.gitignore` rules, different `build.gradle.kts` shape, and different CI secrets. |
| 🔴 Must | **`.gitignore` rules** | Add `android/key.properties`, `android/*.jks`, `android/*.keystore`, and `build/` (which also holds the debug symbols) before the first build runs — once a keystore is committed it is a security incident. |
| 🔴 Must | **Side-by-side install policy** | Decide if `dev` and `prod` must install simultaneously on the same device. Yes → Android needs `applicationIdSuffix = ".dev"`, iOS needs separate bundle IDs, Windows needs distinct MSIX `identity_name`. No → simpler flavor setup. |
| 🔴 Must | **Toolchain pinning** | Use the latest stable Flutter and the AGP / KGP / Gradle versions it generates; **Java 17** minimum, iOS deployment target 15 minimum. The project pins its own versions (`settings.gradle.kts`, Gradle wrapper, `pubspec.lock`, CI image) so contributors and CI agree. |
| 🔴 Must | **iOS UIScene migration plan** | If `AppDelegate` is customized (analytics, deep links, custom plugin registration), plan the migration to `FlutterImplicitEngineDelegate` and `didInitializeImplicitFlutterEngine` before the first iOS prod release on Xcode 26. Mandatory for App Store submissions from April 2026. |
| 🔴 Must | **Android 16 KB compliance audit** | If targeting Android 15+ (and you must, for new submissions), audit every dependency that bundles `.so` files for 16 KB-page alignment before tagging the first prod release. Mandatory from Nov 1 2025; broader cutoff May 31 2026. |
| 🟡 Soon | **iOS provisioning profile plan** | Confirm the prod bundle ID has an App Store distribution profile and the dev bundle ID has a development profile. Confirm before the first `flutter build ipa --flavor prod`. |
| 🟡 Soon | **Windows distribution channel** | Sideloaded MSIX (your code-signing certificate trusted by target machines) vs Microsoft Store (Partner Center signing). Decide before generating the first MSIX. |
| 🟡 Soon | **ProGuard keep rules** | Add the Flutter engine and `sqflite` keep rules to `android/app/proguard-rules.pro` before the first `flutter build apk --flavor prod --release`. |
| 🟡 Soon | **Debug symbol storage location** | Decide where `--split-debug-info` symbols are archived (e.g. `releases/v1.2.3/symbols/`) before the first prod release build runs. Without symbols, crash reports from that version are undecodable forever. |
| 🟢 Later | **MSIX identity strategy** | If side-by-side install on Windows is required, plan how to switch `msix_config.identity_name` between flavors (separate `pubspec_dev.yaml`, build script substitution, etc.). |
| 🟢 Later | **Flavor-specific assets** | Dev app icon (with badge) vs prod app icon, dev splash screen vs prod splash screen. Add when the visual differentiation is actually needed. |

---

## How can AI use this document? What do you need to do?

### How AI uses it

When `docs/guidelines/flutter_build_flavors_guide.md` is referenced from `CLAUDE.md`, an AI coding assistant reads it and can:

- Know that `--dart-define=FLUTTER_APP_FLAVOR=<value>` always fails the build on Flutter ≥ 3.19, and correctly use `--flavor` on Android/iOS and `--dart-define=APP_FLAVOR=<value>` on Windows.
- Know that the `AppFlavorConfig` must read both `APP_FLAVOR` and `FLUTTER_APP_FLAVOR` and never suggest a single-variable implementation that works only on mobile.
- Know that `android/key.properties` and `android/*.jks` must be in `.gitignore` and refuse to commit them even if asked.
- Know which Android signing strategy is in force for the project and produce `build.gradle.kts` that matches it (rather than blending strategies).
- Know that `freezed` and `json_serializable` do not need ProGuard keep rules — and not bloat `proguard-rules.pro` with unnecessary rules.
- Know that `--target-platform` is not a substitute for `--split-per-abi` and produce the correct command for split APK builds.
- Know the Windows-specific `sqfliteFfiInit()` requirement and place it correctly in the `main()` initialization sequence (after `WidgetsFlutterBinding.ensureInitialized()`, before any database open).
- Know that iOS xcconfig files MUST NOT set `DART_DEFINES=FLUTTER_APP_FLAVOR%3D...` and refuse to add that line even if asked to "make iOS pass the flavor explicitly."
- Know that `--obfuscate` is not a strong security boundary on its own — and not oversell it when explaining release hardening.
- Know the symbol archive policy and remind the user to archive `build/symbols/` after every prod release build.
- **Know that `flutter pub run` is deprecated** and use `dart run` (notably `dart run msix:create` and `dart run build_runner build --delete-conflicting-outputs`).
- **Look up toolchain versions, never guess them.** Read `flutter --version`, `android/settings.gradle.kts` and the Gradle wrapper before touching build files, and use what Flutter generates — never a version from memory. Know the Java 17 / iOS 15 minimums, and never write a `kotlinOptions { }` block on AGP 9+ (use `kotlin { compilerOptions { } }`).
- **Know that on iOS, custom `AppDelegate` code that previously lived in `application:didFinishLaunchingWithOptions:` may need to move to `didInitializeImplicitFlutterEngine`** after the UIScene migration in Flutter 3.38+. Flavor-specific analytics keys, deep-link handlers, and plugin registration are typical examples.
- **Know that Android 15+ targets require 16 KB page-size alignment** for Play Store submissions from Nov 1 2025 (extended deadline May 31 2026), and recommend `--analyze-size` review of bundled `.so` files before each release.

Without this guide, the AI applies its own assumptions — which on a topic with this many platform-specific gotchas frequently produces commands that look correct but fail at `kernel_snapshot_program` or silently read the wrong flavor at runtime.

### What you need to do

**Step 1 — Use it from the guidelines submodule (do not copy it)**

```
<project_root>/docs/guidelines/flutter_build_flavors_guide.md
```

**Step 2 — Add a rule to your `CLAUDE.md`**

```
Rule N: Before suggesting any build, run, signing, or flavor-related command, read
        docs/guidelines/flutter_build_flavors_guide.md.

  Build command rules:
  - Android/iOS: --flavor <name> only. NEVER pass --dart-define=FLUTTER_APP_FLAVOR=...
  - Windows/Linux/macOS: --dart-define=APP_FLAVOR=<name> only.
  - All prod release builds: --obfuscate --split-debug-info=build/symbols/<platform>-<flavor>-<version>/
  - Always use `dart run` — `flutter pub run` is deprecated (e.g. `dart run msix:create`,
    `dart run build_runner build --delete-conflicting-outputs`).

  Toolchain rules:
  - Use the latest stable Flutter. Use the AGP / KGP / Gradle versions it generates.
  - Read versions from `flutter --version`, android/settings.gradle.kts and the Gradle
    wrapper. Never write a toolchain version from memory.
  - Java 17 minimum for Android.
  - On AGP 9+ never use kotlinOptions { }. Use kotlin { compilerOptions { } }.
  - On Gradle 9+ there is no project.exec { } — custom tasks use ExecOperations.
  - iOS deployment target: iOS 15 minimum (since Flutter 3.47).
  - Android 15+ targets must verify 16 KB page-size alignment of bundled .so files
    before Play Store submission.

  iOS UIScene rules (Flutter ≥ 3.38, default since 3.41, required to launch with Xcode 27):
  - Custom AppDelegate code (analytics init, deep-link handlers, plugin registration)
    moves to didInitializeImplicitFlutterEngine.
  - AppDelegate must adopt FlutterImplicitEngineDelegate when customized.
  - Info.plist must include UIApplicationSceneManifest.

  Signing rules:
  - Never suggest committing android/key.properties, android/*.jks, or android/*.keystore.
  - Match the project's documented signing strategy (architecture.md §15).

  Initialization rules:
  - Windows/Linux: sqfliteFfiInit() before any database open in main().
  - Always use the two-variable AppFlavorConfig pattern (APP_FLAVOR + FLUTTER_APP_FLAVOR).
```

**Step 3 — Reference specific sections when asking for code**

Instead of:

> *"Set up Android flavors"*

Say:

> *"Following flutter_build_flavors_guide.md Android Flavor Setup section, signing Strategy A, set up dev and prod flavors with applicationIdSuffix '.dev' for dev. Include the Gradle execution-time guard for prod releases."*

Instead of:

> *"Add a Windows build command"*

Say:

> *"Following flutter_build_flavors_guide.md Windows Desktop Flavor Setup section, give me the prod release build command including obfuscation and split-debug-info path."*

**Step 4 — Verify AI output against the guide's gotchas**

After the AI gives you build commands or flavor configuration, quickly verify:

- Does any command pass `--dart-define=FLUTTER_APP_FLAVOR=...`? (Wrong — will fail the build.)
- Does the Windows command use `--flavor`? (Wrong — not supported on desktop.)
- Does the `AppFlavorConfig` read only one variable? (Wrong — desktop will read `prod` regardless.)
- Does the iOS xcconfig set `DART_DEFINES`? (Wrong — same `kernel_snapshot_program` failure.)
- Does any release command lack `--obfuscate --split-debug-info=...`? (Security and symbolication regression.)
- Does any command use `flutter pub run`? (Deprecated — should be `dart run`.)
- Does any Android file use AGP / KGP / Gradle versions older than what the project's Flutter generates, or a `kotlinOptions { }` block on AGP 9+? (Wrong — use the versions Flutter generates; AGP 9 rejects `kotlinOptions`.)
- Does any iOS suggestion put plugin registration or flavor-conditional init in `application:didFinishLaunchingWithOptions:` for a UIScene-migrated project? (Wrong — should be in `didInitializeImplicitFlutterEngine`.)
- Does the iOS `Podfile` declare a deployment target below `15.0`? (Wrong — the minimum is 15 since Flutter 3.47.)
- Is Java 17 actually installed and selected? (Required; Java 11 builds fail.)

If any check fails, cite the section and ask for a correction.

**Step 5 — The guide is stable; your project's choices live elsewhere**

Unlike `architecture.md` and `security.md`, this guide does not change as your project evolves — it documents Flutter framework behavior, not your project's decisions. Your project-specific choices (which signing strategy, where symbols are archived, whether dev/prod install side by side) belong in `architecture.md §15` and `release_process.md §5`. The guide is the *reference*; your docs are the *record*. Once referenced from `CLAUDE.md` (at `docs/guidelines/`), every future session reads it. You do not re-explain it. Per task you indicate which sections are most relevant — this focuses the AI precisely rather than asking it to apply all sections equally to every small change.

---

## How this fits with the other documents

`README.md` lists every document in the guideline set and which ones apply to an app, by profile.
In short: the **references** (engineering standard, flavors guide, store gates, `guideline.md`) say
*how* to build and ship; the app's filled-in **templates** in `docs/` (`PROJECT_PROFILE.md`,
`architecture.md`, `security.md`, `release_process.md`) record *what this app decided*.
