# Change log: Fix the conflicts, wrong facts and gaps from the full review

Implements: [plans/20261027_194447_review-conflicts-errors-gaps.md](../plans/20261027_194447_review-conflicts-errors-gaps.md)

No section was renumbered. No Flutter version numbers were added.

## Results of the "verify first" items

| Item | Check | Result |
|---|---|---|
| B7 `formattingLocale` | Read `DateFormat.localeExists` in the `intl` source; ran a test in a scratch app outside the repo | **Confirmed wrong.** It looks up the exact key, and keys use `en_US`, not `en-US`. Fixed. |
| B8 `proguard-rules.pro` | Built a release APK in a scratch app with a `proguardFiles(...)` line and no such file | **Confirmed wrong.** The build fails: "Supplied proguard configuration does not exist". Fixed. Gradle daemons were stopped afterwards. |
| B9 Linux CMake flavor variable | Read the Flutter tool code that writes `generated_config.cmake` | **Text was right.** The tool writes `FLUTTER_APP_FLAVOR` with `PARENT_SCOPE`, so the top-level file sees it. No change. |

## A. Conflicts

| # | Change | Files |
|---|---|---|
| A1 | The short `security.md` form now includes §12 (OWASP) when the Production App Extension applies. Profile tables name the OWASP sign-off in the Production row. §14 evidence line and standard §23.2 now cover Production. | `security.md`, `DOCS_FOLDER_GUIDELINE.md` §6, `README.md`, `GUIDELINES_MANIFEST.md`, `release_process.md` §14, standard §23.2, `docs/security_README.md` |
| A2 | APK paths now use `build/app/outputs/flutter-apk/…` (bash and PowerShell), with a note that the build log prints the exact names and that no-flavor apps drop `-prod`. The merged-manifest path in `architecture.md` no longer names one fixed folder. | `release_process.md` §6.4, §6.6, §9C; `guideline.md` §2.4; `architecture.md` §7 |
| A3 | `AGENTS.md` intro no longer says Gemini reads it automatically; tools with their own file name are pointed at `AGENTS.md` (`AI_AGENT_START_HERE.md` §4 step 3). "OpenDevin" → "OpenHands". | `AGENTS_MD_GUIDELINE.md` |
| A4 | §11C Web added to the release-step lists. | `AI_AGENT_START_HERE.md` §4, `release_process.md` §5, `docs/release_process_README.md` |
| A5 | §1.8: single-value places (Windows `Runner.rc`, MSIX `display_name`, Snap `title`) use the template-language name; on iOS/macOS prod builds MUST show translated names, dev builds may lose the suffix in translated languages. Flavors guide explains the trade-off and the build-script option. | `guideline.md` §1.8, `flutter_build_flavors_guide.md` |
| A6 | `cupertino_ui`: needed if Cupertino widgets are used **or** the 8.3.1 fallback delegates exist (see "Changes from the plan"). Migration tests start at the oldest installed schema version. "Zero warnings" → "no new warnings (zero is the goal)". §17.1 icon row allows icon fonts. | standard §6.1, §8.1, §8.3.1, §17.1, §18.2; `release_process.md` §8; CLAUDE/AGENTS templates |

## B. Wrong facts

- B1 §10.3: `ListView.separated` is lazy and fine for long lists.
- B2 `security.md` §8.1: obfuscation does not hide string literals.
- B3 standard §15.2 and `security.md` §10: "Screen-capture protection" — Android blocks with
  `FLAG_SECURE`; iOS hides the app-switcher snapshot, hides content while the screen is captured,
  and may react to the screenshot notification.
- B4 OWASP M3 renamed "Insecure Authentication/Authorization" (standard §15.3, `security.md` §12).
- B5 MSIX version mapping `X.Y.Z+N` → `X.Y.Z.0` for the Store (standard §5.5.1,
  `platform_store_readiness.md` §4.2).
- B6 `AppFlavorConfig` comments updated (standard §5.2).
- B7 `formattingLocale` uses `Intl.canonicalizedLocale`, then the language code, then the
  template language; note on loaded date symbols (standard §8.3.2).
- B8 Flavors guide Step 4 no longer lists `proguardFiles(...)`; the ProGuard section says a
  named file must exist.
- B10 §7.7: WebAIM checker kept; the Material page is described as background reading.

## C. Gaps

- C1 `kIsWeb` guards in the §5.5 sqflite and window samples and in `AppLogger.init()`; Web icons
  in §17.5; Web in `security.md` §1 and the `architecture.md` platform-differences table.
- C2 New `platform_store_readiness.md` §2.5A (Android developer verification for direct APKs);
  Direct APK row in `release_process.md` §12 points to it.
- C3 EU trader status in `platform_store_readiness.md` §2.5 and §3.1.
- C4 "Security impact" line added to workflow rule 1 in both CLAUDE/AGENTS templates.
- C5 Parity test fails when a display row in `app_config.json` is a plain string in a
  multi-language app (technical keys `email`, `website`, `privacyPolicy` excepted).
- C6 `security.md` §1 lists the Production profile; example profile has the third-party SDK row;
  the `allowBackup` template example now points to `docs/security.md` §10.
- C7 Tag step added to the Windows, macOS, Linux and Web release steps.

## Changes from the plan

- **A6 `cupertino_ui`:** the plan said to mark the Cupertino fallback delegate "only if Cupertino
  is used". On iOS/macOS, Material's adaptive text-selection menu also reads
  `CupertinoLocalizations`, so dropping the fallback could break a fallback language. The
  Cupertino fallback stays, and `cupertino_ui` is needed when the 8.3.1 fallback delegates exist.
- **B10:** the unverified `color_contrast_checker` package name was removed as well.
- **B8:** the `isMinifyEnabled = true` line was also removed from Step 4, because the Flutter
  Gradle plugin turns R8 on itself.
- **B9:** dropped (text was right).
- `docs/flutter_project_engineering_standard_README.md` needed no change.

## Checks done

- Searched the repository (outside `plans/` and `change_log/`) for the old wording:
  `apk/prod/release`, `apk\prod\release`, `9–11B`, `11B Linux).`, `OpenDevin`,
  `Insecure Authentication |`, `required under the Sensitive Data Extension`,
  `string literals from the binary`, `optional and harmless`, `toLanguageTag`,
  `color_contrast_checker`, `keep in step with pubspec`, `zero warnings`. No matches.
- Copied the changed Dart samples (`formattingLocale`, `AppLogger`, the desktop `main` samples)
  into a scratch app outside the repository: `flutter analyze` reports no issues; a test of
  `formattingLocale` (`en_US`, `ml`, `sa` → `en`) passes.
- The CLAUDE.md and AGENTS.md templates still differ only in tool-specific wording.
- After the Gradle test build: `gradlew --stop` ran, and no `GradleDaemon` or
  `KotlinCompileDaemon` process is left.
