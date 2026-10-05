# History and Checkpoints

The goal is that any material past state of the workspace can be recovered and reviewed: which hypotheses were active, what evidence existed, which claims were authorized, which questions were open, which decisions had been made, what challenges had been raised, and why each proposition was promoted, weakened, demoted, or superseded.

Three mechanisms can provide this, in order of preference: Git, reliable host-provided version history, or `epistemic/CHANGELOG.md`. What matters is recoverable state, not the tool.

> **Checkpoint changed epistemic state, not conversational activity.**

---

## Consent and separation

Commits change the user's repository history, so follow the Workspace Conventions in `ledger.md`:

- If commits are not allowed, edit files only and list the changed epistemic files in your final summary.
- If commits are allowed only on a dedicated branch, never commit epistemic changes elsewhere.
- If the conventions do not say, ask before the first commit and record the answer.

Keep epistemic commits separate from code commits, even when they relate to the same change, unless the conventions explicitly ask for combined commits. The code commit records what changed in the artifact; the epistemic commit records what changed in the beliefs.

```text
fix: invalidate cache after successful writes      ← code
claim: promote H017 to C012 after T021              ← epistemic
```

---

## What to checkpoint

Commit at transitions a reviewer might want to return to:

- a material hypothesis is introduced, or a plausible alternative emerges;
- a challenge exposes an assumption, scope failure, or flawed test;
- important evidence is added, or a prediction succeeds or fails;
- confidence changes materially;
- a hypothesis is promoted, or a claim is weakened, demoted, rejected, or superseded;
- a decision is made, reversed, or narrowed;
- an important question is resolved;
- a source or evidence item is invalidated;
- the workspace changes mode;
- a natural boundary is reached: a session ends, an experiment batch finishes, or work is about to be handed off.

Do not checkpoint every model message, routine self-reflection, stylistic feedback, repeated agreement, trivial test runs, drafts that changed no state, or every subagent call. Bundle formatting-only changes with substantive work. If five critic comments produce one new question and one adversarial test, that is one checkpoint.

**Promotion and demotion should each be a distinct checkpoint.** They mark the boundary of what downstream work was authorized to depend on: before the commit it was not, after it was (or the reverse). A promotion commit includes the new claim, the hypothesis status change, the verification summary, and any Current View update. A demotion commit includes the claim status change, the reactivated hypothesis, the evidence that caused it, a review of dependent decisions, and the Current View. If reconsidering those decisions takes substantial new work, demote first and open a question for them. Never leave an invalid claim authorized because downstream updates are inconvenient.

---

## Preserve the arc

For important investigations, the most valuable history is the sequence showing how a conclusion survived challenge:

```text
hypothesis: add H017 cache invalidation hypothesis
hypothesis: challenge H017 edge-cache assumption
question: add Q009 and discriminating test T024
evidence: add E041 from T021
claim: promote H017 to C012
```

Do not collapse a materially important sequence into one final commit. Equally, do not split mechanically: if challenge, test, and evidence happen in one tightly coupled operation, one checkpoint is enough.

In TDD workflows, do not mirror every red/green/refactor cycle. Routine iterations belong in code history. Checkpoint when a test reproduces the phenomenon, discriminates between hypotheses, falsifies an explanation, materially strengthens a claim, exposes a regression that changes a decision, or forces a demotion.

In multi-agent workflows, record state transitions, not agent chronology. Two critics participating does not mean two commits. **The commit graph should describe epistemic evolution, not organizational topology.**

---

## Commit messages

The subject line names the epistemic transition, prefixed with the record type it most affects:

| Prefix | Use for |
|---|---|
| `hypothesis:` | adding, challenging, strengthening, weakening, rejecting, or superseding hypotheses |
| `evidence:` | adding, disputing, or invalidating evidence |
| `claim:` | promoting, weakening, demoting, or superseding claims |
| `decision:` | making, narrowing, superseding, or reversing decisions |
| `question:` | opening, blocking, or resolving questions |
| `workspace:` | conventions, mode changes, restructuring, reconstruction |

Good:

```text
hypothesis: add H017 cache invalidation hypothesis
hypothesis: weaken H018 after E002
evidence: add E041 from T021
evidence: invalidate E041 after test configuration error
claim: demote C012 after E052
decision: adopt explicit cache invalidation (D006)
question: resolve Q004
workspace: move evidence to large mode
```

