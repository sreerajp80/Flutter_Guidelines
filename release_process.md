# Release Process

Use this document for repositories that ship builds to QA, external testers, enterprise
distribution, or public app stores.

If the repository is not release-tracked yet, keep this file short and mark the current release
scope clearly.

---

## 1. Release Scope

- App: `<app name>`
- Release profile: `internal`, `beta`, `public`, or `not yet shipping`
- Supported release platforms:
  - `Android`
  - `iOS`
  - `Windows`
  - `<other>`
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
| `prod` | `release` | Final release artifact | See section 8 for full commands with all required flags |

Adjust the matrix if the project uses `staging`, `qa`, or no flavors.

> **Flavor signal at build time.** On Android and iOS, `--flavor <name>` is sufficient —
> the Flutter tool auto-injects `FLUTTER_APP_FLAVOR` and rejects any attempt to set it via
> `--dart-define`. On Windows desktop, `--flavor` is not supported; pass
> `--dart-define=APP_FLAVOR=<name>` instead. See
> `docs/flutter_build_flavors_guide.md` and `docs/flutter_project_engineering_standard.md §5.2`
> for the full rationale and the matching `AppFlavorConfig` reader.

---

## 6. Release Build Hardening

All production release builds MUST include the following flags. Omitting any of them is a
release-blocking issue.

### 6.1 Obfuscation And Debug Symbols

```bash
--obfuscate
--split-debug-info=build/symbols/<platform>-<version>/
```

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
- Symbols MUST NOT be committed to source control (add to `.gitignore`).
- Store them alongside the release artifact: e.g. `releases/v1.2.3/symbols/`.
- Without the symbols, stack traces from that version are permanently unreadable.

### 6.2 ProGuard / R8 (Android)

Android release builds run R8 code shrinking. Verify `proguard-rules.pro` is present and covers:
- Flutter engine classes: `io.flutter.**`
- Any package using reflection (sqflite, Firebase, etc.)
- JSON serialization annotations

Always perform a full release build test after adding a new dependency, as R8 can silently strip
classes only accessed via reflection. Symptoms: `ClassNotFoundException` or `NoSuchMethodException`
only in release builds.

Reference: `android/app/proguard-rules.pro` and `docs/flutter_build_flavors_guide.md`.

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
```

Record the output in the release evidence section. Compare against the previous release.
A size increase of more than 10% without a documented justification is a review item.

Size budgets (from engineering standard):

| Platform | Target | Hard Limit |
|----------|--------|------------|
| Android APK arm64 | < 30 MB | 50 MB |
| Android AAB download | < 20 MB | 40 MB |
| Windows MSIX | < 80 MB | 150 MB |

### 6.4 Debuggable And Backup Verification (Android)

Verify that both `android:debuggable` and `android:allowBackup` are explicitly safe in the merged
release manifest before every production release.

1. **`android:debuggable=false`**: A debuggable release build allows an attacker to attach a
   debugger via ADB, inspect memory, execute arbitrary code, and bypass application controls.
   It is also a Google Play policy violation.
2. **`android:allowBackup=false`**: If `allowBackup` is enabled (`true`), an attacker with physical
   or ADB access to an unlocked device can execute `adb backup` and pull the entire application sandbox
   (including databases, shared preferences, and internal files) without root. For apps handling
   sensitive user data, `android:allowBackup` MUST be `false` (or strictly restricted via `fullBackupContent`).

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
- `allowBackup`: `android:allowBackup(0x...)=0x0` (false) or absent with explicit application-level exclusion.

Alternatively, inspect in Android Studio:
Build → Analyze APK → Select APK → AndroidManifest.xml → confirm `debuggable` is absent/false and `allowBackup` is false.

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
- [ ] `android:allowBackup=false` (or strict exclusion) verified in merged release manifest (Android).
- [ ] Cleartext traffic disabled (`usesCleartextTraffic=false`) and network security config verified.
- [ ] Pre-release asset audit passed — no `.env`, keys, or mock data bundled in APK `assets/` (§6.6).
- [ ] Manifest component export audit completed — no accidental `android:exported="true"` (§6.7).
- [ ] Manifest and permission review completed — no unnecessary permissions.
- [ ] OWASP Mobile Top 10 checklist reviewed (see `docs/security.md`).
- [ ] Secrets, keys, and backup settings reviewed if applicable.
- [ ] Sensitive-data flows revalidated if applicable.
- [ ] Data retention and purge behavior verified.

### Localization

- [ ] `app_en.arb`, `app_ml.arb` and `app_sa.arb` all present; ARB key parity test passes.
- [ ] No untranslated English value left in the Malayalam or Sanskrit file.
- [ ] Sanskrit Hindi-marker gate passes and glossary terms are used (engineering standard §8.5).
- [ ] Short-label length budget respected in all three languages (§8.6).
- [ ] Every screen opened in `en`, `ml` and `sa` on a clean device — no missing glyphs, no
      overflow, no clipped Malayalam/Devanagari ascenders (§8.3.3).
- [ ] In-app language picker works: System default / English / മലയാളം / संस्कृतम्, persists across
      restart, applies without restart (§8.4).
- [ ] Date pickers and dialogs verified under `sa` (the framework-delegate fallback, §8.3.1).
- [ ] Every icon-only control has a localized tooltip (§7.8).
- [ ] About screen shows the "Made with ❤️ from India" badge, localized and centered
      (`docs/guideline.md` §1.7).

### Google Play Store Readiness (Android)

- [ ] Full §9A gate completed for this release.
- [ ] `targetSdkVersion` meets Play's current target API level policy (re-checked, not assumed).
- [ ] `versionCode` strictly greater than every previously uploaded build.
- [ ] App Bundle built; Play App Signing enabled; native debug symbols uploaded.
- [ ] Permissions justified; sensitive-permission declarations completed in the console.
- [ ] Privacy policy URL live; Data safety form matches actual behavior; content rating done.
- [ ] Store listing assets ready at the required sizes (icon, feature graphic, screenshots).
- [ ] English and Malayalam listings complete with localized screenshots.
- [ ] Internal-testing upload done and pre-launch report clean.
- [ ] Staged rollout percentage chosen and vitals monitoring planned.

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
8. Verify `android:debuggable=false` and `android:allowBackup=false` in the merged manifest (§6.4).
9. Perform pre-release asset extraction audit to ensure no secrets were packaged in `assets/` (§6.6).
10. Verify artifact naming, installability, and environment on a physical or emulated device.
11. Archive debug symbols from `build/symbols/` to the secure archive location.
12. Complete the Google Play readiness gate (§9A) before uploading to Play.
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
# Verify no debuggable flag and verify allowBackup=false
aapt2 dump badging build/app/outputs/apk/prod/release/app-arm64-v8a-prod-release.apk | grep -i debuggable
aapt2 dump xmltree build/app/outputs/apk/prod/release/app-arm64-v8a-prod-release.apk --file AndroidManifest.xml | grep -i allowBackup

# Audit asset bundle for unencrypted secrets
unzip -l build/app/outputs/apk/prod/release/app-arm64-v8a-prod-release.apk "assets/*"
```

