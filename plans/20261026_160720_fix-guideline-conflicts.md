# Plan: Fix the conflicts between guideline files

**Status:** completed

## The issue

A full review of the guideline set found rules that say different things in different files.
An AI agent that reads one file can do the opposite of what another file says. There are 12 main
conflicts (C1–C12) and 17 smaller ones (S1–S17). This plan fixes all of them.

The fixes build on the current working copy, which already holds uncommitted edits from the
"clarify About badge" and "privacy policy URL" plans.

## Files to change

| File | Items |
|---|---|
| `flutter_project_engineering_standard.md` | C3, C6, C7, C8, C11, C12, S4, S5, S6, S14, S17 |
| `guideline.md` | C3, C5, C7, S5, S11 |
| `release_process.md` | C3, C4, C5 |
| `platform_store_readiness.md` | C1, C3, S1, S11 |
| `flutter_build_flavors_guide.md` | C5, C8 |
| `AI_AGENT_START_HERE.md` | C3, C6 |
| `PROJECT_PROFILE_TEMPLATE.md` | C6, S1 |
| `CLAUDE_MD_GUIDELINE.md` | C5, C7, S5, S13, S15 |
| `AGENTS_MD_GUIDELINE.md` | same as `CLAUDE_MD_GUIDELINE.md` (the two templates must stay identical) |
| `architecture.md` | S1 |
| `DOCS_FOLDER_GUIDELINE.md` | S2, S16 |
| `README.md` | S3 |
| `GUIDELINES_MANIFEST.md` | S3 |
| `language_packs/sanskrit_malayalam.md` | C1, S7, S8, S9 |
| `docs/platform_store_readiness_README.md` | C1 |
| `docs/release_process_README.md` | C2, C4 |
| `docs/flutter_project_engineering_standard_README.md` | C9, S10 |
| `docs/security_README.md` | C10 |

No app project is changed. Nothing outside this repository is changed.

---

## Decisions that need your answer

The plan uses the recommended choice for each of these. Tell me if you want a different one.

| # | Question | Recommended choice (used in this plan) |
|---|---|---|
| D1 | Is CI required for every repository (C6)? | CI checks are **required on GitHub**. A project on another git host runs the same checks in that host's CI, or locally before every merge. The profile row records which. |
| D2 | Keystore backup rule (C5)? | **At least two separate secure backups, and at least one of them offline** (not in cloud storage). |
| D3 | What to do about Web (C3)? | Keep Web declarable. Make the obfuscation rules name the **native** platforms only, add a short **§11C Web** to `release_process.md`, and a short **Web** note in `platform_store_readiness.md` (no store gate; §1 rules still apply when public). |
| D4 | About-screen privacy policy row (S11)? | **MUST** for apps on a public store, to match the store gate. |

---

## Main conflicts — the fix

### C1. "Update this file" vs. the read-only submodule

- `platform_store_readiness.md` intro (line 14) and §7 (line 476): replace "update this file (or
  the project's local copy)" with: "tell the user, so they can update this file in the guidelines
  repository (`DOCS_FOLDER_GUIDELINE.md` §1.1). Record the new rule in the release evidence
  meanwhile."
- `docs/platform_store_readiness_README.md` line 37: "the store wins — tell the user so the
  guideline file can be updated."
- `language_packs/sanskrit_malayalam.md` §4.4 (line 215): "When a term is missing, the project
  writes a note for the user (`DOCS_FOLDER_GUIDELINE.md` §1.1) so the term can be added to this
  pack. Until then, record it in the project's `docs/glossary.md`, marked 'needs native-reader
  review'."

### C2. Size analysis after the release build

`docs/release_process_README.md`:
- Rule N (line 99): "A size-analysis build is optional. If you run it, run it **before** the
  hardened build — it writes into the same folders. Every prod release build MUST be followed by
  a symbol archive step."
- Example request (line 125): replace "post-build size-analysis step" with "the size-analysis
  step before it, if wanted".

### C3. Web has no release path

- Standard §15.1, `guideline.md` §4 checklist (line 785), `AI_AGENT_START_HERE.md` §6, and
  `platform_store_readiness.md` §1 "Obfuscation and symbols": say the `--obfuscate
  --split-debug-info` rule applies to every declared **native** platform (Android, iOS, Windows,
  macOS, Linux). For Web: "use the hardening flags that `flutter build web -h` lists for your
  Flutter; Web builds do not produce the same symbol files."
- `release_process.md`: add `Web — hosting: <...>` to §1, and a short **§11C Web Release Steps**
  (build with `flutter build web --release`, check `flutter build web -h` for options, deploy to
  the declared host over HTTPS, verify in each supported browser). Add a Web row to §12.
- `platform_store_readiness.md`: add a short note after the section table: "Web has no store
  gate. §1 rules (privacy policy, data disclosure, real testing) still apply to a public site."

