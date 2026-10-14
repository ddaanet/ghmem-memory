---
name: ledger-schema-changes-are-free
description: "changing a `corpus/fetch.jsonl` or `corpus/routing.jsonl` event shape, or a sqlite model: no migration, no optional field, no back-compat for landed entries — `fetch.jsonl` regenerates by a full re-sync, `routing.jsonl` is the one file that cannot be regenerated"
metadata:
  type: feedback
---

A ledger or corpus schema change carries no back-compat: no optional field
kept for entries already landed, no migration, no reader that accepts both
shapes. My human partner's call.

What regeneration costs decides how freely each file changes:

- The sqlite mirror is a derived cache that `ghmem rebuild` recreates from
  the ledgers. Free.
- `corpus/fetch.jsonl` is regenerable from GitHub at the price of a full
  re-sync — about 30k events on the micro corpus, one rate-limit window.
- `corpus/routing.jsonl` is the one file a schema change cannot undo: the
  verdicts it records come from triage runs, not from GitHub.
