# Plan — Agent intake for new projects (collect facts, never assume)

**Status:** completed

## The issue

The user starts new apps with an AI agent (Claude Code, Gemini or Codex). The user gives the
agent only a short brief: what the app is for, which tools they use, the app name, and so on.
The agent must create everything else.

`AI_AGENT_START_HERE.md` already exists, but it does not do this job well enough:

1. **No intake step.** §3 step 1 says "fill in the profile with the user", but it does not tell
   the agent *what* to ask, *how* to ask, or what to do with an answer like "you decide". So
   each agent asks different things, or quietly fills gaps with guesses.
2. **The profile misses facts an agent needs before it can design the app.** There is no place
   for: what the app does (main features, target users), whether it works offline, needs a
   backend, accounts, network, ads, in-app purchases, analytics, or device permissions. These
   decide the profiles (Production / Sensitive Data), the store gates and the architecture.
3. **No record of the tools.** Which AI agents work on the project decides which instruction
   files to write (`CLAUDE.md`, `AGENTS.md`, and a Gemini file if Gemini is used). The IDE and
   AI names also feed the About screen (`aiUsed`, `ideUsed`). The developer's OS decides which
   platforms can be built locally (iOS and macOS builds need a Mac).
4. **No "step 0".** A new folder may not yet be a git repository or may not have the
   `docs/guidelines/` submodule. Only the user may add the submodule. The page does not say
   how to check this first.
5. **The `flutter create` command is wrong for this setup.** `flutter create ... <app_name>`
   makes a *new subfolder*. But the guidelines submodule is already at the project root
   (`docs/guidelines/`), so the app must be created *in the current folder*:
   `flutter create --org <prefix> --project-name <dart_package_name> --platforms=<...> .`
6. **No ready-made message for the user.** The user has to write their own first prompt for
   each agent.

## Files to change

| File | Change |
|---|---|
| `AI_AGENT_START_HERE.md` | Add §0 "Before anything: check the folder", a new §3 "New app — intake (collect facts first)", fix the `flutter create` command, add the agent instruction-file rule, renumber later sections. |
| `PROJECT_PROFILE_TEMPLATE.md` | Add §0 "Product brief" and §7 "Tools and workflow"; add a "Source" note for values the agent proposed and the user confirmed. |
| `profiles/example_en_ml_sa_profile.md` | Fill in the two new sections so the worked example stays complete. |
| `README.md` | Replace the one-line agent tip with a copy-paste **starter message** the user can send to any agent. |
| `GUIDELINES_MANIFEST.md` | Only if it lists the profile sections — keep it in step (check during work). |

No change to app code (this repository has none). No submodule involved.

## The fix in detail

### A. `AI_AGENT_START_HERE.md`

**New §0 — Before anything: check the folder** (agent runs read-only checks, changes nothing):
- Is it a git repository? Is `docs/guidelines/` present and filled? If not, stop and ask the
  user to run `git init` / `git submodule add <REPOSITORY_URL> docs/guidelines` themselves.
- Is there already a `pubspec.yaml`? Yes → existing app (adoption sequence). No → new app.
- Is `docs/PROJECT_PROFILE.md` present? Are any values still `<...>`?
- Run `flutter --version` and `flutter doctor` to learn what is installed. Report problems;
  do not install or upgrade anything without asking.

**New §3 — New app intake (collect facts first).** The rules:
1. **Write down what the user already said.** Copy each fact from the user's brief into the
   profile. Do not ask again for something the user already gave.
2. **Ask for everything else, in one organised list.** Group the questions (see the table
   below). Number them so the user can answer "3: yes, 5: no".
3. **Never fill a gap silently.** A value is either *given by the user* or *proposed by the
   agent and confirmed by the user*. When proposing, say it is a suggestion and give the
   reason, e.g. "Suggested app id: `com.<you>.notes` — please confirm or change it."
4. **"You decide" is not a final answer for permanent or public values.** The agent may
   propose, but must get a clear "yes" for: app id, Dart package name, publisher name, public
   contact, platforms, stores, languages, licence, privacy policy URL. For technical choices
   (state management, structure tier, flavors) the agent may pick the documented default and
   record "default — chosen by agent, user agreed".
5. **Unknown is allowed, but must be marked.** If the user does not know yet (e.g. privacy
   policy URL), write `TODO — needed before <gate>` instead of inventing a value. The gate that
   needs it lists it as a blocker.
