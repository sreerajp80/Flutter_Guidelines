# Guideline: How to Create Files in a Project's `docs/` Folder

This document tells you how to create a file inside a Flutter project's `docs/` folder so that
every project's documentation looks and works the same way.

It was built by studying the `docs/` folders of nine existing Flutter projects (Devi, PDFApp,
ContactSphere, TextApp, MantraJapaCounter, todo, qr_reader, youtube_shortcut, Authenticator).

It pairs with [CLAUDE_MD_GUIDELINE.md](CLAUDE_MD_GUIDELINE.md) and [AGENTS_MD_GUIDELINE.md](AGENTS_MD_GUIDELINE.md), which cover how to write the mandatory project-root `CLAUDE.md` and `AGENTS.md`. This one covers the files that live under `docs/`.

---

## 1. Scope — what this guideline covers (and what it does not)

This guideline applies to the **per-app documentation files** you create in a project's
`docs/` folder.

It does **not** apply to, and you must **not** hand-write or edit:

- **`docs/GUIDELINES_MANIFEST.md`** — the shared portable pointer file. It is copied in
  unchanged from the standard set. Do not rewrite it per project.
- **`docs/guidelines/`** — the shared guidelines **Git submodule**. Never edit files inside
  the submodule from within a project. Change those only in their own repository. From a
  project you only ever *read* or *link to* them.

Everything else you add under `docs/` (architecture, security, release, and so on) follows the
rules below.

---

## 2. Local copy vs the submodule — where a doc belongs

Some documents exist both as a shared template in the submodule (`docs/guidelines/…`) and as a
filled-in local file (`docs/…`). The rule (from `GUIDELINES_MANIFEST.md`) is:

- **The local copy wins.** If the project has its own `docs/architecture.md`, that is the truth
  for this app; the submodule's `architecture.md` is only the template.
- **Do not duplicate a submodule doc locally unless you are filling it in for this app.**
  Blueprint templates (`architecture.md`, `security.md`) are *meant* to be copied down and
  filled in. Reference docs (`flutter_project_engineering_standard.md`,
  `flutter_build_flavors_guide.md`) are *not* — link to the submodule copy instead of copying
  them into `docs/`. (Some older apps copied them locally; do not repeat that for new apps.)
- When in doubt: **fill in blueprints locally, link to references.**

---

## 3. Naming convention

- **Case: `snake_case`.** Lowercase, words joined by underscores. Example:
  `release_process.md`, `known_gaps.md`, `implementation_plan.md`.
- Older files use `kebab-case` (hyphens), e.g. `security-rules.md`. Those are tolerated — do
  **not** rename them just for style — but **every new file uses `snake_case`.**
- Name by content, not by date. A `docs/` file describes a lasting part of the app.
- **No date prefix.** Date-prefixed names (`yyyymmdd_hhMMss_*`) are only for `plans/` and
  `change_log/` at the project root, never for `docs/`. All `plans/` and `change_log/` files MUST
  use relative repository paths only (no absolute system paths like `C:\...` or `l:\...`) and MUST NOT
  contain **local system details** — OS user name, computer/host name, home or drive-letter paths,
  network share names, LAN/internal IP addresses, local server URLs with ports, device serial
  numbers, personal email addresses — or any secret (API keys, tokens, passwords, keystore
  passphrases, credentials, PII). These files are committed and may become public, so write them
  as if a stranger will read them. The full rule, with bad → good examples, is in
  `flutter_project_engineering_standard.md` §21.1.1.
- Keep the name short and obvious: `architecture.md`, not `app_architecture_overview_v2.md`.
- For a point-in-time record that is genuinely one-off (an audit of a specific phase), it is
  fine to include a phase marker in the name, e.g. `security_audit_phase13.md`. Put the date
  *inside* the file (see §5), not in the name.

---

## 4. Standard file anatomy

Every `docs/` file follows the same skeleton. Not every part is required, but the order is
always the same.

1. **`# H1` title.** One title line. For app-specific documents, append the app name with an
   em dash: `# Architecture — AppName`. Shared/generic docs omit the app name.
2. **Purpose paragraph.** One short paragraph directly under the title saying what the file is
   and when to read it (e.g. "Read this before changing any security-sensitive code.").
3. **"Read first" links.** If the reader should open something else first, link it here with a
   relative path — typically `../.agents/AGENTS.md` (or `../CLAUDE.md`), a sibling doc, and any
   relevant `guidelines/…` submodule doc.
4. **`---` separator**, then **numbered `##` sections.** Number the main sections (`## 1.`,
   `## 2.`, …) and separate major blocks with a `---` rule, matching the existing files.
5. **Body**, using these shared conventions:
   - **Simple English.** Short sentences, common words. Explain any jargon.
   - **Callouts** for danger. Use a `>` blockquote for secrets/security warnings, e.g. the
     keystore warning in `release_signing.md`.
   - **Code fences with a language tag** (e.g. `powershell`, `bash`, `dart`). Give
     both PowerShell and bash when a command differs by OS.
   - **Tables** for structured lists (packages, colors, profiles, routes).
   - **Checklists** (`- [ ]` / `- [x]`) for progress trackers and backup/verification lists.
   - **Relative links** for every cross-reference so they stay clickable in the repo.

---

## 5. Point-in-time vs living documents

Two kinds of docs live in `docs/`:

- **Living documents** describe how the app *is* and are kept current: `architecture.md`,
  `security.md`, `release_process.md`, `workflow_rules.md`, `dependencies.md`,
  `project_structure.md`. No date line — they are always "now".
- **Point-in-time documents** record a moment: `implementation_progress.md`,
  `security_audit_phase*.md`, `known_gaps.md`. These **start with a `**Date:**` line** (and
  often a `**Scope:**` / `**Result:**` line) right under the purpose paragraph, and are not
  rewritten later — you add a new dated entry or a new file instead.

