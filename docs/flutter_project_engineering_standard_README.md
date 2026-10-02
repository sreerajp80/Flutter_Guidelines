## What does `flutter_project_engineering_standard.md` say?

> **Last checked against Flutter 3.47 / Dart 3.13 (2026-10).** The standard does not pin
> tool versions — it says "use the latest stable Flutter and the Android versions it
> generates". When the SDK moves on, refresh the "last checked" reference snapshot and the
> "since 3.xx" conditions (UIScene, 16 KB pages, `TextScaler`, Java 17, iOS 15 minimum,
> `material_ui` / `cupertino_ui`, etc.) first.

It is a **master rulebook** for building any Flutter application to a consistent, maintainable, and shippable standard. Unlike `architecture.md` (which describes *your specific project*) and `security.md` (which covers *your specific threat model*), the engineering standard is **project-agnostic** — it defines rules that apply to *every* Flutter app you build.

It has 24 sections across every dimension of Flutter development. Here is the map:

**Sections 1–4 — Foundations.** How to apply the standard (conformance language: `MUST`/`SHOULD`/`MAY`, three applicability profiles, repository types), nine core principles, when to use Tier 1 (layer-first) vs Tier 2 (feature-first) structure, the state management rules, the canonical Widget → State → Service → Repository → Datasource data flow, and the recommended `main()` initialization sequence (logging first, then platform bindings, storage, database, config).

**Sections 5–9 — App-level engineering.** Build flavors and when they are required, the `AppFlavorConfig` pattern, **toolchain requirements (latest stable Flutter and the AGP / KGP / Gradle versions it generates, look up versions instead of guessing, Java 17 minimum, `kotlin { compilerOptions { } }` instead of `kotlinOptions` on AGP 9+, iOS 15 minimum), an upgrade checklist for any new Flutter release (with extra steps when crossing 3.47), Android 16 KB page-size compliance, iOS UIScene lifecycle migration**, desktop setup for Windows (MSIX, Store vs code-signed download), macOS (App Sandbox, entitlements, Developer ID notarization) and Linux (GTK build packages, `APPLICATION_ID`, Snap/Flatpak packaging), and which artifact to ship on each platform. UI/UX baseline: theme tokens (Material 3 default since 3.16), **Material and Cupertino as the standalone `material_ui` / `cupertino_ui` packages (Flutter 3.47) and the `dart fix --apply --code=migrate_design_widgets` migration**, screen-state patterns (loading/empty/success/error), feedback component rules (SnackBar vs AlertDialog vs BottomSheet), animation duration tokens with easing curves, haptic feedback rules, keyboard/scroll behavior, and safe area handling (edge-to-edge enforced on Android 15+ targeting SDK 35+). Accessibility: touch targets, contrast ratios, semantics, **font scaling via `TextScaler` at 1.0×/1.5×/2.0×** (the deprecated `textScaleFactor` API has been replaced), focus and keyboard navigation, screen reader testing, **and a mandatory tooltip on every icon-only control**. Localization: **every app ships the languages it declares in `docs/PROJECT_PROFILE.md`** (strings always in ARB files, even for one language), with an in-app language picker when there are two or more, translation-quality rules, optional **language packs** for languages that need extra rules, and short-label budgets. App lifecycle management with `WidgetsBindingObserver` (with the modern `AppLifecycleListener` alternative for lifecycle-only concerns).

**Sections 10–15 — Quality and safety.** Performance: **Impeller as the default renderer (iOS, Android API 29+, and desktop since 3.47)**, frame budget targets, widget rebuild optimization, list rendering rules, image memory rules, isolates for background work, startup targets, and the app size budget table. Error handling: global boundaries in `main()`, four-tier classification (recoverable/degraded/session/fatal), repository-layer error wrapping, and UI error presentation. Code generation: `build_runner` commands (using `dart run`, since `flutter pub run` is deprecated) and the generated file policy. Database: versioned migration strategy, WAL mode, index strategy, integrity constraints. Logging: six-level taxonomy, what to never log, what to log per layer, log rotation. Security: core rules for all apps, Sensitive Data Extension requirements, OWASP Mobile Top 10 checklist.

