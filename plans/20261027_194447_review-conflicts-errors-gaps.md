# Plan: Fix the conflicts, wrong facts and gaps found in the full review

**Status:** completed

## Background

A full re-read of the guideline set (2026-10-27) found three kinds of problems:

- **A. Conflicts** — two files say different things.
- **B. Wrong facts** — a statement or command that is incorrect.
- **C. Gaps** — something the rules need but do not say.

These were checked against current docs and found correct, so they are **not** changed:
`material_ui` / `cupertino_ui`, `dart fix --apply --code=migrate_design_widgets`,
`MaterialUiCompatibilityBridge`, `GlobalMaterialLocalizations.delegates` including Cupertino and
Widgets, the Impeller default on desktop and its opt-out switches.

Items marked **(verify first)** are not confirmed yet. Before editing them, check the source named
in the item. If the check shows the current text is right, drop the item and say so in the change
log.

No section is renumbered, so existing cross-references keep working. No Flutter version numbers
are added (version-agnostic rule).

## Files to be changed

- `flutter_project_engineering_standard.md`
- `guideline.md`
- `release_process.md`
- `security.md`
- `architecture.md`
- `platform_store_readiness.md`
- `flutter_build_flavors_guide.md`
- `AI_AGENT_START_HERE.md`
- `AGENTS_MD_GUIDELINE.md`
- `CLAUDE_MD_GUIDELINE.md`
- `DOCS_FOLDER_GUIDELINE.md`
- `README.md`
- `GUIDELINES_MANIFEST.md`
- `profiles/example_en_ml_sa_profile.md`
- `docs/release_process_README.md`, `docs/security_README.md`,
  `docs/flutter_project_engineering_standard_README.md` (explainers, to match)

---

## A. Conflicts

### A1. OWASP sign-off vs the short `security.md`

**Issue.** Standard §15.3, `security.md` §12 and `release_process.md` §8 make the OWASP checklist
a MUST for Production apps. But a Production-only app keeps a short `security.md`, and the short
form's minimum (§1.1, §3.1, §10, §11, §13) does not include §12. `release_process.md` §14 still
says "required under the Sensitive Data Extension". Standard §23.2 does not mention OWASP.

**Fix.**
- `security.md` intro and `DOCS_FOLDER_GUIDELINE.md` §6: short form also includes §12 when the
  Production App Extension applies.
- `README.md` and `GUIDELINES_MANIFEST.md` profile tables: Production row adds "`security.md` §12
  OWASP sign-off".
- `release_process.md` §14: "required under the Production App Extension and the Sensitive Data
  Extension".
- Standard §23.2: add "OWASP checklist items affected by the change were re-verified (15.3)".
- `docs/security_README.md`: match.

### A2. Wrong APK output paths

**Issue.** `build/app/outputs/apk/prod/release/app-arm64-v8a-prod-release.apk` is used in
`release_process.md` §6.4, §6.6, §9C, `guideline.md` §2.4 Example 3 and `architecture.md` §7.
Gradle names that file `app-prod-arm64-v8a-release.apk`. The name used only exists in
`build/app/outputs/flutter-apk/` (as the flavors guide already says). The checks fail as written.

**Fix.** Use `build/app/outputs/flutter-apk/app-arm64-v8a-prod-release.apk` everywhere (and
`app-arm64-v8a-release.apk` for apps without flavors). Add one short note: "Flutter copies the
APKs to `build/app/outputs/flutter-apk/`; check the build log for the exact name."
In `architecture.md` §7, change the merged-manifest path note the same way.

### A3. Which file Gemini reads

**Issue.** `AGENTS_MD_GUIDELINE.md` says Gemini reads `AGENTS.md` automatically;
`AI_AGENT_START_HERE.md` §4 step 3 says it reads a different file by default. "OpenDevin" is now
"OpenHands".

**Fix.** `AGENTS_MD_GUIDELINE.md` intro and template: say that many agents read `AGENTS.md`;
some (for example Gemini CLI) read their own file by default, and are pointed at `AGENTS.md` as
in `AI_AGENT_START_HERE.md` §4 step 3. Replace "OpenDevin" with "OpenHands". Do not name a
default file name for any tool — tell the reader to check the tool's docs.

