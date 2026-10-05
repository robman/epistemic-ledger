# Record Formats

Contents:

1. Identifiers
2. Frontmatter
3. Small Mode example
4. Hypothesis
5. Claim
6. Evidence
7. Question
8. Decision
9. Status vocabularies
10. Worked example: promotion and demotion
11. Retrospective records

Examples use the qualitative confidence scale. If the workspace uses a numeric scale, substitute values in [0, 1].

---

## 1. Identifiers

Use the prefixes H, C, E, Q, and D with zero-padded numbers: `H001`, `C014`, `E031`. These are the only five namespaces. Test, run, and experiment identifiers (`T021`, `EXP004`) are external references recorded as provenance, and reviewer or model identities are never IDs.

IDs are permanent. Never renumber or reuse one, even after rejection, supersession, or deletion of an accidental draft.

- **Single writer or coordinator** (preferred): take the next unused number in the category.
- **Multiple writers sharing a working copy:** reserve the ID by adding the index row (or the heading, in Small Mode) in the same edit that creates the record.
- **Multiple writers on separate branches or copies:** give each writer a short tag and include it in the ID (`H-a017`, `E-b011`). Record the tag-to-writer mapping in Workspace Conventions, and never renumber on merge.

---

## 2. Frontmatter

In Medium and Large Mode, records carry YAML frontmatter for machine-readable metadata, with human-readable meaning in the Markdown body. In Small Mode, plain `Status:` lines are easier to read.

| Field | Used by | Meaning |
|---|---|---|
| `id` | all | Permanent identifier |
| `type` | all | hypothesis, claim, evidence, question, decision |
| `status` | all | From the vocabulary for that type (section 9) |
| `confidence` | H, C | On the workspace's single scale |
| `created`, `updated` | all | ISO dates |
| `question` | H | The question this hypothesis answers; groups competing hypotheses |
| `evidence` | H | Supporting evidence |
| `challenges` | H | Evidence against, unresolved objections, or failed predictions |
| `depends_on` | C, D | Records relied upon |
| `supersedes` | any | Records this one replaces |
| `open_questions` | D | Questions accepted as unresolved at decision time |
| `priority` | Q | Optional: low, medium, high |
| `source_type` | E | experiment, document, tool-output, log, interview, test-run, and so on |
| `producer_role` | E | Optional: dev, critic, executor, human |
| `agent`, `provider` | E | Optional model or agent identity and provider, where it matters to provenance |
| `tool`, `run_id` | E | Optional tool and run, test, or job identifier |
| `reconstructed`, `original_date`, `provenance` | any | For retrospective records (section 11) |

Add optional fields only when they improve reconstructability. All link fields point outward from the record doing the depending; never add a field that requires another record to be updated in step.

---

## 3. Small Mode example

```markdown
# Epistemic Ledger

## Workspace Conventions

- Mode: small
- Confidence scale: qualitative (low / moderate / high)
- History: git; epistemic commits allowed on current branch
- Tracked in repo: yes
- Writers: single coordinator
- Review: dev/critic challenge for load-bearing claims

## Questions

### Q001: What is causing intermittent stale reads?
Status: open

## Hypotheses

### H001: Cache invalidation is failing after writes
Status: active · Confidence: moderate · Answers: Q001
Evidence: E001 · Challenges: E003
Prediction: bypassing the cache should eliminate stale reads.
Falsifier: stale reads persist with all cache layers bypassed.
Next test: reproduce with cache bypassed under production-like replica delay.

### H002: Replica lag is causing stale reads
Status: weakened · Confidence: low · Answers: Q001
Challenges: E002
Prediction: increasing replica delay should increase stale reads even with cache bypassed.
- 2026-10-04: moderate → low after E002.

## Claims

(none yet)

## Evidence

### E001: Cache bypass removes stale reads
Source: experiment EXP004, staging, git:abc123, 2026-10-04 · Producer: executor
Observation: 100/100 cache-bypassed reads returned fresh values.
Interpretation: consistent with the cache contributing to stale reads.
Limitations: staging traffic only.

### E002: Injected replica delay does not reproduce stale reads with cache bypassed
Source: experiment EXP005, staging, git:abc123, 2026-10-04 · Producer: executor
Observation: 0/100 reads stale with 500 ms replica delay and cache bypassed.
Limitations: single region tested.

### E003: Stale reads persist after selective cache bypass
Source: test T021, staging, git:def456, 2026-10-04 · Producer: executor
Observation: stale reads occurred when only one cache layer was bypassed.
Interpretation: challenges the scope of H001; full bypass not yet isolated.
Limitations: does not distinguish application cache from edge cache.

## Decisions

### D001: Prototype explicit invalidation
Status: proposed · Basis: H001, E001
Open question: Q002 (does the edge cache independently reproduce the failure?)
```

---

## 4. Hypothesis