### C4. Git tag format

`release_process.md`: keep `vX.Y.Z` (§3). In §9 step 14, §10 step 11 and §15, write
`git tag v<X.Y.Z>` and add: "`<X.Y.Z>` is the version without the `+build` part." Same change
in `docs/release_process_README.md` (line 76).

### C5. Keystore backup

One rule everywhere (D2): "Keep at least two separate secure backups, at least one of them
offline (not cloud storage)."
- `guideline.md` §2.3 (the source of truth) and §2.5 wording.
- `flutter_build_flavors_guide.md` Step 1 (line 282): replace "(cloud storage + physical)" with a
  pointer to `guideline.md` §2.3.
- `release_process.md` §7 (line 282): point to `guideline.md` §2.3.
- `CLAUDE_MD_GUIDELINE.md` / `AGENTS_MD_GUIDELINE.md` signing section: "Keep the backups required
  by `guideline.md` §2.3."

### C6. Is CI required?

(D1)
- Standard §19 intro and §19.1: "Every repository hosted on GitHub MUST have the CI workflow. On
  another git host, run the same checks in that host's CI, or locally before every merge, and
  record which in `docs/PROJECT_PROFILE.md` §7."
- `PROJECT_PROFILE_TEMPLATE.md` §7 "CI checks" row: `<GitHub CI — analyze/test only (required
  on GitHub) / other host's CI / local checks before merge>`.
- `AI_AGENT_START_HERE.md` §3.2 "Tools and workflow": change "whether CI checks are wanted" to
  "where the CI checks run (required on GitHub)".

### C7. About text vs. "all text from AppLocalizations"

Name `app_config.json` as the one allowed second source of user-visible text:
- Standard §8.2 exceptions: add a line under the table: "About-screen values come from
  `assets/config/app_config.json` locale maps (`guideline.md` §1), not ARB. Their labels still
  come from ARB."
- `guideline.md` §3 rule (line 701) and §4 checklist (line 762): "...from `AppLocalizations`
  (About values: from `app_config.json`, §1)".
- `CLAUDE_MD_GUIDELINE.md` / `AGENTS_MD_GUIDELINE.md` localization section: same exception.

### C8. No single source for the app name

Add a short rule to `guideline.md` §1 (new §1.8 "The app name"):
- In-app name (About screen, window title) comes from `appName` in `app_config.json`.
- The OS / launcher name is set in each platform's own files (Android `strings.xml` per
  language, iOS/macOS `InfoPlist.strings` / `CFBundleDisplayName`, `Runner.rc`, `msix_config`,
  the Linux `.desktop` file) and MUST match `appName` for each declared language.

Then fix the examples:
- Standard §8.2 ARB example: rename `appTitle` to `titleHome` ("Home"), so the example does not
  suggest a second app-name source.
- Standard §5.2 `AppFlavorConfig`: replace `appName` with `nameSuffix` (`' Dev'` for dev, `''`
  for prod), to be added to the localized name.
- Standard §5.5 window example: set the title after the config and language are loaded, from
  `config.appName.resolve(lang)` (plus the flavor suffix), not the literal `'MyApp'`.
- `flutter_build_flavors_guide.md` Android: replace `resValue("string", "app_name", ...)` with
  per-flavor `res/values/strings.xml` and `res/values-<code>/strings.xml`, so the name can be
  translated (`platform_store_readiness.md` §2.1). Update the "Flavor-Specific Resource Files"
  tree to show a `values-<code>/` folder.

### C9. Animation advice reversed

`docs/flutter_project_engineering_standard_README.md` (lines 111 and 138): "prefer transition
widgets (`FadeTransition`, `ScaleTransition`, `SlideTransition`) or `AnimatedOpacity`; never
rebuild a plain `Opacity` / `Transform` with `setState` every frame" — matching standard §6.5.

### C10. Security explainer assumes an offline app

`docs/security_README.md`:
- §11 row (line 54): "List every permission. If the app is offline, verify `INTERNET` is absent."
- Rule 3 (line 102): make the INTERNET line conditional: "If the app is offline: never add the
  INTERNET permission."
- The "before coding" table: say the 🔴 rows apply in full under the Sensitive Data Extension;
  other apps fill §10, §11 and §13 (as `security.md` says).

### C11. Creating the profile file

Standard §22.1 (line 3333): "If it is missing, run the intake in `AI_AGENT_START_HERE.md` §3 and
get the user's approval before creating it."

### C12. `flutter create` example

Standard §3.3 (line 190): change to
`flutter create --org com.example --project-name my_app --platforms=<declared platforms> .`
and add: "Run it in the project root (the folder that holds `docs/guidelines/`). The final `.`
means this folder."

---

## Smaller inconsistencies — the fix

