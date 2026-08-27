# pitfalls-review

An [Agent Skill](https://agentskills.io/home) that reviews one LLM-security research paper
for nine methodological pitfalls and returns a readable, evidence-backed Markdown report.

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
It produces a summary table followed by one detailed assessment for each pitfall, supported by
verbatim evidence from the paper.

The review is deliberately paper-only: it judges what the paper says, not its repository,
released code, external commentary, or prior reviews.

## Reading the PDF

Given a PDF, the skill reads it with whatever tooling you have, and does not stop at the running
text — prompt templates, model versions, and dataset pipelines are routinely printed as tables and
figures and nowhere else. Quotations still come only from the running text, whose word order is the
paper's own; anything read off a table or figure is cited by number instead.

## Credit

The protocol is adapted from the reviewer guidelines in
[*Chasing Shadows: Pitfalls in LLM Security Research*](https://www.ndss-symposium.org/ndss-paper/chasing-shadows-pitfalls-in-llm-security-research/)
(Evertz et al., NDSS 2026). Licensed under MIT.
