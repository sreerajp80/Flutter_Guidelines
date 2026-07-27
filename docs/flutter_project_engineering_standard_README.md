## What does `flutter_project_engineering_standard.md` say?

> **Reflects Flutter 3.44 / Dart 3.12 (mid 2026).** When the SDK moves on, the
> sections that name a specific Flutter version (UIScene, 16 KB pages, `TextScaler`,
> Java 17, AGP 9 status, iOS 13 minimum, etc.) are the ones to revisit first.

It is a **master rulebook** for building any Flutter application to a consistent, maintainable, and shippable standard. Unlike `architecture.md` (which describes *your specific project*) and `security.md` (which covers *your specific threat model*), the engineering standard is **project-agnostic** — it defines rules that apply to *every* Flutter app you build.

It has 24 sections across every dimension of Flutter development. Here is the map:

**Sections 1–4 — Foundations.** How to apply the standard (conformance language: `MUST`/`SHOULD`/`MAY`, three applicability profiles, repository types), nine core principles, when to use Tier 1 (layer-first) vs Tier 2 (feature-first) structure, the state management rules, the canonical Widget → State → Service → Repository → Datasource data flow, and the mandatory `main()` initialization sequence.

**Sections 5–9 — App-level engineering.** Build flavors and when they are required, the `AppFlavorConfig` pattern, **toolchain requirements (Java 17 minimum, AGP 8.x — not 9.x, iOS 13 minimum), Android 16 KB page-size compliance, iOS UIScene lifecycle migration**, MSIX packaging for Windows, which Android artifact to use when. UI/UX baseline: theme tokens (Material 3 default since 3.16), screen-state patterns (loading/empty/success/error), feedback component rules (SnackBar vs AlertDialog vs BottomSheet), animation duration tokens with easing curves, haptic feedback rules, keyboard/scroll behavior, and safe area handling (edge-to-edge enforced on Android 15+ targeting SDK 35+). Accessibility: touch targets, contrast ratios, semantics, **font scaling via `TextScaler` at 1.0×/1.5×/2.0×** (the deprecated `textScaleFactor` API has been replaced), focus and keyboard navigation, screen reader testing. Localization minimum setup. App lifecycle management with `WidgetsBindingObserver` (with the modern `AppLifecycleListener` alternative for lifecycle-only concerns).

**Sections 10–15 — Quality and safety.** Performance: **Impeller as the default renderer (iOS, Android API 29+)**, frame budget targets, widget rebuild optimization, list rendering rules, image memory rules, isolates for background work, startup targets, and the app size budget table. Error handling: global boundaries in `main()`, four-tier classification (recoverable/degraded/session/fatal), repository-layer error wrapping, and UI error presentation. Code generation: `build_runner` commands (using `dart run`, since `flutter pub run` is deprecated) and the generated file policy. Database: versioned migration strategy, WAL mode, index strategy, integrity constraints. Logging: six-level taxonomy, what to never log, what to log per layer, log rotation. Security: core rules for all apps, Sensitive Data Extension requirements, OWASP Mobile Top 10 checklist.

**Sections 16–24 — Standards and process.** Coding standards including the recommended `analysis_options.yaml` lint rules (with pinned `flutter_lints`). Asset management (WebP/SVG formats, 2×/3× variants, **per-platform asset bundling new in Flutter 3.41**, font licensing). Testing levels per profile, plus the experimental Widget Previewer (Flutter ≥ 3.35). CI minimum and production checklists. Git hygiene. Documentation requirements. AI coding assistant instructions. The Definition of Done checklists. Practical one-liner lessons.

---

## How do you use it in a Flutter project?

Think of it as three things simultaneously.

**1. A decision eliminator before you start.** The standard resolves most common architectural debates in advance. You do not need to debate `ListView` vs `ListView.builder` for a 100-item list, what error handler goes in `main()`, how to initialize `sqflite` on Windows, or which easing curve to use for an entering element. The standard answers those questions once. You follow the answer and move on.

**2. A quality gate during development.** Section 23 (Definition of Done) gives you a concrete checklist before every task is closed. `flutter analyze` clean, tests updated, generated files current, no secrets staged. This turns vague "is it done?" into a binary check.

**3. A profile selector for different project types.** Not every rule applies to every project. The three profiles are:

| Profile | Applies To |
|---------|-----------|
| `Core Baseline` | Every Flutter app, no exceptions |
| `Production App Extension` | Apps shipped to real users, QA, or a store |
| `Sensitive Data Extension` | Apps handling secrets, health, financial, or PII data |

