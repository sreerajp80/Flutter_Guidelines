# Release Process

Fill in this document in full for repositories that ship builds to QA, external testers, enterprise
distribution, or public app stores.

Every app has this file (`DOCS_FOLDER_GUIDELINE.md` §6). If the `Production App Extension` does
not apply yet, fill in §1 only and write "not yet shipping" as the release profile.

---

## 1. Release Scope

- App: `<app name>`
- Release profile: `internal`, `beta`, `public`, or `not yet shipping`
- Supported release platforms (copy from `docs/PROJECT_PROFILE.md`; delete the rest):
  - `Android` — channels: `<Google Play / direct APK>`
  - `iOS` — channels: `<App Store>`
  - `Windows` — channels: `<Microsoft Store / signed direct download>`
  - `macOS` — channels: `<Mac App Store / Developer ID notarized download>`
  - `Linux` — channels: `<Snap Store / Flathub / AppImage / .deb / .rpm>`
- Store gates in force: the sections of `docs/guidelines/platform_store_readiness.md` for each channel above.
- Engineering standard profiles in force:
  - `Core Baseline`
  - `Production App Extension`
  - `Sensitive Data Extension` if applicable

---

## 2. Roles And Responsibilities

| Role | Responsibility | Owner |
|------|----------------|-------|
| Release owner | Coordinates release readiness and final sign-off | `<name/team>` |
| Engineering | Code freeze, fixes, validation | `<name/team>` |
| QA | Test execution and regression sign-off | `<name/team>` |
| Store or distribution owner | Uploads artifacts and manages release metadata | `<name/team>` |

---

## 3. Versioning Policy

- Version format: `MAJOR.MINOR.PATCH+BUILD`
- Source of truth: `pubspec.yaml`
- Build-number increment rule: `<rule>`
- Git tag format: `vX.Y.Z`

---

## 4. Branch And Merge Policy

- Release branch strategy: `<main only / release branches / trunk-based>`
- Hotfix strategy: `<strategy>`
- Required checks before merge:
  - `<ci checks>`
  - `<review requirements>`

---

## 5. Environment And Flavor Matrix

| Flavor | Mode | Purpose | Example Command |
|--------|------|---------|-----------------|
| `dev` | `debug` | Local development | `flutter run --flavor dev` |
| `dev` | `release` | Release-like QA | `flutter build apk --flavor dev --release` |
| `prod` | `release` | Final release artifact | See sections 9–11B for full commands with all required flags |

Adjust the matrix if the project uses `staging`, `qa`, or no flavors.

> **Flavor signal at build time.** On Android and iOS, `--flavor <name>` is sufficient —
> the Flutter tool auto-injects `FLUTTER_APP_FLAVOR` and rejects any attempt to set it via
> `--dart-define`. On Windows, macOS and Linux desktop, pass
> `--dart-define=APP_FLAVOR=<name>` instead. See
> `docs/guidelines/flutter_build_flavors_guide.md` and `docs/guidelines/flutter_project_engineering_standard.md §5.2`
> for the full rationale and the matching `AppFlavorConfig` reader.

---

## 6. Release Build Hardening

All production release builds MUST include the following flags. Omitting any of them is a
release-blocking issue.

### 6.1 Obfuscation And Debug Symbols

```bash
--obfuscate
--split-debug-info=build/symbols/<platform>-<flavor>-<version>/
```

`<platform>` is `android`, `ios`, `windows`, `macos` or `linux`; drop `-<flavor>` when the app has no
flavors; `<version>` is the full `pubspec.yaml` version, e.g. `1.4.0+27`.

`--obfuscate` renames Dart class and method names in the compiled binary to meaningless
identifiers. This serves two purposes:
- **Security**: prevents trivial reverse engineering of application logic from the release binary.
- **Size**: reduces binary size marginally.

`--split-debug-info` extracts the debug symbol mapping to a separate directory. This is
mandatory when `--obfuscate` is used because the symbols are required to decode stack traces
from crash reports.

