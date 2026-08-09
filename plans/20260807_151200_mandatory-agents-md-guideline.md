# Plan: Mandatory AGENTS.md Guideline & Multi-LLM Standard

**Date:** 2026-08-07  
**Status:** Approved  

## Goal
Establish `AGENTS.md` as a mandatory Core Baseline requirement (**MUST**) alongside `CLAUDE.md` for all Flutter projects. Create `AGENTS_MD_GUIDELINE.md` to define standards for `AGENTS.md` targeted at non-Claude AI coding tools and LLMs (Gemini, Antigravity, Cursor, Windsurf, Codex, etc.), and update all guideline index documents and standards to enforce this requirement.

## Proposed Changes

1. **New Guideline Document**:
   - `AGENTS_MD_GUIDELINE.md`: Comprehensive guideline for creating and maintaining project-root `AGENTS.md` files (Thin vs Thick profiles, section order, template, dual-file alignment with `CLAUDE.md`).

2. **Master Guideline Updates**:
   - `CLAUDE_MD_GUIDELINE.md`: Cross-reference `AGENTS_MD_GUIDELINE.md` and state that both `CLAUDE.md` and `AGENTS.md` are mandatory (**MUST**).
   - `GUIDELINES_MANIFEST.md`: Index `AGENTS_MD_GUIDELINE.md` in Core documents, update Core Baseline profile matrix and startup guide.
   - `README.md`: Index `AGENTS_MD_GUIDELINE.md` in document table, update Core Baseline matrix and startup instructions.
   - `guideline.md`: Update §3 (Root layout) and §4 (Quick checklist) to make root `AGENTS.md` mandatory (**MUST**).
   - `flutter_project_engineering_standard.md`: Update §21.1 (Required Documents) and §22.1 (Before Writing Code) to include `AGENTS.md` (via `AGENTS_MD_GUIDELINE.md`) as mandatory (**MUST**).
   - `DOCS_FOLDER_GUIDELINE.md`: Update intro to state pairing with both `CLAUDE_MD_GUIDELINE.md` and `AGENTS_MD_GUIDELINE.md`.

3. **Change Log**:
   - `change_log/20260807_151200_mandatory-agents-md-guideline.md`: Document all created and updated files.
