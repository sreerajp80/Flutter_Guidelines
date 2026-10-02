# Guideline: How to Write an `AGENTS.md` for a Flutter Project

Every Flutter project **MUST** have an `AGENTS.md` file at its root. This guideline defines the mandatory standards and structure for creating and maintaining `AGENTS.md` across all Flutter projects (both new and migrated existing apps).

`AGENTS.md` is read by AI coding assistants, agentic models, and LLMs (such as Gemini, Antigravity, Cursor, Windsurf, Codex, OpenDevin, and custom AI agents) automatically or upon session initialization in that project. It is the primary, tool-agnostic instruction file, ensuring non-Claude LLMs operate with full alignment to the project's architecture, security, build standards, and mandatory workflow rules.

Every Flutter repository MUST maintain both `CLAUDE.md` (for Claude Code native conventions) and `AGENTS.md` (for open/agentic LLM ecosystems) at the project root as mandatory Core Baseline requirements (**MUST**).

---

## 1. First choose a style: Thin or Thick

Pick one of two styles before you write anything.

### Thin pointer style
Use this for every app that has the baseline `docs/` set — `architecture.md`, `security.md`,
`release_process.md`, and so on.

- `AGENTS.md` stays short. It gives identity, commands, and a short rule summary.
- It **points** to the `docs/` files for detail. It does not repeat them.
- Rule: if a detail lives in a `docs/` file, do not copy it into `AGENTS.md` — link to it.
- In Thin-style repositories, `AGENTS.md` and `CLAUDE.md` share identical architecture and rule summaries.

### Thick self-contained style
Use this only for an existing app that has **no** `docs/` folder yet, so `AGENTS.md` must hold
everything itself.

- `AGENTS.md` inlines the detail: schema tables, provider/route tables, full dos-and-don'ts.

### How to choose
- **New apps use Thin.** Every new app has the baseline `docs/` set (`DOCS_FOLDER_GUIDELINE.md` §6), so the detail already has a home.
- **Thick** is only for an existing app that has no `docs/` folder yet. Move it to Thin when the `docs/` set is created.

---

## 2. Canonical section order

Write sections in this order. Skip the ones that do not apply (see the checklist in §3).

1. Title + read-first banner
2. Project identity / tech stack
3. Doc references (Thin style) — the "read these before working" table
4. Hard / non-negotiable rules
5. Architecture rules (layers, boundaries, state, navigation, database)
6. Build & run commands
7. Build flavors (dev / prod)
8. Signing / keystore
9. Security rules
10. Localization rules
11. Code style / naming conventions
12. Testing rules
13. Dependency constraints
14. Where things live (project tree)
15. Workflow rules (plan → approve → log) — from this guideline set
16. Communication rules (simple English) — from this guideline set
17. Dos & Don'ts ("What AI agents must always / never do")

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
| 10 | Localization rules | **Always** | inline | inline |
| 11 | Code style / naming | **Always** | short | full table |
| 12 | Testing rules | **Always** | short + link | full |
| 13 | Dependency constraints | If constrained | link | inline allow/block lists |
| 14 | Where things live (tree) | Recommended | short | full |
| 15 | Workflow rules (plan/log) | **Always** | inline (see §6) | inline (see §6) |
| 16 | Communication rules | **Always** | inline (see §6) | inline (see §6) |
| 17 | Dos & Don'ts | Recommended | optional | **yes** |

Every Flutter app **MUST** have a root `AGENTS.md` file. All "Always" sections in the table above must appear in every project's `AGENTS.md`, regardless of style.

---

## 4. Fill-in-the-blanks template

Copy this into a new project's `AGENTS.md` and replace every `<...>` placeholder. Delete any
section that the checklist marks optional and the project does not need. Notes in `> ` blocks are
guidance — remove them from the final file.

````markdown
# AGENTS.md — <App Name>

This file is read by AI agents and LLM coding assistants (Gemini, Antigravity, Cursor, Windsurf, Codex, etc.) at the start of every session in this repository.
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

> Keep this table honest and current. It is the fastest way for AI agents to orient.

---

## Read these docs before working   <!-- Thin style only -->

| Document | Read when |
|----------|-----------|
| docs/architecture.md | Changing structure, screens, state, services, models, repositories |
| docs/security.md | Touching permissions, logging, storage, crypto, manifest |
| docs/release_process.md | Building a release, versioning, release checklist |
| docs/guidelines/flutter_build_flavors_guide.md | Build config, signing, flavors, Gradle, ProGuard |
| docs/guidelines/flutter_project_engineering_standard.md | Any code change — layers, naming, testing |
| docs/PROJECT_PROFILE.md | Always — platforms, stores, languages, identity, About options |
| docs/guidelines/guideline.md | About screen, Android keystore, `lib/` layout |
| docs/guidelines/platform_store_readiness.md | Before any release; when touching permissions, entitlements, store-listed behavior |
| docs/GUIDELINES_MANIFEST.md | The shared Flutter guidelines index |

> `docs/<file>` = this app's filled-in copy (templates). `docs/guidelines/<file>` = the shared
> reference in the submodule; never copy it into `docs/`.

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

> Android and iOS: `--flavor <name>` only — Flutter sets `FLUTTER_APP_FLAVOR` itself; never pass it
> with `--dart-define`. Desktop: `--dart-define=APP_FLAVOR=<name>`. `AppFlavorConfig` reads both
> (engineering standard §5.2).

