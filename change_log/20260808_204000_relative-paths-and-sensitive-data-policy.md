# Change Log: Enforced Relative Paths & Sensitive Data Exclusion Policy

**Plan Reference:** [plans/20260808_204000_relative-paths-and-sensitive-data-policy.md](../plans/20260808_204000_relative-paths-and-sensitive-data-policy.md)

## Summary of Changes

Updated all core Flutter guideline documents to explicitly require that all change logs (`change_log/`) and plans (`plans/`) use **relative repository paths only** (never absolute local machine/system paths like `C:\...`, `l:\...`, or `file:///...`) and contain **no sensitive information** suitable for public internet sharing (secrets, API keys, tokens, passwords, keystore passphrases, local absolute paths, internal IPs, credentials, or PII).

### Files Modified

1. **`CLAUDE_MD_GUIDELINE.md`**:
   - Updated §4 (Workflow rules template), §5 (Verbatim sections), and §8 (Final self-check) to mandate relative paths and sensitive data sanitization in all plan and change log files.

2. **`AGENTS_MD_GUIDELINE.md`**:
   - Updated §4 (Workflow rules template), §6 (Verbatim sections), and §9 (Final self-check) to mandate relative paths and sensitive data sanitization for non-Claude LLMs and AI agents.

3. **`DOCS_FOLDER_GUIDELINE.md`**:
   - Updated §3 (Naming convention / path rules) to state that date-prefixed plan and change log entries are strictly governed by relative path and privacy requirements.

4. **`flutter_project_engineering_standard.md`**:
   - Updated Sections 21.1, 22.2, 22.3, and 23.1 (Definition of Done) to mandate relative repository paths only and zero sensitive data in `plans/` and `change_log/`.

5. **`guideline.md`**:
   - Updated §3 (Standard `lib/` layout rules) and §4 (Checklist) to enforce relative paths and zero sensitive data in plans and change logs.

6. **`change_log/20260807_150000_mandatory-claude-md-and-doc-consistency.md`**:
   - Replaced an absolute `file:///l:/...` link URI with a relative repository link (`../plans/...`).

## Verification
- Performed grep searches across all repository files for absolute paths (`C:\`, `l:\`, `file:///`) and sensitive data patterns to ensure 100% compliance.
