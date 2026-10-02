# Platform Store Readiness

This document holds the **release gates** for every platform and distribution channel a Flutter
app can ship to. A project applies **only** the sections for the channels it declares in
`docs/PROJECT_PROFILE.md` (engineering standard §1.2.1).

Each gate MUST pass **before the first upload** to that channel and MUST be re-checked before
every production release. Items marked *(one-time)* are set up once and only re-verified
afterwards.

> **Store rules change often.** Apple, Google, Microsoft, Canonical (Snap Store) and Flathub
> update their requirements every year — target SDK levels, required Xcode versions, screenshot
> sizes, privacy declarations. Numbers below were last checked in **2026-10** and are a starting
> point. Before each release, check the store's current documentation and update this file (or
> the project's local copy) when a rule has changed. Never assume a value already in the project
> is still accepted.

| Section | Channel |
|---|---|
| 1 | Rules for every store |
| 2 | Google Play (Android) |
| 3 | Apple App Store (iOS) |
| 4 | Windows — Microsoft Store and direct download |
| 5 | macOS — Mac App Store and Developer ID direct download |
| 6 | Linux — Snap Store, Flathub, and direct packages |
| 7 | Recording the result |

---

## 1. Rules for every store

These apply to every declared public distribution channel.

- **Identity** *(one-time)*: the reverse-DNS app id from `docs/PROJECT_PROFILE.md` is used
  everywhere the store allows. It can never be changed after the first publish on Google Play,
  the App Store, the Mac App Store, the Snap Store (snap name) or Flathub.
- **Versioning**: `pubspec.yaml` `version: X.Y.Z+N` is the single source. The build number `N`
  strictly increases on every upload to any store and is never reused, even for a rejected build.
- **Privacy policy**: a public, reachable, app-specific URL. Required by Google Play, the App
  Store and the Microsoft Store for most apps; strongly recommended everywhere. Link it from the
  About screen (`guideline.md` §1.6).
- **Data disclosure** matches reality — Play Data safety, Apple App Privacy, Microsoft Store
  privacy declaration, Flathub/Snap permissions. Include what bundled SDKs and plugins collect.
- **Permissions are least-privilege** on every platform (Android permissions, iOS/macOS usage
  strings and entitlements, MSIX capabilities, Snap plugs, Flatpak `finish-args`). Every one is
  used and explained.
- **Account deletion**: if the app lets users create an account, it MUST also let them delete it
  from inside the app (Google Play and the App Store both enforce this).
- **Age / content rating** questionnaire completed for each store.
- **Listing text and screenshots** show the real app, in each listing language, with no
  placeholder text, no other platform's UI (no Android screenshots on the App Store), and no
  misleading claims.
- **Release build tested**: the exact artifact you upload was installed and its primary flow
  completed on a clean device or VM — not the debug build.
- **Obfuscation and symbols**: every release uses `--obfuscate --split-debug-info=...`, and the
  symbols are archived (`release_process.md` §6.1).
- **Support contact**: a public support address or URL is set in every store.

---

## 2. Google Play (Android)

Applies when Google Play is a declared channel for Android. Build commands: `guideline.md` §2.4
and `release_process.md` §9.

### 2.1 Application identity and versioning

| Item | Requirement |
|---|---|
| `applicationId` *(one-time)* | Reverse-DNS, owned domain, lowercase, permanent. It can never be changed after the first publish. Flavors may append a suffix (`.dev`), but the production id MUST have no suffix. |
| `versionCode` | Strictly increasing integer on every upload, never reused — even for a rejected or rolled-back build. |
| `versionName` | Matches `pubspec.yaml` (`<version>+<build>` → `versionName+versionCode`). |
| App name | Set in `android/app/src/main/AndroidManifest.xml` via a localized `@string/app_name`, matching the store listing. |
| Package visibility | If the app queries other packages, declare `<queries>` — Play rejects silent package enumeration. |

### 2.2 API level, ABI, and compatibility

