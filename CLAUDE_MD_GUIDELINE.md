# Guideline: How to Write a `CLAUDE.md` for a Flutter Project

Every Flutter project **MUST** have a `CLAUDE.md` file at its root, alongside a mandatory `AGENTS.md` file (see [AGENTS_MD_GUIDELINE.md](AGENTS_MD_GUIDELINE.md) for non-Claude LLMs). This guideline defines the mandatory standards and structure for creating and maintaining `CLAUDE.md` across all Flutter projects (both new and migrated existing apps).

`CLAUDE.md` is read by Claude Code (and AI coding assistants) automatically at the start of every session in that project. It is the first document the AI inspects, so it MUST be clear, concise where possible, and complete where required. Every Flutter repository MUST maintain both `CLAUDE.md` and `AGENTS.md` at project root as mandatory Core Baseline requirements (**MUST**).

This guideline was built by studying real `CLAUDE.md` files from Flutter apps of different sizes — from small single-screen tools to shipped, security-sensitive apps.

---

## 1. First choose a profile: Thin or Thick

Pick one of two styles before you write anything.

### Thin pointer profile
Use this when the project already has (or will have) a full `docs/` folder — `architecture.md`,
`security.md`, `release_process.md`, and so on.

- `CLAUDE.md` stays short. It gives identity, commands, and a short rule summary.
- It **points** to the `docs/` files for the detail. It does not repeat them.
- Rule: if a detail lives in a `docs/` file, do not copy it into `CLAUDE.md` — link to it.
- Typical fit: medium or large apps, shipped apps, security-sensitive apps.

### Thick self-contained profile
Use this when the project is small or has **no** `docs/` folder, so `CLAUDE.md` must hold
everything itself.

- `CLAUDE.md` inlines the detail: schema tables, provider/route tables, full dos-and-don'ts.
- Typical fit: small utilities, counters, single-purpose tools.

### How to choose
- Has a `docs/` set already, or the project is medium/large → **Thin**.
- No `docs/`, small project, one developer → **Thick**.
- When unsure, start **Thin** and grow. It is easier to add detail than to trim a wall of text.

---

## 2. Canonical section order

Write sections in this order. Skip the ones that do not apply (see the checklist in §3).

1. Title + read-first banner
2. Project identity / tech stack
3. Doc references (Thin profile) — the "read these before working" table
4. Hard / non-negotiable rules
5. Architecture rules (layers, boundaries, state, navigation, database)
6. Build & run commands
7. Build flavors (dev / prod)
8. Signing / keystore
9. Security rules
10. Code style / naming conventions
11. Testing rules
12. Dependency constraints
13. Where things live (project tree)
14. Workflow rules (plan → approve → log) — from this guideline set
15. Communication rules (simple English) — from this guideline set
16. Dos & Don'ts ("What Claude must always / never do")

---

## 3. Section checklist (required vs optional)

| # | Section | Required? | Thin | Thick |
|---|---------|-----------|------|-------|
| 1 | Title + read-first banner | **Always** | short line | short line |
| 2 | Project identity / tech stack | **Always** | short list/table | full table |
| 3 | Doc references table | Thin only | **yes** | n/a (no docs) |
| 4 | Hard / non-negotiable rules | If any exist | summarize + link | inline full |
| 5 | Architecture rules | **Always** | 1 paragraph + link | inline full |
| 6 | Build & run commands | **Always** | inline | inline |
| 7 | Build flavors | If flavors used | inline or link | inline |
| 8 | Signing / keystore | If it ships releases | link | inline |
| 9 | Security rules | **Always** | summarize + link | inline |
| 10 | Code style / naming | **Always** | short | full table |
| 11 | Testing rules | **Always** | short + link | full |
| 12 | Dependency constraints | If constrained | link | inline allow/block lists |
| 13 | Where things live (tree) | Recommended | short | full |
| 14 | Workflow rules (plan/log) | **Always** | inline (see §5) | inline (see §5) |
| 15 | Communication rules | **Always** | inline (see §5) | inline (see §5) |
| 16 | Dos & Don'ts | Recommended | optional | **yes** |

Every Flutter app **MUST** have a root `CLAUDE.md` file. All "Always" sections in the table above must appear in every project's `CLAUDE.md`, regardless of profile.

---

## 4. Fill-in-the-blanks template

Copy this into a new project's `CLAUDE.md` and replace every `<...>` placeholder. Delete any
section that the checklist marks optional and the project does not need. Notes in `> ` blocks are
guidance — remove them from the final file.

````markdown
# CLAUDE.md — <App Name>

This file is read by Claude Code at the start of every session in this repository.
Read it before making any change. <If Thin: See the docs table below for full detail.>

---

## Project identity