You declare the active profiles in `architecture.md §1`. Rules marked "under Production App Extension" only activate once that profile is declared.

---

## What should you fill out before starting the project?

The engineering standard is **not a fill-in-the-blanks template**. You do not write into it. You **read it and make decisions from it**, then record those decisions in `architecture.md` and `security.md`.

### Part 1 — Decisions to make before writing code

| Priority | Section | Decision to Make and Record |
|----------|---------|----------------------------|
| 🔴 Must | **§1 Profiles** | Declare which profiles apply. Record in `architecture.md §1`. |
| 🔴 Must | **§3 Structure** | Choose Tier 1 or Tier 2. Record in `architecture.md §4` with the reason. |
| 🔴 Must | **§4.1 State management** | Confirm one primary package. Record in `architecture.md §8`. |
| 🔴 Must | **§4.2 Data flow** | Confirm the canonical chain or document any omitted layer. Record in `architecture.md §9`. |
| 🔴 Must | **§4.5 Init sequence** | Plan the exact `main()` order. Record in `architecture.md §5`. |
| 🔴 Must | **§5 Build config** | Decide if flavors are needed now. Plan the `AppFlavorConfig` if yes. Record in `architecture.md §15`. |
| 🔴 Must | **§13 Database** | Confirm WAL mode on, FK enforcement on, migration strategy. Record in `architecture.md §14`. |
| 🔴 Must | **§14 Logging** | Confirm logger package and what must never be logged. Record in `architecture.md §17`. |
| 🟡 Soon | **§11 Error handling** | Draft the sealed `AppException` hierarchy before the first repository is written. Record in `architecture.md §10`. |
| 🟡 Soon | **§15 Security** | Decide sensitivity level and active profile. Feed decisions into `security.md §1–§4`. |
| 🟡 Soon | **§16.1 Lints** | Copy the recommended `analysis_options.yaml` into the project root before writing the first Dart file. |
| 🟡 Soon | **§19 CI** | Set up the minimum CI workflow before the first merge to main. |
| 🟢 Later | **§6 UI/UX** | Reference animation tokens and feedback component rules as each screen is built. |
| 🟢 Later | **§7 Accessibility** | Verify touch targets, contrast, and semantics as each widget is built. |
| 🟢 Later | **§10 Performance** | Reference frame budget and list rules as each screen is built. |
| 🟢 Later | **§23 Done checklist** | Walk through before marking any task complete. |

### Part 2 — Setup tasks the standard mandates before coding

**Copy `analysis_options.yaml` lint rules** — Section 16.1 provides the full recommended rule set. Create this file before the first Dart file. Lint problems are cheapest to fix before patterns embed.

**Set up CI with minimum checks** — Section 19.1: `flutter pub get` → `build_runner` → `dart format` → `flutter analyze` → `flutter test`. Set this up before the first code lands, not after.

**Configure `flutter_localizations` in `MaterialApp`** — Section 8.1. Without `GlobalMaterialLocalizations` delegates, certain Material widgets render incorrectly on non-English system locales. A one-line fix at project creation that prevents a confusing bug later.

**Run the dependency audit for offline apps** — Section 16.5. For fully offline apps, `dart pub deps --style=tree` must be run before adding any package. This verifies no transitive HTTP dependency is introduced. Architecturally mandatory, not optional.

**Create `docs/architecture.md` and `docs/security.md`** — Section 21.1 lists these as required documents. Fill out all 🔴 Must sections in both before coding begins.

---

## How can AI use this document? What do you need to do?

### How AI uses it

When the standard is placed in `docs/` and referenced in `CLAUDE.md`, an AI coding assistant reads it and gains a complete picture of *how* code should be written — not just what to build.

**Structure** — It knows the tier, avoids a second `utils/` folder, and never invents a second state management system.

**Data flow** — It knows Widget → State → Service → Repository → Datasource. It will not put SQL inside a widget or put navigation logic inside a service.

**Init order** — It knows `sqfliteFfiInit()` before database open, database open before `runApp()`. Wrong ordering causes silent release-only crashes; the standard explains why.

**List rendering** — It knows `ListView(children:[...])` is prohibited for lists over ~20 items. It uses `ListView.builder` with `itemExtent` when item height is fixed.

**Error handling** — It knows to catch `SqliteException` at the repository layer, re-throw typed `AppException`, and never expose stack traces or exception class names to the user.

