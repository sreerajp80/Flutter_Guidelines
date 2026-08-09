# Plan: Mandate CLAUDE.md & Enforce Documentation Consistency

**Status:** Completed

## Objective
Ensure that `CLAUDE.md` guideline is established as a mandatory requirement (**MUST**) for all Flutter projects across all guideline files (`CLAUDE_MD_GUIDELINE.md`, `flutter_project_engineering_standard.md`, `GUIDELINES_MANIFEST.md`, `README.md`, `guideline.md`, `DOCS_FOLDER_GUIDELINE.md`). Eliminate documentation contradictions and remove unnecessary redundant information.

## Proposed Changes

### Guideline & Standard Updates
1. `CLAUDE_MD_GUIDELINE.md`:
   - Declare root `CLAUDE.md` a mandatory requirement (**MUST**) for all Flutter apps (new and migrated).
   - Mark `CLAUDE.md` creation and its required sections as a MUST in §3 checklist.

2. `flutter_project_engineering_standard.md`:
   - Move `CLAUDE.md` from Section 21.2 ("Recommended Documents") to Section 21.1 ("Required Documents For App Repositories") as a mandatory requirement (**MUST**).
   - Update Section 3.3 ("Recommended Root Layout For App Repositories") to include `CLAUDE.md` (mandatory root AI rules file), `assets/config/app_config.json`, `plans/`, `change_log/`, and `docs/GUIDELINES_MANIFEST.md`.
   - Update Section 22.1 ("AI Coding Assistant Instructions") to mandate that AI coding assistants read and strictly adhere to the project's root `CLAUDE.md`.

3. `GUIDELINES_MANIFEST.md`:
   - Update Core documents table, Core Baseline profile definition, and "Where to start" section to emphasize that root `CLAUDE.md` (via `CLAUDE_MD_GUIDELINE.md`) is a mandatory Core Baseline requirement (**MUST**).

4. `README.md`:
   - Update document index, startup instructions, and profile matrix to explicitly list `CLAUDE.md` creation/maintenance per `CLAUDE_MD_GUIDELINE.md` as mandatory (**MUST**).

5. `guideline.md`:
   - Update §3 (Root & `lib/` layout) and §4 (Checklist) to include the root `CLAUDE.md` following `CLAUDE_MD_GUIDELINE.md` as a mandatory requirement (**MUST**).

6. `DOCS_FOLDER_GUIDELINE.md`:
   - Update references to state that `CLAUDE.md` is mandatory at the project root.

## Verification
- Verified that all documentation files align with 0 contradictions regarding `CLAUDE.md` being a mandatory requirement (MUST).
- Verified that no redundant information or dead links exist across all guidelines.