**Symbol archive policy:**
- The symbols directory MUST be archived securely after every production release build.
- Symbols MUST be retained for the lifetime of the released version.
- Symbols MUST NOT be committed to source control (`build/` is already in `.gitignore`).
- Copy them to the secure archive **outside the repository**, next to the release artifact (for
  example `<archive>/v1.2.3/symbols/`), and record the location in §14.
- Without the symbols, stack traces from that version are permanently unreadable.

### 6.2 ProGuard / R8 (Android)

Android release builds run R8 code shrinking. Verify `proguard-rules.pro` is present and covers:
- Flutter engine classes: `io.flutter.**`
- Any package using reflection (sqflite, Firebase, etc.)
- JSON serialization annotations

Always perform a full release build test after adding a new dependency, as R8 can silently strip
classes only accessed via reflection. Symptoms: `ClassNotFoundException` or `NoSuchMethodException`
only in release builds.

Reference: `android/app/proguard-rules.pro` and `docs/guidelines/flutter_build_flavors_guide.md`.

### 6.3 App Size Analysis

Run size analysis before every release to catch dependency bloat early:

```bash
# Android
flutter build apk --flavor prod --release \
  --analyze-size

# iOS
flutter build ipa --flavor prod --release \
  --analyze-size

# Windows
flutter build windows --release \
  --dart-define=APP_FLAVOR=prod \
  --analyze-size

# macOS
flutter build macos --release \
  --dart-define=APP_FLAVOR=prod \
  --analyze-size

# Linux
flutter build linux --release \
  --dart-define=APP_FLAVOR=prod \
  --analyze-size
```

Run only the commands for declared platforms.

Record the output in the release evidence section. Compare against the previous release.
A size increase of more than 10% without a documented justification is a review item.

Size budgets (from engineering standard):

| Platform | Target | Hard Limit |
|----------|--------|------------|
| Android APK arm64 | < 30 MB | 50 MB |
| Android AAB download | < 20 MB | 40 MB |
| iOS App Store download | < 40 MB | 80 MB |
| Windows MSIX | < 80 MB | 150 MB |
| macOS `.app` / DMG | < 80 MB | 150 MB |
| Linux bundle / package | < 80 MB | 150 MB |

### 6.4 Debuggable And Backup Verification (Android)

Verify `android:debuggable` and `android:allowBackup` in the merged release manifest before every
production release.

1. **`android:debuggable=false`**: A debuggable release build allows an attacker to attach a
   debugger via ADB, inspect memory, execute arbitrary code, and bypass application controls.
   It is also a Google Play policy violation.
2. **`android:allowBackup=false`**: If `allowBackup` is enabled (`true`), an attacker with physical
   or ADB access to an unlocked device can execute `adb backup` and pull the entire application sandbox
   (including databases, shared preferences, and internal files) without root. For apps handling
   sensitive user data (Sensitive Data Extension), `android:allowBackup` MUST be `false` (or strictly
   restricted via `fullBackupContent`). Other apps choose deliberately and record the choice in
   `docs/security.md` §10.

Check via `aapt2`:

```bash
# bash / zsh
APK="build/app/outputs/apk/prod/release/app-arm64-v8a-prod-release.apk"
# Check debuggable (must be absent or false)
aapt2 dump badging "$APK" | grep -i debuggable
# Check allowBackup (search manifest)
aapt2 dump xmltree "$APK" --file AndroidManifest.xml | grep -i allowBackup
```

```powershell
# PowerShell (Windows)
$APK = "build\app\outputs\apk\prod\release\app-arm64-v8a-prod-release.apk"
# Check debuggable (must be absent or false)
aapt2 dump badging $APK | Select-String -Pattern "debuggable"
# Check allowBackup
aapt2 dump xmltree $APK --file AndroidManifest.xml | Select-String -Pattern "allowBackup"
```

**Expected results:**
- `debuggable`: No `application-debuggable` line present (defaults to false when absent).
- `allowBackup`: matches the choice in `docs/security.md` §10. Under the Sensitive Data Extension:
  `android:allowBackup(0x...)=0x0` (false), or explicit exclusion rules.

