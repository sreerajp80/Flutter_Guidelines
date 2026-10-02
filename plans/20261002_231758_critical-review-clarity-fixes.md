# Critical review — fix errors, contradictions and unclear wording

**Status:** completed

## 1. Goal

Make every document in the guideline set correct, consistent with the others, and sharp. A reader
(human or AI agent) must never have to guess which of two rules wins, which file a path points to,
or whether a code sample works.

## 2. How the review was done

Every non-plan, non-change-log file was read in full: the index files, the agent-file guidelines,
`guideline.md`, the engineering standard, the store gates, the release process, the flavors guide,
the two blueprints, the language pack, the example profile and the six explainers. Findings are
grouped by how much harm they cause.

---

## 3. Findings and fixes

### A. Wrong — code that breaks, or facts that mislead

| # | Where | Problem | Fix |
|---|---|---|---|
| A1 | `flutter_build_flavors_guide.md` Step 4; `guideline.md` §2.2 | `rootProject.file("android/key.properties")` inside `android/app/build.gradle.kts` points to `android/android/key.properties` (the root project **is** `android/`). `storeFile = file(...)` resolves from `android/app/`, but the keystore lives in `android/`. Signing never loads. `guideline.md` §2.2 also hedges ("if Gradle resolves it that way"). | Use `rootProject.file("key.properties")` and `storeFile = rootProject.file(props.getProperty("storeFile"))`. State once, firmly: `storeFile` is relative to `android/`. |
| A2 | Standard §4.5, §11.1, §14.2; `guideline.md` §1.5; `architecture.md` §5 | Logging is initialized at step 6, but the global error handlers (§11.1, set first) and every earlier init step call `AppLogger`. `AppLogger._logger` is `late`, so an early error throws `LateInitializationError`. `ConfigService` uses `debugPrint`, which §14.3 bans. | Make `AppLogger` safe before `init()` (start with a console logger, swap in the file logger in `init()`). Move logging init to step 2, right after `ensureInitialized()` (flavor values are compile-time constants, so nothing blocks it). Replace `debugPrint` in `ConfigService` with `AppLogger.debug`. Update the `architecture.md` §5 table to match. |
| A3 | Flavors guide "Xcode Scheme And xcconfig Setup"; standard §5.4 | Flutter needs Xcode **build configurations** named `Debug-<flavor>`, `Release-<flavor>`, `Profile-<flavor>`. The guide only creates schemes on the plain `Debug`/`Release` configurations, so per-flavor xcconfig files cannot be attached. `#include "Generated.xcconfig"` from `ios/Flutter/dev/` does not resolve. `Info.plist` hard-codes the bundle id instead of `$(PRODUCT_BUNDLE_IDENTIFIER)`. | Rewrite the iOS steps: create the per-flavor build configurations, attach the xcconfig to each, set `PRODUCT_BUNDLE_IDENTIFIER` per configuration, fix the include paths, link the official flavor guide. Shorten standard §5.4 to the rules and point to the guide. |
| A4 | Standard §12.1, §12.4 | Freezed sample uses `class Todo with _$Todo` and `^2.5.0`; current Freezed needs `abstract class` / `sealed class`, and Dart 3 `switch` replaces `when`/`maybeWhen`. | Update the sample and the rule ("use an exhaustive `switch`"). |
| A5 | Many code samples (`material_ui ^1.5.0`, `intl`, `logger`, `msix`, `window_manager`, `build_runner`, `flutter_lints ^6.0.0`, …) | Concrete version numbers invite copying — the exact thing §5.3 forbids ("never write a version from memory"). | Replace with `^<current>` plus a short "take the current version from pub.dev" comment. |
| A6 | `security.md` §13 (iOS) | Says Keychain items are removed on uninstall when `kSecAttrAccessibleAfterFirstUnlockThisDeviceOnly` is used. Accessibility classes do not control that; items persist. | State that Keychain items persist across uninstall; keep the "purge on first launch after reinstall" rule. |
| A7 | `security.md` §10 (Android) | Suggests SafetyNet, which Google has shut down. | Name Play Integrity API only. |
| A8 | `language_packs/sanskrit_malayalam.md` §4.1 | The forbidden-marker list includes `आपका आपकी आपके हमारा मेरा`, but the CI regex does not check them. | Add them to the regex. |
| A9 | `profiles/example_en_ml_sa_profile.md` | Sanskrit gender errors that break the pack's own agreement rule: `एतत् अनुप्रयोगः` (`अनुप्रयोगः` is masculine) and neuter `पुस्तकालयानि` (masculine noun). | Correct to masculine forms and mark them "needs native-reader review" in the change log. |
| A10 | `release_process.md` §5 | "See section 8 for full commands" — §8 is the checklist; commands are in §9. | Point to §9–§11B. |
| A11 | Engineering-standard explainer | Invents a rule ("`compute()` for work over ~4 ms"); says profiles are declared in `architecture.md` §1 (they live in `docs/PROJECT_PROFILE.md`); says §21.1 requires `security.md` (it does not); calls the init order "mandatory" (the standard says "recommended"). | Correct each statement to match the standard. |
| A12 | Standard §19.2 CI example | Android job uses `--flavor prod`; the iOS job does not, yet its symbols folder is named `ios-prod`. | Make both jobs use the same flavor pattern. |
| A13 | Flavors guide "ABI deprecation note" | Says Play deprecated `armeabi-v7a` for new apps in 2024 and new apps should be 64-bit only, while other files list `armeabi-v7a` output. The claim is not backed by a source. | Remove the claim; keep the plain list of `--split-per-abi` outputs. |
| A14 | Standard §8.1 | Offers `synthetic-package: true` (the `flutter_gen` package) as a choice. That package is deprecated in current Flutter. | Keep only "generated code goes in `lib/`" (verify against current docs during the change). |

