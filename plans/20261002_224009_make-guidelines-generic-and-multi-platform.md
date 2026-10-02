# Make the guidelines generic and ready for all store platforms

**Status:** completed

## 1. Goal

1. Anyone should be able to drop this guideline set into a Flutter project, tell an AI agent
   "follow these guidelines", and get a correct app — not an app with one person's name,
   languages and branding baked in.
2. Following the guidelines should give an app that can be **published**:
   - Android → Google Play (already well covered),
   - iOS → App Store (partly covered),
   - Windows → Microsoft Store and direct download (partly covered),
   - macOS → Mac App Store and direct download (Developer ID + notarization) (**missing**),
   - Linux → Snap Store / Flathub / direct package (**missing**).

## 2. What the analysis found

### 2.1 Personal / project-specific content (blocks "generic")

| Where | Problem |
|---|---|
| `guideline.md` §1.2 | Sample JSON has the owner's real name, real email, a personal app name, and fixed AI / IDE names. |
| `guideline.md` §1.7, standard §22.2, §23.1, release checklist, CLAUDE/AGENTS templates | "Made with ❤️ from India" badge is **mandatory in every app** and "MUST NOT be removed". |
| standard §8 (≈ 950 lines), `guideline.md` §3–§4, CLAUDE/AGENTS templates, release §8 / §9A.7 | **English + Malayalam + Sanskrit are mandatory for every app.** No other language set is allowed. Sanskrit/Malayalam glossary, Hindi-marker CI gate, Malayalam Play listing are all hard rules. |
| `README.md`, `guideline.md` intro | Written as "my apps", "my personal conventions". |
| `CLAUDE_MD_GUIDELINE.md` line 7 + §6 | Names nine private app repos ("learned from the nine files"). |
| CLAUDE/AGENTS templates §"Workflow rules" | Say "from global rules" — refers to the owner's private global `CLAUDE.md`, which other users do not have. |
| `guideline.md` §1.2 `aiUsed` / `ideUsed` | Required display fields that only make sense for the owner. |

### 2.2 Platform gaps (blocks "deploy to all stores")

| Platform | What exists | What is missing |
|---|---|---|
| Android | Strong: signing, R8, 16 KB pages, Play readiness gate (§9A). | Only small items: app icon / adaptive icon generation, a generic (not Malayalam) listing-language rule. |
| iOS | Flavors, UIScene, deployment target, `flutter build ipa`. | No **App Store readiness gate**: bundle id, App Store Connect record, privacy manifest (`PrivacyInfo.xcprivacy`), App Privacy labels, export compliance (`ITSAppUsesNonExemptEncryption`), usage-description strings, screenshots sizes, TestFlight, review notes, account deletion. |
| Windows | Flavors via `APP_FLAVOR`, MSIX config, release steps. | No **Microsoft Store gate** (Partner Center identity names, reserved name, age rating, store listing), no code-signing rule for direct-download MSIX/EXE (trusted certificate), no installer alternative (e.g. Inno Setup) note. |
| macOS | Only "deployment target macOS 12" and one `Platform.isMacOS` line. | **Everything**: bundle id, signing, entitlements + App Sandbox (`DebugProfile.entitlements` / `Release.entitlements`, network client entitlement), hardened runtime, Developer ID + **notarization** (`notarytool`, `stapler`) for direct download, DMG packaging, Mac App Store submission, desktop flavors on macOS, release steps, security controls. |
| Linux | sqflite FFI init only. | **Everything**: build deps (`clang`, `cmake`, `ninja`, `gtk3`), `.desktop` file + icons, `APPLICATION_ID`, packaging (Snap / Flatpak / AppImage / .deb), store submission (Snap Store, Flathub), release steps, security controls. |
| All | — | No single "**target platforms**" declaration per project, so an AI does not know which platform folders, gates and CI jobs to set up. No cross-platform CI matrix (Linux runner for Android/Linux, macOS runner for iOS/macOS, Windows runner for Windows). No app-icon / splash generation rule. |

