## What does `flutter_build_flavors_guide.md` say?

> **Version-independent.** The guide does not pin tool versions and does not tie a rule to one
> Flutter release. It says "use the latest stable Flutter and the Android versions it
> generates", and where tools differ it tells you how to check your own toolchain
> (`flutter --version`, `flutter <command> -h`, the files `flutter create` writes). Version
> numbers appear only in its dated "Reference snapshot — not a rule" table.

It's a **platform-by-platform technical reference** for setting up Flutter build flavors. Unlike `architecture.md` and `security.md` (project-specific templates) or `flutter_project_engineering_standard.md` (a universal rulebook), this guide is **task-focused**: it tells you exactly how to wire flavors into Android, iOS, Windows, macOS and Linux builds, and explains the framework-level rules that constrain those choices.

It has the following major sections:

- **Toolchain Prerequisites** — the minimums you must meet (the JDK your AGP needs, the Flutter minimum iOS / macOS versions, Linux build packages, the Xcode / SDK App Store Connect accepts, Play's target API level and 16 KB pages) with **where to find the current value** for each, plus a dated "reference snapshot" of what Flutter generated when last checked (not a rule). The toolchain rules themselves live in engineering standard §5.3.
- **Flavor basics** — what a flavor represents, the typical `dev`/`prod` matrix, and the four flavor × mode combinations (debug/release × dev/prod) with their signing implications.
- **How the flavor reaches Dart code** — `FLUTTER_APP_FLAVOR` is owned by the framework and must never be set by hand. One rule for every platform: use `--flavor` where `flutter build <platform> -h` lists it and the native setup exists; otherwise fall back to `--dart-define=APP_FLAVOR=<name>`. Two ways to read the flavor at runtime (`appFlavor` constant vs `String.fromEnvironment`), and why a project that uses the fallback needs the two-variable `AppFlavorConfig`.
- **Android flavor setup** — Kotlin DSL is what `flutter create` writes (Groovy DSL note for older projects), the **Android toolchain baseline** (where `settings.gradle.kts` and the wrapper set the versions, `kotlin { compilerOptions { } }` instead of the legacy `kotlinOptions { }`, keeping the `gradle.properties` flags Flutter writes, built-in Kotlin as an optional separate migration, and `ExecOperations` instead of the removed `project.exec`), product flavors in Gradle, two signing strategies (local file, separate keystores per flavor; signing is never done in CI), step-by-step keystore creation with `keytool`, the `key.properties` pattern, the `.gitignore` rules, the full `build.gradle.kts` example with a Gradle execution-time guard for prod releases, and the caveats of that guard.
- **ProGuard / R8 rules** — R8 full mode is the default; Flutter already adds the engine's keep rules and the app's `proguard-rules.pro` by itself; plugins ship their own rules; add a rule only when a package documents it; the Dart side (including `freezed` / `json_serializable` output) is never processed by R8.
- **Which Android artifact to use** — split APKs vs AAB, and the important distinction between `--target-platform` and `--split-per-abi`.
- **16 KB page size compliance** — a pointer to the rule and current deadline in engineering standard §5.3.1, and how to check the real artifact (`zipalign -c -P 16`, Android Studio's APK Analyzer, a 16 KB emulator system image).
- **iOS flavor setup** — at least the Flutter minimum deployment target, **the UIScene lifecycle (required by Apple for apps built with the SDK after iOS 26; auto-migration for unmodified `AppDelegate`; manual steps and a correct `didInitializeImplicitFlutterEngine(_ engineBridge: FlutterImplicitEngineBridge)` example for customized ones)**, the `Debug-<flavor>` / `Release-<flavor>` build configurations and matching schemes Flutter needs, the bundle ID and display name per build configuration, the Podfile mapping, why Xcode settings MUST NOT set `DART_DEFINES=FLUTTER_APP_FLAVOR%3D...`, and provisioning profile rules per flavor.
- **Desktop flavors — which flag to use** — a one-time check per desktop platform (`flutter build <platform> -h`) that decides between `--flavor` and the `APP_FLAVOR` fallback, recorded in `architecture.md §15`.
- **Windows desktop flavor setup** — `--flavor` needs no CMake changes when the tool supports it; MSIX packaging via `dart run msix:create` with **`build_windows: false`, so it packages the hardened build instead of making its own unobfuscated one**; one MSIX per architecture; side-by-side MSIX identity. Desktop setup that is not flavor-specific (`sqflite_common_ffi`, window size) is in engineering standard §5.5.
- **macOS desktop flavor setup** — `--flavor` with a per-flavor `AppInfo` xcconfig and Xcode scheme when dev and prod must install side by side, the `APP_FLAVOR` fallback otherwise, and keeping debug-only entitlements out of `Release.entitlements`.
- **Linux desktop flavor setup** — `--flavor` or the fallback, and a CMake snippet that gives a dev build its own `APPLICATION_ID` in either case (placed after `add_subdirectory(${FLUTTER_MANAGED_DIR})`).
- **Recommended release matrix** — a complete table of platform × flavor × mode × signing × command for the typical Flutter project, with `--tree-shake-icons` behavior explained.
- **Debug symbol management** — what `--obfuscate` actually does (and doesn't do), the symbol archive policy, why losing the symbols means crash reports become undecodable.
- **Notes for new projects** — a per-platform setup checklist to run when bootstrapping a new repository, including UIScene and 16 KB requirements.

Unlike the templates, this guide is **read-only reference** — you do not fill anything in. You consult it when wiring up build configuration, then record your actual decisions in `architecture.md §15` and `release_process.md §5`.

---

## How do you use it in a Flutter project?

Think of it as three things simultaneously.

**1. A pre-flight checklist before the first `flutter build` command.** Before running any flavored build, verify that your `AppFlavorConfig` reads both `APP_FLAVOR` and `FLUTTER_APP_FLAVOR` (the two-variable pattern), your Android `key.properties` is in `.gitignore`, your iOS xcconfig files don't try to set `DART_DEFINES=FLUTTER_APP_FLAVOR%3D...`, and each desktop platform uses the flag your "which flag to use" check selected (never `FLUTTER_APP_FLAVOR` as a dart-define). Getting any of these wrong produces the `kernel_snapshot_program failed` build error or — worse — a build that succeeds but reads the wrong flavor at runtime.

**2. A copy-paste reference during setup.** When you are configuring `android/app/build.gradle.kts`, the example block is ready to drop in. When you are creating Xcode build configurations and schemes, every step is spelled out. When you are packaging Windows with MSIX, the `pubspec.yaml` config is provided. You read the section, copy the relevant pattern, adapt it to your project's names.

**3. A decision recorder.** Every section has decision points: which Android signing strategy (A or B), whether to commit `key.properties` to source control (no — Strategy A documented), which flavor flag each desktop platform uses, whether dev and prod must install side by side on Windows (different `identity_name` if yes), whether to use split APKs or AAB (channel-dependent). The guide forces those decisions to be made consciously and points you to record them in `architecture.md §15`.

---

## What should you decide BEFORE running the first flavored build?

The build flavors guide is **not a fill-in-the-blanks template**. You read it, make decisions from it, then record those decisions in your project's `architecture.md §15` and `release_process.md §5`.

These decisions must be made before the first flavored build is attempted, because each one shapes file layout that is expensive to change later:

| Priority | Decision | Why Before First Build |
|----------|----------|-----------------------|
| 🔴 Must | **Flavor names** | Decide `dev` and `prod` (or your variant). The names propagate to Gradle product flavors, Xcode schemes, xcconfig folders, `applicationIdSuffix` values, and `AppFlavorConfig` enum cases. Renaming later means touching every native project. |
| 🔴 Must | **Two-variable `AppFlavorConfig`** | Implement the pattern from engineering standard §5.2: read `APP_FLAVOR` first, fall back to `FLUTTER_APP_FLAVOR`. A single-variable version breaks silently on any platform that uses the `APP_FLAVOR` fallback: the flavor reads as `prod` regardless of the build command. |
| 🔴 Must | **Flavor flag per desktop platform** | Run `flutter build <platform> -h` for each declared desktop platform: `--flavor` if listed (and, on macOS, per-flavor schemes exist), otherwise `--dart-define=APP_FLAVOR`. Record the result in `architecture.md §15` and in the `CLAUDE.md` / `AGENTS.md` command blocks. |
| 🔴 Must | **Android signing strategy** | Choose Strategy A (local `key.properties`) or B (separate keystores per flavor). Each implies different `.gitignore` rules and a different `build.gradle.kts` shape. Signing is always local — GitHub CI never signs or holds signing secrets. |
| 🔴 Must | **`.gitignore` rules** | Add `android/key.properties`, `android/*.jks`, `android/*.keystore`, and `build/` (which also holds the debug symbols) before the first build runs — once a keystore is committed it is a security incident. |
| 🔴 Must | **Side-by-side install policy** | Decide if `dev` and `prod` must install simultaneously on the same device. Yes → Android needs `applicationIdSuffix = ".dev"`, iOS needs separate bundle IDs, Windows needs distinct MSIX `identity_name`. No → simpler flavor setup. |
| 🔴 Must | **Toolchain pinning** | Use the latest stable Flutter and the AGP / KGP / Gradle versions it generates, the JDK that AGP needs, and at least the Flutter minimum iOS / macOS deployment targets. The project pins its own versions (`settings.gradle.kts`, Gradle wrapper, `pubspec.lock`, CI) so contributors and CI agree. |
| 🔴 Must | **iOS UIScene migration plan** | If `AppDelegate` is customized (analytics, deep links, custom plugin registration, method channels), plan the migration to `FlutterImplicitEngineDelegate` and `didInitializeImplicitFlutterEngine` before you build with an Apple SDK that requires the UIScene lifecycle. |
| 🔴 Must | **Android 16 KB compliance check** | If targeting Android 15 (API 35) or higher, check the release artifact's native libraries for 16 KB alignment before the first prod release, and before each later one. The current Play deadline is in engineering standard §5.3.1. |
| 🟡 Soon | **iOS provisioning profile plan** | Confirm the prod bundle ID has an App Store distribution profile and the dev bundle ID has a development profile. Confirm before the first `flutter build ipa --flavor prod`. |
| 🟡 Soon | **Windows distribution channel** | Sideloaded MSIX (your code-signing certificate trusted by target machines) vs Microsoft Store (signed by the Store). Decide before generating the first MSIX, and set `build_windows: false` in `msix_config`. |
| 🟡 Soon | **ProGuard keep rules** | Usually none are needed: Flutter adds the engine rules and plugins ship their own. Create `android/app/proguard-rules.pro` only for rules a package documents, and test the first `flutter build apk --flavor prod --release`. |
| 🟡 Soon | **Debug symbol storage location** | Decide where `--split-debug-info` symbols are archived (e.g. `releases/v1.2.3/symbols/`) before the first prod release build runs. Without symbols, crash reports from that version are undecodable forever. |
| 🟢 Later | **MSIX identity strategy** | If side-by-side install on Windows is required, plan how to switch `msix_config.identity_name` between flavors (command-line overrides in a dev build script, config swap, etc.). |
| 🟢 Later | **Flavor-specific assets** | Dev app icon (with badge) vs prod app icon, dev splash screen vs prod splash screen. Add when the visual differentiation is actually needed. |

---

## How can AI use this document? What do you need to do?

### How AI uses it

When `docs/guidelines/flutter_build_flavors_guide.md` is referenced from `CLAUDE.md`, an AI coding assistant reads it and can:

- Know that `--dart-define=FLUTTER_APP_FLAVOR=<value>` always fails the build, and pick `--flavor` or the `APP_FLAVOR` fallback per platform by checking `flutter build <platform> -h` (or by reading the choice recorded in `architecture.md §15`).
- Know that the `AppFlavorConfig` must read both `APP_FLAVOR` and `FLUTTER_APP_FLAVOR` and never suggest a single-variable implementation.
- Know that `android/key.properties` and `android/*.jks` must be in `.gitignore` and refuse to commit them even if asked.
- Know which Android signing strategy is in force for the project and produce `build.gradle.kts` that matches it (rather than blending strategies).
- Know that Flutter already adds the engine's R8 keep rules, and not bloat `proguard-rules.pro` with blanket or unnecessary rules.
- Know that `--target-platform` is not a substitute for `--split-per-abi` and produce the correct command for split APK builds.
- Know the Windows/Linux `sqfliteFfiInit()` requirement and place it correctly in the `main()` initialization sequence (after `WidgetsFlutterBinding.ensureInitialized()`, before any database open).
- Know that iOS xcconfig files MUST NOT set `DART_DEFINES=FLUTTER_APP_FLAVOR%3D...` and refuse to add that line even if asked to "make iOS pass the flavor explicitly."
- Know that `--obfuscate` is not a strong security boundary on its own — and not oversell it when explaining release hardening.
- Know that `dart run msix:create` must package the hardened build (`build_windows: false`), never rebuild it without `--obfuscate`.
- Know the symbol archive policy and remind the user to archive `build/symbols/` after every prod release build.
- **Know that `flutter pub run` is deprecated** and use `dart run` (notably `dart run msix:create` and `dart run build_runner build --delete-conflicting-outputs`).
- **Look up toolchain versions, never guess them.** Read `flutter --version`, `android/settings.gradle.kts`, `android/gradle.properties` and the Gradle wrapper before touching build files, and use what Flutter generates — never a version from memory. Never write a `kotlinOptions { }` block (use `kotlin { compilerOptions { } }`).
- **Know that on iOS, custom `AppDelegate` code that lived in `application:didFinishLaunchingWithOptions:` moves to `didInitializeImplicitFlutterEngine`** with the UIScene lifecycle. Flavor-specific analytics keys, deep-link handlers, method channels and plugin registration are typical examples.
- **Know that apps targeting API 35+ need 16 KB page-size alignment** for Play, and check the release artifact with `zipalign -c -P 16` or the APK Analyzer — not with `--analyze-size`.

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
  - NEVER pass --dart-define=FLUTTER_APP_FLAVOR=...
  - Flavor flag per platform (from docs/architecture.md §15):
      Android: --flavor   iOS: --flavor
      Windows: <--flavor | --dart-define=APP_FLAVOR>
      macOS:   <--flavor | --dart-define=APP_FLAVOR>
      Linux:   <--flavor | --dart-define=APP_FLAVOR>
  - All prod release builds: --obfuscate --split-debug-info=build/symbols/<platform>-<flavor>-<version>/
  - --analyze-size is a separate throw-away build; run it before the release build, never ship it.
  - Windows: msix_config has build_windows: false; run msix:create right after the hardened build.
  - Always use `dart run` — `flutter pub run` is deprecated (e.g. `dart run msix:create`,
    `dart run build_runner build --delete-conflicting-outputs`).

  Toolchain rules:
  - Use the latest stable Flutter. Use the AGP / KGP / Gradle versions it generates.
  - Read versions from `flutter --version`, android/settings.gradle.kts and the Gradle
    wrapper. Never write a toolchain version from memory.
  - Use the JDK the project's AGP needs (`flutter doctor -v`).
  - Never use kotlinOptions { }. Use kotlin { compilerOptions { } }.
  - Keep the android/gradle.properties flags Flutter writes.
  - On Gradle 9+ there is no project.exec { } — custom tasks use ExecOperations.
  - iOS / macOS deployment targets: at least what a fresh `flutter create` writes.
  - Apps targeting API 35+ must verify 16 KB alignment of bundled .so files
    before Play Store submission.

  iOS UIScene rules:
  - Custom AppDelegate code (analytics init, deep-link handlers, plugin registration,
    method channels) moves to didInitializeImplicitFlutterEngine(_ engineBridge:).
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

> *"Following flutter_build_flavors_guide.md Windows Desktop Flavor Setup section, give me the prod release build command including obfuscation and split-debug-info path, and the MSIX step."*

**Step 4 — Verify AI output against the guide's gotchas**

After the AI gives you build commands or flavor configuration, quickly verify:

- Does any command pass `--dart-define=FLUTTER_APP_FLAVOR=...`? (Wrong — will fail the build.)
- Does a desktop command use a flavor flag other than the one recorded for that platform? (Wrong — `--flavor` fails on a toolchain that does not list it, and on macOS without per-flavor schemes.)
- Does the `AppFlavorConfig` read only one variable? (Wrong — a platform on the `APP_FLAVOR` fallback will read `prod` regardless.)
- Does the iOS xcconfig set `DART_DEFINES`? (Wrong — same `kernel_snapshot_program` failure.)
- Does any release command lack `--obfuscate --split-debug-info=...`? (Security and symbolication regression.)
- Does `msix:create` run with `build_windows` left at its default? (Wrong — it ships an unobfuscated rebuild.)
- Is `--analyze-size` combined with `--split-debug-info`, or run after the release build? (Wrong — the tool refuses the first; the second overwrites the hardened artifact.)
- Does any command use `flutter pub run`? (Deprecated — should be `dart run`.)
- Does any Android file use AGP / KGP / Gradle versions older than what the project's Flutter generates, or a `kotlinOptions { }` block? (Wrong — use the versions Flutter generates and `kotlin { compilerOptions { } }`.)
- Does any iOS suggestion put plugin registration or flavor-conditional init in `application:didFinishLaunchingWithOptions:` for a UIScene-migrated project? (Wrong — should be in `didInitializeImplicitFlutterEngine`.)
- Does the iOS `Podfile` declare a deployment target below what a fresh `flutter create` writes? (Wrong — raise it.)
- Is the JDK that AGP needs installed and selected? (`flutter doctor -v`.)

If any check fails, cite the section and ask for a correction.

**Step 5 — The guide is stable; your project's choices live elsewhere**

Unlike `architecture.md` and `security.md`, this guide does not change as your project evolves — it documents Flutter framework behavior, not your project's decisions. Your project-specific choices (which signing strategy, which flavor flag each desktop uses, where symbols are archived, whether dev/prod install side by side) belong in `architecture.md §15` and `release_process.md §5`. The guide is the *reference*; your docs are the *record*. Once referenced from `CLAUDE.md` (at `docs/guidelines/`), every future session reads it. You do not re-explain it. Per task you indicate which sections are most relevant — this focuses the AI precisely rather than asking it to apply all sections equally to every small change.

---

## How this fits with the other documents

`README.md` lists every document in the guideline set and which ones apply to an app, by profile.
In short: the **references** (engineering standard, flavors guide, store gates, `guideline.md`) say
*how* to build and ship; the app's filled-in **templates** in `docs/` (`PROJECT_PROFILE.md`,
`architecture.md`, `security.md`, `release_process.md`) record *what this app decided*.
