# Change log: Fix the findings from the second full review

Implements: [plans/20261027_200137_second-review-fixes.md](../plans/20261027_200137_second-review-fixes.md)

No section was renumbered. No Flutter version numbers were added.

## Results of the "check first" items

| Item | Check | Result |
|---|---|---|
| B4 16 KB date | Read Android's page-size guide (`developer.android.com/guide/practices/page-sizes`) | **Text was right.** The guide says that from February 1, 2027, app updates without 16 KB support cannot be released. No change. |
| D7 agent tool names | Read the AGENTS.md site's list of supported tools (`agents.md`) | Codex, Cursor, Windsurf and GitHub Copilot's coding agent are listed; **Antigravity and OpenHands are not.** Those two names were removed. The existing "check the tool's current docs" rule stays. |

## A. Conflicts

- A1 `DOCS_FOLDER_GUIDELINE.md` §6: `release_process.md` is in full under the Production App
  Extension; `security.md` is in full under the Sensitive Data Extension. The short-form section
  list is unchanged. `docs/security_README.md` already matched; no change there.
- A2 Language pack §4.1: `ध्वनिः` = Sound, `श्रव्यम्` = Audio, never `शब्दः` (Word). Now matches
  the glossary.

## B. Wrong facts

- B1 `security.md` §13: iOS Keychain purge uses first-run detection — a marker in
  `SharedPreferences` / `UserDefaults` (deleted on uninstall); if it is missing at startup,
  delete the Keychain items, then write the marker.
- B2 `release_process.md` §10 step 8 and `platform_store_readiness.md` §3.2: upload the `.ipa`
  with Transporter, or the `Runner.xcarchive` through Xcode Organizer → Distribute App.
- B3 `release_process.md` §6.5: cleartext is off by default for apps that target API 28 or
  higher; set it explicitly anyway.
- B4: no change (see above).

## C. Mismatches

- C1 `platform_store_readiness.md` §5.3: `--deep` removed from the `codesign` command; the
  comment says to sign nested frameworks first.
- C2 Flavors guide: dev `values-<code>/strings.xml` carries the dev suffix too.
- C3 Flavors guide: `msix_config` `languages` comment — one entry per declared language.
- C4 Flavors guide "Notes for New Projects" (iOS): per-flavor build configurations, with the
  bundle id and display name in Build Settings (or xcconfig files).
- C5 `release_process.md` §9 B: bash `VERSION` line strips `\r`.
- C6 `release_process.md` §7: signing config is `android/key.properties` on the release machine
  or a local secrets manager, never CI.
- C7 Standard §7.7: hit areas checked with `debugPaintPointersEnabled`.
- C8 Language pack §4.2: `ഓഡിയോ`, `വീഡിയോ`, `ഇമെയിൽ`, `ഫോൺ`, `ലോഡ്` added to the accepted loanwords.
- C9 `security.md` §12 M7: "R8 / ProGuard rules verified (Android)".

## D. Gaps

- D1 `guideline.md` §1.8: Snap `title` and Web (`<title>`, `manifest.json` `name` /
  `short_name`) added; they are single-value places. Note that `flutter create` writes no Linux
  `.desktop` file.
- D2 `release_process.md` §8, §9 and the step 3 lines of §10–§11C: code generation runs only if
  the project uses it.
- D3 `platform_store_readiness.md` §3.3: `PrivacyInfo.xcprivacy` must be in the Runner target.
- D4 `architecture.md` §6: `hidden` row added.
- D5 `security.md` §10 (iOS) and standard §15.2: `UITraitCollection.sceneCaptureState` first,
  `UIScreen.isCaptured` for older code; detection is best-effort.
- D6 Standard §20.4: `.env` / `.env.*` ignored; §12.2 Option B lists `*.gr.dart`.
- D7 `AGENTS_MD_GUIDELINE.md`: see above.

## Needs native-reader review

- Sanskrit: the §4.1 rule `ध्वनिः` (Sound) / `श्रव्यम्` (Audio).
- Malayalam: the new accepted loanwords `ഓഡിയോ`, `വീഡിയോ`, `ഇമെയിൽ`, `ഫോൺ`, `ലോഡ്`.

## Checks done

- Searched the repository (outside `plans/` and `change_log/`) for the old wording:
  `install UUID`, `Upload the .ipa with Xcode Organizer`, `enforced by default on Android 9`,
  `ProGuard applied`, `--deep --options`, `for audio/sound`,
  `or the Production App Extension applies`, `Antigravity`, `OpenHands`. No matches.
- The CLAUDE.md and AGENTS.md guidelines still differ only in tool-specific wording.
- Only prose and comments changed, so no build was run and no Gradle daemon was started.
