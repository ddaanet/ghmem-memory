---
name: sqlalchemy-uow-hides-mutations
description: "a planned mutation reds no test; same-PK delete+add emits one `UPDATE`; `Session.merge` autoflushes unconditionally; `session.execute` of an `on_conflict_do_update` leaves the identity map stale and the weak map hides it — `populate_existing`; `session.execute` vs `session.connection().execute`"
metadata:
  type: reference
---

SQLAlchemy behaviours that make a plausible reducer defect impossible to
inject, or invisible once injected. Measured against `sqlalchemy 2.0.51` on
the routing reducer (`src/ghmem/routing_reduce.py`).

**Same-PK delete-then-add within one flush collapses into a single `UPDATE`.**
`session.delete(obj)` followed by `session.add(SameClass(pk=same))` emits no
`DELETE` and no `INSERT` at all, for any mapped class. The classic hazard — a
pending `DELETE` landing behind an `INSERT` and wiping the row — is therefore
not constructible through the ORM. Reaching it requires *leaving* the unit of
work: an insert issued on `session.connection()` while an ORM `DELETE` is
pending does lose the row, and that is the only construction that produces it.

**`Session.merge(load=True)` autoflushes unconditionally** — `if load:
self._autoflush()`, not conditional on an identity-map miss. Any pending write
is already on the connection by the time the merge's own statement runs.
`session.get` autoflushes only when it must emit a `SELECT`, which it does
whenever the caller holds no reference to the row (the identity map is weak).
So an ordering that looks saved by `get` is usually saved by `merge` one call
earlier; relying on `get` is incidental, relying on `merge` is not.

**A Core upsert writes past the ORM, and the weak identity map hides it.**
`session.execute(sqlite_insert(Entity)...on_conflict_do_update(...))` does not
refresh an identity-mapped instance: the row changes in the database while the
instance keeps its old value *and* stays persistent-clean, so a later
assignment of that old value emits no `UPDATE` at all. Fix with
`.returning(Entity).execution_options(populate_existing=True)` and
`session.scalars(stmt).one()` — also the only ORM-coherent upsert SQLAlchemy
offers, since there is no dialect-agnostic one and every plain ORM form is the
read-then-write an upsert exists to avoid. It is near-invisible because the
identity map is weak: an instance no caller keeps is collected between two
events of one session, and the next `get` re-reads the true row, so a test
over the realistic sequence passes whatever the code does. Pin the instances —
`event.listen(Entity, "load", …)` appending to a list, scoped by a context
manager — and it reproduces.

**How to apply.** Run a planned mutation before writing prose asserting what
it discriminates — discrimination is a measurement, never a deduction from how
the code reads. A mutation that reds nothing has two readings that look
identical in the pytest output and lead opposite ways: a weak test gets
strengthened, an impossible defect gets its *claim* retired while the test may
still earn its place on the coverage it genuinely has. So probe whether the
defect is reachable at all before touching the test — and when the framework
is what saves the code, name which mechanism, because "collected in time" and
"written correctly" are different guarantees.