6. **Show the filled profile and get approval** before `flutter create` or any other file.
7. **Questions may be asked in rounds.** First the must-have list; later rounds only for
   things that came up from earlier answers (e.g. picked iOS → ask about a Mac and Apple
   developer account).

The question checklist (grouped, short). The agent asks only what the brief did not answer:

| Group | Ask about |
|---|---|
| Product | What the app does in one line; main features (first version); who uses it; what is *out of scope* |
| Identity | App display name; publisher / organisation; reverse-DNS app id; Dart package name; public support contact; website; copyright owner; licence of the code |
| Platforms & stores | Which platforms ship; which store or channel for each; minimum OS if the user has a need; the developer's OS (only to know what can be built locally — not stored as a machine detail) |
| Languages | Template language; every other UI language; store-listing languages; who reviews each translation |
| Data & privacy | Offline only or online; backend / API (own or third-party); user accounts / sign-in; which data is stored, and where; any personal, health, finance or secret data; analytics or crash reporting; ads; in-app purchases or subscriptions; device permissions (camera, location, files, notifications, …); privacy policy URL |
| Look & feel | Brand colours or "use the default theme"; app icon source (user supplies / agent makes a placeholder); light / dark mode; About screen rows and the optional signature badge |
| Tools & workflow | Which AI agents will work on the project (Claude Code, Gemini, Codex, other); IDE; git hosting (GitHub or other); whether CI is wanted; who owns releases (a role, not a personal name) |
| Technical defaults | State management, structure tier, flavors — offer the documented default and ask for a yes |

The answers then decide, and the agent says so in the summary:
- applicability profiles (Production App Extension, Sensitive Data Extension),
- which store gates in `platform_store_readiness.md` apply,
- which language packs apply,
- which instruction files to write.

**Instruction files per agent** (new short rule in the bootstrap step): always `CLAUDE.md` and
`AGENTS.md` (both mandatory). If Gemini is used, also make Gemini read the same rules — a short
`GEMINI.md` that points to `AGENTS.md`, or the tool's setting that makes it read `AGENTS.md`.
Check the tool's current docs for which file it reads; do not assume. Never keep two different
copies of the rules.

**Fix the create command** in the bootstrap step:

```bash
flutter create --org <reverse-dns-prefix> --project-name <dart_package_name> --platforms=<declared platforms> .
```

with a note: run it in the project root (where `docs/guidelines/` already is), so the app is
not created in a subfolder.

**Renumber** the old §3 → §4 (bootstrap, starts with "intake done and approved"), §4 → §5,
§5 → §6. Fix every cross-reference to these numbers (I will grep the whole set).

### B. `PROJECT_PROFILE_TEMPLATE.md`

- **New §0 "Product brief"**: one-line purpose, main features for version 1, target users, out
  of scope, and a "Data & privacy" table (offline/online, backend, accounts, data stored,
  sensitive data, analytics, crash reporting, ads, purchases, permissions with a reason each).
- **New §7 "Tools and workflow"**: AI agents used, IDE, git hosting, CI wanted (checks only),
  release owner (role). Move "Release owner" here from §6.
- **"Source" rule** in the "How to use" note: next to an agent-proposed value write
  `(proposed, confirmed by user)`; for unknowns write `TODO — needed before <gate>`.
- Keep it short; delete-what-does-not-apply stays the rule.

### C. `profiles/example_en_ml_sa_profile.md`

Add a filled §0 and §7 that match the example app (offline PDF reader, no accounts, no ads,
file access permission, Claude Code + VS Code, GitHub, CI checks only).

### D. `README.md`

Add a copy-paste starter message under "AI agents: start here", for example:

> Read `docs/guidelines/AI_AGENT_START_HERE.md` and follow it. This is a new app.
> Here is what I know: <purpose>, <app name>, <platforms/stores>, <languages>, <tools>.
> Ask me for anything else you need. Do not assume anything.

### E. Consistency check

- Grep all files for references to `AI_AGENT_START_HERE.md` section numbers and the profile
  section numbers; update them.
- Keep the rules version-agnostic (no Flutter version numbers).
- Write the change log in `change_log/` afterwards.

## Out of scope

- No change to `CLAUDE_MD_GUIDELINE.md` / `AGENTS_MD_GUIDELINE.md` beyond a cross-reference, if
  any is needed. (If you want a full Gemini section there, that can be a separate plan.)
