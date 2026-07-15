# Plan: Single "guidelines manifest" document with full paths

**Status:** completed

## What the user wants

A **single document** that lists the **relative paths** to every guideline document in
this repository. The user will drop this one file into each of their Flutter projects. From
there, that project (and Claude working in it) can point back to the master source-of-truth
guidelines living in the repository root, keeping every app consistent.

## The issue

Today the guidelines live only inside this repository. Other projects have no single, portable
pointer back to them. Copying all documents into every app is heavy and drifts out of date. A
lightweight manifest file that carries the canonical relative paths solves this: one file to copy,
always pointing at the live source.

## Files to be changed

- **New:** `GUIDELINES_MANIFEST.md` — the single portable
  document. Contains:
  - A short "how to use this file" note (copy it into your project root, reference it from
    the project's `CLAUDE.md`).
  - A table of every guideline document with its **relative path** in this repository, plus
    a one-line description (reused from `README.md`).
  - The plain-English explainer paths in `docs/`.
  - The applicability-profile summary so an app knows which docs apply to it.
  - A note that these are relative paths to the master copies; when a doc is instead copied
    into an app's own `docs/`, the local copy wins.

## Plan for the content

1. List the seven core documents (guideline, engineering standard, architecture, build
   flavors, release process, security, my_flutter_apps) with relative paths + descriptions.
2. List the five `docs/*_README.md` explainers with relative paths.
3. Include the README index path itself.
4. Add the profile table (Core Baseline / Production App Extension / Sensitive Data Extension).
5. Add usage instructions at the top.

No existing files are modified. After approval and implementation, a change log will be written
to `change_log/`.

## Open question

Paths will be relative to the repository root. If you also
want a submodule setup description, I will include it.
