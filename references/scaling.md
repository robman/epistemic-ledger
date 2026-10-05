# Workspace Scaling

The workspace grows in three steps. At each step, existing files keep their paths and their role as entry points; new layers are added beneath them and nothing is replaced. Record the current mode in the Workspace Conventions section of `ledger.md`.

Scale storage only when the current structure causes operational problems. Storage, coordination, and review scale independently: a small ledger may need high-assurance review, and a large one may hold mostly low-risk evidence. Review strength and agent patterns are covered in `collaboration.md`; this file covers storage only.

---

## Small Mode

```text
epistemic/
  ledger.md
```

Everything lives in `ledger.md`, in sections for conventions, questions, hypotheses, claims, evidence, and decisions (example in `record-formats.md` section 3). Stay here while the whole state is easy to read in one pass and update in one edit. Most investigations never leave Small Mode, and a dev/critic workflow does not by itself require more files.

---

## Medium Mode

```text
epistemic/
  ledger.md
  questions.md
  hypotheses.md
  claims.md
  evidence.md
  decisions.md
```

Each category file holds many records. `ledger.md` keeps the Workspace Conventions and becomes a short current-state summary that routes to the category files:

```markdown
# Epistemic Ledger

## Workspace Conventions
- Mode: medium
- ...

## Current View
Primary hypothesis H017 (moderate). C012 authorizes remediation planning. Q004 open.

## Records
- [Questions](questions.md)
- [Hypotheses](hypotheses.md)
- [Claims](claims.md)
- [Evidence](evidence.md)
- [Decisions](decisions.md)
```

Keep the Current View to a few lines. It answers "where do things stand?" at a glance; detail belongs in the records.

---

## Large Mode

```text
epistemic/
  ledger.md
  hypotheses.md
  hypotheses/
    H017.md
    H018.md
  evidence.md
  evidence/
    E019.md
  ...
```

Category files become compact indexes and record detail moves into per-record files:

```markdown
# Hypotheses

| ID | Summary | Question | Confidence | Status | Updated |
|---|---|---|---|---|---|
| [H017](hypotheses/H017.md) | Cache invalidation causes stale results | Q004 | moderate | active | 2026-10-04 |
| [H018](hypotheses/H018.md) | Replica lag causes stale results | Q004 | low | weakened | 2026-10-04 |
```

The Question column shows competing hypotheses without each record listing its rivals. Indexes summarize and are not authoritative: if an index and a record disagree, the record wins and the index is fixed. Keep indexes free of narrative and agent metadata.

Categories move to per-record files independently. If evidence is large but there are four hypotheses, split only evidence.

---

## When to move up

Use operational signals, not fixed counts.

**Small → Medium** when `ledger.md` is hard to scan, single edits keep touching unrelated sections, partial edits become error-prone, or several workstreams or agents regularly need different categories of state.

**Medium → Large** for a category when it is too large to navigate reliably, its records have long lifecycles or substantial challenge and provenance histories, record-level `git log` would be valuable, several writers may edit different records concurrently, or loading individual records would materially reduce context use.

When moving up:

1. move records verbatim;
2. keep every ID;
3. preserve revision history and provenance;
4. update the category index;
5. checkpoint the restructure as its own `workspace:` commit, without mixing in content changes.

---

## Stable entry points and read order

These paths never change once they exist:

```text
epistemic/ledger.md
epistemic/questions.md
epistemic/hypotheses.md
epistemic/claims.md
epistemic/evidence.md
epistemic/decisions.md
```

Read in this order, stopping as soon as you have what the task needs:

1. `epistemic/ledger.md`
2. the relevant category file
3. individual record files

Loading the whole workspace for a task that touches two records wastes context. The routing layers exist to make selective loading possible.

---

## Atomic updates

Keep the workspace consistent after every edit:

- **Small Mode:** one edit to `ledger.md`, applying related changes together (new evidence, the affected hypothesis revision, the Current View).
- **Medium Mode:** the smallest set of category files, usually one or two, plus the Current View if it changed.
- **Large Mode:** the primary record, then its index, then directly affected dependent records, then the Current View, then checkpoint.

Never edit extra files just to maintain backlinks; outward-only references make that unnecessary. When several agents are involved, have them produce results and let one coordinator apply each update atomically (`collaboration.md` sections 4 and 8).