- `targetSdkVersion` MUST meet Play's current target API level policy (Play requires new apps and
  updates to target an API level within one year of the latest major Android release; the deadline
  is typically 31 August each year). Check the current requirement before each release rather than
  trusting the value already in the project.
- `compileSdkVersion` ≥ `targetSdkVersion`.
- `minSdkVersion` is a deliberate, documented product decision — record it in `docs/architecture.md`.
- 64-bit native code is mandatory: ship an App Bundle, or split APKs including `arm64-v8a`.
- 16 KB page-size compliance is required for Android 15+ devices (see the engineering standard).
- Edge-to-edge behavior verified when targeting SDK 35+.

### 2.3 Signing and upload

- Ship an **Android App Bundle (`.aab`)**, not an APK, to Play.
- **Language splitting MUST be disabled** when the app has an in-app language picker (`bundle { language { enableSplit = false } }` in
  `android/app/build.gradle.kts`). Without this, Play downloads only the phone's system language,
  breaking the in-app language picker (engineering standard §8.1).
- **Play App Signing** MUST be enabled *(one-time)*. Keep the upload key backed up offline; losing
  the upload key is recoverable through Play support, losing a pre-App-Signing release key is not.
- Signing config points at `android/key.properties` (see `docs/guideline.md` §2) and is **never**
  committed.
- `flutter build appbundle --release --obfuscate --split-debug-info=...` — all three flags, always.
- Upload the native debug symbols (`build/symbols/`) to Play so crash traces de-obfuscate, and
  archive them alongside the release evidence.

### 2.4 Manifest, permissions, and policy declarations

- Every permission in the merged manifest is justified and used. Remove anything inherited from a
  dependency that the app does not need (`tools:node="remove"`).
- Sensitive permissions require an in-console declaration and are commonly rejected: all-files
  access, exact alarms, accessibility service, SMS/call log, background location, camera/microphone
  in the background, `QUERY_ALL_PACKAGES`.
- Foreground services declare a `foregroundServiceType` and a use-case declaration in the console.
- `android:debuggable=false`, `android:allowBackup` decided deliberately, `usesCleartextTraffic=false`
  (`release_process.md` §6.4, §6.5).
- No accidental `android:exported="true"` (`release_process.md` §6.7).
- Ads, payments, and analytics SDKs are declared where the console asks for them.

### 2.5 Store account declarations

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

### 2.6 Store listing assets

| Asset | Requirement |
|---|---|
| App icon | 512 × 512 PNG, 32-bit, no alpha-dependent design |
| Feature graphic | 1024 × 500 PNG/JPG |
| Phone screenshots | 2–8, PNG/JPG, 16:9 or 9:16, min 320 px, max 3840 px on the longest side |
| Tablet screenshots | Required if the app is distributed to tablets (7-inch and 10-inch sets) |
| Short description | ≤ 80 characters |
| Full description | ≤ 4000 characters |
| App title | ≤ 30 characters, no keyword stuffing, no store badges or price in the title |

### 2.7 Localization of the listing

- Provide the Play listing in **every declared language that Play supports as a listing
  language**, including localized screenshots. Check each language against Play Console's list;
  if a declared language is not a Play listing language, ship it inside the app only and note
  this in `docs/PROJECT_PROFILE.md` (language packs record known cases).
- Screenshots MUST show real app UI in the language of that listing — not template-language
  screenshots under another language's listing.

### 2.8 Pre-launch verification

- Upload to **internal testing** first; run the Play Console **pre-launch report** and resolve all
  crashes, ANRs, and flagged accessibility and security items.
- Verify the app installs, launches, and completes its primary flow from a Play-served build (not
  just a locally installed APK), in every declared language.
- Android vitals thresholds reviewed after each rollout (crash rate, ANR rate).
- Production rollout starts as a **staged rollout** (e.g. 10% → 50% → 100%) with vitals checked at
  each step.


---

## 3. Apple App Store (iOS)