**Sections 16–24 — Standards and process.** Coding standards including the recommended `analysis_options.yaml` lint rules (with pinned `flutter_lints`). Asset management (WebP/SVG formats, 2×/3× variants, **per-platform asset bundling new in Flutter 3.41**, font licensing). Testing levels per profile, plus Widget Previews (stable since Flutter 3.47). CI minimum and production checklists. Git hygiene. Documentation requirements. AI coding assistant instructions. The Definition of Done checklists. Practical one-liner lessons.

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

You declare the active profiles in `docs/PROJECT_PROFILE.md` §2 (and repeat them in `architecture.md` §1). Rules marked "under Production App Extension" only activate once that profile is declared.

---

## What should you fill out before starting the project?

The engineering standard is **not a fill-in-the-blanks template**. You do not write into it. You **read it and make decisions from it**, then record those decisions in `architecture.md` and `security.md`.

### Part 1 — Decisions to make before writing code

| Priority | Section | Decision to Make and Record |
|----------|---------|----------------------------|
| 🔴 Must | **§1 Profiles** | Declare which profiles apply. Record in `docs/PROJECT_PROFILE.md` §2 and `architecture.md` §1. |
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

**Configure the framework localization delegates in `MaterialApp` and disable Play language splitting** — Section 8.1. Since Flutter 3.47 `GlobalMaterialLocalizations` comes from `material_ui` (its `.delegates` list also covers Cupertino and Widgets). Without these delegates, certain Material widgets render incorrectly on non-English system locales. In addition, `bundle.language.enableSplit = false` must be set in `android/app/build.gradle.kts` so Google Play does not strip non-system languages on download.

**Create `l10n.yaml` and one ARB file per declared language** — Section 8.2. ARB files are Flutter's version of Android's `strings.xml`. Put every user-visible string in the ARB files and read it with `AppLocalizations.of(context)`; no raw text inside widgets. Logs, non-UI exception messages, asset paths, route names, and map keys may stay as plain literals.

**Ship exactly the declared languages** — Section 8.3. The project lists its languages in `docs/PROJECT_PROFILE.md`; `supportedLocales` equals that list, and every key exists in every declared language's ARB file with a genuine translation — a feature whose strings exist only in the template language is not finished (Section 8.7, verified by the translation parity test across ARB files, `app_config.json`, and content assets). Two traps to know before you start: Flutter ships **no framework translation for some languages** (Sanskrit is one), so you must register the small fallback delegates in 8.3.1 or date pickers and dialogs will crash; and `intl` has no date/number data for some languages, so formatting falls back to the template language (8.3.2). Bundle fonts that cover every non-Latin script you declare, or those languages render as empty boxes on a clean device (8.3.3).

**Let the user pick the language inside the app** — Section 8.4. When two or more languages are declared, the app starts in the system language when it is one of them and in the template language otherwise, and Settings has a picker — System default plus each language in its own script — whose choice is saved, survives a restart, and applies immediately without one.

**Translation quality and language packs** — Section 8.5. Every non-template language is reviewed by a fluent reader; AI or machine translation is only a draft, and unsure strings are flagged in the change log. Never substitute a related language for a declared one. Languages that need strict extra rules have a **language pack** in `language_packs/` — for example `sanskrit_malayalam.md` holds the Sanskrit-not-Hindi rules and CI gate, the Malayalam vocabulary rules, and a shared tri-lingual UI glossary. A pack is mandatory only for projects that declare its languages.

**Keep menus and labels short; only prose may be long** — Section 8.6. Buttons, menus, tabs, labels and tooltips get a one-to-two-word budget in each language; help text, empty states, descriptions, and About row labels are exempt. Characters are visible grapheme clusters counted with `package:characters` (`string.characters.length`), so vowel signs and viramas count with their base letter. ARB key prefixes (`action…`, `label…`, `tooltip…` vs `desc…`, `help…`, `aboutDetail…`) make the rule checkable by a test.

**Every icon-only button needs a tooltip** — Section 7.8. An icon with no label is a guess for a sighted user and silence for a screen reader. The tooltip text comes from ARB like every other string, names the action rather than the icon, and a short widget test fails the build when one is missing.

