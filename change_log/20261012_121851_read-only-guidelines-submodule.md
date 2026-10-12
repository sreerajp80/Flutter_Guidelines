# Change log — the guidelines submodule is strictly read-only

Implements plan: [../plans/20261012_114117_read-only-guidelines-submodule.md](../plans/20261012_114117_read-only-guidelines-submodule.md)

## Why

The old rule only said "never edit files inside the submodule". In one project, an AI agent
fast-forwarded the submodule and left the new version uncommitted. GitHub Desktop then showed
"Submodule changes", and it looked as if the project had changed this repository. The rule now
covers every way a project can change the submodule.

## What changed

- **`DOCS_FOLDER_GUIDELINE.md` §1** — the `docs/guidelines/` bullet is now a strict, no-exceptions
  rule. A project and its AI agents must not:
  - edit, add or delete files in the submodule;
  - pull, fetch-and-merge, check out or `git submodule update` it;
  - stage or commit a new submodule version;
  - commit, branch or push inside it.

  The bullet also says:
  - If Git shows "Submodule changes", tell the user and don't touch them.
  - Only the user adds the submodule (`git submodule add`) and fills it after a fresh clone
    (`git submodule update --init`).
  - An agent that finds the folder empty or missing stops and asks.
- **`DOCS_FOLDER_GUIDELINE.md` new §1.1 "When the guidelines need a change"** — the project writes a
  note; the user changes the guidelines repository in its own session (plan, approval, change
  log, commit, push); the user then updates the submodule version in each project.
- **`GUIDELINES_MANIFEST.md`** — step 2 now says the user adds the submodule. A new step 3 says the
  submodule is read-only and only the user updates its version, with a link to §1.1. Later steps
  are renumbered 4–6.
- **`CLAUDE_MD_GUIDELINE.md` and `AGENTS_MD_GUIDELINE.md`** — the same changes in both:
  - **Template:** new item 4 in the mandatory "Workflow rules" block. It says `docs/guidelines/` is
    read-only: never edit, pull, check out or commit it or its version, and write a note for the
    user instead.
  - **"Keep verbatim in spirit" section:** now lists the read-only rule.
  - **Anti-patterns:** new line for changing the submodule from the project.
  - **Final self-check:** new box for the rule.
- **`AI_AGENT_START_HERE.md` §1** — new ground rule 6: never change the guidelines submodule.
- **`README.md`** — one sentence in "Templates and references" saying the submodule is read-only
  for the project and only the user updates its version.

## Decision on the open point

The user chose the strict option. Adding the submodule and filling it after a fresh clone are
also done only by the user.

## Not done here

Existing apps still have their old `CLAUDE.md` / `AGENTS.md`. Adding workflow rule 4 to each app
is separate work, done in that app's own session.