```markdown
---
id: H017
type: hypothesis
status: active
confidence: moderate
created: 2026-10-04
updated: 2026-10-04
question: Q004
evidence: [E031, E034]
challenges: [E037]
---

# H017: Cache invalidation is causing stale results

## Proposition
Stale results are caused by cache invalidation failing after writes.

## Scope
Requests served from the shared application cache after successful writes.

## Assumptions
- The stale result is produced after the write completed successfully.
- The test path exercises the same cache behavior as the affected production path.

## Predictions
- Bypassing the cache should eliminate stale results.
- Fresh processes should reproduce the problem less often.

## Falsification Criteria
Materially weakened if stale results still occur with all relevant cache layers bypassed under otherwise equivalent conditions.

## Challenges
- E037 shows one stale read during a partial cache-bypass test.
- The edge cache has not yet been isolated from the application cache.

## Next Test
T021: full cache bypass under injected replica delay.

## Revision Notes
- 2026-10-04: low → moderate after E034.
```

Predictions are the most useful part of a hypothesis: one that predicts nothing distinguishable from its alternatives cannot be tested well. Falsification criteria need only say what result would materially weaken the proposition or force a narrower version.

Keep `Challenges` to evidence IDs and concise objections, never critic transcripts. If a critic raises an alternative, create a hypothesis; if it identifies missing information, create a question.

---

## 5. Claim

```markdown
---
id: C012
type: claim
status: provisional
confidence: high
created: 2026-10-04
updated: 2026-10-04
depends_on: [H017, E031, E034, E041]
supersedes: [C008]
---

# C012: Cache invalidation failure is the primary cause of stale reads

## Statement
The primary cause of the observed stale reads is failed cache invalidation after writes.

## Scope
The current application environment, on the tested request path.

## Known Limitations
Replica lag has not been ruled out in every deployment region.

## Verification
- H017 correctly predicted E041.
- T021 exercised complete cache bypass under injected replica delay.
- An independent critic found no unresolved alternative within the tested scope.
- Multi-region behavior remains unverified.

## Dependency Authorization
Safe to rely on when choosing the remediation strategy for the tested request path.

## Revisit If
- Stale reads persist with the cache bypassed.
- A region shows stale reads without cache involvement.
- The tested path proves unrepresentative of the production failure.
```

Claims are `provisional` by default; use `established` only once a claim has survived meaningful attempts to break it and its scope is well characterized. The Verification section is recommended for load-bearing claims. Never write "verified by critic": state what challenge was performed and what resulted. Strong external evidence can justify promotion without a critic, and a critic's approval cannot justify promotion when evidence is weak.

---

## 6. Evidence

```markdown
---
id: E019
type: evidence
status: valid
source_type: test-run
producer_role: executor
tool: pytest
run_id: T021
created: 2026-10-04
---

# E019: Full cache bypass removes stale-read behavior

## Source
Test: T021 · Environment: staging · Code version: git:abc123
Observed at: 2026-10-04T10:22:00Z
Command: `pytest tests/cache/test_stale_reads.py::test_full_cache_bypass`

## Observation
With all known cache layers bypassed, all 100 test writes were followed by fresh reads.

## Interpretation
Consistent with cache behavior materially contributing to stale reads.

## Limitations
- Did not reproduce production traffic volume.
- Only one deployment region was tested.
- Does not establish which cache layer caused the original failures.
```

**Evidence is append-mostly.** After creation, only complete provenance, fix clerical errors, change status, clarify limitations, or add correction notes. If the substantive observation changes, create a new record that supersedes the old one.

**Choose the useful subset of provenance:** source identity and type, author, source and retrieval timestamps, URL or path, commit or content hash, dataset version, configuration, environment, tool invocation, and run ID. For tests, capture what was run, against what version, in what environment, with what configuration, and what was observed.

**Agent output** is evidence only when the statement itself is the observation (see `collaboration.md` section 9). "The critic thinks race conditions are likely" is a hypothesis or interpretation; "running that path reproduced the failure 8/10 times" is evidence.

---

## 7. Question

```markdown
---
id: Q004
type: question
status: open
priority: medium
created: 2026-10-04
---

# Q004: What is causing intermittent stale reads?

## Why It Matters
Remediation differs completely depending on whether the cache or replication is at fault.

## Discriminating Evidence Needed
A test that independently varies cache participation and replica delay on the affected path.

## Resolution Method
1. Full cache bypass with normal replica behavior.
2. Full cache bypass with injected replica delay.
3. Normal cache with replication forced synchronous.

## Resolution
(filled in when resolved, citing the records that resolved it)
```

Questions mark missing information and act as the anchor competing hypotheses point to. If a question's hypotheses are exhaustive and mutually exclusive, say so, because it changes how numeric confidences can be read. For contentious questions, `Discriminating Evidence Needed` usually does more than another agent's opinion. An optional `Candidate Explanations` list can aid readability, but the authoritative grouping is the `question` field on each hypothesis.

---

## 8. Decision

