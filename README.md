# Epistemic Ledger

A skill that keeps a persistent, Git-versioned record of hypotheses, evidence, claims, questions and decisions, so reasoning across agents and sessions stays auditable, challengeable and revisable.

> **Markdown holds the epistemic state. Version history holds how it changed.**

## Why

LLMs are increasingly used for work where it matters not only *what* gets concluded but *how*: debugging with competing explanations, root-cause analysis, research with conflicting sources, design decisions made under uncertainty. In that kind of work the usual failure isn't a lack of intelligence. It's losing track: an assumption quietly becomes a premise, an inference gets remembered as a fact, a plausible alternative disappears, and nobody can later say why a conclusion was trusted.

This skill gives an agent (or a team of agents, or a human working with them) an explicit, inspectable epistemic state that lives outside the model, separates proposing a conclusion from deciding whether it is safe to depend on, and lets tests and observations, not agent agreement, decide what changes.

The background argument is in the post [Don't be fooled - LLMs really do reason](https://flux.robman.fyi/p/dont-be-fooled-llms-really-do-reason): the LLM is a kernel for a larger inference platform, and that platform can be shaped to be trustworthy.

## How it works

The ledger records five kinds of thing:

| Record | Question it answers |
|---|---|
| **Evidence** (E) | What did we actually observe or retrieve? |
| **Hypothesis** (H) | What candidate explanations are we considering? |
| **Claim** (C) | What is other work currently authorised to depend on? |
| **Question** (Q) | What is materially unresolved? |
| **Decision** (D) | What did we commit to, on what basis? |

A few rules do most of the work:

- **Observations are kept apart from interpretations**, so an inference can't quietly harden into a fact.
- **Hypotheses state predictions and falsifiers**, so they can actually be tested.
- **Proposal and challenge are separate roles.** A proposer (dev) suggests, a critic tries to falsify, an executor runs the tests, and a coordinator records the results. One agent can play every role, but it switches between them explicitly.
- **Tests and observations outrank agent agreement.** More agents agreeing doesn't make anything true. Reality gets a vote.
- **Claims are revocable.** A claim is a licence to depend on something within a stated scope, with known limitations and conditions for revisiting it. It is demoted when new evidence contests it.
- **History is never destroyed.** Status changes, not deletions, and every belief change cites its cause.

As the skill puts it: *the ledger models what is known, not the topology of the agents reasoning about it.* Swap models, add critics or change providers, and the epistemic state stays put.

## Example

A small investigation lives entirely in one file, `epistemic/ledger.md`:

```markdown
### H001: Cache invalidation is failing after writes
Status: active · Confidence: moderate · Answers: Q001
Evidence: E001 · Challenges: E003
Prediction: bypassing the cache should eliminate stale reads.
Falsifier: stale reads persist with all cache layers bypassed.

### E003: Stale reads persist after selective cache bypass
Source: test T021, staging, git:def456 · Producer: executor
Observation: stale reads occurred when only one cache layer was bypassed.
Interpretation: challenges the scope of H001; full bypass not yet isolated.
Limitations: does not distinguish application cache from edge cache.
```

As the work grows, the workspace scales by indirection, from one file to category files to per-record files, without moving anything a reader already relies on.

## Install

The skill follows the [Agent Skills](https://agentskills.io) open standard, so it should work in any harness that supports skills. These instructions assume this repository's root is the skill folder.

**Claude Code (all your projects):**

```bash
git clone https://github.com/robman/epistemic-ledger ~/.claude/skills/epistemic-ledger
```

**Claude Code (one project only):**

```bash
git clone https://github.com/robman/epistemic-ledger .claude/skills/epistemic-ledger
```

If the `skills` directory didn't exist when your session started, restart Claude Code so it gets picked up. See the [Claude Code skills docs](https://code.claude.com/docs/en/skills) for details.

**Claude apps:** zip the folder and upload it as a custom skill. See the [Claude Help Center](https://support.claude.com) for the current steps.

## Using it

Just describe the work. The skill triggers on multi-step investigations where conclusions may change, or you can ask for it directly:

> Use the epistemic ledger to track this investigation into the intermittent stale reads.

On first use, the agent asks where to put the workspace, whether to commit it, and on which branch, then records your answers so it doesn't ask again. Skip the skill for quick one-shot fixes and simple questions; it is designed to stay proportionate.

For best results add instructions to your AGENTS.md or CLAUDE.md file.

## What's in the repo

```text
SKILL.md                      Core rules, record types and the operating loop
references/
  collaboration.md            Roles, challenge loop, TDD and agent patterns (always read)
  record-formats.md           IDs, templates, statuses and a worked example
  history.md                  Checkpoints, commit conventions and changelog fallback
  scaling.md                  Small, Medium and Large workspace modes
```

## Make it your own

These are the patterns I've chosen to make the idea tangible. They aren't the only way to do it. The rules are currently enforced softly, through instructions, review and history, and it is straightforward to harden them: schema checks on frontmatter, commit hooks that reject a claim without its dependencies, CI that validates the record graph, or external audits. An agentic-coding harness can build most of that for you in minutes or hours.

Issues and pull requests are welcome, especially reports of where the ledger helped, where it got in the way, and where an agent still found a way to launder an inference into a fact.

## Credits

Designed by [Rob Manson](https://flux.robman.fyi). Developed in a proposer/critic loop with Claude (Anthropic).

## License

See [LICENSE](LICENSE).