Not useful, because they describe agent activity rather than state change:

```text
critic finished review
dev responded to critic
model-b agrees with model-a
update confidence
```

Use the commit body for which agent, model, test, or run originated the change, when that matters to reconstruction:

```text
hypothesis: add H024 cache/replication interaction

Raised in cross-model review by critic/model-b; not raised by the
model-a dev or critic. No supporting evidence yet.
```

Confidence changes always name their cause (`hypothesis: H017 moderate → high after E041`). Another agent agreeing, the same rationale repeated, time passing, or the proposer becoming more convinced are not causes. A review that merely approves or agrees normally produces no commit at all, and approval is never evidence even when the review is a required formal artifact.

---

## Historical integrity

Do not rewrite shared epistemic history. No force-pushing away material transitions, squashing proposal → challenge → test → revision sequences, deleting failed hypotheses, narrowing a past claim after the fact, or editing old evidence to hide an invalid test. Squashing "H017 strengthened, C012 promoted, C012 demoted after E052" into "final cache hypothesis" erases exactly what a reviewer needs.

Correct mistakes by adding records and status changes, not by editing the past. On a private branch that has not been shared, ordinary hygiene may still squash trivial noise, but never material transitions.

**Reconstructed history.** When the ledger is introduced after work has happened, reconstruct important prior state from commits, issues, test runs, transcripts, or design documents, and mark those records as reconstructed (see `record-formats.md` section 11). Never create backdated commits to simulate contemporaneous recording:

```text
workspace: reconstruct H017 and E019 from issue #42
```

---

## Tags and branches

Tag sparingly, for milestones someone will want to find again, and only if commits are allowed: `pre-decision-D006`, `epistemic-review-2026-10-04`, `post-incident-review`. Good points are before expensive or irreversible decisions, after formal review, and before deploying on a provisional claim.

Competing hypotheses belong side by side as records answering the same question, not on separate branches. Branch only when the work itself must diverge, for example two prototypes with incompatible experimental code, and still express both as alternatives under one question. Writers on separate branches use tagged IDs (`record-formats.md` section 1).

When branches merge, never resolve conflicting epistemic records by taking the newer side. Preserve all unique evidence, competing hypotheses, and material challenges; reconcile status changes explicitly against the evidence; and open a question wherever the merge exposes unresolved disagreement. If one branch has H017 `supported` and the other has H017 `weakened after E052`, check whether E052 is valid and record the reconciled status with a revision note.

---

## Changelog fallback

Without Git or reliable host history, append to `epistemic/CHANGELOG.md` for every transition listed under "What to checkpoint". Record epistemic consequences, not agent activity, and make entries specific enough that the state on each date can be reconstructed from the records and their revision notes.

```markdown
# Epistemic Changelog

## 2026-10-04
- Added Q004 (cause of intermittent stale reads), H017 (cache invalidation), H018 (replica lag).
- Critic review challenged whether existing tests fully bypass the edge cache; added T021.
- Added E041: 0/100 stale reads with full cache bypass and injected replica delay.
- H017 moderate → high; H018 moderate → low, both after E041.
- Promoted H017 to C012; accepted D006 (prototype explicit invalidation).

## 2026-10-09
- Added E052: stale reads in eu-west with all cache layers bypassed.
- Demoted C012 (scope overstated); reactivated H017 as weakened; added H023.
- D006 holds only for the previously tested region.
```

The changelog records transitions. It does not replace the structured current state in the records.

---

## Before checkpointing

Did the epistemic state actually change, and materially enough that someone might want this point back? Does the message describe the transition rather than the agent action? Are observations separate from interpretations, critic objections represented, test results linked to the hypotheses they affect, causes recorded for status and confidence changes, and dependent claims and decisions updated? Would the checkpoint make sense to someone who never saw the transcript? If not, fix the state first.

At any checkpoint, a reviewer should be able to answer: what was believed, what alternatives remained, what evidence existed, what challenges had been raised, what changed the state, why a claim was or was no longer safe to depend on, what decisions followed, and what uncertainty remained. If history cannot answer those, it records too little. If answering them means reading hundreds of agent messages, it records the wrong things.
