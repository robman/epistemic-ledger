---
name: epistemic-ledger
description: Maintain a persistent, reviewable record of hypotheses, evidence, claims, open questions, and decisions in an epistemic/ Markdown workspace, versioned with Git when available. Use this for investigations that span many steps or sessions where conclusions may change, such as debugging with competing explanations, root-cause analysis, research with conflicting sources, architecture or design decisions made under uncertainty, or any work the user wants to be auditable, reviewable, or resumable later. Separates proposing a conclusion from challenging and testing it, using critics, subagents, tools, and executable tests where the harness supports them. Skip it for quick one-shot fixes, simple factual questions, and tasks without meaningful uncertainty.
compatibility: Requires a persistent filesystem. Git is preferred for history but optional. Works best in harnesses that support subagents, tools, or executable tests, but none are required.
---

# Epistemic Ledger

This skill keeps an explicit, durable record of what is currently believed, how strongly, on what basis, and how that changed over time. The record lives in Markdown files under `epistemic/`, and version history (Git when available) provides the timeline.

> **Markdown holds the epistemic state. Version history holds how it changed.**

The ledger lets anyone picking up the work later (a future session, a colleague, a reviewer, or another agent) see what is known, what is still open, what has been challenged, and why decisions were made, without rereading a transcript. It records durable state and the material transitions that changed it, not every reasoning step. It also separates *proposing* an explanation from *deciding whether it is safe to depend on*: where an independent check is available, no single reasoning pass should generate, evaluate, and authorize its own conclusions.

---

## Required reading

**Before doing any ledger work, read `references/collaboration.md`.** It defines the proposer, critic, executor, and coordinator roles and the challenge loop this skill depends on. It applies on every path, including a single agent in Small Mode, where one agent plays every role in sequence.

Read the other references when their trigger occurs:

- `references/record-formats.md`: before creating records in Medium or Large Mode, when unsure of a status or field, or before a first promotion or demotion.
- `references/history.md`: before the first commit, or when Git is unavailable.
- `references/scaling.md`: when `ledger.md` is getting hard to work with, or when more than one writer is editing records.

---

## Keep it proportionate

The ledger costs attention and context, so it pays off only when losing track of the state would hurt: several plausible explanations, evidence that might conflict, decisions later work will depend on, or a user who wants the reasoning to be auditable.

An investigation with three hypotheses needs a short `ledger.md`, not a directory tree. Create records for material findings only, never to fill out a template. Scale review the same way: a trivial change needs no second agent, while a load-bearing claim, risky change, or expensive decision may justify a separate critic, stronger tests, or model diversity.

---

## Starting work

**If `epistemic/ledger.md` exists**, read it first, including its Workspace Conventions section, and follow those conventions. They record choices the user already made, so do not ask again.

**If it does not exist**, ask the user before creating anything, because the workspace and any commits land in their repository. Propose defaults so they can confirm in one reply:

- **Location:** `epistemic/` at the repository root.
- **Tracking:** committed to the repo, or gitignored and kept local.
- **Commits:** whether you may commit at all, and if so, on the current branch or a dedicated one.

Then create `epistemic/ledger.md` with a Workspace Conventions section recording the answers, begin in Small Mode (everything in `ledger.md`), migrate only existing state that matters, mark anything reconstructed after the fact as reconstructed, and checkpoint if commits are allowed. Do not build the full directory structure up front.

```markdown
# Epistemic Ledger

## Workspace Conventions

- Mode: small
- Confidence scale: qualitative (low / moderate / high)
- History: git; epistemic commits allowed on the current branch, kept separate from code commits
- Tracked in repo: yes
- Writers: single coordinator
- Review: dev/critic challenge for load-bearing claims
```

---

## The five record types

| Ask yourself | Record type |
|---|---|
| Did I observe or retrieve something? | **Evidence** (E) |
| Do I have a candidate explanation? | **Hypothesis** (H) |
| Is this safe for other work to build on? | **Claim** (C) |
| Is something materially unanswered? | **Question** (Q) |
| Are we committing to a direction or spending resources? | **Decision** (D) |

> **Evidence is observed; hypotheses explain; claims authorize dependency; decisions commit action; questions mark what is missing.**

An assumption that materially affects further work is recorded as a hypothesis, so it stays visible and testable instead of quietly becoming a premise.

