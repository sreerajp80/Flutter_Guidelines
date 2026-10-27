# Change Log: Make the privacy policy rule clear (file vs URL)

Implements plan `plans/20261026_154922_privacy-policy-url-clarity.md`.

## Why

The engineering standard said "`PRIVACY.md` or hosted privacy policy", which read as if a repo
file alone was enough. Stores need a public URL. The guidelines also did not say how to use a
`PRIVACY.md` in a public GitHub repo as that URL, or that one URL must be used everywhere.

## What changed

| File | Change |
|---|---|
| `flutter_project_engineering_standard.md` | §21.1 required-files row: now "Privacy policy (text + public URL)". Stores need a URL; a repo file counts only if reachable at a public URL; `PRIVACY.md` in a public repo is a common setup. |
| `platform_store_readiness.md` | §1 "Privacy policy" rule: added "Where to host it" (website, GitHub Pages, or `PRIVACY.md` in a public repo via its branch URL), the "must be public" rule, what breaks the link, the "one URL everywhere" rule, and matching the store data disclosures. |
| `PROJECT_PROFILE_TEMPLATE.md` | "Privacy policy URL" row: added a GitHub URL example and the "same URL in `app_config.json` and every store console" reminder. |

## Not changed

- No app project.
- No privacy policy text template was added.