### B. Contradictions between documents

| # | Conflict | Fix |
|---|---|---|
| B1 | **Path convention.** About 39 references write reference docs as `docs/<file>` (e.g. `docs/flutter_project_engineering_standard.md`, `docs/guideline.md`). But `DOCS_FOLDER_GUIDELINE.md` §2 says reference docs are never copied into `docs/` — they live in `docs/guidelines/`. The explainers even tell users to copy the standard into `docs/`. | Define two classes once, in `README.md` and `GUIDELINES_MANIFEST.md`: **Templates** (copied to `docs/` and filled in: `PROJECT_PROFILE_TEMPLATE.md` → `PROJECT_PROFILE.md`, `architecture.md`, `security.md`, `release_process.md`) and **References** (never copied; read at `docs/guidelines/<file>`). Rewrite every path to match. |
| B2 | **Mandatory docs.** `DOCS_FOLDER_GUIDELINE.md` §6 makes 9 docs mandatory for every app, including `security.md` and `release_process.md`. The standard §21.1 and the README profile table make `security.md` Sensitive-only and `release_process.md` Production-only. | See decision D-1 below. |
| B3 | `DOCS_FOLDER_GUIDELINE.md` | Still names nine private app repos (missed by the "make generic" change). Links to `../.agents/AGENTS.md`, but `AGENTS.md` lives at the project root. Its naming rule ("lowercase `snake_case`") conflicts with the mandatory `PROJECT_PROFILE.md`. | Remove the repo names; use `../AGENTS.md`; add an explicit exception for `PROJECT_PROFILE.md` and `GUIDELINES_MANIFEST.md`. |
| B4 | **`allowBackup`.** `release_process.md` §8 checklist requires `false` for every app; §6.4 says MUST only for sensitive apps; store gate §2.4 says "decided deliberately". | **Decided by the user:** `false` is MUST only under the Sensitive Data Extension; for other apps it is a deliberate choice recorded in `security.md` §10. Align all three. |
| B5 | **Toolchain rules are hidden in an optional section.** Standard §5 says it is "optional for Core Baseline", and the toolchain rules sit under "5.3 Android Flavor Setup" — yet `guideline.md` §4 and `AI_AGENT_START_HERE.md` treat them as Core MUST. | Retitle §5.3 "Toolchain Requirements (every app)"; move the Gradle product-flavor snippet into §5.2. State at the top of §5 which sub-sections apply to every app (5.3–5.5, for declared platforms) and which are Production only. Section numbers stay the same. |
| B6 | **Desktop `--flavor`.** §5.2 table and §5.5 say desktop never accepts `--flavor`; §5.5.2 and the flavors guide say macOS may. | One statement: Windows and Linux do not accept it; macOS may with Xcode schemes; the rule for all desktops is still `APP_FLAVOR`. |
| B7 | Touch-target size is stated in §6.9 (Production) and §7.1 (Core). | Keep it in §7.1 only. |
| B8 | §7.6 requires TalkBack and Narrator whatever the platforms; `architecture.md` §16 lists Orca too. | "Test with the screen reader of each declared platform: TalkBack, VoiceOver, Narrator, Orca." |
| B9 | **ListView.** §10.3: ≤ 20 items OK, beyond that "consider" a builder. §22.2: "never" for more than 20. | One rule everywhere (§10.3, §22.2, explainer): a list that is unbounded or can grow past 20 items MUST use `ListView.builder` / `GridView.builder` / a sliver builder. A list with a fixed, known size of 20 items or fewer MAY use `ListView(children: [...])`. Drop "consider" and the frame-time judgement call. |
| B10 | **Data retention.** §15.4 makes every app that stores user data document retention in `docs/security.md`, which the README tables give only to Sensitive apps. | Resolved by D-1: every app has `security.md`; retention is always recorded in its §13. |
| B11 | **OWASP checklist.** §15.3 reads as required for every production release; DoD §23.3 lists it under Sensitive only. | MUST under the Sensitive Data Extension (sign-off in `security.md` §12 before every production release). SHOULD for other production apps, which may mark items `n/a`. Update standard §15.3, the `release_process.md` §8 checklist and §14 evidence line, and DoD §23.3 to say this. |
| B12 | **Debug-symbol folder.** At least 8 different patterns (`android-release/`, `android-prod/`, `android-$VERSION/`, `<platform>-<version>/`, `<platform>-prod-<version>/`, …). The `.gitignore` entry `*.symbols/` does not match any of them. | One pattern everywhere: `build/symbols/<platform>-<flavor>-<version>/`, where `<platform>` is `android`/`ios`/`windows`/`macos`/`linux`, `<flavor>` is dropped when the app has no flavors, and `<version>` is the full pubspec value `X.Y.Z+N` (e.g. `build/symbols/android-prod-1.4.0+27/`). Bash and PowerShell examples read the version the same way. Remove `*.symbols/` from `.gitignore` (`build/` already covers it). The archived copy lives **outside** the repo, in the place named in `release_process.md` §14. |
| B13 | CLAUDE/AGENTS templates and standard §22.2 list the long-prose key prefixes without `aboutDetail…`, which §8.6 includes. | Add `aboutDetail…` everywhere the list appears. |
| B14 | §6.1 bans colour literals in widgets; the badge widget hard-codes `Color(0xFFE53935)` and also allows "or the theme's error/red accent". | One rule: the heart uses a named design token set to `#E53935`. Drop the "or". |
| B15 | MSIX capabilities: flavors guide sets `capabilities: runFullTrust`; store gate says the package adds it automatically; standard uses `internetClient`. | Use the store-gate wording everywhere; drop `runFullTrust` from the sample. |
| B16 | CLAUDE/AGENTS guidelines | The template has a "Localization rules" section, but the canonical order (§2) and checklist (§3) leave it out, while the self-check requires it and "order matches §2". | Add it to §2 and §3 (after Security). |
| B17 | `AGENTS_MD_GUIDELINE.md` §5 | Says both files hold identical rules, then says `AGENTS.md` may just cross-reference `CLAUDE.md`. | Pick one: `AGENTS.md` writes the same rules out in full (not only a link), as the self-check already says. |
| B18 | Store gate §1 says a privacy policy is required by Play "for most apps"; §2.5 says "every app". | Use "every app" for Google Play. |