### 2.3 Missing "AI entry point"

There is no single short, ordered document that tells an AI agent: *"Read these files in this
order, fill these templates, run `flutter create --platforms=...`, then pass these gates."* The
AI has to discover the order from a 3,300-line standard.

## 3. Approach

**Keep the owner's conventions, but move them out of the generic core into an optional
"project profile".** Nothing the owner uses today is lost; it just stops being forced on
everyone.

- Generic documents talk about *"the project's declared languages"*, *"the project's declared
  target platforms"*, *"the optional About badge"* — never a fixed person, country or language.
- A new fill-in **project profile** file holds the per-owner / per-project choices (author,
  languages, badge text, target platforms, stores, package id prefix).
- The owner's current choices (English/Malayalam/Sanskrit, the India badge, author details)
  are saved as a ready-made **example profile** plus a separate **language pack** that keeps
  all the Sanskrit/Malayalam quality rules, glossary and CI gate unchanged.

## 4. Files to change

### New files

| File | Purpose |
|---|---|
| `AI_AGENT_START_HERE.md` | Short ordered entry point for any AI agent: read order, how to pick profiles and platforms, bootstrap steps (`flutter create --org <reverse-dns> --platforms=<list>`), then the per-platform store gates. |
| `PROJECT_PROFILE_TEMPLATE.md` | Fill-in template: app name, org / package id, author, contact, languages (template locale + others), About badge (on/off + text), target platforms, distribution channels per platform, applicability profiles. Copied to the app as `docs/PROJECT_PROFILE.md`. |
| `profiles/example_en_ml_sa_profile.md` | The owner's current choices as a worked example (languages en/ml/sa, "Made with ❤️ from India" badge, About fields). No personal email. |
| `language_packs/sanskrit_malayalam.md` | Moved, unchanged in meaning: standard §8.3.1 Sanskrit delegate, §8.5 quality rules, glossary, Hindi-marker gate, Malayalam rules. Applies only when a project declares `ml` / `sa`. |
| `platform_store_readiness.md` | One place for every store gate: Google Play (moved from release §9A), **App Store (new)**, **Microsoft Store (new)**, **Mac App Store + Developer ID notarization (new)**, **Snap Store / Flathub / direct Linux packages (new)**. |
| `docs/platform_store_readiness_README.md` | Plain-English explainer for the new file. |

### Changed files