Alternatively, inspect in Android Studio:
Build → Analyze APK → Select APK → AndroidManifest.xml → confirm `debuggable` is absent/false and `allowBackup` matches the recorded choice.

### 6.5 Network Security Configuration And Cleartext Traffic

Verify that cleartext HTTP traffic is disabled for production builds:
- Ensure `android:usesCleartextTraffic="false"` in the merged manifest (enforced by default on Android 9+ / API 28+).
- If `android:networkSecurityConfig` is defined in `res/xml/network_security_config.xml`, verify:
  - `<base-config cleartextTrafficPermitted="false">` is in place.
  - In production builds, no `<trust-anchors>` allow user-installed certificates (which would enable easy MITM proxy inspection via tools like Charles, Proxyman, or Burp Suite).

### 6.6 Pre-Release Asset And Secret Leak Audit

While `--obfuscate` scrambles compiled Dart logic in `libapp.so`, **files in the APK's `assets/` and `res/` directories remain completely unencrypted**. Any party with access to the APK can inspect its contents with `unzip` or `apktool`.

Before releasing, audit the bundled assets in the APK:

```bash
# bash / zsh
unzip -l build/app/outputs/apk/prod/release/app-arm64-v8a-prod-release.apk "assets/*"
```

```powershell
# PowerShell (Windows)
tar -tf build\app\outputs\apk\prod\release\app-arm64-v8a-prod-release.apk | Select-String "assets/"
```

**Audit checklist:**
- [ ] No `.env`, `.env.production`, or secret credentials files are packaged in `assets/`.
- [ ] No private keys, `.pem`, `.p12`, or test keystores exist in assets.
- [ ] No raw database files containing seed customer or internal test data are bundled.
- [ ] Only declared runtime config (e.g., `assets/config/app_config.json`) is included.

### 6.7 Exported Component Audit

Inspect all declared activities, services, and broadcast receivers in the merged manifest:
- Every component with an `<intent-filter>` automatically requires an explicit `android:exported` attribute.
- Components intended strictly for internal app usage MUST specify `android:exported="false"`.
- If an activity or receiver must be exported (e.g., deep linking or OAuth callback), ensure it enforces proper input validation and permission controls.

---

## 7. Signing And Secret Handling

> Keystore location, `key.properties` naming, and the `.gitignore` rules: see
> `guideline.md §2` (the source of truth).

- Signing config location: `<environment variables / secrets manager / CI secret store>`
- Keystore or certificate ownership: `<owner>`
- Secret rotation process: `<brief process>`
- Rules:
  - Signing material must not live in source control.
  - Local signing helpers must not expose secrets in committed files.
  - CI logs must not print signing secrets.
  - Keystore files MUST be backed up in at least two separate secure locations.
    Losing the keystore means being unable to publish updates to the Play Store for that app.
  - The same rules apply to Apple certificates and provisioning profiles, App Store Connect API
    keys, notarization credentials, Windows code-signing certificates, and store upload tokens
    (Snapcraft, Partner Center). See `guideline.md §2.5`.

---

## 8. Release Checklist

Complete these items before every release.

### Code And Quality

- [ ] Required CI checks passed.
- [ ] `dart format --output=none --set-exit-if-changed .` passed.
- [ ] `flutter analyze` passed with zero warnings.
- [ ] `flutter test` passed.
- [ ] Integration tests passed if applicable.
- [ ] No critical or release-blocking bugs remain open.
- [ ] Code generation is current: `dart run build_runner build --delete-conflicting-outputs` run
      and all generated files up to date.

### Performance

- [ ] Release build profiled for jank on primary user flow.
- [ ] App size analyzed and within budget (see section 6.3).
- [ ] Startup time verified under 2 seconds on a mid-range device.

### Security

- [ ] `--obfuscate` and `--split-debug-info` applied to all release builds.
- [ ] Debug symbols archived securely for this version.
- [ ] ProGuard / R8 rules verified (Android).
- [ ] `android:debuggable=false` confirmed in merged release manifest (Android).
- [ ] `android:allowBackup` matches the choice recorded in `docs/security.md` §10; `false` (or strict
      exclusion) is MUST under the Sensitive Data Extension (Android).
