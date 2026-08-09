# Change Log: Mandatory AGENTS.md Guideline & Multi-LLM Standard

**Date:** 2026-08-07  
**Plan:** `plans/20260807_151200_mandatory-agents-md-guideline.md`  

## Changes Made

1. **New Guideline Document**:
   - [AGENTS_MD_GUIDELINE.md](../AGENTS_MD_GUIDELINE.md): Comprehensive guideline for writing project `AGENTS.md` files for other LLMs and AI coding agents (Gemini, Antigravity, Cursor, Windsurf, Codex, etc.). Defines profile selection (Thin vs Thick), canonical section order, checklist, full template, and dual-file alignment with `CLAUDE.md`.

2. **Master Guideline Updates**:
   - [CLAUDE_MD_GUIDELINE.md](../CLAUDE_MD_GUIDELINE.md): Cross-referenced `AGENTS_MD_GUIDELINE.md` and clarified that every Flutter project MUST maintain both `CLAUDE.md` and `AGENTS.md` at root (**MUST**).
   - [GUIDELINES_MANIFEST.md](../GUIDELINES_MANIFEST.md): Added `AGENTS_MD_GUIDELINE.md` relative path to Core documents, updated Core Baseline profile matrix and startup guide.
   - [README.md](../README.md): Indexed `AGENTS_MD_GUIDELINE.md` in documents table, updated Core Baseline matrix and startup instructions.
   - [guideline.md](../guideline.md): Updated §3 layout rules and §4 quick checklist to enforce mandatory root `AGENTS.md` (**MUST**) per `AGENTS_MD_GUIDELINE.md`.
   - [flutter_project_engineering_standard.md](../flutter_project_engineering_standard.md): Updated §21.1 (Required Documents Table) and §22.1 (Before Writing Code) to include `AGENTS.md` (via `AGENTS_MD_GUIDELINE.md`) as mandatory (**MUST**).
   - [DOCS_FOLDER_GUIDELINE.md](../DOCS_FOLDER_GUIDELINE.md): Updated intro to state pairing with both `CLAUDE_MD_GUIDELINE.md` and `AGENTS_MD_GUIDELINE.md`.
