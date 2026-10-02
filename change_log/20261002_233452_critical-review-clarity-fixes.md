# Change log — critical review: errors, contradictions and unclear wording

Implements plan: `plans/20261002_231758_critical-review-clarity-fixes.md`

## Decisions used (from the user)

- All 9 baseline `docs/` files stay mandatory for every app. `security.md` and
  `release_process.md` are kept short when their profile does not apply.
- Thin/Thick "profile" in the agent-file guidelines is now "Thin style / Thick style".
- `android:allowBackup=false` is MUST only under the Sensitive Data Extension.
- Lists: a builder is MUST for lists that are unbounded or can exceed 20 items.
- OWASP checklist: MUST under the Sensitive Data Extension, SHOULD otherwise.
- Debug symbols: one folder pattern, `build/symbols/<platform>-<flavor>-<version>/`.

## What changed

### Paths and document classes (B1)

- `README.md` and `GUIDELINES_MANIFEST.md` now define **templates** (copied into `docs/`) and
  **references** (read at `docs/guidelines/`). The path rules are written out once.
- 38 references such as `docs/flutter_project_engineering_standard.md` and `docs/guideline.md`
  now point to `docs/guidelines/<file>`.
- `DOCS_FOLDER_GUIDELINE.md` §2 uses the same two classes. The explainers no longer tell users
  to copy reference docs into `docs/`.

### Code and facts (A-items)

- **Android signing (flavors guide Step 4, `guideline.md` §2.2):** `rootProject.file("key.properties")`
  and `rootProject.file(storeFile)`. The old path pointed to `android/android/key.properties`.
- **Startup order (standard §4.5, §11.1, §14.2; `architecture.md` §5):** logging and the error
  handlers now come at step 2. `AppLogger` starts with a console logger, so a log call before
  `init()` no longer throws. `ConfigService` uses `AppLogger.debug` instead of `debugPrint`.
- **iOS flavors (standard §5.4, flavors guide):** rewritten around the `Debug-<flavor>` /
  `Profile-<flavor>` / `Release-<flavor>` build configurations Flutter needs. Bundle id and name
  are set per configuration, `Info.plist` uses `$(PRODUCT_BUNDLE_IDENTIFIER)`, and the Podfile
  mapping was added. The broken xcconfig include paths are gone.
- **Freezed (standard §12.4):** `abstract class` and an exhaustive `switch` instead of `when`.
- **Version numbers in samples:** replaced with `^<current>` ("take it from pub.dev").
- **`l10n.yaml` (standard §8.1):** removed `synthetic-package`. Checked against the Flutter
  breaking-change page: since Flutter 3.32 the localization code is always generated into the
  source tree.
- **`security.md`:** fixed the iOS Keychain claim (items persist across uninstall whatever their
  accessibility class) and replaced SafetyNet with Play Integrity API.
- **Language pack:** the Hindi-marker CI regex now also checks `आपका आपकी आपके हमारा मेरा`. The
  self-test samples still pass, and the new words are caught. Legitimate Sanskrit samples
  (`स्थाप्यताम्`, `यथा तथा कथा`, `विन्यासः`) do not trigger it.
- **`release_process.md` §5:** now points to §9–§11B for the commands (it pointed to §8).
- **Standard §19.2 CI:** the iOS job now uses `--flavor prod`, like Android.
- **Flavors guide:** removed the unsourced "ABI deprecation" claim.
- **Engineering-standard explainer:** removed the invented "~4 ms" rule. It now says profiles are
  declared in `docs/PROJECT_PROFILE.md`, describes the init order as recommended, and gives the
  correct §21.1 list.

### Contradictions (B-items)

- Mandatory docs: standard §21.1, the README and manifest profile tables,
  `AI_AGENT_START_HERE.md`, both template intros and `architecture.md` §22 now match
  `DOCS_FOLDER_GUIDELINE.md` §6.
- `DOCS_FOLDER_GUIDELINE.md`: removed the nine private repo names, replaced `../.agents/AGENTS.md`
  with `../AGENTS.md`, and added the upper-case name exception for `PROJECT_PROFILE.md` and
  `GUIDELINES_MANIFEST.md`.
- `allowBackup`: the store gate §2.4, `release_process.md` §6.4/§8/§9 and `security.md` §10 now
  agree.
- Standard §5: says which sub-sections apply to every app. §5.3 is now "Toolchain Requirements
  (Every App)", and the Android product-flavor snippet moved into §5.2. The desktop `--flavor`
  rule is one statement (Windows/Linux: no; macOS: only with schemes; `APP_FLAVOR` always).
  Section numbers did not change.
- Touch targets live only in §7.1. Screen-reader testing covers the reader of each declared
  platform (TalkBack, VoiceOver, Narrator, Orca).
- List rule unified in §10.3, §22.2 and the explainer.
- OWASP scope unified in standard §15.3/§23.3, `security.md` §12, `release_process.md` §8/§14 and
  the explainers.
- Debug-symbol folder unified everywhere. `*.symbols/` was removed from `.gitignore` samples
  (`build/` covers it), and the archive copy is stated to live outside the repo.
- `aboutDetail…` added to every list of long-prose key prefixes.
- Badge heart colour: one design token (`AppColors.heart` = `#E53935`); the "or theme accent"
  option is gone.
- MSIX samples no longer list `runFullTrust` (the package adds it).
- Agent-file guidelines: "Localization rules" added to the section order and checklist (now 17
  sections). New apps use Thin style; Thick is only for an existing app with no `docs/`. In
  `AGENTS_MD_GUIDELINE.md` the "see §5" references now say §6, and `AGENTS.md` must write the
  rules out in full, not just link to `CLAUDE.md`.
- Store gate §1 privacy-policy wording matches §2.5.

### Wording (C-items)

- "Pinned with `^`" became "with a caret (`^`) constraint". Bare `§` numbers in the agent-file
  templates now say "engineering standard". The badge rule was rewritten in plain terms. The
  §8.6 length table has three non-overlapping rows. `AppLocalizations.of(context)!` was removed.
  The ARB snippets no longer contain `//` comments. The App Bundle claim about "Play Integrity"
  was removed. Store gate §3–§6 now have an "Applies when" line.

### Trimming (D-items)

- Deleted the `release_process.md` §9A "Moved" stub.
- The flavors guide no longer repeats the toolchain rules or the desktop `sqflite` / window
  setup; it links to the standard (the single source).
- The explainers' stale "five documents" sections were replaced by a short pointer to
  `README.md`.
- `architecture.md` §7 points to `docs/dependencies.md` instead of an undefined file.
  Formatting fix in the flavors guide notes.

## Needs native-reader review

- `profiles/example_en_ml_sa_profile.md`: the two corrected Sanskrit sentences
  (`एषः अनुप्रयोगः किं करोति इति एकपङ्क्तिवर्णनम्।`,
  `सर्वे प्रयुक्ताः पुस्तकालयाः मुक्तस्रोतसः सन्ति।`).

## Not verified by running code

This repository holds documents only, so the changed Gradle, Xcode and Dart samples were not
built here. The Freezed and iOS-flavor changes follow the current official docs as I understand
them. Check them once in a real project.