```markdown
---
id: D006
type: decision
status: accepted
created: 2026-10-04
depends_on: [C012, E019]
open_questions: [Q007]
---

# D006: Replace passive cache expiry with explicit invalidation

## Decision
Implement explicit cache invalidation after successful writes.

## Basis
C012 is dependency-authorized for the affected request path.

## Alternatives Considered
- Shorter TTL: rejected; reduces but does not eliminate stale reads.
- Disable cache: rejected; disproportionate performance cost.
- Wait for multi-region analysis: rejected; evidence suffices for the scoped, reversible change.

## Review
A critic challenged whether E019 isolated the application cache from the edge cache; T024 resolved that within the current environment.

## Dissent
H018 remains plausible outside the tested region. This limits generality but does not block the scoped decision.

## Accepted Risk
Q007 (multi-region replica behavior) remains open.

## Revisit If
- Replica lag proves dominant in any affected region.
- Explicit invalidation fails to remove stale reads on the production path.
```

`Review` and `Dissent` are optional: use Review when a challenge changed or constrained the decision, and Dissent for unresolved alternatives that still bear on it. A decision should never rest on "three agents agreed" when its underlying claim is not otherwise authorized.

---

## 9. Status vocabularies

Use only these. If a status seems missing, the situation usually fits one of these with an explanatory note.

| Type | Statuses |
|---|---|
| Hypothesis | `active`, `supported`, `weakened`, `promoted`, `rejected`, `superseded` |
| Claim | `provisional`, `established`, `weakened`, `demoted`, `rejected`, `superseded` |
| Evidence | `valid`, `disputed`, `invalidated`, `superseded` |
| Question | `open`, `blocked`, `resolved`, `obsolete` |
| Decision | `proposed`, `accepted`, `superseded`, `reversed` |

- `supported` (H): strengthened by evidence but still under test or comparison.
- `promoted` (H): a claim now carries this proposition; the hypothesis stays as history.
- `weakened` (C): still authorized but under material pressure; review dependent work soon.
- `demoted` (C): dependency authorization withdrawn and the hypothesis reactivated.
- `rejected`: believed false within its stated scope.
- `disputed` (E): observation, provenance, or method materially contested but not yet invalidated.
- `blocked` (Q): cannot currently be answered; say what would unblock it.

Never add statuses such as `reviewed`, `verified`, or `critic-approved`. Review is process and provenance, not an epistemic state.

---

## 10. Worked example: promotion and demotion

**Start.** Q004 asks what causes stale reads. H017 (cache invalidation) and H018 (replica lag) are both `active`. E019 and E034 support H017, and the proposer suggests it is strong enough to guide remediation.

**Challenge.** A critic is asked to falsify H017 and propose discriminating observations. It finds that existing bypass tests may not disable the edge cache, and that replica delay was never varied independently. Neither is evidence: the first becomes a challenge on H017, the second motivates T021.

**Test.** T021 bypasses all cache layers while injecting 500 ms replica delay, producing E041: 0/100 stale reads, one region tested. E041 supports H017 and weakens H018 within that scope.

**Promote.** Create C012 with `depends_on: [H017, E019, E034, E041]` and a Verification section noting the borne-out prediction, the addressed edge-cache challenge, and the unverified multi-region scope. Set H017 to `promoted`:

```text
2026-10-04: promoted to C012 after E041 addressed the cache-isolation challenge.
```

D006 then records the remediation decision with `depends_on: [C012]`.

**Contradiction.** E052 later shows stale reads in eu-west with all cache layers bypassed.

**Demote.** Set C012 to `demoted` ("2026-10-09: demoted after E052; scope overstated"). Reactivate H017 as `weakened` with E052 in its challenges. Create H023 ("replica lag causes stale reads in eu-west", `question: Q004`, `evidence: [E052]`). Mark D006 `superseded`, or note that it holds only for the tested region.

**Further challenge.** A critic on a different model proposes H024: cache failure dominates generally, but replica lag independently causes the eu-west failures. That proposal is not evidence. Record H024 if it is distinct and testable, and design tests that discriminate it from H023.

**Re-promotion.** If later evidence supports the narrower scope, create C015 with `supersedes: [C012]`. C012 is never edited back to life.

No record was deleted and no ID reused, so history shows the full arc: proposal → challenge → test → evidence → promotion → contradiction → demotion → revised hypotheses.

---

## 11. Retrospective records

When a record is reconstructed after the fact, for example while initializing a ledger from earlier work, mark it:

```yaml
---
id: H017
type: hypothesis
status: active
reconstructed: 2026-10-04
original_date: approximately 2026-09-28
provenance: commit abc123 and issue #42
---
```

Presenting reconstructed history as contemporaneous misleads reviewers about what was known when. A reconstructed critic challenge is acceptable if earlier transcripts show it happened and it is marked as reconstructed. Never retroactively invent reviews, falsification criteria, tests, model diversity, or confidence values that did not exist at the time.
