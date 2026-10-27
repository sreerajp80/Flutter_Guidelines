# Plan: Make the About signature badge rules clear

**Status:** completed

## The issue

When a project follows the guidelines to show "Made with ❤ from India" on the About screen in
English, Malayalam and Sanskrit, the agent gets confused. The causes:

1. **Where the text lives is unclear.** `guideline.md` §1.7 says the text "comes only from the
   project profile… not from `app_config.json`", and then says it comes from the ARB key
   `madeWithLove`. The rest of the About screen is translated through `app_config.json` (§1.6).
   So the text can end up in the profile, the ARB files or `app_config.json`.
2. **The §1.7 example uses "by <Team>" only.** The "from <Place>" form is shown only in the
   example profile, which an agent may not read.
3. **The ❤️ emoji appears in the docs.** §1.7 opens with "Made with ❤️ by <Team>", and the example
   profile says "Made with ❤️ from India". An agent may type the emoji into the ARB file instead of
   `{heart}`, even though §1.7 says the heart must be an icon.
4. **The three example translations follow different patterns, with no explanation.**
   - English `Made with {heart} from India`: the heart stands for the word "love".
   - Malayalam `സ്നേഹത്തോടെ {heart} ഇന്ത്യയിൽ നിന്ന്`: "With love ❤ from India". The word
     "love" is written out and there is no word for "made".
   - Sanskrit `सस्नेहं निर्मितम् {heart} भारततः`: "Made with love ❤ from India". The word
     "love" is written out.
   An agent or reviewer may "fix" these differently each time.
5. **The tests check too little.** The parity test (standard §8.7) checks only that
   `madeWithLove` keeps `{heart}`, and only if the key exists. It does not check that
   `madeWithLoveA11y` exists in every language, or that no ❤ emoji is in the text.

## Files to change

| File | Change |
|---|---|
| `guideline.md` | §1.7: state clearly where the text lives, how it gets into the app, the "from <Place>" form, the no-emoji rule and the translation rule. Remove the emoji from the prose examples. |
| `profiles/example_en_ml_sa_profile.md` | Write the badge as `{heart}` in the intro, not ❤️. Add an English back-translation for each language and a note on the translation pattern. Update the Malayalam text only if you choose option B below. |
| `PROJECT_PROFILE_TEMPLATE.md` | §5: add a "Badge screen-reader text" row (the example has it, the template does not). Add a short note: "use `{heart}`, never an emoji; the agent copies these texts into the ARB files". |
| `flutter_project_engineering_standard.md` | §8.7 parity test: when `madeWithLove` exists in the template, require `madeWithLove` and `madeWithLoveA11y` in every language, require `{heart}`, and fail if a ❤ emoji character is present. |
| `language_packs/sanskrit_malayalam.md` | Add the badge row to the About-screen table, so the language pack and the example profile give the same text. |

The CLAUDE/AGENTS templates and `release_process.md` already say "only if the profile enables it".
They do not change.

## The fix in detail

### 1. One clear flow for the text (`guideline.md` §1.7)

Replace the "text comes only from the project profile" bullet with:

> **Where the text lives.** `docs/PROJECT_PROFILE.md` §5 is the *decision record*: it says whether
> the badge is on and gives the exact text for each declared language. The app reads the text
> **only from the ARB files** (`madeWithLove` and `madeWithLoveA11y` in every `app_<code>.arb`). It
> never comes from `app_config.json` and is never a literal in Dart. When the profile changes,
> update the ARB files to match.

### 2. Show both forms

Show both `Made with {heart} by <Team>` and `Made with {heart} from <Place>` in the §1.7 ARB
example. Point to the worked example profile for a three-language version.

### 3. No emoji rule

Add a bullet: the ARB text and the profile text MUST use the `{heart}` placeholder, never a ❤ or ❤️
character. In the prose, write the example as "Made with {heart} from <Place>".

### 4. Translation rule (DECISION NEEDED)

- **Option A (recommended): keep the current Malayalam and Sanskrit texts, and explain them.**
  Add to §1.7: "The heart does not have to replace a word. In some languages it reads better after
  a written word for love. That is allowed. The only rule is that `madeWithLoveA11y` must read
  naturally without the heart." Add back-translations to the example profile. This does not change
  any text a native reader already approved.
- **Option B: add the missing word "made" to Malayalam.** For example
  `ഇന്ത്യയിൽ നിന്ന് സ്നേഹത്തോടെ നിർമ്മിച്ചത് {heart}`. This must be checked by a fluent Malayalam
  reader before release, and the screen-reader text changes to match. Sanskrit already has
  "made", so it stays the same.

### 5. Stronger parity test (standard §8.7)

Replace the single `{heart}` test with one that, when the template ARB has `madeWithLove`:
- requires `madeWithLove` and `madeWithLoveA11y` in every locale's ARB,
- requires `{heart}` in every `madeWithLove`,
- fails if `madeWithLove` or `madeWithLoveA11y` contains `❤` (U+2764) or the heart emoji forms.

## Out of scope

- No change to the `MadeWithLove` widget code (it already works).
- No change to any app project. Each project then updates its own profile and ARB files.

## After the change

Write a change log in `change_log/` and set this plan's status to `completed`.
