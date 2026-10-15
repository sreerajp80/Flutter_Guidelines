# Change log — Agent intake for new projects

Implements plan: `plans/20261015_154004_agent-intake-for-new-projects.md`

## Why

New apps are built by AI agents (Claude Code, Gemini, Codex) from a short user brief. The agent
must collect every other fact from the user and never guess. The guidelines did not say what to
ask, how to ask, or how to record "you decide" and "I don't know yet".

## What changed

### `AI_AGENT_START_HERE.md`

- New **§0 "Before anything: check the folder"**: read-only checks — git repository, guidelines
  submodule present (only the user adds it), new or existing app, profile placeholders,
  `flutter --version` / `flutter doctor`.
- New **§3 "New app — intake"**:
  - §3.1 rules: record what the user said, ask the rest in one numbered list, never fill a gap
    silently, "you decide" needs a clear yes for permanent or public values, mark unknowns as
    `TODO — needed before <gate>`, ask in rounds, no private details, approve the profile first.
  - §3.2 question table: product, identity, platforms and stores, languages, data and privacy,
    look and feel, tools and workflow, technical defaults.
  - §3.3 what the answers decide: profiles, store gates, language packs, instruction files.
- Old §3 → **§4 bootstrap**, now starting only after the approved intake:
  - `flutter create` now runs in the project root with `--project-name` and a final `.`, so the
    app is not created in a subfolder next to `docs/guidelines/`.
  - Step 3: `CLAUDE.md` and `AGENTS.md` always; for an agent that reads another file by default
    (for example Gemini), a pointer file or the tool's setting, so there is only one copy of the
    rules. The agent must check the tool's current docs for the file name.
  - Step 7: features follow the product brief order; a new fact (permission, backend, language)
    means ask the user and update the profile first.
- Old §4 → **§5 adoption**: runs the intake for gaps the code cannot answer.
- Old §5 → **§6 ready for the stores**: new check that the profile has no placeholders and no
  `TODO` for this release. The last line now says to tell the user about changed store rules,
  instead of editing `platform_store_readiness.md` (an app must not edit the submodule).

### `PROJECT_PROFILE_TEMPLATE.md`

- "How to use" note: how to mark agent-proposed values `(proposed, confirmed by user)` and
  unknown values `TODO — needed before <gate>`; points to the intake.
- New **§0 Product brief** with **§0.1 Data and privacy** (network, backend, accounts, stored
  data, sensitive data, analytics, crash reporting, ads, purchases, permissions with reasons).
- §6: added brand colours, theme modes, app icon source. "Release owner" moved to §7.
- New **§7 Tools and workflow**: AI agents used, IDE, can build iOS/macOS locally, git hosting,
  CI checks, release owner.
- Existing section numbers §1–§6 are unchanged, so no outside references broke.

### `profiles/example_en_ml_sa_profile.md`

- Filled in §0, §0.1, the new §6 rows and §7 for the example offline PDF app.

### `README.md`

- Copy-paste **starter message** for a new app that works with any agent, plus the setup steps
  before it (folder, `git init`, submodule). Updated the two document descriptions.

### Small description updates (keep in step with the template)

- `GUIDELINES_MANIFEST.md`, `DOCS_FOLDER_GUIDELINE.md`, `CLAUDE_MD_GUIDELINE.md`,
  `AGENTS_MD_GUIDELINE.md`: the one-line description of `PROJECT_PROFILE.md` now includes the
  product brief and tools; the manifest and docs-folder guideline point to the intake (§3).
  (The plan named only the manifest; the other three hold the same description line.)

## Checks

- Every section referenced in the new text exists (engineering standard §1.2, §17.5, §19.1–§19.3,
  §21.1.1; `guideline.md` §1.7, §4; `DOCS_FOLDER_GUIDELINE.md` §1, §1.1, §6).
- No other file referenced `AI_AGENT_START_HERE.md` section numbers, so the renumbering broke
  nothing.
- No Flutter version numbers were added.