Write absolute dates (`2026-07-18`), never "today" or "last week".

---

## 6. Mandatory baseline docs for a new project

When generating or initializing the `docs/` folder for a new Flutter app, **always create the complete baseline set** of documents below. Do not omit any document from this initial set:

1. **`architecture.md`** (Living) — Technical design, layers, data models, and component boundaries.
2. **`security.md`** (Living) — Security rules, permissions, threat model, and offline boundaries.
3. **`release_process.md`** (Living) — Keystore signing, versioning, build commands, and release runbook.
4. **`workflow_rules.md`** (Living) — Plan-before-changing, explicit user approval gate, and log-after-changing rules.
5. **`dependencies.md`** (Living) — Approved baseline packages and explicit prohibited dependencies list.
6. **`project_structure.md`** (Living) — Project file tree and directory responsibility layout.
7. **`implementation_plan.md`** (Point-in-time) — Phase-by-phase build roadmap with objectives and action steps (starts with `**Date:**`).
8. **`implementation_progress.md`** (Point-in-time) — Live status checklist by phase (`- [x]` / `- [ ]`, starts with `**Date:**`).

Together with `GUIDELINES_MANIFEST.md` and the `guidelines/` submodule, these 8 documents form the complete baseline documentation suite required for every new project.

---

## 7. Catalog of recognized doc types

When you need a doc, first check whether it is one of these standard types and use its
skeleton. Prefer these names over inventing new ones.

| File (`snake_case`) | Kind | What it is | Typical sections |
| --- | --- | --- | --- |
| `architecture.md` | Living | The app's technical design — layers, packages, folder tree, key flows. Fill in from the submodule blueprint. | Project config · Design goals · Layered architecture · Packages · Folder structure · Theme · Key interactions · Non-functional requirements |
| `security.md` (or `security_rules.md`) | Living | Security rules and/or blueprint — threat model, permissions, crypto, data handling. Short "rules" file may link to a fuller blueprint. | Boundaries/offline rules · Minimal permissions · Input validation · Secrets handling · OWASP/threat checklist |
| `release_process.md` / `release_signing.md` | Living | How to build, sign, and ship a release. | Secrets warning callout · What the build expects · Generate keystore · `key.properties` · Build commands · Verify signature · Backup checklist |
| `workflow_rules.md` | Living | Plan-before-changing and log-after-changing rules for this project (mirrors the global workflow). | Plan before changing · Approval gate · Log after changing |
| `<app>_idea.md` | Living | The product concept and requirements — the "why" and "what". | Core concept · Experience/UX · Design system · Navigation · Development phases |
| `implementation_plan.md` | Point-in-time | Phase-by-phase build plan with objectives and action steps. | One `##` per phase (Objective + Action steps) · Verification / Definition of Done |
| `implementation_progress.md` | Point-in-time | Live checklist of what is done. | Status overview · Detailed task checklist by phase (`- [x]`) |
| `known_gaps.md` | Point-in-time | What is declared but not integrated, and resolved items. | Resolved (dated) · Still open / not integrated · Out of scope |
| `dependencies.md` | Living | Notable packages and their integration status. | Grouped bullet list by concern (declared vs integrated) |
| `project_structure.md` | Living | The project file tree, for quick orientation. | A single fenced tree, or short notes + tree |
| `security_audit_phase*.md` | Point-in-time | A dated, rule-by-rule audit record. | Date/Scope/Result header · How to re-run checks · Rule-by-rule findings |

If what you need is **not** in this table, it is probably a section inside an existing doc
(see §8) rather than a new file.

---

## 8. When to create a new doc vs extend an existing one

Keep the `docs/` set small and predictable.

- **Prefer a new `##` section** in an existing doc over a new file. Audio lifecycle rules go in
  `architecture.md`; a new permission rule goes in `security.md`.
- **Create a new file only when** the topic is a whole recognized type from §7, or it is a
  standalone point-in-time record (an audit, a one-off migration note).
- Do not create per-feature files (`login.md`, `settings.md`). Those belong as sections.
- One concept, one home. Do not describe the release process in both `architecture.md` and
  `release_process.md` — put it in one and link from the other.

---

## 9. Cross-linking rules

- Always use **relative markdown links**, so they work when the repo is cloned anywhere.
- Link "up" to the mandatory project rules file: `../CLAUDE.md` (or `../.agents/AGENTS.md`).
- Link "sideways" to siblings: `[architecture.md](architecture.md)`.
- Link to the submodule when referring to a shared template or reference:
  `docs/guidelines/security.md`.
- When a short rules file exists alongside a fuller blueprint, the short file should say where
  the full detail lives (e.g. `security_rules.md` → "Full detail is in the Security
  Architecture section of `architecture.md`.").

---

## 10. Checklist before saving a new `docs/` file

- [ ] It is **not** `GUIDELINES_MANIFEST.md` and **not** inside `guidelines/` (§1).
- [ ] Name is `snake_case`, lowercase, descriptive, no date prefix (§3).
- [ ] It is a filled-in blueprint, not a copied-down reference doc (§2).
- [ ] Starts with an `# H1` title (+ app name for app-specific docs) and a one-paragraph
      purpose (§4).
- [ ] "Read first" links present where useful; all cross-links are relative (§4, §9).
- [ ] Numbered `##` sections separated by `---`; simple English throughout (§4).
- [ ] Secrets/danger use a `>` callout; commands are in language-tagged fences (§4).
- [ ] If point-in-time, it has a `**Date:**` line with an absolute date (§5).
- [ ] It matches one of the recognized types in §7, or it genuinely could not be a section of
      an existing doc (§8).