### 3.1 Account and identity

| Item | Requirement |
|---|---|
| Apple Developer Program *(one-time)* | Paid membership for the publisher in `docs/PROJECT_PROFILE.md` (individual or organization). |
| Bundle id *(one-time)* | Explicit App ID registered in Certificates, Identifiers & Profiles; equals `PRODUCT_BUNDLE_IDENTIFIER` for the prod flavor. Permanent after first publish. |
| App Store Connect record *(one-time)* | App name (≤ 30 characters, unique on the store), primary language, bundle id, SKU. |
| Version | `CFBundleShortVersionString` = pubspec `X.Y.Z`; `CFBundleVersion` = build number `N`, higher than every previous upload of that version. Flutter sets both from `pubspec.yaml`. |
| Capabilities | Only those the app uses (Push, iCloud, Sign in with Apple, …), enabled on the App ID **and** in Xcode Signing & Capabilities. |

### 3.2 Build and toolchain

- Build with the Xcode / iOS SDK version App Store Connect currently requires (check before each
  release; Apple raises it every spring).
- Deployment target and UIScene migration as in engineering standard §5.4.
- Signing: Apple Distribution certificate + App Store provisioning profile for the prod bundle id
  (automatic signing in Xcode is fine). Certificates and profiles never enter git.
- Build: `flutter build ipa --release --obfuscate --split-debug-info=build/symbols/ios-prod-<version>/`
  (add `--flavor prod` when flavors are used). The `.ipa` is in `build/ios/ipa/`.
- Upload with Xcode Organizer or Apple's Transporter app. Wait for processing to finish before
  submitting.
- **Device family**: if `TARGETED_DEVICE_FAMILY` includes iPad, every screen MUST work on iPad in
  all orientations the app allows, and iPad screenshots are required. If the app is not designed
  for iPad, set the target to iPhone only.

### 3.3 `Info.plist` and privacy

- **Usage-description strings** (`NSCameraUsageDescription`, `NSPhotoLibraryUsageDescription`,
  `NSMicrophoneUsageDescription`, `NSLocationWhenInUseUsageDescription`, …) for every permission the
  app **or any plugin** can request. A missing string crashes the app or gets it rejected. Explain
  the real reason in plain words, and localize them with `InfoPlist.strings` for each declared
  language.
- **Privacy manifest** — `ios/Runner/PrivacyInfo.xcprivacy` declares the data the app collects and
  the "required reason" APIs it uses (for example `UserDefaults`, used by `shared_preferences`, and
  file timestamps). Plugins and SDKs MUST ship their own manifests; use versions that do.
- **App Privacy** answers in App Store Connect match the privacy manifest and the real behavior.
- **Export compliance**: set `ITSAppUsesNonExemptEncryption` in `Info.plist` (`false` when the app
  uses only exempt encryption such as standard HTTPS and OS-provided crypto). If it uses its own
  non-exempt crypto, complete the export compliance documentation.
- **Tracking**: if the app tracks users across other companies' apps or sites, it MUST use App
  Tracking Transparency and declare it.
- **Sign-in**: if the app offers third-party or social login, check the current App Review rule on
  offering an equivalent privacy-focused login option (such as Sign in with Apple).
- `CFBundleLocalizations` lists the declared languages (engineering standard §8.1).

### 3.4 Listing assets

| Asset | Requirement (check current sizes in App Store Connect help) |
|---|---|
| App icon | 1024 × 1024 PNG in the asset catalog, no transparency, no rounded corners added by you |
| iPhone screenshots | Required for the largest current iPhone display class (6.9-inch); 1–10 per language |
| iPad screenshots | Required when the app runs on iPad (13-inch class) |
| Subtitle | ≤ 30 characters |
| Promotional text | ≤ 170 characters (can change without a new build) |
| Description | ≤ 4000 characters |
| Keywords | ≤ 100 characters, comma-separated, no competitor names |
| Support URL | Required |
| Privacy policy URL | Required |
| Age rating | Questionnaire completed |
| Review notes | How to reach every feature; a demo account if login is required |