- **S1. Minimum OS has four homes.** `docs/PROJECT_PROFILE.md` §3 is the one home.
  `architecture.md` §15 "Minimum OS" column and §19 line: "see `docs/PROJECT_PROFILE.md` §3".
  `platform_store_readiness.md` §2.2: record `minSdk` and its reason in
  `docs/PROJECT_PROFILE.md` §3. Profile template: note that the macOS arm64-only decision
  (standard §5.5.2) also goes in §3.
- **S2. Short `security.md`.** `DOCS_FOLDER_GUIDELINE.md` §6: "at least §10, §11 and §13", to
  match `security.md`.
- **S3. The manifest is copied.** `README.md` (templates table) and `GUIDELINES_MANIFEST.md`
  ("Templates vs. references"): add that `GUIDELINES_MANIFEST.md` is the one pointer file that is
  copied unchanged; every other non-template file is never copied.
- **S4. Logger name.** Standard §8.2 table: `AppLogger.d(...)` → `AppLogger.debug(...)`.
- **S5. `lib/` layout.** `guideline.md` §3 tree: add `core/logging/` (`app_logger.dart`).
  CLAUDE/AGENTS template "Architecture rules": `core/config/` instead of `config/`. Standard §6.1:
  token file "e.g. `lib/theme/tokens.dart` (Tier 1) or `lib/app/theme/tokens.dart` (Tier 2)".
- **S6. ARB prefixes.** Standard §8.6: "Prefix every key, except the fixed keys named in
  `guideline.md` (`madeWithLove`, `madeWithLoveA11y`, `aboutDetail<Key>`)". §7.8 examples:
  `actionDelete` → `tooltipDelete`, `actionShare` → `tooltipShare`.
- **S7. Sanskrit button length.** `sanskrit_malayalam.md` §5: target column "1 word — nominal for
  titles and labels, imperative for buttons"; and "A button is one imperative verb, optionally
  with one indeclinable when the meaning needs it (`सर्वं चीयताम्`, `पुनः प्रयत्यताम्`)."
- **S8. Badge text in the pack.** `sanskrit_malayalam.md` §4.4 badge rows: add "Example wording.
  The project profile is the record for the badge text (`guideline.md` §1.7)."
- **S9. Hindi gate vs. a project that also declares Hindi.** `sanskrit_malayalam.md` §4.1: add
  "If the project also declares Hindi (`hi`), remove `app_config.json` from the file list and
  check its `sa` entries with a JSON-aware script or by review."
- **S10. Language split.** `docs/flutter_project_engineering_standard_README.md` line 73: "when
  two or more languages are declared".
- **S11. Privacy policy wording.** `platform_store_readiness.md` §1: "Required by Google Play and
  the App Store for every app, and by the Microsoft Store for almost every app." `guideline.md`
  §1.6: privacy-policy row is **MUST** for apps on a public store (D4).
- **S12. Screenshot blocking.** Standard §15.2: "MUST be enabled on every screen that shows
  sensitive data" (matches `security.md` §10).
- **S13. Coverage target.** CLAUDE/AGENTS templates: replace `<Coverage target: e.g. 80% ...>`
  with "No fixed coverage % (engineering standard §18.3); name the critical areas below."
- **S14. `CHANGELOG.md` in upgrades.** Standard §5.3 upgrade step 5: "record the upgrade in the
  change log (`change_log/`), and in `CHANGELOG.md` if the project keeps one."
- **S15. Thick style vs. baseline docs.** CLAUDE/AGENTS §1 "Thick": add "This is a temporary
  state while an existing app is migrated (`guideline.md`: existing apps SHOULD migrate). The
  baseline `docs/` set is still the goal."
- **S16. `DOCS_FOLDER_GUIDELINE.md` wording.**
  - `implementation_progress.md` becomes a **Living** doc with a `**Last updated:**` line (it is
    a status checklist that changes). Update §5, §6 item 9 and the §7 table.
  - §7 table: remove "(or `security_rules.md`)" and "/ `release_signing.md`"; add one line: "Older
    apps may have `security_rules.md` or `release_signing.md`; new apps use the baseline names
    from §6." Adjust the §4 and §9 examples to match.
- **S17. Standard README matches.** After the standard changes above, check its explainer for
  any sentence that now disagrees (logger, app name, CI) and update it.

---

## Out of scope

- No app project is changed.
- No new rules beyond what is needed to make the files agree.
- Wording of rules that already agree is not touched.

## After the change

- Search the repository again for the old wording (e.g. `AppLogger.d(`, `cloud storage +
  physical`, `appTitle`, `post-build size`, `git tag v<version>`) to confirm nothing is left.
- Check that `CLAUDE_MD_GUIDELINE.md` and `AGENTS_MD_GUIDELINE.md` templates still match.
- Write a change log in `change_log/` and set this plan's status to `completed`.
