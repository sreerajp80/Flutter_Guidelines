# Change Log: Mandated CLAUDE.md & Enforced Documentation Consistency

**Plan Reference:** [plans/20260807_150000_mandatory-claude-md-and-doc-consistency.md](../plans/20260807_150000_mandatory-claude-md-and-doc-consistency.md)

## Summary of Changes

Updated all core Flutter guideline documents to establish `CLAUDE.md` as an explicit, non-negotiable **MUST** (mandatory core baseline requirement) across all Flutter projects, resolved documentation contradictions, and eliminated redundant descriptions.

### Files Modified

1. **`CLAUDE_MD_GUIDELINE.md`**:
   - Explicitly declared root `CLAUDE.md` a mandatory requirement (**MUST**) for all Flutter projects (both new and existing/migrated apps).
   - Updated §3 checklist to affirm that every Flutter app MUST have a root `CLAUDE.md` file containing all required baseline sections.

2. **`flutter_project_engineering_standard.md`**:
   - Moved `CLAUDE.md` from Section 21.2 ("Recommended Documents") to Section 21.1 ("Required Documents For App Repositories") as a mandatory requirement (MUST).
   - Updated Section 3.3 ("Recommended Root Layout For App Repositories") diagram to explicitly include `CLAUDE.md` (mandatory root AI rules file), `assets/config/app_config.json`, `plans/`, `change_log/`, and `docs/GUIDELINES_MANIFEST.md`.
   - Updated Section 22.1 ("AI Coding Assistant Instructions") to mandate that AI coding assistants read and strictly adhere to the project's root `CLAUDE.md`.

3. **`GUIDELINES_MANIFEST.md`**:
   - Updated Core documents table, Core Baseline profile definition, and "Where to start" section to emphasize that root `CLAUDE.md` (via `CLAUDE_MD_GUIDELINE.md`) is a mandatory Core Baseline requirement (**MUST**).

4. **`README.md`**:
   - Updated document index table, startup instructions, and profile matrix to explicitly list `CLAUDE.md` creation/maintenance per `CLAUDE_MD_GUIDELINE.md` as mandatory (**MUST**).

5. **`guideline.md`**:
   - Updated §3 (Root & `lib/` layout) and §4 (Checklist) to include the root `CLAUDE.md` following `CLAUDE_MD_GUIDELINE.md` as a mandatory requirement (**MUST**).

6. **`DOCS_FOLDER_GUIDELINE.md`**:
   - Updated references to state that `CLAUDE.md` is mandatory at the project root.

## Verification
- Audited all markdown files to confirm complete cross-document consistency, alignment on MUST requirements, valid relative links, and zero contradictions.