- [ ] Cleartext traffic disabled (`usesCleartextTraffic=false`) and network security config verified.
- [ ] Pre-release asset audit passed — no `.env`, keys, or mock data bundled in APK `assets/` (§6.6).
- [ ] Manifest component export audit completed — no accidental `android:exported="true"` (§6.7).
- [ ] Manifest and permission review completed — no unnecessary permissions.
- [ ] Other declared platforms: iOS/macOS usage strings and entitlements, MSIX capabilities, Snap
      plugs and Flatpak `finish-args` are least-privilege (`docs/security.md` §10).
- [ ] OWASP Mobile Top 10 checklist signed off in `docs/security.md` §12 (MUST under the Sensitive
      Data Extension; SHOULD otherwise).
- [ ] Secrets, keys, and backup settings reviewed if applicable.
- [ ] Sensitive-data flows revalidated if applicable.
- [ ] Data retention and purge behavior verified.

### Localization

All references are to the engineering standard unless noted. "Declared" means listed in
`docs/PROJECT_PROFILE.md`.

- [ ] One ARB file per declared language; translation parity test passes (ARB keys,
      `app_config.json`, and help assets; §8.7).
- [ ] No untranslated template-language value left in any other language's file.
- [ ] Every applicable language pack's checklist passes (§8.5, `docs/guidelines/language_packs/`).
- [ ] New or changed translations reviewed by a fluent reader (§8.5).
- [ ] Short-label length budget respected in every declared language (§8.6).
- [ ] Every screen opened in every declared language on a clean device — no missing glyphs, no
      overflow, no clipped tall scripts (§8.3.3).
- [ ] With two or more languages: in-app language picker works — System default + each language
      by its endonym, persists across restart, applies without restart (§8.4).
- [ ] With two or more languages and Android declared: language splitting disabled in Gradle
      (`bundle.language.enableSplit = false`) so Play users receive every language (§8.1).
- [ ] Date pickers and dialogs verified under every fallback-delegate language (§8.3.1).
- [ ] Every icon-only control has a localized tooltip (§7.8).
- [ ] If the project profile enables the About signature badge: it shows, localized and centered
      (`docs/guidelines/guideline.md` §1.7).

### Store Readiness (Every Declared Channel)

- [ ] `docs/guidelines/platform_store_readiness.md` §1 (rules for every store) passes.
- [ ] Google Play: §2 gate completed — target API level re-checked, `versionCode` increased, App
      Bundle + Play App Signing, Data safety, listings and screenshots, internal testing and
      pre-launch report clean, staged rollout planned.
- [ ] Apple App Store: §3 gate completed — required Xcode/SDK, usage strings, privacy manifest,
      App Privacy, export compliance, screenshots, TestFlight build verified.
- [ ] Windows: §4 gate completed — WACK passed; Store identity values exact and `store: true`, or
      direct-download package code-signed and timestamped; clean-VM install/upgrade/uninstall.
- [ ] macOS: §5 gate completed — sandbox and entitlements verified in the release build; Mac App
      Store upload via TestFlight, or Developer ID signed + notarized + stapled DMG verified with
      `spctl` on a clean Mac.
- [ ] Linux: §6 gate completed — desktop file and AppStream metadata validated; Snap / Flatpak /
      direct packages tested on clean VMs under Wayland and X11.

### Product And Documentation

- [ ] Version in `pubspec.yaml` updated.
- [ ] Changelog or release notes updated.
- [ ] User-visible behavior changes documented.
- [ ] Required store metadata ready.

### Artifact Validation

- [ ] Intended release artifact built successfully.
- [ ] Artifact installs and launches correctly on a clean device / VM.
- [ ] Flavor and environment correct in the built artifact
      (verify via flavor banner or app title if `dev`, or absence of both in `prod`).
- [ ] Version name and build number correct.
- [ ] Release build tested end-to-end (not just debug build).

---

## 9. Android Release Steps