### A4. Web steps missing from release lists

**Issue.** `AI_AGENT_START_HERE.md` §4 step 8, `release_process.md` §5 ("sections 9–11B") and
`docs/release_process_README.md` leave out §11C.

**Fix.** Add "§11C Web" / "sections 9–11C" in all three.

### A5. App name rules on iOS and single-value places

**Issue.** The flavors guide sets `CFBundleDisplayName = $(FLAVOR_APP_NAME)`; `guideline.md` §1.8
asks for a translated name in `InfoPlist.strings`. The `.strings` value replaces the plist value,
so the dev build loses " Dev". §1.8 also says "MUST match in each declared language", but
Windows `Runner.rc` and MSIX `display_name` hold one value only.

**Fix.**
- `guideline.md` §1.8: places that hold one value (Windows `Runner.rc`, MSIX `display_name`,
  Snap `title`) use the template-language name. Translations apply where the platform supports
  them.
- Flavors guide, "Xcode Build Configurations And Schemes": when `InfoPlist.strings` translates
  the name, give each flavor its own translated `.strings` values (or keep the display name in
  `InfoPlist.strings` only for prod and accept the template name plus suffix for dev). Say which
  is the default: dev builds MAY show the template-language name with " Dev".

### A6. Smaller clashes

- **`cupertino_ui`.** Standard §8.1 pubspec and the §8.3.1 fallback code: mark `cupertino_ui`
  and `FallbackCupertinoLocalizationsDelegate` "only if the app uses Cupertino widgets or
  delegates", matching §6.1.
- **Migration test start.** §18.2: "from the oldest schema version still installed by users
  (usually version 1)", matching §13.1.
- **Zero warnings.** `release_process.md` §8 and CLAUDE/AGENTS templates: "no new analyzer
  warnings (zero is the goal)", matching standard §16.1. (Or raise §16.1 to MUST — this plan
  takes the softer wording.)
- **Icon format.** §17.1 "Icons" row: "Material/Cupertino icon fonts (`Icons.*`) or SVG via
  `flutter_svg` for custom icons".

---

## B. Wrong facts

### B1. `ListView.separated`
§10.3 last bullet: replace with "`ListView.separated` is also lazy and is fine for long lists."

### B2. Obfuscation and string literals
`security.md` §8.1: remove "string literals" from the list; add "Obfuscation does not hide string
literals — never put secrets in strings."

