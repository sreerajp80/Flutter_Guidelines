# Change Log: Dynamic About-screen details

**Implements plan:** `plans/20260712_163714_dynamic-about-details.md`
**Date:** 2026-07-12

## What was done

Updated `guideline.md` to require the About screen to be data-driven.

### Files changed

- `guideline.md`:
  - Added **§1.6 About screen — render `details` dynamically**: the About screen MUST loop
    over `AppConfig.details` and render one row per entry (skipping empty key/value),
    never hard-coding field names. Any field added to `details` in `app_config.json`
    appears automatically. Includes a reference snippet from the reference Text App and an
    optional `mailto:` nicety for an `email` key.
  - Added one line to the §4 checklist covering dynamic `details` rendering.

### Not changed

No app source files were modified. The reference Text App already implements this pattern;
the guideline now states it as a rule for all apps.