| Field | Value |
|-------|-------|
| App name | <App Name> |
| Type | <one-line description of what the app does> |
| Platforms | <declared platforms from `docs/PROJECT_PROFILE.md` — e.g. Android (minSdk <NN>, targetSdk <NN>), iOS <NN>, Windows, macOS <NN>, Linux> |
| Stores / channels | <e.g. Google Play, App Store, Microsoft Store, Mac App Store / Developer ID, Snap Store / Flathub> |
| Languages | <en (template), ...> |
| Package / org id | <com.example.app> |
| Flutter SDK | <project's pinned version — from `flutter --version`> |
| Dart SDK | <project's pinned version — from `flutter --version`> |
| State management | <Riverpod / Provider + ChangeNotifier / ...> |
| Navigation | <go_router / named routes> |
| Database | <sqflite / sqflite_sqlcipher / shared_preferences / none> |
| Orientation | <portrait only / both> |
| Connectivity | <fully offline — no INTERNET / online optional / online> |

> Keep this table honest and current. It is the fastest way for the AI to orient.

---

## Read these docs before working   <!-- Thin profile only -->

| Document | Read when |
|----------|-----------|
| docs/architecture.md | Changing structure, screens, state, services, models, repositories |
| docs/security.md | Touching permissions, logging, storage, crypto, manifest |
| docs/release_process.md | Building a release, versioning, release checklist |
| docs/flutter_build_flavors_guide.md | Build config, signing, flavors, Gradle, ProGuard |
| docs/flutter_project_engineering_standard.md | Any code change — layers, naming, testing |
| docs/PROJECT_PROFILE.md | Always — platforms, stores, languages, identity, About options |
| docs/platform_store_readiness.md | Before any release; when touching permissions, entitlements, store-listed behavior |
| docs/GUIDELINES_MANIFEST.md | The shared Flutter guidelines index |

> If a doc is copied into this project's own `docs/`, the local copy wins over the master.

---

## Hard rules (must follow — these override convenience)

1. <e.g. Open source only. No commercial/source-available SDKs. Check a package license first.>
2. <e.g. Offline-first. The app works fully offline; online parts are optional.>
3. <e.g. Scoped storage only — system file picker / share intents; no broad storage permission.>
4. <e.g. Never crash on bad input. Every parser has a failure path with a friendly message.>
5. <e.g. Atomic / copy-on-write saves. Never modify the original file in place.>

> Only list rules that are truly non-negotiable for this app. Drop this section if there are none.

---

## Architecture rules

- Layout: <Tier 1 layer-first under `lib/` — config/ models/ providers/ repositories/
  screens/ services/ widgets/ main.dart>. Do not restructure without instruction.
- Layer boundaries: widgets must not know <SQL / SharedPreferences keys / file paths / intents>.
  Services must not know <BuildContext / navigation routes / UI strings>.
- Dependency direction: <screens → providers → repositories/services → database → models>.
- Models are immutable (<freezed / const + copyWith>). Never mutate in place.
- <No direct DB access from widgets — go through the repository/provider layer.>

---

## Build & run commands

```bash
flutter pub get                        # install dependencies
flutter run --flavor dev               # daily development
flutter run --flavor prod              # production build with debug tooling
flutter analyze                        # static analysis (must be clean)
flutter test                           # run all tests
dart format .                          # format before committing

# Production release APK (split per ABI)
flutter build apk --flavor prod --release \
  --obfuscate --split-debug-info=build/symbols/android-prod-<version>/ --split-per-abi

# Production Play Store bundle
flutter build appbundle --flavor prod --release \
  --obfuscate --split-debug-info=build/symbols/android-prod-<version>/

# Other declared platforms — keep only the ones this app ships
flutter build ipa --flavor prod --release \
  --obfuscate --split-debug-info=build/symbols/ios-prod-<version>/
flutter build windows --release --dart-define=APP_FLAVOR=prod \
  --obfuscate --split-debug-info=build/symbols/windows-prod-<version>/
flutter build macos --release --dart-define=APP_FLAVOR=prod \
  --obfuscate --split-debug-info=build/symbols/macos-prod-<version>/
flutter build linux --release --dart-define=APP_FLAVOR=prod \
  --obfuscate --split-debug-info=build/symbols/linux-prod-<version>/
```

> If the app defines flavors, a bare `flutter run` fails — always pass `--flavor`.

---

## Build flavors   <!-- if flavors are used -->

| Flavor | App ID | Display name | Signing |
|--------|--------|--------------|---------|
| dev | <id>.dev | <App Name> Dev | Debug keystore (automatic) |
| prod | <id> | <App Name> | Release keystore (android/key.properties) |

> Flutter ≥ 3.19 sets `FLUTTER_APP_FLAVOR` for you; read it with
> `String.fromEnvironment('FLUTTER_APP_FLAVOR')`. Do not pass it explicitly.

