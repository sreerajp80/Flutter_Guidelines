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

### 6.4 Debuggable Verification (Android)

Verify that `android:debuggable` is `false` in the merged release manifest before every
production release. A debuggable production build is a security vulnerability and a Google Play
policy violation.

Check via `aapt2`:

```bash
# bash / zsh
aapt2 dump badging build/app/outputs/apk/prod/release/app-arm64-v8a-prod-release.apk \
  | grep -i debuggable
```

```powershell
# PowerShell (Windows)
aapt2 dump badging build\app\outputs\apk\prod\release\app-arm64-v8a-prod-release.apk `
  | Select-String -Pattern debuggable
```

Expected output: no `application-debuggable` line present (the attribute defaults to false when
absent). If it appears, investigate `buildTypes.release.isDebuggable` in `build.gradle.kts`.

Alternatively, inspect the merged manifest in Android Studio:
Build → Analyze APK → Select APK → AndroidManifest.xml → confirm `debuggable` is absent or false.

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
- [ ] ProGuard rules verified (Android).
- [ ] `android:debuggable=false` confirmed in merged release manifest (Android).
- [ ] Manifest and permission review completed — no unnecessary permissions.
- [ ] OWASP Mobile Top 10 checklist reviewed (see `docs/security.md`).
- [ ] Secrets, keys, and backup settings reviewed if applicable.
- [ ] Sensitive-data flows revalidated if applicable.
- [ ] Data retention and purge behavior verified.

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
6. Build the required Android artifacts with all hardening flags.
7. Run size analysis and record output.
8. Verify `android:debuggable=false` in the merged manifest.
9. Verify artifact naming, installability, and environment on a physical or emulated device.
10. Archive debug symbols from `build/symbols/`.
11. Upload to the intended distribution channel.
12. Tag the release in git: `git tag v<version>` and push.

### Android Build Commands

```bash
flutter pub get
dart run build_runner build --delete-conflicting-outputs
dart format --output=none --set-exit-if-changed .
flutter analyze
flutter test

# Split APKs for direct distribution
flutter build apk \
  --flavor prod \
  --release \
  --obfuscate \
  --split-debug-info=build/symbols/android-prod-$(cat pubspec.yaml | grep '^version:' | cut -d' ' -f2)/ \
  --split-per-abi

# App Bundle for Google Play
flutter build appbundle \
  --flavor prod \
  --release \
  --obfuscate \
  --split-debug-info=build/symbols/android-prod-$(cat pubspec.yaml | grep '^version:' | cut -d' ' -f2)/

# Size analysis
flutter build apk --flavor prod --release \
  --analyze-size
```

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
