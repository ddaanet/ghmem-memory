---
name: docformatter-pin-and-wrap-dead-zone
description: "`just green` rewraps untouched committed files or breaks mid-hyphenated-word: docformatter 1.7.8, pin `docformatter==1.7.7`; a `--diff` run that edits the tree (`in-place = true` leaks); ruff **D205** on a summary docformatter just wrapped: one-line summaries of 81–88 chars have no fixed point between `wrap-summaries = 80` and line-length 88"
metadata:
  type: reference
---

## The gate rewraps files nobody touched

docformatter must be **1.7.7** (`uv tool install --python 3.13
docformatter==1.7.7`, as CI does). 1.7.8 rewraps every file in the repo,
committed and untouched ones included, and breaks lines in the middle of a
hyphenated word. `docformatter --version` settles which one is on PATH; the
justfile's `require-docformatter` guard checks presence only, and its install
hint names no version.

## `docformatter --diff` edits the tree

`[tool.docformatter] in-place = true` in `pyproject.toml` leaks into a
`--diff` run, which then edits the tree.

## D205 on a docstring the formatter produced

`wrap-summaries = 80` against ruff's line-length 88 leaves a dead zone with
no fixed point: a one-line summary of 81–88 characters passes ruff,
docformatter reflows it to two lines, and the wrapped form trips ruff
**D205**. Keep a one-line summary to 80 characters or fewer.

The 80 is deliberate and not to be raised to 88: it targets about 72
characters of prose at typical method indent, which docformatter cannot
express as a content width.