| File | Change |
|---|---|
| `README.md` | Remove "my apps"; describe as a reusable set; add new files to the table; add "AI agents start at `AI_AGENT_START_HERE.md`". |
| `GUIDELINES_MANIFEST.md` | Same updates; list the new files; generic wording. |
| `guideline.md` | Rename idea to "Common app conventions". §1.2 sample JSON uses placeholders (`<Your App>`, `<Author>`, `<contact>`), languages come from the project profile. `aiUsed` / `ideUsed` become optional examples. §1.7 badge becomes **optional**, text set in the profile (India badge kept as an example). §2 keystore rules unchanged. §2.4 build examples kept. §3/§4 checklist uses "declared languages" and "declared platforms", and points to `platform_store_readiness.md` for every declared platform, not only Play. |
| `flutter_project_engineering_standard.md` | §1.2 add "declare target platforms". §3.3 root layout lists `macos/`, `linux/` with when-required notes and adds `AGENTS.md`. §5.5 becomes **Desktop Build Setup (Windows, macOS, Linux)** with macOS entitlements/sandbox and Linux build deps + `.desktop`/application id. §5.6 artifact table for all 5 platforms. §8 rewritten generic: English (or chosen template) + declared languages, parity test, in-app picker, label length — Sanskrit/Malayalam specifics moved to the language pack (left as a short pointer). New §17.x app icons + splash for every platform (`flutter_launcher_icons`, adaptive icon, macOS/Windows/Linux icons). §19.2 CI as a per-platform matrix with correct runners. §21 adds `docs/PROJECT_PROFILE.md`. §22 / §23 replace en/ml/sa and India-badge rules with "declared languages" and "badge if enabled in profile"; add "the store gate of each declared platform still holds". |
| `release_process.md` | §1 platforms list adds macOS and Linux. §5 flavor note covers all desktops. §8 localization checklist made generic; store section points to the gate file for each declared platform. §9A moved to `platform_store_readiness.md` (leave a pointer). §10 iOS steps expanded (archive, upload to App Store Connect with Xcode Organizer or Transporter, TestFlight, submit for review). **New §11A macOS release steps** (build, sign, notarize, staple, DMG or Mac App Store upload). **New §11B Linux release steps** (build on oldest supported distro / in container, package as Snap / Flatpak / AppImage / deb, test on clean VM). §11 Windows adds code signing. §12 distribution table pre-filled with one row per store. |
| `flutter_build_flavors_guide.md` | "Windows Desktop Flavor Setup" becomes "Desktop Flavor Setup" with macOS (Xcode schemes optional, `APP_FLAVOR` define, bundle id per flavor) and Linux (`APPLICATION_ID` per flavor in `linux/CMakeLists.txt`) subsections; release matrix covers all 5 platforms. |
| `security.md` | §10 adds **macOS** (App Sandbox, hardened runtime, Keychain, entitlements least-privilege) and **Linux** (libsecret for secure storage, file permissions, Flatpak/Snap confinement). |
| `architecture.md` | §15 environment / build model lists all target platforms; add a "target platforms and distribution" table. |
| `CLAUDE_MD_GUIDELINE.md`, `AGENTS_MD_GUIDELINE.md` | Remove names of private repos and "nine files". "From global rules" → "from this guideline set". Localization section uses the declared languages from the profile; Sanskrit/Malayalam lines move to an "if the language pack applies" note. About-badge rule becomes conditional. Identity table gets "Target platforms" and "Stores". Template build commands include macOS/Linux examples. Self-check updated. |
| `DOCS_FOLDER_GUIDELINE.md` | Add `PROJECT_PROFILE.md` to mandatory baseline docs; add new files to the catalog. |
| `docs/*_README.md` explainers | Update to match each changed document (generic wording, new platforms). |

## 5. What stays the same

- Keystore + `key.properties` rules, obfuscation and debug-symbol rules.
- The plan → approve → change-log workflow and the privacy rule for `plans/` / `change_log/`
  (kept as part of this guideline set, no longer described as "the owner's global rules").
- All Sanskrit/Malayalam rules — moved, not deleted.
- "Latest stable Flutter, do not pin versions from memory" policy from the two earlier plans
  today.

## 6. Order of work

1. Create `PROJECT_PROFILE_TEMPLATE.md`, example profile, language pack (move text).
2. Generalise `guideline.md` and engineering standard §8, §22, §23.
3. Write `platform_store_readiness.md` (move Play gate, add iOS / Windows / macOS / Linux gates).
4. Update `release_process.md`, flavors guide, `security.md`, `architecture.md`.
5. Update `CLAUDE_MD_GUIDELINE.md`, `AGENTS_MD_GUIDELINE.md`, `DOCS_FOLDER_GUIDELINE.md`.
6. Write `AI_AGENT_START_HERE.md`; update `README.md`, manifest, explainers.
7. Consistency pass: grep for `Malayalam`, `Sanskrit`, `India`, `ml`, `sa`, personal names,
   `Windows` lists missing macOS/Linux, and broken `§` cross-references.
8. Write the change log.

## 7. Risks

- **Large change (~10 files, several thousand lines touched).** Section numbers in the standard
  will shift; every `§` cross-reference must be re-checked in step 7.
- Store rules change often (Apple / Google / Microsoft / Snap / Flathub). New gate text will say
  "check the current store rule before each release" rather than hard-coding numbers that go
  stale, and will give a "last checked" date like the existing toolchain snapshot.
- Existing apps that point at old section numbers (e.g. `§8.5`) will need their `CLAUDE.md`
  links refreshed. The language pack keeps the same sub-headings to make that easy.
