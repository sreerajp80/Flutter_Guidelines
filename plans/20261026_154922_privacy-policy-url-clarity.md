# Plan: Make the privacy policy rule clear (file vs URL)

**Status:** completed

## The issue

1. `flutter_project_engineering_standard.md` §21.1 (the required-files table) says
   "`PRIVACY.md` **or** hosted privacy policy". The "or" reads as if a file in the repo is enough
   by itself. Agents then ask "a URL or a file?". But `platform_store_readiness.md` and
   `PROJECT_PROFILE_TEMPLATE.md` both say the store needs a **public, reachable URL**. A file
   alone does not satisfy Google Play.
2. No guideline says that a `PRIVACY.md` in a **public** GitHub repo can serve as that URL, which
   link to use, or what breaks the link.
3. The guidelines do not say that the same URL must be used in every place (profile, About screen,
   store console).

## Files to change

| File | Change |
|---|---|
| `flutter_project_engineering_standard.md` | §21.1 table row: replace "`PRIVACY.md` or hosted privacy policy" with a row that says a public privacy policy **URL** is required, and that the text may live in `PRIVACY.md` in the repo. Point to `platform_store_readiness.md`. |
| `platform_store_readiness.md` | Under the shared "Privacy policy" rule (around line 39), add a short "Where to host it" note: options, the GitHub rules, and the "one URL everywhere" rule. |
| `PROJECT_PROFILE_TEMPLATE.md` | "Privacy policy URL" row: add an example of the GitHub form and a reminder that the same URL goes into `app_config.json` and the store consoles. |

## The fix in detail

### 1. Standard §21.1 row

From:

> `PRIVACY.md` or hosted privacy policy | Required for any app on a public store

To:

> Privacy policy (text + public URL) | Required for any app on a public store. Stores need a
> public **URL**; a file in the repo counts only if it is reachable at a public URL. A common setup
> is `PRIVACY.md` in a public repo, linked by its URL (`platform_store_readiness.md` §1).

### 2. New note in `platform_store_readiness.md`

> **Where to host it.** Any public `https://` page works. Common choices:
> - Your website, or GitHub Pages.
> - `PRIVACY.md` at the root of a **public** GitHub repo, linked by its branch URL:
>   `https://github.com/<owner>/<repo>/blob/<default-branch>/PRIVACY.md`.
>   Use the branch link, not a link to one commit, so later edits show up.
>
> Rules for any choice:
> - The page MUST be public: no sign-in, not a private repo, not a PDF download, and not limited
>   to some countries.
> - Do not rename or move the file, change the default branch name, or make the repo private —
>   any of these breaks the link, and a broken policy link is a store policy problem.
> - Use **one URL everywhere**: `docs/PROJECT_PROFILE.md`, `privacyPolicy` in
>   `assets/config/app_config.json` (About screen), and every store console.
> - The policy text MUST match the store data disclosures (Play Data safety, Apple App Privacy,
>   Microsoft Store privacy declaration).

### 3. Profile template row

> `<https://...>` — required before any public store release. Example for a public GitHub repo:
> `https://github.com/<owner>/<repo>/blob/main/PRIVACY.md`. Use the same URL in
> `app_config.json` (`privacyPolicy`) and every store console.

## Out of scope

- No app project is changed.
- No privacy policy text template is added (could be a later, separate change).

## After the change

Write a change log in `change_log/` and set this plan's status to `completed`.
