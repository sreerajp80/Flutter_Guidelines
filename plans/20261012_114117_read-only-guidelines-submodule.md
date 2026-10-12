# Make the guidelines submodule strictly read-only for projects

**Status:** completed — see [../change_log/20261012_121851_read-only-guidelines-submodule.md](../change_log/20261012_121851_read-only-guidelines-submodule.md)

## 1. The issue

Projects add this repository as a Git submodule at `docs/guidelines/`. Today the only rule is in
`DOCS_FOLDER_GUIDELINE.md` §1: "Never edit files inside the submodule from within a project."

That is not strict enough. It only talks about *editing files*. In one project, an AI agent
fast-forwarded the submodule folder to a newer version and left that new version uncommitted.
GitHub Desktop then showed "Submodule changes", and it looked as if the project had changed this
repository.

The rule must cover every way a project can change the submodule, not just file edits. It must
also say what to do when a guideline really needs a change. And the short rule must be in every
app's `CLAUDE.md` and `AGENTS.md`, because many agents never open `DOCS_FOLDER_GUIDELINE.md`.

## 2. Files to change

| File | Change |
|---|---|
| `DOCS_FOLDER_GUIDELINE.md` | Rewrite the `docs/guidelines/` bullet in §1 as a strict rule; add a part "When the guidelines need a change". |
| `GUIDELINES_MANIFEST.md` | Add a line under "How to use this file": the submodule is read-only; only the user updates its version. |
| `CLAUDE_MD_GUIDELINE.md` | Add the rule to the mandatory template block, to §5 (sections kept in every file), to §7 anti-patterns and to the §8 self-check. |
| `AGENTS_MD_GUIDELINE.md` | The same changes (template, §6, §8, §9), so the two files stay in step. |
| `AI_AGENT_START_HERE.md` | Add the rule to §1 "Ground rules (always)". This list sums up the rules every agent must follow. |
| `README.md` | Add one short sentence in "Templates and references" that the submodule is read-only for the project. |

No other file states or sums up this rule. I searched every `.md` file for "submodule" and
`docs/guidelines`. The other hits are only paths that point to a guideline file; they need no change.

## 3. The fix in detail

### 3.1 `DOCS_FOLDER_GUIDELINE.md` §1

Replace the current `docs/guidelines/` bullet with:

> - **`docs/guidelines/`** — the shared guidelines **Git submodule**. It is **read-only** for the
>   project. A project, and any AI agent working in it, must **never change the submodule in any
>   way**. No exceptions:
>   - no editing, adding or deleting files inside `docs/guidelines/`;
>   - no `git pull`, fetch-and-merge, `git checkout`, or `git submodule update` on it;
>   - no staging or committing a new submodule version (the "pointer" — the commit of the
>     guidelines repo that the project records) in the project;
>   - no commits, branches or pushes inside `docs/guidelines/`.
>
>   The project only reads the guideline files and links to them. If `git status` or GitHub
>   Desktop shows "Submodule changes" or a new commit for `docs/guidelines`, do not stage it, do not
>   commit it and do not try to "fix" it. Tell the user.

Then add a new short part at the end of §1:

> ### 1.1 When the guidelines need a change
>
> 1. The project writes a **note**: in chat, or as a file in the project's own `plans/` folder.
>    The note says which guideline file to change, what to change, and why.
> 2. The **user** makes the change in a separate session in the guidelines repository, with its
>    own plan, approval and change log. Then the user commits and pushes it.
> 3. The **user** then updates the submodule version in each project themselves.

### 3.2 `GUIDELINES_MANIFEST.md`

Under "How to use this file", after step 2 (adding the submodule), add a new item:

> 3. **The submodule is read-only for the project.** Never edit, pull, check out or commit it or
>    its version from the project. Only the user updates its version. If a guideline needs a
>    change, see `docs/guidelines/DOCS_FOLDER_GUIDELINE.md` §1.1 "When the guidelines need a change".

The later items are renumbered (3 → 4, 4 → 5, 5 → 6).

### 3.3 `CLAUDE_MD_GUIDELINE.md` and `AGENTS_MD_GUIDELINE.md`

The rule goes where the "must carry" rules already live: the mandatory **Workflow rules** block,
which every app's file must keep inline.

- **Template, "Workflow rules (mandatory)"** — add item 4, using the exact wording you gave:

  > 4. **The guidelines submodule is read-only.** `docs/guidelines/` is read-only. Never edit,
  >    pull, check out or commit the submodule or its version. If a guideline needs a change,
  >    write a note for the user instead (`docs/guidelines/DOCS_FOLDER_GUIDELINE.md` §1.1).

- **§5 (CLAUDE) / §6 (AGENTS) "Sections you must always keep verbatim in spirit"** — add to the
  Workflow rules bullet: "…and the read-only rule for the `docs/guidelines/` submodule."
- **§7 (CLAUDE) / §8 (AGENTS) Anti-patterns** — add: "Changing the `docs/guidelines/` submodule
  from the project (editing, pulling, checking out, or committing a new version)."
- **§8 (CLAUDE) / §9 (AGENTS) Final self-check** — add:
  "- [ ] The read-only rule for the `docs/guidelines/` submodule is present, inline."

### 3.4 `AI_AGENT_START_HERE.md` §1 Ground rules

Add a new item 6:

> 6. **Never change the guidelines submodule.** In an app, `docs/guidelines/` is read-only: never
>    edit, pull, check out or commit it or its version. If a guideline needs a change, write a
>    note for the user instead (`DOCS_FOLDER_GUIDELINE.md` §1.1).

### 3.5 `README.md`

In "Templates and references", after the sentence that says a project adds this repo as a
submodule, add:

> The submodule is **read-only** for the project: a project never edits, pulls, checks out or
> commits it. Only the user updates its version (see `DOCS_FOLDER_GUIDELINE.md` §1.1).

## 4. One open point — decided: keep it strict (only the user runs these commands)

Two submodule commands are not covered by your list:

- **Adding the submodule the first time** (`git submodule add`, `GUIDELINES_MANIFEST.md` step 2)
  during a new-app bootstrap.
- **Filling an empty submodule folder after a fresh clone** (`git submodule update --init`). This
  only checks out the version the project already records. It does not change the version.

Your rule forbids `git submodule update` with no exceptions. My suggestion is to keep it strict
and say that **the user** runs both of these commands too. An agent that finds `docs/guidelines/`
empty or missing then stops and asks the user. If you would rather allow the agent to run these
two commands, tell me and I'll change the plan.

## 5. After the change

- Write the change log to `change_log/`.
- Set this plan's status to `completed`.
- Existing apps keep their old `CLAUDE.md` / `AGENTS.md` until someone updates them. Each app's
  update is separate work in that app's own session. It is not part of this plan.