### 3.5 Localization of the listing

Provide the listing (name, subtitle, description, keywords, screenshots) in every declared
language that App Store Connect supports. A declared language the store cannot list ships inside
the app only.

### 3.6 Pre-launch verification

- Upload to **TestFlight**; test internally first, then with external testers (external builds
  need Beta App Review).
- Verify the TestFlight build installs, launches and completes its primary flow on a real device,
  in every declared language, on the oldest supported iOS version and the newest.
- Submit for review with complete review notes. Use **phased release** for updates when the change
  is risky.

---

## 4. Windows — Microsoft Store and direct download

### 4.1 Common to both

- App metadata in `windows/runner/Runner.rc` and real app icon (engineering standard §5.5.1, §17.5).
- MSIX built with `dart run msix:create` from `msix_config` in `pubspec.yaml`. `msix_version` is
  four parts and increases on every release.
- **Capabilities**: the `msix` package declares `runFullTrust`, which a Flutter desktop app needs.
  Add only the other capabilities the app uses (`internetClient`, `webcam`, `microphone`,
  `location`, …).
- Install, launch, upgrade from the previous version (user data kept) and uninstall cleanly on a
  **clean Windows VM**, not the developer machine. Test on x64 and, if shipped, arm64.
- Run the **Windows App Certification Kit (WACK)** against the package and fix every failure.

### 4.2 Microsoft Store

| Item | Requirement |
|---|---|
| Partner Center account *(one-time)* | Developer account for the publisher. |
| Reserved app name *(one-time)* | Reserve the name in Partner Center. |
| Product identity *(one-time)* | Copy **Package/Identity/Name**, **Package/Identity/Publisher** and **Package/Properties/PublisherDisplayName** exactly into `identity_name`, `publisher` and `publisher_display_name`. |
| Signing | Set `store: true`. The Store signs the package — do not sign it yourself. |
| Version | Store packages need the last (revision) part of `msix_version` to be `0`, e.g. `1.2.3.0`. |
| Architectures | x64 at minimum; add arm64 for Windows on Arm devices (`.msixbundle` or one package per architecture). |
| Privacy policy URL | Required when the app accesses the internet or personal data — which is almost every app. |
| Age rating | IARC questionnaire completed. |
| Store listing | Description, at least one screenshot per listing language (1366 × 768 or larger recommended), app icon/logos, category, support contact. One listing per declared language the Store supports. |
| Submission notes | How to test the app; test account if login is required. |

### 4.3 Direct download (outside the Store)

- The MSIX, or an installer (`.exe` from Inno Setup, WiX, or similar) wrapping the release folder,
  MUST be **code-signed** with a certificate trusted on users' machines (an OV/EV code-signing
  certificate or a managed signing service), and **timestamped** so it stays valid after the
  certificate expires.
- An unsigned or self-signed package cannot be installed as MSIX on normal machines and an unsigned
  `.exe` triggers SmartScreen warnings.
- Publish SHA-256 checksums next to the download, served over HTTPS.
- Decide how users get updates (Store, App Installer `.appinstaller` file, or an in-app update
  check) and record it in `release_process.md`.

---

## 5. macOS — Mac App Store and Developer ID direct download

### 5.1 Common to both

