# Collaboration and Challenge

This file is required reading for every use of the skill. It defines the roles that separate proposing a conclusion from deciding whether it is safe to depend on, and how to run them with one agent or many.

Contents:

1. Roles
2. Working alone
3. Prompting and briefing critics
4. The coordinator's translation job
5. TDD and executable verification
6. Collaboration patterns
7. Selective context
8. Concurrency
9. Model and agent provenance

---

## 1. Roles

The roles are functions, not permanent agents. They can be separate subagents, separate prompts, different models or providers, humans, tools, or one agent switching stance deliberately.

**Proposer (dev)**

1. State the candidate explanation or intended behavior clearly.
2. Name the assumptions that materially affect it.
3. Make predictions that distinguish it from alternatives, and say what result would weaken or falsify it.
4. Propose the smallest useful test, experiment, retrieval, or implementation.
5. Do not promote your own proposition because the implementation worked.

**Critic (reviewer)**

1. Try to falsify the proposition, not improve its presentation.
2. Surface hidden assumptions, competing explanations, edge cases, counterexamples, and scope failures.
3. Check whether proposed tests actually discriminate between hypotheses, and propose stronger ones where warranted.
4. Keep objections distinct from observations, and do not treat disagreement itself as evidence.

**Executor**

Runs tests, tools, experiments, retrieval, and measurements. Its output becomes evidence only through normal provenance: what was run, against what artifact or version, in what environment, and what was observed.

**Coordinator**

Integrates outputs into the ledger (see section 4). Where several agents participate, prefer one logical coordinator as the only writer. This prevents duplicate records, ID collisions, inconsistent statuses, interpretation being recorded as evidence, and critics editing away the proposition they are reviewing. Reviewers rarely need write access.

---

## 2. Working alone

When one agent plays every role, the separation is weaker but still worth enforcing. Self-review is the weakest form of challenge, so do not rely on it alone for load-bearing claims when a subagent, test, or tool check is available.

To keep the roles distinct:

1. **Write predictions and falsifiers into the ledger before checking them.** A prediction recorded after the result is not a prediction.
2. **Switch to the critic stance explicitly.** Take the hypothesis as a target, list at least one competing explanation and one way the evidence could mislead, and record any that are material.
3. **Let the executor's output speak first.** Record the observation before writing the interpretation.
4. **Promote only on evidence,** never on having become convinced while proposing.

---

## 3. Prompting and briefing critics

Frame the task as falsification:

> Try to falsify H017, identify alternative explanations, and propose observations that would discriminate between them.

not:

> Verify that H017 is correct.

Give the critic enough context to understand the problem and inspect the artifact (the question, the proposition under challenge, the relevant evidence, known alternatives), but do not anchor it on the proposer's preferred conclusion or reasoning. Unnecessary framing produces agreement, not challenge.

---

## 4. The coordinator's translation job

The coordinator does not decide which agent sounds most convincing. It translates workflow output into the five record types:

- plausible alternatives → hypotheses;
- missing information → questions;
- objections → challenges on the hypotheses they bear on;
- test and tool results → evidence;
- dependency authorization → claims, only when the dependency test is satisfied.

Material disagreement is preserved until evidence resolves it. Never resolve it by majority vote.

```text
critic: "The edge cache may invalidate H017."

coordinator records:
  - a challenge on H017
  - Q009 if isolation is unresolved
  - T024 as the proposed discriminating test

executor runs T024.

coordinator records:
  - E061 with the observed result
  - a revision note on H017 citing E061
```

If a dev/critic exchange reduces to three material ledger changes, record those three changes, not the dialogue.

---

## 5. TDD and executable verification

In software work, hypotheses can often be turned into executable predictions, which makes test-driven development a particularly strong epistemic pattern.

```text
expected behavior → failing or discriminating test → implementation
  → critic / adversarial tests → execution → evidence → hypothesis or claim update
```

The proposer states the expected behavior before implementing, creates a test that currently fails for the relevant reason, makes the smallest change intended to satisfy it, and says what result would show the explanation was wrong.

