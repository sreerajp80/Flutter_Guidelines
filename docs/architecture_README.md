## What does `architecture.md` say?

It's a **living blueprint template** for your Flutter app. It has 22 sections covering every architectural decision a Flutter project needs to make explicit:

- What the app is and what platforms it targets (Section 1–2)
- How the codebase is structured — Tier 1 flat vs Tier 2 feature-first (Section 3–4)
- The exact startup sequence in `main()` and why order matters (Section 5)
- How the app behaves when backgrounded, killed, or memory-pressured (Section 6)
- Whether the app needs internet and how that's enforced (Section 7)
- State management choice and boundaries (Section 8)
- How data flows from Widget → State → Service → Repository → DB (Section 9)
- How errors are classified and surfaced (Section 10)
- Your SQLite schema, migration history, and indexes (Section 11)
- DI, navigation, persistence, build flavors, UI system, logging, testing (Sections 12–18)
- Performance constraints, key decisions, and known risks (Sections 19–21)

Right now it's a **blank template** — every field says `<placeholder>`. You fill it with your project's actual decisions.

---

## How do you use it in a Flutter project?

Think of it as three things simultaneously:

**1. A decision-forcing checklist before you write code**
Going through each section forces you to make explicit choices early — state management, error strategy, DB schema, offline enforcement — rather than discovering conflicts mid-build.

**2. A living reference during development**
Every time a major decision is made or changed — new migration, new route added, new risk identified — you update the relevant section. It stays current with the codebase.

**3. An onboarding document**
Anyone (including your future self six months later) can read it and understand the entire system without reading the code. Section 5 alone tells you the exact `main()` sequence needed to avoid release-only crashes.

---

## What should you fill out BEFORE starting the project?

These sections must be decided and written **before writing a single line of code**, because they shape every file you create:

| Priority | Section | Why Before Coding |
|----------|---------|-------------------|
| 🔴 Must | **§1 Scope** | Locks down platforms, profiles in force |
| 🔴 Must | **§2 Goals / Non-Goals** | Prevents scope creep decisions mid-build |
| 🔴 Must | **§3 Architecture Summary** | Your one-paragraph north star |
| 🔴 Must | **§4 Repository Structure** | Tier choice + folder layout — affects every file |
| 🔴 Must | **§5 Initialization Sequence** | Wrong order = release-only crashes |
| 🔴 Must | **§7 Offline Behavior** | Often the hardest constraint — shapes all dependencies |
| 🔴 Must | **§8 State Management** | Choose one primary pattern — document its boundaries now |
| 🔴 Must | **§9 Data Flow** | Widget → State → Service → Repo → DB pattern |
| 🔴 Must | **§11 Domain Model** | Schema v1, table list, and migration plan |
| 🔴 Must | **§13 Navigation** | Routing package — route file location and protected-route strategy |
| 🟡 Soon | **§10 Error Handling** | Sealed `AppException` hierarchy — before first repository |
| 🟡 Soon | **§14 Persistence** | WAL mode, FK enforcement, sqflite_common_ffi setup |
| 🟡 Soon | **§15 Environment & Build** | `dev`/`prod` flavor model, dart-define pattern |
| 🟢 Later | **§6 Lifecycle** | Fill as you implement `AppLifecycleService` |
| 🟢 Later | **§17 Logging** | Fill when `AppLogger` is implemented |
| 🟢 Later | **§18 Testing** | Fill as test suite grows |
| 🟢 Later | **§20 Decisions** | Record as tradeoffs are made |
| 🟢 Later | **§21 Risks** | Populate from your project's risk register |

---

## How can AI use this document? What do you need to do?

This is where you get significant leverage. Here's the exact mechanism:

### How AI uses it
When `architecture.md` is populated and placed in your project, an AI assistant reads it and can:
- Know the exact folder structure before creating any file (no wrong-tier mistakes)
- Know the initialization order before writing `main()` (no ordering bugs)
- Know the offline constraint and refuse to suggest packages that perform network access
- Know the schema version before writing any migration (no version conflicts)
- Know the error hierarchy before writing any repository method
- Know which state-management package is in force (and not suggest alternatives)
- Know which navigation package is in force and use the correct route definition style

Without it, the AI has to guess or ask — and sometimes guesses wrong silently.

### What you need to do

**Step 1 — Place it in the right location**
```
<project_root>/docs/architecture.md
```
Your `CLAUDE.md` (the hard rules file) should already reference it, but if not, add a rule like:

```
Rule 2: Read docs/architecture.md before creating any file, writing any migration,
        or suggesting any package.
```

**Step 2 — Fill out the 🔴 Must sections completely before coding**
Once that's done, the document is your project's ground truth.

**Step 3 — Keep it current as the project evolves**
Every time you add a migration, tell the AI: *"Update §11 schema version to 2, migration adds the new column to its table."* This keeps the document accurate so future sessions don't make stale decisions.

**Step 4 — Reference it explicitly when asking for code**
Instead of:
> *"Write a repository for X"*

Say:
> *"Following docs/architecture.md §9 data flow and §11 schema, write the XRepository"*

This pins the AI to your documented decisions rather than its defaults.

---

## How this fits with the other documents

`README.md` lists every document in the guideline set and which ones apply to an app, by profile.
In short: the **references** (engineering standard, flavors guide, store gates, `guideline.md`) say
*how* to build and ship; the app's filled-in **templates** in `docs/` (`PROJECT_PROFILE.md`,
`architecture.md`, `security.md`, `release_process.md`) record *what this app decided*.
