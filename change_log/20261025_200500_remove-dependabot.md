# Remove Dependabot from the guidelines

Implements plan: `plans/20261025_194912_remove-dependabot.md`

## What changed

Dependabot is no longer part of the guidelines. CI is now "checks only", and dependencies are
updated by hand.

- `flutter_project_engineering_standard.md`
  - Folder tree: `.github/` no longer lists `dependabot.yml`.
  - §19 title is now "CI Standard (Checks Only)"; the intro says CI runs only the 19.1 checks.
  - §19.1 example: the `actions/checkout` comment now says to update action versions by hand.
  - §19.2 is replaced by "Dependency Updates (Manual)": do not add `.github/dependabot.yml` or
    any other update bot; update packages by hand following §16.6; update GitHub Action
    versions by hand at the same time; GitHub security alerts may stay on.
  - §19.3: the last bullet no longer mentions Dependabot.
  - Section numbers did not change, so links to §19.3 and §19.4 still work.
- `AGENTS_MD_GUIDELINE.md`, `CLAUDE_MD_GUIDELINE.md`
  - Workflow rule 5 is now "CI is checks only", and says no Dependabot or other update bot.
  - Summary line and checklist item updated to match.
- `AI_AGENT_START_HERE.md`: CI step and final checklist item updated.
- `docs/flutter_project_engineering_standard_README.md`: summary, quick-start table row and
  §19 note updated.
- `flutter_build_flavors_guide.md`, `guideline.md`, `release_process.md`,
  `PROJECT_PROFILE_TEMPLATE.md`, `profiles/example_en_ml_sa_profile.md`: short mentions changed
  to "checks only".

## Check

A search for "dependabot" outside `plans/` and `change_log/` now finds only the new text that
says not to use it.

## Not changed

- Old plans and change logs (history).
- Existing projects that already have `.github/dependabot.yml`. They keep it until it is
  removed in that project as its own change.
