# pitfalls-review

An [Agent Skill](https://agentskills.io/home) that reviews one LLM-security research paper
for nine methodological pitfalls and returns a structured, evidence-backed JSON verdict.

## Install

```bash
npx skills add caohch-1/LLMSec-PitfallsReview
```

Then ask your agent to use `pitfalls-review` on a paper, or invoke `$pitfalls-review` in
clients that support explicit skill invocation.

## What it checks

The skill evaluates data poisoning, LLM-generated label inaccuracy, data leakage, model
collapse, spurious correlations, context truncation, prompt sensitivity, surrogate fallacy,
and model ambiguity.

The review protocol is self-contained in [`pitfalls-review/SKILL.md`](pitfalls-review/SKILL.md).
It emits exactly one assessment for each pitfall, supported by verbatim evidence from the paper.
The output contract is the machine-readable
[`review.schema.json`](pitfalls-review/assets/review.schema.json).

The review is deliberately text-only: it judges what the paper says, not its repository,
released code, external commentary, or prior reviews.

## Credit

The protocol is adapted from the reviewer guidelines in
[*Chasing Shadows: Pitfalls in LLM Security Research*](https://arxiv.org/abs/2512.09549)
(Evertz et al., NDSS 2026). Licensed under MIT.