1. Pull the intended release commit and verify it is clean (`git status`).
2. Verify the version in `pubspec.yaml`.
3. Fetch dependencies: `flutter pub get`.
4. Run code generation: `dart run build_runner build --delete-conflicting-outputs`.
5. Run format, analyze, and test checks.
6. Build the required Android production artifacts with all hardening flags (`--release`, `--obfuscate`, `--split-debug-info`, `--split-per-abi` or `appbundle`).
7. Run size analysis and record output.
8. Verify `android:debuggable=false` and the recorded `android:allowBackup` value in the merged manifest (§6.4).
9. Perform pre-release asset extraction audit to ensure no secrets were packaged in `assets/` (§6.6).
10. Verify artifact naming, installability, and environment on a physical or emulated device.
11. Archive debug symbols from `build/symbols/` to the secure archive location.
12. Complete the Google Play readiness gate (`docs/guidelines/platform_store_readiness.md` §2) before uploading to Play.
13. Upload to the intended distribution channel (Play Store console or secure internal repository).
14. Tag the release in git: `git tag v<version>` and push.

### Android Build Commands & Examples

#### A. Multi-Flavor App (`prod` Flavor)

**Bash / macOS / Linux:**
```bash
# 1. Pre-build checks
flutter pub get
dart run build_runner build --delete-conflicting-outputs
dart format --output=none --set-exit-if-changed .
flutter analyze
flutter test

# 2. Build Split APKs for direct distribution
VERSION=$(grep '^version:' pubspec.yaml | cut -d' ' -f2)
flutter build apk \
  --flavor prod \
  --release \
  --obfuscate \
  --split-debug-info=build/symbols/android-prod-$VERSION/ \
  --split-per-abi

# 3. Build App Bundle for Google Play Store
flutter build appbundle \
  --flavor prod \
  --release \
  --obfuscate \
  --split-debug-info=build/symbols/android-prod-$VERSION/

# 4. Size analysis
flutter build apk --flavor prod --release --analyze-size
```

**PowerShell (Windows):**
```powershell
# 1. Pre-build checks
flutter pub get
dart run build_runner build --delete-conflicting-outputs
dart format --output=none --set-exit-if-changed .
flutter analyze
flutter test

# 2. Extract version from pubspec.yaml
$VERSION = (Get-Content pubspec.yaml | Select-String '^version:').ToString().Split(' ')[1].Trim()

# 3. Build Split APKs for direct distribution
flutter build apk `
  --flavor prod `
  --release `
  --obfuscate `
  --split-debug-info="build/symbols/android-prod-$VERSION/" `
  --split-per-abi

# 4. Build App Bundle for Google Play Store
flutter build appbundle `
  --flavor prod `
  --release `
  --obfuscate `
  --split-debug-info="build/symbols/android-prod-$VERSION/"

# 5. Size analysis
flutter build apk --flavor prod --release --analyze-size
```

#### B. Standard App (No Flavors)

**Bash / macOS / Linux:**
```bash
VERSION=$(grep '^version:' pubspec.yaml | cut -d' ' -f2)

# Build Split APKs
flutter build apk \
  --release \
  --obfuscate \
  --split-debug-info=build/symbols/android-$VERSION/ \
  --split-per-abi

# Build App Bundle for Google Play
flutter build appbundle \
  --release \
  --obfuscate \
  --split-debug-info=build/symbols/android-$VERSION/
```

**PowerShell (Windows):**
```powershell
$VERSION = (Get-Content pubspec.yaml | Select-String '^version:').ToString().Split(' ')[1].Trim()

# Build Split APKs
flutter build apk `
  --release `
  --obfuscate `
  --split-debug-info="build/symbols/android-$VERSION/" `
  --split-per-abi

# Build App Bundle for Google Play
flutter build appbundle `
  --release `
  --obfuscate `
  --split-debug-info="build/symbols/android-$VERSION/"
```

#### C. Post-Build APK Verification Commands