---

## Signing / keystore   <!-- if the app ships releases -->

- Keystore file: <path>. Alias: <alias>. Keep at least one offline backup.
- Create `android/key.properties` (gitignored — never commit).
- `.gitignore` must include: `key.properties`, `*.jks`, `*.keystore`, `build/symbols/`.
- Other platforms: <Apple certificates/profiles, Windows code-signing certificate, notarization
  credentials> live outside the repo (`guideline.md` §2.5). Never commit them.

---

## Security rules

- Never log secrets, keys, tokens, or decrypted data — even in debug builds.
- <Store sensitive data in flutter_secure_storage; never in SharedPreferences.>
- Request only the permissions the app needs; <never add INTERNET if the app is offline>.
- <android:allowBackup="false" must remain in the manifest.> (if applicable)

---

## Localization rules   <!-- mandatory for every app -->

- Declared languages (from `docs/PROJECT_PROFILE.md`): **<English (`en`, template), ...>**. Every
  feature and every screen works in every declared language.
- All user-visible text comes from `lib/l10n/*.arb` via `AppLocalizations` — never a raw string
  literal in a widget (even in a single-language app).
- `l10n.yaml` (project root) and one ARB file per declared language (`app_<code>.arb`) must exist.
  Run `flutter gen-l10n` after editing any `.arb` file.
- Every new key goes into **every** declared language's file with a real translation. Never leave
  the template-language value sitting in another language's file.
- Every ARB key needs an `@key` description entry in the template file.
- Language packs in force: <none / `language_packs/<name>.md` — follow its rules, gates and
  glossary>. Flag any translation you are unsure of for native-reader review in the change log.
- `supportedLocales` equals the declared languages; <fallback delegate installed for: `<codes>` /
  none needed> (§8.3.1). Format dates and numbers with the `formattingLocale(...)` helper (§8.3.2).
- <If 2+ languages:> The language is user-selectable in Settings (System default + each language
  by its endonym), persisted, and applied without restarting the app.
- Menu, button, label, tab and tooltip strings stay short in every declared language (§8.6); only
  `desc…`/`help…`/`empty…`/`error…`/`body…` keys may be long prose.
- Every icon-only control has a localized `tooltip:` (§7.8).
- Material and Cupertino come from the `material_ui` / `cupertino_ui` packages (pinned with `^`).
  Import `package:material_ui/material_ui.dart`, never `package:flutter/material.dart` (§6.1).
- The About screen is data-driven and localized<; it ends with the signature badge "<badge text>"
  — only if the project profile enables it>.
- Literals are allowed only for logs, non-UI exception messages, asset paths, route names, and
  map/JSON keys.

---

## Code style / naming

- Files `snake_case.dart`; classes `PascalCase`; variables/methods `camelCase`;
  providers `camelCase` + `Provider` suffix.
- Use `package:` imports, not relative. Prefer `const` constructors, `final` locals, single quotes.
- Run `dart format .` and keep `flutter analyze` at zero warnings before every commit.

---

## Testing rules

- Mirror `lib/` structure in `test/` (e.g. `test/services/`, `test/data/`).
- <Coverage target: e.g. 80% on lib/data and lib/domain.>
- Critical areas that must be covered before release: <list them>.
- Add or update a test whenever you add or change a service/DAO/use-case.

---

## Dependency constraints   <!-- if constrained -->

- Blocked (never add, never accept as transitive dep): <http clients, cloud/BaaS, analytics,
  crash reporting, ads, network-status — for an offline app>.
- Before adding any new package: check its `pubspec.yaml` for networking deps, state why it is
  needed, and confirm it fits the hard rules.

---

## Where things live

```
CLAUDE.md            # this file — project rules
docs/                # design docs (Thin profile)
plans/               # one plan per change (see workflow rules)
change_log/          # one log per implemented change
lib/                 # app source
test/                # tests
```

---

## Workflow rules (mandatory)

Every change follows plan-before-changing and log-after-changing:

1. **Plan before changing.** Write a full plan to `plans/` named
   `yyyymmdd_hhMMss_<short-slug>.md` with a `**Status:**` line, the files to change, the issue,
   and the fix. Then **STOP and get explicit approval** before editing/creating/deleting any
   project file (other than the plan). A question or ambiguous reply is not approval.
2. **Log after changing.** After implementing, write a change log to `change_log/` named
   `yyyymmdd_hhMMss_<short-slug>.md` describing what changed and referencing its plan.