---

## 9A. Google Play Store Readiness (Mandatory Gate)

Every app is built to be publishable on Google Play. This gate MUST pass **before the first upload**
and MUST be re-checked before every production release. Items marked *(one-time)* are set up once
and only re-verified afterwards.

### 9A.1 Application identity and versioning

| Item | Requirement |
|---|---|
| `applicationId` *(one-time)* | Reverse-DNS, owned domain, lowercase, permanent. It can never be changed after the first publish. Flavors may append a suffix (`.dev`), but the production id MUST have no suffix. |
| `versionCode` | Strictly increasing integer on every upload, never reused — even for a rejected or rolled-back build. |
| `versionName` | Matches `pubspec.yaml` (`<version>+<build>` → `versionName+versionCode`). |
| App name | Set in `android/app/src/main/AndroidManifest.xml` via a localized `@string/app_name`, matching the store listing. |
| Package visibility | If the app queries other packages, declare `<queries>` — Play rejects silent package enumeration. |

### 9A.2 API level, ABI, and compatibility

- `targetSdkVersion` MUST meet Play's current target API level policy (Play requires new apps and
  updates to target an API level within one year of the latest major Android release; the deadline
  is typically 31 August each year). Check the current requirement before each release rather than
  trusting the value already in the project.
- `compileSdkVersion` ≥ `targetSdkVersion`.
- `minSdkVersion` is a deliberate, documented product decision — record it in `docs/architecture.md`.
- 64-bit native code is mandatory: ship an App Bundle, or split APKs including `arm64-v8a`.
- 16 KB page-size compliance is required for Android 15+ devices (see the engineering standard).
- Edge-to-edge behavior verified when targeting SDK 35+.

### 9A.3 Signing and upload

- Ship an **Android App Bundle (`.aab`)**, not an APK, to Play.
- **Play App Signing** MUST be enabled *(one-time)*. Keep the upload key backed up offline; losing
  the upload key is recoverable through Play support, losing a pre-App-Signing release key is not.
- Signing config points at `android/key.properties` (see `docs/guideline.md` §2) and is **never**
  committed.
- `flutter build appbundle --release --obfuscate --split-debug-info=...` — all three flags, always.
- Upload the native debug symbols (`build/symbols/`) to Play so crash traces de-obfuscate, and
  archive them alongside the release evidence.