### C. Unclear wording and terms

| # | Problem | Fix |
|---|---|---|
| C1 | "Profile" means three different things: applicability profiles, the project profile file, and the Thin/Thick "profile" of a `CLAUDE.md`. | Rename Thin/Thick to **Thin style / Thick style**. |
| C2 | "Pinned with `^`" — a caret is a range, not a pin; §16.6 uses "pin" for exact versions. | Say "with a caret (`^`) constraint"; keep "pin" for exact versions. |
| C3 | CLAUDE/AGENTS templates cite bare `§8.3.1`, `§7.8`, … — the reader cannot tell which document. | Prefix with "engineering standard". |
| C4 | `guideline.md` §1.7 badge: "Identical in every app that enables it for the same owner … MUST NOT be removed … without changing the project profile first." | "The text comes only from the project profile. Do not change or remove it unless the profile changes first." |
| C5 | §8.6 length table: "English (and other Latin-script)" vs "Any other declared language" overlap. | Three clear rows: Latin script 20; CJK 10; any other script 22; a language pack may override. |
| C6 | §8.2 shows `AppLocalizations.of(context)!` as an option, though `l10n.yaml` sets `nullable-getter: false`. | Show only `AppLocalizations.of(context)`. |
| C7 | JSON code fences contain `//` comments (invalid JSON), e.g. `guideline.md` §1.7. | Use a `jsonc` fence or move the file name above the fence. |
| C8 | `guideline.md` §2.4 claims App Bundles enable "Play Integrity protection". | Remove; App Bundles enable Play App Signing and per-device delivery. |
| C9 | Store gate §3–§6 have no "Applies when …" line; §2 does. | Add the same line to each. |