---

## Signing / keystore   <!-- if the app ships releases -->

- Keystore file: <path>. Alias: <alias>. Keep at least one offline backup.
- Create `android/key.properties` (gitignored — never commit).
- `.gitignore` must include: `android/key.properties`, `android/*.jks`, `android/*.keystore`, and `build/` (which also holds the debug symbols).
- Other platforms: <Apple certificates/profiles, Windows code-signing certificate, notarization
  credentials> live outside the repo (`docs/guidelines/guideline.md` §2.5). Never commit them.

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
  none needed> (engineering standard §8.3.1). Format dates and numbers with the `formattingLocale(...)` helper (§8.3.2).
- <If 2+ languages:> The language is user-selectable in Settings (System default + each language
  by its endonym), persisted, and applied without restarting the app.
- Menu, button, label, tab and tooltip strings stay short in every declared language (engineering standard §8.6); only
  `desc…`/`help…`/`empty…`/`error…`/`body…`/`aboutDetail…` keys may be long.
- Every icon-only control has a localized `tooltip:` (engineering standard §7.8).
- Material and Cupertino come from the `material_ui` / `cupertino_ui` packages (with a caret `^` constraint).
  Import `package:material_ui/material_ui.dart`, never `package:flutter/material.dart` (engineering standard §6.1).
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
AGENTS.md            # this file — project rules for AI agents / LLMs
CLAUDE.md            # Claude Code native project rules
docs/                # design docs (Thin style)
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

## What AI agents must always / never do   <!-- recommended, esp. Thick style -->

**Always:** <read this file first; state the target layer before adding a class; run analyze +
test after changes; keep main.dart thin.>

**Never:** <put business logic in a widget; call a DAO from a widget; edit generated
`*.freezed.dart` / `*.g.dart`; add a blocked dependency; log secrets.>
````

---

## 5. Dual Alignment: `AGENTS.md` and `CLAUDE.md`

In all Flutter repositories, `CLAUDE.md` and `AGENTS.md` exist side-by-side as mandatory root instruction files:
- Both files contain the same rules, identity values and commands.
- `AGENTS.md` writes these rules out in full. It MUST NOT be only a pointer to `CLAUDE.md`: many agents do not follow links. Only tool-specific wording (for example "Claude" vs "AI agents") may differ.
- When updating project commands, security rules, architecture boundaries, or identity properties, update BOTH `CLAUDE.md` and `AGENTS.md` so that all AI coding assistants stay synchronized.

---

## 6. Sections you must always keep verbatim in spirit

Two sections are part of this guideline set's working method and must appear in **every** `AGENTS.md`, both
styles, worded the same in meaning:

- **Workflow rules** — plan → approve → log, with relative repository paths only, no local system
  details and no sensitive internet-inappropriate data, `plans/` and `change_log/` naming, and the
  hard approval gate.
- **Communication rules** — always simple English.

Do not shorten these into a single link. Keep the short inline version shown in the template so AI agents always see them, even in a Thin file.

---

## 7. Per-section writing tips

- **Read-first banner**: one or two lines. State that the file is auto-loaded and must be read before any change.
- **Identity table**: a table beats prose. Include SDK versions, minSdk, org id, and connectivity stance.
- **Hard rules**: number them and phrase each as a testable "must".
- **Architecture**: name exact folders and dependency directions.
- **Commands**: copy-paste ready lines. Mention flavor constraints if applicable.
- **Workflow & Communication**: keep inline and verbatim in spirit.

---

## 8. Anti-patterns to avoid

- Creating `CLAUDE.md` without creating `AGENTS.md` (both are mandatory).
- Divergent rules between `CLAUDE.md` and `AGENTS.md` (e.g. different build commands or architecture layers).
- Dropping workflow or communication rules.

---

## 9. Final self-check before saving a new `AGENTS.md`

- [ ] Style chosen (Thin or Thick) and the file matches it.
- [ ] Both `AGENTS.md` and `CLAUDE.md` exist at the project root (**MUST**).
- [ ] All "Always" sections from the §3 checklist are present.
- [ ] Identity table filled with real versions, org id, connectivity stance, and minSdk if Android is declared.
- [ ] Build commands are copy-paste ready and match the project's flavors.
- [ ] Workflow rules (plan/approve/log) and simple-English rule are present, inline.
- [ ] `plans/` and `change_log/` entries use relative paths only and contain zero local system details and zero sensitive data — safe to publish on the internet.
- [ ] A localization rule is present: all user-visible text comes from `lib/l10n/*.arb` via `AppLocalizations`.
- [ ] The declared languages are named, matching `docs/PROJECT_PROFILE.md`, with key parity across their ARB files.
- [ ] Any language pack in force is named, and its key rule is summarized.
- [ ] With two or more languages: the in-app language picker rule is present.
- [ ] The declared platforms and stores are named, matching `docs/PROJECT_PROFILE.md`, with build
      commands for each and a pointer to `docs/guidelines/platform_store_readiness.md`.
- [ ] The tooltip rule (every icon-only control) and the short-label rule are present.
- [ ] The About-screen rule is present, including the signature badge only if the profile enables it.
- [ ] No personal data (names, personal emails) is written into the file beyond what the project
      profile publishes.
- [ ] Every `<...>` placeholder from the template is replaced or its section deleted.
- [ ] Rules in `AGENTS.md` match `CLAUDE.md` exactly.
