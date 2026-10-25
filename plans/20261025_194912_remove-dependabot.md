# Remove Dependabot from the guidelines

**Status:** completed

## The issue

The guidelines say every project MUST have `.github/dependabot.yml` (engineering standard
§19.2), and the CI rule is "checks + Dependabot only". The user does not want Dependabot.
It makes many small pull requests and bumps one Flutter package at a time, which often does
not work well. So new projects should not get it.

Manual dependency updates are already covered by §16.6 (`flutter pub outdated` monthly and
before each release), so nothing new is needed to replace it.

## Files to change

1. `flutter_project_engineering_standard.md`
2. `AGENTS_MD_GUIDELINE.md`
3. `CLAUDE_MD_GUIDELINE.md`
4. `AI_AGENT_START_HERE.md`
5. `docs/flutter_project_engineering_standard_README.md`
6. `flutter_build_flavors_guide.md`
7. `guideline.md`
8. `release_process.md`
9. `PROJECT_PROFILE_TEMPLATE.md`
10. `profiles/example_en_ml_sa_profile.md`

Not changed: old files in `plans/` and `change_log/`. They are history.

## The fix

### 1. `flutter_project_engineering_standard.md`

- Line 174 (folder tree): `.github/` comment becomes `workflows/ci.yml (checks only, §19)`.
- §19 title: `CI Standard (Checks And Dependabot Only)` → `CI Standard (Checks Only)`.
- §19 intro: CI does one thing, the analyze/test checks (19.1). Dependencies are updated by
  hand (19.2). Nothing else (19.3).
- §19.1 example: remove the comment `# Dependabot keeps action versions current (19.2)`.
- §19.2: replace the whole Dependabot section (rules + `dependabot.yml` example) with a short
  section **"19.2 Dependency Updates (Manual)"**:
  - Do not add `.github/dependabot.yml` or any other bot that opens update pull requests.
  - Update packages by hand, following §16.6 (`flutter pub outdated`, major versions one at
    a time, plan → approve → log).
  - When you update packages, also check the versions of the GitHub Actions in
    `.github/workflows/` (for example `actions/checkout`) and update them by hand.
  - GitHub's security alerts (repository setting) MAY stay on; they do not need a file in
    the repository.
- §19.3 last bullet: `besides the 19.1 checks and Dependabot` → `besides the 19.1 checks`.
- Section numbers stay the same (19.3 and 19.4 keep their numbers), so links in other files
  do not break.

### 2 and 3. `AGENTS_MD_GUIDELINE.md`, `CLAUDE_MD_GUIDELINE.md`

- Workflow rule 5: `CI is checks + Dependabot only` → `CI is checks only`. Remove the
  `plus Dependabot (...)` part. Add: no Dependabot or other update bots; update dependencies
  by hand.
- Summary line: `checks-and-Dependabot-only CI rule` → `checks-only CI rule`.
- Checklist: `The CI rule (checks + Dependabot only)` → `The CI rule (checks only)`.

### 4. `AI_AGENT_START_HERE.md`

- Line 165: `runs only the analyze/test checks (§19.1) and Dependabot (§19.2)` →
  `runs only the analyze/test checks (§19.1). No Dependabot; dependencies are updated by hand (§19.2).`
- Line 199: `runs only the checks and Dependabot` → `runs only the checks`.

### 5. `docs/flutter_project_engineering_standard_README.md`

- Line 18: `CI that runs only analyze/test checks and Dependabot` → `CI that runs only analyze/test checks (dependencies updated by hand, no Dependabot)`.
- Line 61: remove `and .github/dependabot.yml`.
- Line 71: heading → `Set up CI with checks only`; replace the Section 19.2 part with
  `Section 19.2: update dependencies by hand, no Dependabot`.

### 6–10. Short mentions

Change `checks and Dependabot` / `analyze/test and Dependabot only` to `checks only` /
`analyze/test only` in:
- `flutter_build_flavors_guide.md` line 256,
- `guideline.md` line 759,
- `release_process.md` line 280,
- `PROJECT_PROFILE_TEMPLATE.md` line 129,
- `profiles/example_en_ml_sa_profile.md` line 156.

Re-wrap lines where needed to keep the file's line width.

### Check after the change

`grep -ri dependabot` outside `plans/` and `change_log/` should only find the new §19.2
"do not add" text and the matching short rules.

### Note for existing projects

Projects that already have `.github/dependabot.yml` keep it until someone removes it in that
project (its own plan). This change only updates the guidelines.