Agent roles, critiques, tests, and review passes are **not** record types. They are processes that generate, challenge, or validate records. A critic's objection is not evidence: it may become a hypothesis, a question, or a challenge on an existing hypothesis, and the result of any test it motivates may become evidence. Test and run identifiers such as `T021` are external references recorded as provenance, not a sixth ID namespace.

### The hypothesis/claim boundary is dependency

A **hypothesis** is still under test or comparison. A **claim** is a proposition that downstream reasoning, implementation, or decisions are currently authorized to rely on, within a stated scope and with stated limitations. Authorization is not settlement: a claim remains open to challenge, carries the conditions under which it should be revisited, and is demoted or rejected when contested or refuted. The test is "is it safe for other work to build on this, for now?", not "is confidence high?" or "is this proven?" A well-supported hypothesis can remain a hypothesis if building on it is still premature.

### Observations are not interpretations

Evidence records what was observed, measured, retrieved, or reported. What it is thought to imply goes in a separate Interpretation section.

```markdown
## Observation
The service returned HTTP 503 for 18 of 100 requests.

## Interpretation
Consistent with upstream saturation, but does not by itself establish the cause.
```

Recording "the upstream service was overloaded" as the observation, when overload was never measured, is the most common way the ledger goes wrong: an inference becomes a fact and later reasoning treats it as settled. The same boundary stops model interpretation, critic commentary, and agreement between agents from being laundered into evidence.

---

## Principles

**Preserve uncertainty.** Keep plausible alternatives as separate hypotheses and open a question when information is missing. Collapsing alternatives early is the failure this skill exists to prevent.

**Record why beliefs changed.** Every status or confidence change names its cause: new or corrected evidence, a test result, a borne-out or failed prediction, a contradiction, a challenge, a scope change, or an invalidated source. Time passing and agents agreeing are not causes.

**Never destroy history.** Do not delete a material record because it is no longer current; change its status. Every material prior state must stay recoverable through Git, host history, or a changelog.

**Keep provenance, and never invent it.** Evidence identifies its source well enough to verify. Other records link to what they rest on. If provenance is incomplete, say so; a fabricated source is worse than a missing one.

**Point references outward.** Links live in the record doing the depending, so nothing must be kept in sync elsewhere:

```text
Hypothesis → Evidence, and the Question it answers
Claim      → Hypothesis, Evidence
Decision   → Claim, Hypothesis, Evidence, Question
Evidence   → (nothing required)
Question   → (nothing required)
```

Competing hypotheses are grouped by pointing at a shared question, not by listing each other.

**Scale by indirection, not replacement.** `epistemic/ledger.md` is always the entry point. As the workspace grows it points to category files, and those point to per-record files, but nothing a reader already relies on moves.

**Aim for reconstructability, not verbosity.** Record enough that a capable reviewer could reconstruct what was known, what alternatives existed, what challenges were raised, what was tested, and why each conclusion followed. Leave out narration that does not serve that.

---

## Challenge and verification

`references/collaboration.md` covers this in full. The rules that matter on every path:

```text
question → candidate hypotheses → proposal → independent challenge
        → discriminating test → observed evidence → belief revision
        → claim, or continued uncertainty
```

- **Separate proposal from challenge.** Use a distinct critic, subagent, test, or tool check where the harness allows. Working alone, switch roles explicitly and write predictions down before checking them.
- **Ask critics to falsify, not verify.** "Try to falsify H017, identify alternative explanations, and propose observations that would discriminate between them" is far stronger than "verify that H017 is correct."
- **Tests outrank agreement.** Agreement between agents shows a point was considered; it is not evidence for the proposition. When something can be executed, measured, or retrieved, prefer the observed result.
- **A passing test establishes only what it observes,** within its environment and scope. A failing test may weaken a hypothesis without establishing any particular alternative.
- **Never resolve disagreement by vote.** Turn plausible alternatives into hypotheses, missing information into questions, and test results into evidence, then seek a discriminating observation.

---

## Promotion and demotion

These transitions are where the ledger earns its keep, so make them explicit.

**Promote a hypothesis to a claim** when its scope is clear, its important limitations are known, credible alternatives and contradictions have been addressed, and the remaining uncertainty is acceptable for the work that will depend on it. Load-bearing claims should also rest on at least one of: a discriminating test, a borne-out prediction, externally measured or retrieved evidence, or an independent challenge that found no material unresolved objection. Proposer and critic agreeing is never sufficient on its own.