**Logging** — It knows `print` and `debugPrint` are banned in committed code, trace/debug must be gated by flavor config, and all error logs must include the `error` object and `stackTrace`.

**Animation** — It knows `Curves.easeOut` for entering elements, `Curves.easeIn` for leaving elements, `MediaQuery.disableAnimations` must be respected, and that animating layout-affecting properties (`Padding`, `SizedBox` dimensions, `Align` factors) is permitted but requires profiling — `Transform` and `Opacity` are preferred by default because they are GPU-composited and skip layout/paint passes.

**Code generation** — It knows whether generated files are committed or excluded in your project, and will regenerate them after modifying annotated source.

**Definition of done** — It knows a task is not complete until `flutter analyze` is clean, tests are updated, generated files are current, and no secrets are staged.

Without the standard, the AI applies its own defaults — which may be inconsistent with your project's patterns and which can vary between sessions.

### What you need to do

**Step 1 — Place the standard in your docs folder**
```
<project_root>/docs/flutter_project_engineering_standard.md
```

**Step 2 — Add a rule to `CLAUDE.md`**

```
Rule N: Before writing any code, read docs/flutter_project_engineering_standard.md.
        Active profiles: Core Baseline, Production App Extension.

  Structure: Follow architecture.md §4 tier. No second state-management system.
             No packages introducing transitive HTTP or network activity.

  Code rules (every task):
  - Never use ListView(children:[...]) for lists over 20 items.
  - Never use print() or debugPrint(); use AppLogger.
  - Prefer Transform/Opacity over animating layout-affecting properties
    (Padding, SizedBox dimensions, Align factors). Animating layout properties
    is permitted with profiling evidence; the default is to avoid it.
  - Never put SQL, encryption, or HTTP knowledge inside a widget.
  - Always add const to constructors and widget instantiations where possible.
  - Always add a Semantics label to custom interactive widgets.
  - Always use compute() or Isolate for work taking more than ~4 ms.

  Definition of done (every task):
  - flutter analyze clean. Tests added/updated. Generated files current.
  - No secrets, build output, or local machine files staged.
```

**Step 3 — Reference specific sections when asking for code**

Instead of: *"Write the TodoRepository"*
Say: *"Following engineering standard §4.2 data flow and §11.3 repository error handling, write the TodoRepository. Catch SqliteException and re-throw typed StorageException. Never log field content."*

Instead of: *"Build the todo list screen"*
Say: *"Following engineering standard §6.3 screen-state guidance, §6.4 feedback components, and §10.3 list performance rules, build the todo list screen. ListView.builder, skeleton loader, SnackBar with undo for delete."*

**Step 4 — Use §23 as a review checklist after receiving AI output**

After the AI delivers code, quickly verify: `const` constructors used, `ListView.builder` for any sizeable list, no SQL in widgets, no silently swallowed errors, tests added, `flutter analyze` clean. If any check fails, cite the section and ask for a correction.

**Step 5 — The standard is project-level ground truth — not a per-session upload**

Once in `docs/` and referenced in `CLAUDE.md`, every future session reads it automatically alongside `architecture.md` and `security.md`. You do not re-explain it. Per task you indicate which sections are most relevant — this focuses the AI precisely rather than asking it to apply all 24 sections equally to every small change.

---

## How the five documents work together

| Document | Answers |
|----------|---------|
| `flutter_project_engineering_standard.md` | *How* should all Flutter code be written? Universal rules for every project. |
| `flutter_build_flavors_guide.md` | *How* exactly do flavors wire into each platform's native build system? |
| `architecture.md` | *What* did this specific project decide? Tier, packages, schema, routes, signing strategy. |
| `security.md` | *What* does this specific project protect? What is sensitive, what is never logged, how is data encrypted? |
| `release_process.md` | *How* does this specific project ship? The exact commands, the checklist, the evidence trail. |

All five are needed to write correct code and ship it safely. The engineering standard gives the pattern. The build flavors guide gives the platform mechanics for delivering that pattern through Gradle, Xcode, and `flutter build windows`. `architecture.md` gives the type names and layer locations. `security.md` constrains what the logger inside any given method is allowed to emit. `release_process.md` turns the resulting decisions into the exact reproducible commands and pre-release gates each shipped build must pass through. Referencing all five when asking the AI for a non-trivial implementation gives it everything required to produce code that is correct, consistent with your project, safe, and shippable.
