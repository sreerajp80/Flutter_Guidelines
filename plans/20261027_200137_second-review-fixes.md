# Plan: Fix the conflicts, wrong facts and gaps from the second full review

**Status:** completed

Follows the review done after [20261027_194447_review-conflicts-errors-gaps.md](20261027_194447_review-conflicts-errors-gaps.md).
No section is renumbered. No Flutter version numbers are added (the guidelines stay
version-agnostic).

## Files to change

- `DOCS_FOLDER_GUIDELINE.md`
- `language_packs/sanskrit_malayalam.md`
- `security.md`
- `release_process.md`
- `platform_store_readiness.md`
- `flutter_build_flavors_guide.md`
- `flutter_project_engineering_standard.md`
- `guideline.md`
- `architecture.md`
- `AGENTS_MD_GUIDELINE.md` (only if the check in D7 finds the tool list wrong)
- `docs/security_README.md` (only if A1 changes wording it repeats)

---

## A. Conflicts

### A1. When `security.md` / `release_process.md` are filled in full

**Issue.** `DOCS_FOLDER_GUIDELINE.md` §6 says both files are filled in full "when the Sensitive
Data Extension **or** the Production App Extension applies". `README.md`, the manifest and
`security.md` say: Production → full `release_process.md` plus the OWASP sign-off; Sensitive
Data → full `security.md`. §6 then describes a short `security.md` for Production apps, so it
also contradicts itself.

**Fix.** Rewrite the paragraph: `release_process.md` is filled in full under the Production App
Extension; `security.md` is filled in full under the Sensitive Data Extension. Keep the existing
list of the minimum sections for the short forms (including §12 OWASP when Production applies).
Check `docs/security_README.md` uses the same wording.

### A2. Sanskrit word for "audio"

**Issue.** Language pack §4.1 says "Use `ध्वनिः` for audio/sound", but the glossary has
Audio = `श्रव्यम्` and Sound = `ध्वनिः`.

**Fix.** Keep the glossary (it has separate terms for the two meanings). Change the §4.1 line to:
use `ध्वनिः` for Sound and `श्रव्यम्` for Audio (audio content), never `शब्दः`, which means Word.
Mark the change "needs native-reader review" in the change log (pack §4.4 review rule).

---

## B. Wrong facts

### B1. iOS Keychain purge after reinstall (`security.md` §13)

**Issue.** "Compare stored install UUID against a newly generated one" cannot work.

**Fix.** Describe the working method: keep a first-run marker in `SharedPreferences` /
`UserDefaults` (these are deleted on uninstall, Keychain items are not). At startup, if the
marker is missing, delete the app's Keychain items, then write the marker. Use the same wording
for the Windows, macOS and Linux lines that point to "the same first-run detection".

### B2. Uploading to App Store Connect

**Issue.** `release_process.md` §10 step 8 says "Upload the `.ipa` with Xcode Organizer".
Organizer uploads the archive; the `.ipa` goes through Transporter.

**Fix.** `release_process.md` §10 step 8 and `platform_store_readiness.md` §3.2: upload the
`.ipa` from `build/ios/ipa/` with Apple's Transporter app, **or** open
`build/ios/archive/Runner.xcarchive` in Xcode Organizer and use Distribute App.

### B3. Cleartext traffic default (`release_process.md` §6.5)

**Issue.** "Enforced by default on Android 9+" — the default depends on the app's target SDK.

**Fix.** "Cleartext is off by default for apps that target API 28 (Android 9) or higher; set it
explicitly anyway."

### B4. 16 KB page-size date (standard §5.3.1)

**Issue.** The text says updates without 16 KB support cannot be released "from February 1,
2027". I could not confirm this; sources found mention Nov 1, 2025 and an extension to about
May 31, 2026.

**Fix.** During implementation, read Android's page-size guide
(`developer.android.com/guide/practices/page-sizes`). If it confirms a date, keep or correct it
with "when last checked (2026-10)". If it cannot be confirmed, remove the date and keep only the
"check the current enforcement date before each release" rule. Report which case applied in the
change log.

---

## C. Smaller mismatches

