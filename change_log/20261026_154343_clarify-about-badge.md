# Change Log: Make the About signature badge rules clear

Implements plan `plans/20261026_154100_clarify-about-badge.md` (option A: keep the current
Malayalam and Sanskrit texts and explain them).

## Why

Agents were confused when adding "Made with {heart} from India" in three languages. It was unclear
where the text lives, the §1.7 example showed only "by <Team>", the docs used a ❤️ emoji, and the
Malayalam and Sanskrit texts follow a different pattern from English with no explanation.

## What changed

| File | Change |
|---|---|
| `guideline.md` | §1.7: new "Where the text lives" rule (profile = decision record, ARB files = the only runtime source, never `app_config.json` or a Dart literal). New "Use `{heart}`, never an emoji" rule. Prose examples now use `{heart}`. Added a "from <Place>" ARB example. New "Translating the badge" part: the heart may follow a written word for love, and a translation may drop a word such as "made"; this is intended. §4 checklist item now names both ARB keys and the no-emoji rule. |
| `PROJECT_PROFILE_TEMPLATE.md` | §5: added a screen-reader text row and a note to use `{heart}`, never an emoji, and that the texts are copied into the ARB files. |
| `profiles/example_en_ml_sa_profile.md` | Intro uses `{heart}` instead of ❤️. Added a back-translation table for English, Malayalam and Sanskrit, with a note that the differences are intended. The texts themselves are unchanged. |
| `flutter_project_engineering_standard.md` | §8.7 parity test: the badge test now runs when the template ARB has `madeWithLove`. It requires `madeWithLove` and `madeWithLoveA11y` in every language, exactly one `{heart}`, and no heart emoji in either key. |
| `language_packs/sanskrit_malayalam.md` | About-screen table: added the badge and badge screen-reader rows, plus a note that the Malayalam/Sanskrit pattern is intended. |

## Not changed

- The `MadeWithLove` widget code.
- The Malayalam and Sanskrit badge texts.
- The CLAUDE/AGENTS templates and `release_process.md` (already correct).
- No app project was changed. Each project updates its own profile and ARB files.