1. Create a claim with a new ID, `depends_on` the hypothesis and key evidence, and explicit Scope, Known Limitations, and Revisit If sections.
2. Set the hypothesis to `promoted` with a revision note naming the claim.
3. Where challenge or verification supported promotion, record enough to reconstruct it.

**Demote a claim** when contradictory evidence, an overstated scope, unreliable provenance, a failed adversarial test, or a failed assumption undermines reliance on it.

1. Set the claim to `demoted` and record why dependency was withdrawn.
2. Reactivate the hypothesis (`active` or `weakened`) with a note pointing to the demotion. If the proposition needs rewording, create a new hypothesis that supersedes it instead of editing it.
3. Review dependent records, especially decisions, and note which need reconsideration.
4. Preserve the failed test, contradiction, or critic-discovered issue as evidence or an open question.

IDs are never reused. Re-promotion creates a new claim that supersedes the demoted one. A worked example is in `references/record-formats.md`.

---

## Confidence

Confidence is optional and never substitutes for evidence: a confident claim with weak provenance is still weakly grounded. Use one scale per workspace, recorded in the conventions; qualitative (`low`, `moderate`, `high`) is the default. Numeric credences for competing hypotheses need not sum to 1 unless their shared question declares them exhaustive and mutually exclusive. Record every material change with old value, new value, and cause.

---

## Conflicts and corrections

| Situation | Action |
|---|---|
| Evidence conflicts | Keep both. Check scope, timing, method, environment, and source quality. Update affected hypotheses and claims; open a question if unresolved. Never average disagreement away. |
| Agents or critics disagree | Identify the disputed proposition and differing assumptions, then seek a discriminating observation. If none is practical, preserve the alternatives as hypotheses or a question. Reviewer count is not evidence. |
| A source turns out wrong | Mark its evidence `invalidated`, `superseded`, or `disputed` with the reason, add corrected evidence if available, and follow the dependency chain. |
| A test did not test what was assumed | Do not silently reinterpret it. Clarify the limitation or supersede the evidence, then review what depended on it. |
| Two current claims contradict | Check scope, staleness, and evidence quality. Demote one if warranted, or open a question. Keep the contradiction visible. |

---

## Operating loop

1. **Frame the material question** so competing explanations can answer the same thing.
2. **Generate candidate hypotheses** instead of letting the first plausible explanation become an unstated premise. State predictions and falsifiers for material ones.
3. **Challenge** important propositions, in proportion to their consequences.
4. **Run discriminating checks**, preferring tests, tools, measurement, and retrieval over model agreement.
5. **Record material evidence** with provenance, keeping observation, interpretation, and limitations apart.
6. **Update affected hypotheses** with the cause of each change, then promote or demote when "is this safe to build on?" changes.
7. **Record decisions** with their basis, rejected alternatives, accepted risks, material dissent, and revisit conditions. A decision can be sound while uncertainty remains, provided the record says which uncertainty was accepted.
8. **Update `ledger.md`** and any touched index, then checkpoint at meaningful transitions if the conventions allow.

Use the smallest process that preserves the epistemic integrity of the work, and make edits in an order that keeps the workspace consistent: primary record, then index, then checkpoint.

---

## Before finishing substantial work

Fix the workspace before stopping if any of these fail:

- Material findings and status or confidence changes are recorded with their causes.
- Evidence has provenance or states its gaps, and observations are kept apart from interpretations.
- Credible alternatives have not been silently collapsed, and material critic objections are resolved or visible.
- Claims show scope and limitations; none remains authorized after it should have been demoted.
- Load-bearing claims received proportionate challenge or verification.
- Decisions link to their basis and state revisit conditions; important open questions are visible.
- `ledger.md` reflects the current view, and material changes are checkpointed.

**The reconstruction test:** could a capable reviewer, using only the workspace and its referenced sources, reconstruct what was proposed, what alternatives existed, what challenges were raised, what was tested, what evidence resulted, and why the current conclusions were authorized? Common gaps are unrecorded tool output, unstated assumptions, missing alternatives, hidden disagreement, unexplained confidence changes, and external data whose version was not captured.

---

## Core invariants

> **Evidence is observed; hypotheses explain; claims authorize dependency; decisions commit action; questions mark what is missing.**

> **Proposal and challenge are separate roles, even when one agent plays both.**

> **Tests and observations outrank agent agreement.**

> **References point outward from the record that depends on them.**

> **Scale by indirection, not replacement.**

> **Never destroy epistemic history. Keep a concise current view, preserve how it changed, and retain enough provenance to reconstruct any material point-in-time state.**