| # | Where | Issue | Fix |
|---|---|---|---|
| C1 | `platform_store_readiness.md` §5.3 | Comment says Apple advises against `--deep`, command uses `--deep` | Remove `--deep` from the command; keep the comment that nested frameworks are signed first, and that the Xcode export path is the safer way |
| C2 | Flavors guide, "Flavor-Specific Resource Files" | Dev `values-<code>/strings.xml` comment does not add " Dev" | Comment: "app_name in each other declared language, plus the dev suffix" (matches `guideline.md` §1.8) |
| C3 | Flavors guide, MSIX `msix_config` example | `languages: en-us` with no note | Add "one entry per declared language" like standard §5.5.1 |
| C4 | Flavors guide, "Notes for New Projects" → iOS | Says "xcconfig files per flavor"; the iOS steps use Build Settings | Change to "build configurations per flavor with the bundle id and display name set in Build Settings (or xcconfig files)" |
| C5 | `release_process.md` §9 B (bash) | `VERSION=` line lacks `tr -d '\r'` | Add it, same as §9 A |
| C6 | `release_process.md` §7 | "Signing config location: local environment variables / secrets manager" | "`android/key.properties` on the release machine (git-ignored, `guideline.md` §2), or a local secrets manager — never CI" |
| C7 | Standard §7.7 | Uses `debugPaintSizeEnabled` for hit areas; §7.1 uses `debugPaintPointersEnabled` | Use `debugPaintPointersEnabled` for hit areas in both; `debugPaintSizeEnabled` only for layout bounds |
| C8 | Language pack §4.2 | Glossary uses `ഓഡിയോ`, `വീഡിയോ`, `ഇമെയിൽ`, `ഫോൺ`, `ലോഡുചെയ്യുന്നു`, not on the accepted-loan list | Add `ഓഡിയോ`, `വീഡിയോ`, `ഇമെയിൽ`, `ഫോൺ`, `ലോഡ്` to the established-loanword list (mark for native-reader review) |
| C9 | `security.md` §12 M7 | "ProGuard applied" | "R8 / ProGuard rules verified (Android)" |

---

## D. Gaps

| # | Where | Gap | Fix |
|---|---|---|---|
| D1 | `guideline.md` §1.8 | No Web entry; Snap `title` only in the single-value list; Linux `.desktop` is not made by `flutter create` | Add Web: `<title>` in `web/index.html` and `name` / `short_name` in `web/manifest.json` (single value, template language). Add Snap `title` to the OS list. Note that the Linux `.desktop` file is created for packaging |
| D2 | `release_process.md` §8, §9 (steps and commands), §10–§11C step text | `build_runner` run unconditionally | Mark it "only if the project uses code generation" (as standard §19.1) |
| D3 | `platform_store_readiness.md` §3.3 | Privacy manifest must be in the Runner target | Add: "add the file to the Runner target in Xcode (Target Membership); a file only in the folder is not bundled" |
| D4 | `architecture.md` §6 | Lifecycle table lacks `hidden` | Add a `hidden` row (window hidden / minimised on desktop; before `paused` on mobile) |
| D5 | `security.md` §10 iOS, standard §15.2 | Only `UIScreen.isCaptured` named | Name `UITraitCollection.sceneCaptureState` (UIScene apps) first and `UIScreen.isCaptured` for older code; say capture detection is best-effort |
| D6 | Standard §20.4 | No `.env*` line; Option B list lacks `*.gr.dart` (§12.2) | Add `.env` / `.env.*` under a "Local secrets" heading; add `*.gr.dart` to the §12.2 Option B list |
| D7 | `AGENTS_MD_GUIDELINE.md` intro and template banner | Names Antigravity and OpenHands as reading `AGENTS.md`; not verified | Check each tool's current docs during implementation. Keep the names that are confirmed; for any not confirmed, drop the name and rely on the existing "check the tool's current docs" rule. Report the result in the change log |

---

## Checks after the change

- Search the repository (outside `plans/` and `change_log/`) for the old wording:
  `install UUID`, `Upload the .ipa with Xcode Organizer`, `enforced by default on Android 9+`,
  `ProGuard applied`, `--deep --options`, `Use \`ध्वनिः\` for audio`, `or the Production App
  Extension applies`. No matches expected.
- `CLAUDE_MD_GUIDELINE.md` and `AGENTS_MD_GUIDELINE.md` still differ only in tool-specific wording.
- No Dart or build sample changes need a test build (only comments and prose change), so no
  Gradle build is run.
- Write a change log in `change_log/` that lists each item, the result of B4 and D7, and the
  language-pack terms marked "needs native-reader review".