| Item | Requirement |
|---|---|
| Bundle id *(one-time)* | `PRODUCT_BUNDLE_IDENTIFIER` in `macos/Runner/Configs/AppInfo.xcconfig`; SHOULD match iOS when both ship. |
| Version | Flutter sets `CFBundleShortVersionString` / `CFBundleVersion` from `pubspec.yaml`. |
| Category | `LSApplicationCategoryType` set in `macos/Runner/Info.plist` (required by the Mac App Store). |
| Minimum macOS | Podfile + Xcode deployment target ≥ the Flutter minimum (engineering standard §5.4). |
| Architectures | Ship a universal binary (arm64 + x86_64); check with `lipo -archs`. Dropping Intel is a documented decision. |
| Entitlements | Least privilege, set in **both** `DebugProfile.entitlements` and `Release.entitlements`; network client entitlement present if the app uses the network (engineering standard §5.5.2). |
| Usage strings | `NSCameraUsageDescription`, `NSMicrophoneUsageDescription`, etc. for every protected resource the app or a plugin uses. |
| Privacy manifest | `PrivacyInfo.xcprivacy` included for App Store distribution, as for iOS (§3.3). |
| Icon | Full macOS icon set generated (engineering standard §17.5). |
| Menus and window | App menu shows the real app name; About/Quit/Hide work; window has a sensible minimum size. |

### 5.2 Mac App Store

- App Store Connect record with the macOS platform added (the same record as iOS can be used when
  the bundle id is shared).
- **App Sandbox is mandatory** (`com.apple.security.app-sandbox` = `true` in Release).
- Certificates: **Apple Distribution** (app) and **Mac Installer Distribution** (package), plus a
  Mac App Store provisioning profile. Never in git.
- Build: `flutter build macos --release --obfuscate --split-debug-info=build/symbols/macos-prod-<version>/`
  (plus `--dart-define=APP_FLAVOR=prod` when flavors are used), then open
  `macos/Runner.xcworkspace` in Xcode → Product → Archive → Distribute App → App Store Connect.
- Screenshots: 16:10, at one of the accepted sizes (e.g. 1280 × 800, 1440 × 900, 2560 × 1600,
  2880 × 1800); 1–10 per language.
- App Privacy, export compliance (`ITSAppUsesNonExemptEncryption`), age rating, review notes, and
  listing localization as for iOS (§3.3–§3.5).
- Test through **TestFlight for macOS** before submission.

### 5.3 Developer ID direct download (outside the Mac App Store)

Without notarization, Gatekeeper blocks the app on users' Macs. Every direct-download release MUST
be signed, notarized and stapled.

1. Certificate: **Developer ID Application** (and Developer ID Installer if shipping a `.pkg`).
2. Enable **Hardened Runtime**; keep App Sandbox on unless a documented need prevents it.
3. Build the release (`flutter build macos --release ...` as above), then archive in Xcode →
   Distribute App → **Developer ID** → Upload (Xcode notarizes for you), **or** from the command
   line:

   ```bash
   # Sign (Xcode export usually does this) — hardened runtime + secure timestamp.
   codesign --force --deep --options runtime --timestamp \
     --sign "Developer ID Application: <Team Name> (<TEAMID>)" "build/macos/Build/Products/Release/<App>.app"

   # Notarize. Credentials are stored once in the keychain with
   # `xcrun notarytool store-credentials <profile>` — never in scripts or the repo.
   ditto -c -k --keepParent "build/macos/Build/Products/Release/<App>.app" "<App>.zip"
   xcrun notarytool submit "<App>.zip" --keychain-profile "<profile>" --wait

   # Staple the ticket so the app opens offline.
   xcrun stapler staple "build/macos/Build/Products/Release/<App>.app"
   ```

4. Package as a **DMG** (e.g. `hdiutil` or `create-dmg`), sign the DMG, notarize it, and staple it.
5. Verify on a **clean Mac** (or a fresh user account) that has never run the app:

   ```bash
   spctl --assess --type execute -vv "<App>.app"
   spctl --assess --type open --context context:primary-signature -vv "<App>.dmg"
   ```

6. Decide how users get updates (e.g. an in-app updater such as Sparkle, or a manual "new version"
   notice) and record it in `release_process.md`.

---

## 6. Linux — Snap Store, Flathub, and direct packages

### 6.1 Common to all channels