**Run the dependency audit for offline apps** — Section 16.5. For fully offline apps, `dart pub deps --style=tree` must be run before adding any package. This verifies no transitive HTTP dependency is introduced. Architecturally mandatory, not optional.

**Create the baseline `docs/` set** — Section 21.1 and `DOCS_FOLDER_GUIDELINE.md` §6 list the required documents, including `docs/architecture.md` and `docs/security.md`. Fill out all 🔴 Must sections of `architecture.md` before coding begins.

---

## How can AI use this document? What do you need to do?

### How AI uses it

When `docs/guidelines/flutter_project_engineering_standard.md` is referenced in `CLAUDE.md`, an AI coding assistant reads it and gains a complete picture of *how* code should be written — not just what to build.

**Structure** — It knows the tier, avoids a second `utils/` folder, and never invents a second state management system.

**Data flow** — It knows Widget → State → Service → Repository → Datasource. It will not put SQL inside a widget or put navigation logic inside a service.

**Init order** — It knows logging starts first, `sqfliteFfiInit()` runs before the database opens, and the database opens before `runApp()`. Wrong ordering causes silent release-only crashes; the standard explains why.

**List rendering** — It knows `ListView(children:[...])` is prohibited for lists over ~20 items. It uses `ListView.builder` with `itemExtent` when item height is fixed.

**Error handling** — It knows to catch `SqliteException` at the repository layer, re-throw typed `AppException`, and never expose stack traces or exception class names to the user.

**Logging** — It knows `print` and `debugPrint` are banned in committed code, trace/debug must be gated by flavor config, and all error logs must include the `error` object and `stackTrace`.

**Animation** — It knows `Curves.easeOut` for entering elements, `Curves.easeIn` for leaving elements, `MediaQuery.disableAnimations` must be respected, and that animating layout-affecting properties (`Padding`, `SizedBox` dimensions, `Align` factors) is permitted but requires profiling — `Transform` and `Opacity` are preferred by default because they are GPU-composited and skip layout/paint passes.

**Code generation** — It knows whether generated files are committed or excluded in your project, and will regenerate them after modifying annotated source.

**Definition of done** — It knows a task is not complete until `flutter analyze` is clean, tests are updated, generated files are current, and no secrets are staged.

Without the standard, the AI applies its own defaults — which may be inconsistent with your project's patterns and which can vary between sessions.

### What you need to do

**Step 1 — Use it from the guidelines submodule (do not copy it)**
```
<project_root>/docs/guidelines/flutter_project_engineering_standard.md
```

**Step 2 — Add a rule to `CLAUDE.md`**

```
Rule N: Before writing any code, read docs/guidelines/flutter_project_engineering_standard.md.
        Active profiles: Core Baseline, Production App Extension.

  Structure: Follow architecture.md §4 tier. No second state-management system.
             No packages introducing transitive HTTP or network activity.

  Code rules (every task):
  - Use ListView.builder for any list that is unbounded or can exceed 20 items.
  - Never use print() or debugPrint(); use AppLogger.
  - Prefer Transform/Opacity over animating layout-affecting properties
    (Padding, SizedBox dimensions, Align factors). Animating layout properties
    is permitted with profiling evidence; the default is to avoid it.
  - Never put SQL, encryption, or HTTP knowledge inside a widget.
  - Always add const to constructors and widget instantiations where possible.
  - Always add a Semantics label to custom interactive widgets.
  - Move work that can drop a frame (large JSON parsing, crypto on big payloads)
    off the main isolate with compute() or an Isolate; profile first (§10.5).

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

Once referenced in `CLAUDE.md` (at `docs/guidelines/`), every future session reads it alongside `architecture.md` and `security.md`. You do not re-explain it. Per task you indicate which sections are most relevant — this focuses the AI precisely rather than asking it to apply all 24 sections equally to every small change.

---

## How this fits with the other documents

`README.md` lists every document in the guideline set and which ones apply to an app, by profile.
In short: the **references** (engineering standard, flavors guide, store gates, `guideline.md`) say
*how* to build and ship; the app's filled-in **templates** in `docs/` (`PROJECT_PROFILE.md`,
`architecture.md`, `security.md`, `release_process.md`) record *what this app decided*.