### B3. iOS screen capture
Standard §15.2 and `security.md` §10 iOS: say what iOS allows:
- hide the app-switcher snapshot with the overlay (as now);
- iOS cannot block screenshots; the app MAY react to `UIApplication.userDidTakeScreenshotNotification`;
- detect screen recording / mirroring (`UIScreen.isCaptured` or the scene's capture state) and
  hide sensitive content while it is on.
Rename the rule to "Screen-capture protection (Android: block; iOS: hide and detect)".

### B4. OWASP M3 name
Standard §15.3 and `security.md` §12: "Insecure Authentication/Authorization".

### B5. MSIX version mapping
Standard §5.5.1 and `platform_store_readiness.md` §4.1: define the mapping. Default:
`msix_version = X.Y.Z.0` for the Store (revision must be 0), so every Store release MUST raise
`X.Y.Z`; a build-only change (`+N`) is not enough for a Store upload. Direct-download MSIX MAY use
`X.Y.Z.N`, but only if `N` fits in 0–65535. Check the current Partner Center rule before release.

### B6. Stale comments in `AppFlavorConfig`
Standard §5.2: comments say "passed via `--dart-define` on platforms without `--flavor`" and
"injected by Flutter whenever `--flavor` is passed".

### B7. `formattingLocale` locale format — (verify first)
**Check:** `intl` source/docs for `DateFormat.localeExists` and the key format (`en_US` vs
`en-US`), with a scratch test outside the repo.
**Fix if confirmed:** use `Intl.canonicalizedLocale(locale.toString())`, then try the language
code alone, then the template language.

### B8. `proguard-rules.pro` listed but optional — (verify first)
**Check:** build a scratch app outside the repo with `proguardFiles(..., "proguard-rules.pro")`
and no such file. Stop the Gradle daemons afterwards (global rule).
**Fix if it fails:** in flavors guide Step 4, either remove the `proguardFiles(...)` line
(Flutter adds the file itself) or say "create the file (it may be empty) if you keep this line",
and remove "harmless".

### B9. Linux CMake flavor variable scope — (verify first)
**Check:** the Flutter Linux template's `flutter/CMakeLists.txt` — is `FLUTTER_APP_FLAVOR`
visible in the parent scope after `add_subdirectory(${FLUTTER_MANAGED_DIR})`?
**Fix if not visible:** read the flavor from
`flutter/ephemeral/generated_config.cmake` with `include()` in the top-level file, or use the
`APP_ID_SUFFIX` environment way for both cases.

### B10. Contrast tool link
§7.7: replace the Material "how the system works" link with a real checker (keep WebAIM; drop the
Material link or describe it as background reading).

---

## C. Gaps

### C1. Web-safe samples
- Standard §5.5 samples: guard `Platform.*` with `if (!kIsWeb && ...)` and import
  `package:flutter/foundation.dart`.
- Standard §14.2 `AppLogger`: file output only when not Web (`kIsWeb` → console only); note that
  `dart:io` file APIs do not work on Web.
- §17.5: add `web:` icon generation (favicon and `web/icons/`) when Web is declared.
- `security.md` §1: add `Web` to the platform list. `architecture.md` §15 platform-differences
  table: add a Web column.

### C2. Android developer verification
`platform_store_readiness.md` §2 and `release_process.md` §12 (Direct APK row): note that Google
requires apps on certified Android devices to come from verified developers, also outside Play,
rolling out by country. Tell the reader to check the current status on Google's Android developer
site before a direct-APK release. No dates as rules.

### C3. EU trader status
`platform_store_readiness.md` §2.5 and §3.1: add "EU trader status declared (Digital Services
Act) if the app is distributed in the EU". Check the current console wording.

### C4. "Security impact" line in templates
CLAUDE/AGENTS templates, workflow rule 1: plans that touch network, storage, sign-in,
permissions, deep links, WebViews, platform folders or dependencies include a "Security impact"
line (standard §22.2). Keep both templates identical.

### C5. Parity test and display rows
Standard §8.7 parity test: fail when a known display row (`author`, `license`, and any key not
in an allow list of technical keys such as `email`, `website`, `privacyPolicy`) is a plain string
in a multi-language app.

### C6. Template details
- `security.md` §1: add `Production App Extension` to the profiles list.
- `profiles/example_en_ml_sa_profile.md` §0.1: add the "Other third-party SDKs that send data"
  row (`none`).
- CLAUDE/AGENTS templates: replace the `allowBackup="false" must remain` example with "backup and
  device-transfer rules stay as recorded in `docs/security.md` §10".

### C7. Git tag for every platform
`release_process.md` §11, §11A, §11B, §11C: add the same last step as iOS — "Tag the release
(`v<X.Y.Z>`) unless another platform already tagged it."

---

## Not in this plan

- The 16 KB deadline and the dated version snapshots stay as they are (they are dated "last
  checked" boxes, not rules).
- No change to the CLAUDE.md / AGENTS.md two-copy design.

## Verification

- Search the repo (outside `plans/` and `change_log/`) for old wording: `apk/prod/release`,
  `sections 9–11B`, `§11B Linux).`, `OpenDevin`, `Insecure Authentication |`,
  `required under the Sensitive Data Extension`, `string literals from the binary`.
- Copy changed Dart samples into a scratch app outside the repo and run `flutter analyze`.
- Confirm CLAUDE/AGENTS templates still differ only in tool-specific wording.
- Record the result of each (verify first) item in the change log.

**Security impact:** documentation only; it tightens the OWASP and iOS screen-capture rules and
weakens no control.