### D. Trimming

| # | Problem | Fix |
|---|---|---|
| D1 | `release_process.md` §9A is a "Moved" stub with history. The file is a template copied into apps. | Delete §9A; the move is already in the change log. |
| D2 | Toolchain rules, `sqflite` FFI init and `window_manager` setup are written out in both the standard and the flavors guide; the standard also calls the guide "canonical" for desktop setup while carrying its own copy. | The standard is the single source. The flavors guide keeps its minimums table and flavor-specific steps, and links to the standard for the rest. |
| D3 | All six explainers end with "How the five documents work together"; there are now more documents. Each explainer says "place it in `docs/`", also for reference docs. | Replace the table with one current line pointing to `README.md`; fix the "place it" step per B1. |
| D4 | `architecture.md` §7 points to `docs/dependency_audit.md`, which no guideline defines. | Point to `dependencies.md` / the release evidence. |
| D5 | Flavors guide "Notes For New Projects": no blank line before **macOS:**. | Fix the formatting. |

---

## 4. Decisions (answered by the user)

- **D-1 Mandatory `docs/` set (B2): keep all 9 mandatory.** `DOCS_FOLDER_GUIDELINE.md` §6 stays
  the rule. The standard §21.1, the README and manifest profile tables, and
  `AI_AGENT_START_HERE.md` are changed to match it. `security.md` and `release_process.md` are
  created for every app; an app outside the Sensitive / Production profiles keeps them short and
  says so at the top (both templates already allow this). B10 then becomes: data retention is
  always recorded in `security.md` §13.
- **D-2 Thin/Thick rename (C1): rename to "Thin style / Thick style".**

## 5. Files to change

- `README.md`, `GUIDELINES_MANIFEST.md`, `AI_AGENT_START_HERE.md`
- `CLAUDE_MD_GUIDELINE.md`, `AGENTS_MD_GUIDELINE.md`, `DOCS_FOLDER_GUIDELINE.md`
- `guideline.md`
- `flutter_project_engineering_standard.md`
- `flutter_build_flavors_guide.md`
- `platform_store_readiness.md`, `release_process.md`
- `architecture.md`, `security.md`
- `language_packs/sanskrit_malayalam.md`, `profiles/example_en_ml_sa_profile.md`
- `docs/architecture_README.md`, `docs/flutter_build_flavors_guide_README.md`,
  `docs/flutter_project_engineering_standard_README.md`,
  `docs/platform_store_readiness_README.md`, `docs/release_process_README.md`,
  `docs/security_README.md`
- New change log in `change_log/`

## 6. What stays the same

- All rules not listed above, all section numbers (so apps that link to `§x.y` keep working),
  the Sanskrit/Malayalam glossary terms, and the plan → approve → log workflow.

## 7. Order of work

1. B1 path convention (touches most files — do it first).
2. A-items (code and facts).
3. B-items, then C-items, then D-items.
4. Consistency pass: grep for `docs/flutter_`, `docs/guideline.md`, `build/symbols/`,
   `debugPrint`, `Thin profile`, `runFullTrust`, `^1.5.0`, and every `§` cross-reference.
5. Write the change log; set this plan to `completed`.

## 8. Risks

- Many files change at once. Mitigation: the grep pass in step 4, and section numbers do not move.
- A3, A4 and A14 depend on current Flutter / Freezed behavior. During the change, check the
  official docs before writing the new text, and say so in the change log.