The critic checks whether the test exercises the claimed failure mode, whether a different implementation or explanation could also satisfy it, whether the implementation merely overfits the test, and whether edge cases or regressions remain. Tests should encode intended behavior, not the proposed implementation.

**The observed result is the evidence.** The proposer's expectation and the critic's prediction remain hypotheses until checked. A test designed by a critic is not stronger for that reason; what matters is whether it discriminates and whether its execution is trustworthy.

Do not create an evidence record for every routine test pass. Record the epistemically material ones: a first reproduction, a discriminating result, a test that strengthens or weakens a claim, an adversarial test that exposes an assumption, or a later failure that forces demotion. A full regression run can be a single evidence item with provenance pointing to the run.

---

## 6. Collaboration patterns

Choose the lightest pattern that fits the epistemic risk. Review strength should scale with consequences, not with file count or the number of agents available.

| Pattern | Shape | Use when |
|---|---|---|
| Single agent | one agent, explicit role switches (section 2) | Low-risk, reversible, or well-tested work |
| Dev/critic pair | coordinator, dev, critic, executor | Default for material implementation or load-bearing claims |
| Multiple critics | as above, with critics on separate concerns | Correlated failure matters and the cost is justified |
| Cross-model or cross-provider | critics on different models or providers | Stubborn or high-stakes problems, subtle interpretation, weak executable checks, or same-model critics converging without resolving contradictory evidence |

A same-model dev/critic pair is still useful because role, context, and objective differ. Model diversity can reduce shared blind spots but never creates statistical independence, since models can share training data, assumptions, and failure modes. Treat independence as operational: a review is independent if it does not inherit the proposer's conclusion as a premise. A strong external test is often worth more than another model.

Use stronger review when a claim is load-bearing, a mistake would propagate widely, an action is expensive or hard to reverse, evidence is ambiguous, or executable verification is weak. Use lighter review when the claim is local and reversible or a test already discriminates the hypotheses directly.

Changing the harness, adding agents, or switching providers does not require restructuring the ledger. **The ledger models what is known, not the topology of the agents reasoning about it.**

---

## 7. Selective context

Give each role the smallest context that lets it do its job well. This saves context and is also an epistemic safeguard, since extra context can anchor reviewers on prior conclusions.

- **Proposer:** the question, active hypotheses, constraining claims, relevant evidence, and the artifact being changed.
- **Critic:** the question, the proposition under challenge, relevant evidence, known alternatives, and the artifact under review.
- **Executor:** the exact instruction, artifact version, environment and configuration, and what observations to capture.
- **Coordinator:** `ledger.md`, touched indexes, touched records, and the new outputs.

Do not load unrelated records "just in case."

---

## 8. Concurrency

Parallelize challenge and evidence gathering, not edits to shared state.

Good: critic A challenges implementation assumptions, critic B inspects test coverage, and an executor reproduces the failure, all at once.

Poor: three agents rewriting the same hypothesis record.

Prefer agents producing proposals and results that the coordinator applies as one atomic ledger update. If several writers must edit concurrently, follow the ID rules in `record-formats.md`.

---

## 9. Model and agent provenance

Agent, model, and provider identity are provenance, not epistemic authority. Record them when model diversity was used deliberately as a control, when a finding may depend on model-specific behavior, when the agent output is itself the object being studied, or when reproducibility requires it. Keep this metadata on the record that needs it, not in category indexes.

A model's statement is evidence only when the statement itself is what was observed: "Critic model B classified 17 of 20 cases as ambiguous" is evidence about the critic's output, not about the cases. Reviewer identity is never an ID namespace; there are no `R001` or `CRITIC-004` records.

Most reviews collapse into a challenge, a question, a proposed test, evidence, or a revision note. Keep a separate review artifact only when the review has durable value in its own right (a formal security review, a regulated audit, a large architectural review with unresolved dissent), and reference it from the relevant records rather than copying it into the ledger.