3. **Relative paths & privacy only.** `plans/` and `change_log/` files are committed and may become
   public on the internet. They MUST use relative repository paths only (never absolute system
   paths like `C:\...`, `l:\...`, or `file:///...`). They MUST NOT contain any **local system
   details** — OS user name, computer/host name, home or drive-letter paths, network share names,
   LAN/internal IP addresses, local server URLs with ports, device serial numbers, personal email
   addresses — or any secret (API keys, tokens, passwords, keystore passphrases, credentials, PII).
   Write them as if a stranger will read them; nothing should reveal the machine they came from.

Create `plans/` and `change_log/` if they do not exist.

---

## Communication rules

- **Always use simple English.** Write all responses, plans, change logs, and explanations in
  plain, simple English. Short sentences, common words. Explain any jargon you must use.

---

## What Claude must always / never do   <!-- recommended, esp. Thick profile -->

**Always:** <read this file first; state the target layer before adding a class; run analyze +
test after changes; keep main.dart thin.>

**Never:** <put business logic in a widget; call a DAO from a widget; edit generated
`*.freezed.dart` / `*.g.dart`; add a blocked dependency; log secrets.>
````

---

## 5. Sections you must always keep verbatim in spirit

Two sections are part of this guideline set's working method and must appear in **every** `CLAUDE.md`, both
profiles, worded the same in meaning:

- **Workflow rules** — plan → approve → log, with relative repository paths only, no local system
  details and no sensitive internet-inappropriate data, `plans/` and `change_log/` naming, and the
  hard approval gate.
- **Communication rules** — always simple English.

Do not shorten these into a single link. Keep the short inline version shown in the template so the
AI always sees them, even in a Thin file.

---

## 6. Per-section writing tips

- **Read-first banner**: one or two lines. State that the file is auto-loaded and must be read
  before any change. Do not pad it.
- **Identity table**: a table beats prose. Include SDK versions, minSdk, org id, and the
  connectivity stance (offline vs online) — the AI uses these constantly.
- **Hard rules**: number them and phrase each as a testable "must". Put the *why* only when it is
  not obvious. Keep the list to real non-negotiables — a long list of soft rules gets ignored.
- **Architecture**: name the exact folders and the dependency direction. State the one or two
  boundaries that get broken most (widgets ↔ DB is the classic one).
- **Commands**: give copy-paste-ready lines. If the project has flavors, say that a bare
  `flutter run` fails.
- **Security**: lead with "never log secrets" and the permission stance. Link to `docs/security.md`
  in Thin files instead of inlining the threat model.
- **Testing**: name the *critical* areas by feature (migrations, crash recovery, parsers, crypto).
  Generic "write tests" advice is not useful.
- **Dependencies**: for an offline app, an explicit block list is worth its length — it stops a
  networking package sneaking in as a transitive dep.
- **Dos & Don'ts**: most valuable in Thick files. Keep each line one concrete action, not a theme.

---

## 7. Anti-patterns to avoid

- Repeating a `docs/` file inside a Thin `CLAUDE.md`. Link instead — two copies drift.
- A wall of soft "should" rules. Rules that are not enforced train the reader to skip the section.
- Stale versions and tables. If the identity table says the wrong Flutter version, trust drops.
- Dropping the workflow or simple-English rules to "save space". They are mandatory.
- Feature-first restructuring advice with no instruction to do so — say "Tier 1, do not promote
  without discussion".

---

## 8. Final self-check before saving a new `CLAUDE.md`

- [ ] Profile chosen (Thin or Thick) and the file matches it.
- [ ] All "Always" sections from the §3 checklist are present.
- [ ] Identity table filled with real versions, minSdk, org id, connectivity stance.
- [ ] Build commands are copy-paste ready and match the project's flavors.
- [ ] Workflow rules (plan/approve/log) and simple-English rule are present, inline.
- [ ] `plans/` and `change_log/` entries use relative paths only and contain zero local system details and zero sensitive data — safe to publish on the internet.
- [ ] A localization rule is present: all user-visible text comes from `lib/l10n/*.arb` via `AppLocalizations`.
- [ ] The declared languages are named, matching `docs/PROJECT_PROFILE.md`, with key parity across their ARB files.
- [ ] Any language pack in force is named, and its key rule is summarized.
- [ ] With two or more languages: the in-app language picker rule is present.
- [ ] The declared platforms and stores are named, matching `docs/PROJECT_PROFILE.md`, with build
      commands for each and a pointer to `docs/platform_store_readiness.md`.
- [ ] The tooltip rule (every icon-only control) and the short-label rule are present.
- [ ] The About-screen rule is present, including the signature badge only if the profile enables it.
- [ ] No personal data (names, personal emails) is written into the file beyond what the project
      profile publishes.
- [ ] Every `<...>` placeholder from the template is replaced or its section deleted.
- [ ] No `docs/` content is duplicated (Thin) / nothing critical is missing (Thick).
- [ ] Links point to files that exist (or are planned) in this project.
- [ ] Section order matches §2.