```bash
# Verify no debuggable flag, and check allowBackup against docs/security.md §10
aapt2 dump badging build/app/outputs/apk/prod/release/app-arm64-v8a-prod-release.apk | grep -i debuggable
aapt2 dump xmltree build/app/outputs/apk/prod/release/app-arm64-v8a-prod-release.apk --file AndroidManifest.xml | grep -i allowBackup

# Audit asset bundle for unencrypted secrets
unzip -l build/app/outputs/apk/prod/release/app-arm64-v8a-prod-release.apk "assets/*"
```

---

## 10. iOS Release Steps

Build machine: macOS with the Xcode version App Store Connect currently requires.

1. Pull the intended release commit and verify it is clean (`git status`).
2. Confirm signing and provisioning are valid for the prod bundle id (Apple Distribution
   certificate, App Store profile).
3. Run format, analyze, test, and code generation checks.
4. Build the iOS release artifact with all hardening flags (commands below).
5. Run size analysis and record output.
6. Validate `Info.plist` usage strings, privacy manifest, export compliance, and environment
   config (`docs/guidelines/platform_store_readiness.md` §3.3).
7. Archive debug symbols from `build/symbols/`.
8. Upload the `.ipa` with Xcode Organizer or Transporter.
9. Test the build through TestFlight on a real device, in every declared language.
10. Complete the App Store gate (`docs/guidelines/platform_store_readiness.md` §3) and submit for review.
11. Tag the release in git: `git tag v<version>` and push.

### iOS Build Commands

```bash
flutter build ipa \
  --flavor prod \
  --release \
  --obfuscate \
  --split-debug-info=build/symbols/ios-prod-<version>/

flutter build ipa \
  --flavor prod \
  --release \
  --analyze-size
```

Drop `--flavor prod` if the app has no flavors.

---

## 11. Windows Release Steps

Build machine: Windows (x64; arm64 builds need an arm64-capable toolchain).

1. Pull the intended release commit.
2. Verify the version in `pubspec.yaml` and the `msix_config` version in `pubspec.yaml`.
3. Run format, analyze, test, and code generation checks.
4. Build the Windows release.
5. Run size analysis.
6. Create the MSIX package — `store: true` for the Microsoft Store, or signed with the
   code-signing certificate (from the CI secret store) for direct download.
7. Run the Windows App Certification Kit on the package.
8. Verify the MSIX installs, upgrades from the previous version, and uninstalls cleanly on a clean
   Windows environment (not the dev machine).
9. Archive debug symbols.
10. Complete the Windows gate (`docs/guidelines/platform_store_readiness.md` §4), then submit to Partner
    Center or publish the signed download with its checksum.

### Windows Build Commands

```bash
flutter build windows \
  --release \
  --dart-define=APP_FLAVOR=prod \
  --obfuscate \
  --split-debug-info=build/symbols/windows-prod-<version>/

flutter build windows --release \
  --dart-define=APP_FLAVOR=prod \
  --analyze-size

# Use `dart run`; `flutter pub run` is deprecated and prints a warning.
dart run msix:create
```

---

## 11A. macOS Release Steps

Build machine: macOS with a current Xcode.

1. Pull the intended release commit.
2. Verify the version in `pubspec.yaml`, and the bundle id and category in `macos/Runner/`.
3. Run format, analyze, test, and code generation checks.
4. Build the macOS release with all hardening flags (commands below).
5. Run size analysis; confirm a universal binary with `lipo -archs`.
6. Verify the **release** build with its real entitlements: network, file access, and every
   sandboxed feature work (missing entitlements usually fail only in release).
7. Archive debug symbols.
8. Distribute:
   - **Mac App Store**: archive in Xcode (`macos/Runner.xcworkspace`) → Distribute App → App Store
     Connect; test through TestFlight; submit for review.
   - **Developer ID**: sign with hardened runtime, notarize with `xcrun notarytool`, staple with
     `xcrun stapler`, package as a signed, notarized and stapled DMG, and verify with `spctl` on a
     clean Mac (`docs/guidelines/platform_store_readiness.md` §5.3).
9. Complete the macOS gate (`docs/guidelines/platform_store_readiness.md` §5).

### macOS Build Commands

