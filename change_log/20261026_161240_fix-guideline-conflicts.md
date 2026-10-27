# Change log: Fix the conflicts between guideline files

Implements: [plans/20261026_160720_fix-guideline-conflicts.md](../plans/20261026_160720_fix-guideline-conflicts.md)

Decisions used (the plan's defaults, approved by the user): D1 CI required on GitHub; D2 at least
two keystore backups, one offline; D3 Web stays declarable with its own short release section;
D4 privacy-policy row is a MUST for public store apps.

## Main conflicts

| # | Fix | Files |
|---|---|---|
| C1 | "Update this file / add the term here" replaced with "tell the user" (read-only submodule). Missing glossary terms go to the project's `docs/glossary.md` meanwhile. | `platform_store_readiness.md`, `docs/platform_store_readiness_README.md`, `language_packs/sanskrit_malayalam.md` |
| C2 | Size analysis is optional and runs before the hardened build. | `docs/release_process_README.md` |
| C3 | Obfuscation rules name the native platforms. New `release_process.md` §11C Web Release Steps, Web row in §1 and §12, Web note in the store gates, Web row in the architecture table. | standard §15.1, `guideline.md` §4, `release_process.md`, `platform_store_readiness.md`, `AI_AGENT_START_HERE.md` §6, `architecture.md` |
| C4 | Git tag is `v<X.Y.Z>` (no `+build`), once per release. | `release_process.md`, `docs/release_process_README.md` |
| C5 | One backup rule in `guideline.md` §2.3; other files point to it. | `guideline.md`, `flutter_build_flavors_guide.md`, `release_process.md`, CLAUDE/AGENTS guides |
| C6 | CI required on GitHub; other hosts run the same checks in their CI or locally. Profile row and intake question updated. | standard §19.1, `PROJECT_PROFILE_TEMPLATE.md`, `AI_AGENT_START_HERE.md` |
| C7 | `app_config.json` About values named as the one allowed second source of screen text. | standard §8.2, `guideline.md` §3 and §4, CLAUDE/AGENTS guides |
| C8 | New `guideline.md` §1.8 "The app name". `appTitle` → `titleHome`; `AppFlavorConfig.appName` → `nameSuffix`; window title from `config.appName`; Android app name in per-language `strings.xml`, not `resValue`. | `guideline.md`, standard §5.2, §5.5, §8.2, `flutter_build_flavors_guide.md` |
| C9 | Animation advice now matches standard §6.5. | `docs/flutter_project_engineering_standard_README.md` |
| C10 | Offline-only wording made conditional; "before coding" table applies in full only under the Sensitive Data Extension. | `docs/security_README.md` |
| C11 | Missing profile: run the intake and get approval before creating it. | standard §22.1 |
| C12 | `flutter create ... --project-name my_app ... .` in the project root. | standard §3.3 |

## Smaller inconsistencies

- S1: `PROJECT_PROFILE.md` §3 is the one home for minimum OS (architecture §15/§19 and store
  gate §2.2 now point there).
- S2: Short `security.md` = §10, §11, §13 in `DOCS_FOLDER_GUIDELINE.md` §6.
- S3: `GUIDELINES_MANIFEST.md` described as the copied pointer file (`README.md`, manifest,
  `DOCS_FOLDER_GUIDELINE.md` §2).
- S4: `AppLogger.d` → `AppLogger.debug`.
- S5: `core/logging/` added to the `lib/` layout; CLAUDE/AGENTS layout uses `core/config/`;
  token file path per tier.
- S6: ARB prefix exceptions named; tooltip examples use `tooltip…` keys.
- S7: Sanskrit button rule allows one verb plus one indeclinable/object word.
- S8: Badge rows in the language pack marked as example wording.
- S9: Note for projects that also declare Hindi.
- S10: Language-split rule made conditional in the standard explainer.
- S11: App Store privacy policy "every app"; About privacy row MUST for public store apps.
- S12: FLAG_SECURE on every screen that shows sensitive data.
- S13: Coverage % example removed from the CLAUDE/AGENTS templates.
- S14: Upgrade step logs to `change_log/` (and `CHANGELOG.md` if kept).
- S15: Thick style marked as a temporary migration state.
- S16: `implementation_progress.md` is Living with `**Last updated:**`; old alternative names
  moved to a note.
- S17: Standard explainer CI line updated.

## Checks done

- Searched the repository for the old wording (`AppLogger.d(`, `cloud storage + physical`,
  `appTitle`, `post-build size`, `git tag v<version>`, `resValue("string"`, `update this file (or`,
  `Coverage target`, `setTitle('`, and others). No matches outside `plans/` and `change_log/`.
- The CLAUDE.md and AGENTS.md templates differ only in the tool-specific wording.