| Item | Requirement |
|---|---|
| App id *(one-time)* | `APPLICATION_ID` in `linux/CMakeLists.txt` is the reverse-DNS id (engineering standard §5.5.3). |
| Build machine | Release binaries built on the oldest supported distribution (or its container) so `glibc` matches users' systems. |
| Desktop file | `<APPLICATION_ID>.desktop` with `Name`, `Comment`, `Exec`, `Icon=<APPLICATION_ID>`, `Categories`, `Terminal=false`. Validate with `desktop-file-validate`. |
| Icons | `<APPLICATION_ID>.png` at 256 × 256 (and 512 × 512) plus SVG if available, installed under `share/icons/hicolor/`. |
| AppStream metadata | `<APPLICATION_ID>.metainfo.xml` with id, name, summary, description, `launchable`, `project_license`, `metadata_license`, screenshots, `releases`, and OARS `content_rating`. Validate with `appstreamcli validate`. Required by Flathub; used by Snap and software centers. |
| Runtime dependencies | GTK 3 and anything plugins need (e.g. `libsecret` for secure storage) are declared in the package. |
| Testing | Install, launch, and run the primary flow on clean VMs of the main target distros (e.g. Ubuntu LTS and Fedora), under both **Wayland** and **X11**. |

### 6.2 Snap Store

- Snapcraft developer account; register the snap name *(one-time)*: `snapcraft register <name>`.
- `snap/snapcraft.yaml` at the repo root. Example shape — check the current snapcraft Flutter
  documentation for the supported `base` and plugin options:

  ```yaml
  name: my-app
  title: My App
  version: '1.0.0'
  summary: One line, 78 characters or fewer
  description: |
    What the app does.
  base: core24
  grade: stable
  confinement: strict
  apps:
    my-app:
      command: my_app
      extensions: [gnome]
      plugs: [network]          # least privilege — add only what the app uses
  parts:
    my-app:
      source: .
      plugin: flutter
      flutter-target: lib/main.dart
  ```

- Build with `snapcraft`, test with `sudo snap install --dangerous ./<name>_<version>_<arch>.snap`,
  then upload with `snapcraft upload --release=edge` and promote to `beta` / `stable` after testing.
- `confinement: strict` is required for normal store listing. Interfaces that need manual store
  review (for example `personal-files`, `system-files`) SHOULD be avoided.
- Store listing: icon, screenshots, license, website/contact, and privacy policy if the app
  collects data.

### 6.3 Flathub

- App id MUST be a reverse-DNS id under a domain or code-hosting namespace the publisher controls
  (Flathub verifies it).
- Submit a Flatpak manifest (`<APPLICATION_ID>.yml`) by pull request to the Flathub submissions
  repository. Use a current `org.freedesktop.Platform` or `org.gnome.Platform` runtime.
- Flathub builds in its own offline infrastructure. Check the current Flathub requirements for
  whether the manifest must build Flutter from source (vendoring the Flutter SDK and pub cache) or
  may package the release bundle your CI produced; follow whichever they currently accept.
- `finish-args` least privilege: typically `--share=ipc`, `--socket=wayland`,
  `--socket=fallback-x11`, `--device=dri`, plus `--share=network` only if the app uses the
  network. Avoid broad `--filesystem=home`; use portals (file chooser) instead.
- The AppStream metainfo (§6.1) MUST pass Flathub's linter, with screenshots and release notes.

### 6.4 Direct packages (AppImage, `.deb`, `.rpm`)

- Build from the release bundle (`build/linux/x64/release/bundle/`) with a packaging tool such as
  `appimagetool`, `dpkg-deb`, or a Flutter packaging helper.
- Declare runtime dependencies in `.deb` / `.rpm` metadata; bundle them in an AppImage.
- Publish over HTTPS with SHA-256 checksums, and SHOULD sign the checksum file with GPG.
- Decide how users get updates and record it in `release_process.md`.

---

## 7. Recording the result

For each release and each declared channel, record in the release evidence
(`release_process.md` §14): the gate checklist result, the build number uploaded, the store
submission or rollout record, and any rule that changed since the last release (update this file
when it did).