### 9A.4 Manifest, permissions, and policy declarations

- Every permission in the merged manifest is justified and used. Remove anything inherited from a
  dependency that the app does not need (`tools:node="remove"`).
- Sensitive permissions require an in-console declaration and are commonly rejected: all-files
  access, exact alarms, accessibility service, SMS/call log, background location, camera/microphone
  in the background, `QUERY_ALL_PACKAGES`.
- Foreground services declare a `foregroundServiceType` and a use-case declaration in the console.
- `android:debuggable=false`, `android:allowBackup` decided deliberately, `usesCleartextTraffic=false`
  (§6.4, §6.5).
- No accidental `android:exported="true"` (§6.7).
- Ads, payments, and analytics SDKs are declared where the console asks for them.

### 9A.5 Store account declarations

- **Privacy policy URL** — reachable, public, app-specific. Required for every app, whether or not
  it collects data.
- **Data safety form** — completed and matching what the app actually does (including anything a
  bundled SDK collects). A mismatch is a policy violation.
- **Content rating questionnaire** — completed.
- **Target audience and content** — declared; if children may be a target audience, the Families
  policy applies.
- **Ads declaration**, **news app declaration**, **COVID/health declarations** — where applicable.
- **Account deletion** — if the app supports account creation, an in-app and a web-accessible
  deletion path MUST exist and be declared.
- **Developer contact details** and, for personal accounts created recently, Play's testing
  requirements before production access.

### 9A.6 Store listing assets

| Asset | Requirement |
|---|---|
| App icon | 512 × 512 PNG, 32-bit, no alpha-dependent design |
| Feature graphic | 1024 × 500 PNG/JPG |
| Phone screenshots | 2–8, PNG/JPG, 16:9 or 9:16, min 320 px, max 3840 px on the longest side |
| Tablet screenshots | Required if the app is distributed to tablets (7-inch and 10-inch sets) |
| Short description | ≤ 80 characters |
| Full description | ≤ 4000 characters |
| App title | ≤ 30 characters, no keyword stuffing, no store badges or price in the title |

### 9A.7 Localization of the listing

The app itself ships English, Malayalam and Sanskrit (engineering standard section 8).

- The Play listing MUST be provided in **English** and in **Malayalam** (`ml-IN`), including
  localized screenshots.
- **Sanskrit is not an available Play listing language.** It is shipped *inside* the app only; do
  not attempt to add it as a store locale, and do not drop it from the app because the store cannot
  list it.
- Screenshots MUST show real app UI in the language of that listing — not English screenshots under
  the Malayalam listing.

### 9A.8 Pre-launch verification

- Upload to **internal testing** first; run the Play Console **pre-launch report** and resolve all
  crashes, ANRs, and flagged accessibility and security items.
- Verify the app installs, launches, and completes its primary flow from a Play-served build (not
  just a locally installed APK), in all three languages.
- Android vitals thresholds reviewed after each rollout (crash rate, ANR rate).
- Production rollout starts as a **staged rollout** (e.g. 10% → 50% → 100%) with vitals checked at
  each step.

---

## 10. iOS Release Steps

1. Confirm signing and provisioning are valid for the prod flavor bundle ID.
2. Run format, analyze, test, and code generation checks.
3. Build the iOS release artifact.
4. Run size analysis.
5. Validate permissions, metadata, and environment config.
6. Archive debug symbols.
7. Upload through the approved pipeline (Xcode Organizer or `xcrun altool`).
8. Confirm TestFlight or App Store processing.

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

---

## 11. Windows Release Steps

1. Pull the intended release commit.
2. Verify the version in `pubspec.yaml` and the `msix_config` version in `pubspec.yaml`.
3. Run format, analyze, test, and code generation checks.
4. Build the Windows release.
5. Run size analysis.
6. Create the MSIX package.
7. Verify the MSIX installs cleanly on a clean Windows environment (not the dev machine).
8. Archive debug symbols.
9. Distribute.

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

## 12. Distribution Channels

| Channel | Artifact | Audience | Notes |
|---------|----------|----------|-------|
| `<channel>` | `<apk/aab/ipa/msix>` | `<audience>` | `<notes>` |
| `<channel>` | `<artifact>` | `<audience>` | `<notes>` |

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
- OWASP checklist sign-off: `<signed by / date>`

---

## 15. Post-Release Checks

- [ ] Crash and error monitoring reviewed (if applicable; for offline apps: post-install test on
      clean device).
- [ ] Analytics or telemetry sanity checked if applicable.
- [ ] User-reported issues triaged.
- [ ] Release tag created and pushed: `git tag v<version> && git push origin v<version>`.
- [ ] Debug symbols confirmed in secure archive.
- [ ] Follow-up tasks recorded.