```bash
flutter build macos \
  --release \
  --dart-define=APP_FLAVOR=prod \
  --obfuscate \
  --split-debug-info=build/symbols/macos-prod-<version>/

# Output: build/macos/Build/Products/Release/<App>.app
```

---

## 11B. Linux Release Steps

Build machine: the oldest Linux distribution you support (or its container), with the GTK
build packages from engineering standard §5.5.3.

1. Pull the intended release commit.
2. Verify the version in `pubspec.yaml`, `APPLICATION_ID` in `linux/CMakeLists.txt`, and the
   version in packaging files (`snap/snapcraft.yaml`, Flatpak manifest, AppStream `releases`).
3. Run format, analyze, test, and code generation checks.
4. Build the Linux release with all hardening flags (commands below).
5. Run size analysis.
6. Archive debug symbols.
7. Package for each declared channel: `.snap`, Flatpak, AppImage, `.deb`, `.rpm`.
8. Validate the desktop file and AppStream metadata (`desktop-file-validate`,
   `appstreamcli validate`).
9. Install and run each package on clean VMs of the main target distros, under Wayland and X11.
10. Complete the Linux gate (`docs/guidelines/platform_store_readiness.md` §6), then upload
    (`snapcraft upload`, Flathub pull request) or publish the files with checksums.

### Linux Build Commands

```bash
flutter build linux \
  --release \
  --dart-define=APP_FLAVOR=prod \
  --obfuscate \
  --split-debug-info=build/symbols/linux-prod-<version>/

# Output: build/linux/x64/release/bundle/  (arm64: build/linux/arm64/release/bundle/)

# Snap (from the repo root, with snap/snapcraft.yaml)
snapcraft
```

---

## 12. Distribution Channels

Keep one row per declared channel; delete the rest.

| Channel | Artifact | Audience | Notes |
|---------|----------|----------|-------|
| Google Play | `.aab` | Public | Staged rollout; gate §2 |
| Direct APK | split `.apk` | `<audience>` | Signed with the release keystore |
| Apple App Store | `.ipa` | Public | TestFlight first; gate §3 |
| Microsoft Store | `.msix` / `.msixbundle` (`store: true`) | Public | Gate §4.2 |
| Windows direct download | signed `.msix` or installer `.exe` | `<audience>` | Code-signed + timestamped; gate §4.3 |
| Mac App Store | Xcode archive → App Store Connect | Public | Sandbox required; gate §5.2 |
| macOS direct download | notarized + stapled `.dmg` | `<audience>` | Developer ID; gate §5.3 |
| Snap Store | `.snap` | Public | edge → beta → stable; gate §6.2 |
| Flathub | Flatpak (built by Flathub) | Public | Gate §6.3 |
| Linux direct download | AppImage / `.deb` / `.rpm` | `<audience>` | Checksums over HTTPS; gate §6.4 |

---

## 13. Rollback And Hotfix Process

- Rollback trigger: `<what forces rollback>`
- Rollback method: `<store halt / phased rollout pause / hotfix release>`
- Hotfix branch naming: `<pattern>`
- Verification after rollback or hotfix:
  - Full release checklist MUST be completed even for hotfixes.
  - Debug symbols for the hotfix build MUST be archived.

---

## 14. Release Evidence

Store links or references to release evidence here after each release.

- CI run: `<url or identifier>`
- Test report: `<url or identifier>`
- Size analysis output: `<location>`
- Debug symbols archive: `<secure location>`
- Built artifact: `<location>`
- Release notes: `<location>`
- Store submission or rollout record: `<location>`
- OWASP checklist sign-off (required under the Sensitive Data Extension): `<signed by / date>`

---

## 15. Post-Release Checks

- [ ] Crash and error monitoring reviewed (if applicable; for offline apps: post-install test on
      clean device).
- [ ] Analytics or telemetry sanity checked if applicable.
- [ ] User-reported issues triaged.
- [ ] Release tag created and pushed: `git tag v<version> && git push origin v<version>`.
- [ ] Debug symbols confirmed in secure archive.
- [ ] Follow-up tasks recorded.
