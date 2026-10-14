---
name: graphql-query-live-validation
description: "editing a GraphQL query string in `src/ghmem/github.py` or a `sync_*.py` module: fixture tests decode a canned response and pass on a malformed query — validate against the live API (one point per call); after a move-only edit, `ast`-extract the query constants from both trees and diff token streams"
metadata:
  type: feedback
---

A green suite says nothing about an edited GraphQL query. The fixture tests
decode a canned response, so they never send the query string and cannot see
a malformed one; the `e2e` marker that would is excluded from the gate and
from CI, which has no token.

Validate every edited query against the live API before committing it. A
query costs one rate-limit point per call, so validating each edited query
once is cheap.

A move-only edit — a query constant relocated between modules, reindented or
rewrapped — needs no network: extract the query constants from the old and
the new tree with `ast`, normalize each to a token stream, and diff the
streams. That is cheaper than a probe and catches an accidental reshaping a
probe would also accept.
