---
name: pitfalls-review
description: Review one research paper for nine LLM-security methodological pitfalls (data poisoning, label inaccuracy, data leakage, model collapse, spurious correlations, context truncation, prompt sensitivity, surrogate fallacy, model ambiguity) and write an evidence-backed Markdown report. Use when asked to audit, review, or screen a paper for methodological pitfalls or reproducibility problems.
license: MIT
---

# Pitfalls Review

Review one research paper using the protocol below.

If you were given a PDF, read it with whatever PDF tooling the machine has. Do not stop at the
running text: tables and figures routinely carry prompt templates, model versions, dataset
provenance, and per-model results that appear nowhere else, and a plain text dump loses or
scrambles them. Re-render a page to recover a table, and rasterise a page and look at it if you
can read images.

Write the Markdown report to the requested location, or return it directly if none was specified.

Do not consult the paper's repository, released code, external commentary, benchmark ground truth,
prior reviews, or unrelated files. Reviewing artifacts would answer a different question.

# Pitfall Review Protocol

You are reviewing a single research paper to determine whether it exhibits any of nine
methodological pitfalls that arise when large language models are used in security research.

Judge **only what the paper says**. You are not evaluating the paper's released code, repository, or
supplementary artifacts — only the paper itself, including its appendices, tables, and figures.
This is deliberate: a claim that is only true in an unpublished script is not a claim the paper made.
A prompt template or model version printed only in a figure, however, *is* a claim the paper made,
so read the figures rather than only their captions where you are able to.

You may use general knowledge about models, datasets, and tools named in the paper (for example, a
model's known maximum context length, or a dataset's public release date). You may not use knowledge
about *this paper's* reception, replication, or any external commentary on it.

Your goal is accurate classification, not fault-finding. Several of these pitfalls are difficult or
impossible to avoid, and a paper that hits one may still be excellent work. Do not inflate a
judgment because a paper "feels" weak, and do not suppress one because it is well known.

---

## The four screening questions

For every pitfall, in order:

1. **Is the pitfall applicable to the paper?** Could it reasonably have influenced the results?
2. **Is the pitfall present?** Is there clear evidence that it occurs in the paper, even if only to a
   limited extent? If there is strong indirect evidence, or missing information that suggests the
   pitfall is probably present, then it is **likely present**.
3. **If present, is it fully present** (affecting all results) **or partly present** (affecting only
   some results)?
4. **Is the pitfall discussed in the paper?** That is, do the authors themselves acknowledge,
   contextualize, or address the issue — anywhere in the text, including limitations sections?

---

## Decision tree

Follow this exactly. It produces one of nine labels.

```
Is the pitfall APPLICABLE to this paper?
├─ no ........................................ "Does not apply"
├─ unclear ................................... "Unclear from text"
└─ yes → Is it PRESENT?
         ├─ no ............................... "Not present"
         ├─ likely (strong indirect evidence,
         │   or missing info implying it) → Discussed?
         │                                  ├─ yes → "Likely present (but discussed)"
         │                                  └─ no  → "Likely present"
         └─ yes → Is it only PARTLY present
                  (affects some results only)?
                  ├─ yes → Discussed?
                  │        ├─ yes → "Partly present (but discussed)"
                  │        └─ no  → "Partly present"
                  └─ no  → Discussed?
                           ├─ yes → "Present (but discussed)"
                           └─ no  → "Present"
```

### The nine labels

Emit one of these strings **exactly**, character for character:

| Label | Meaning |
|---|---|
| `Does not apply` | Out of scope for this paper |
| `Unclear from text` | Applicable, but the paper lacks the detail needed to judge |
| `Not present` | Applicable, and the paper avoids it |
| `Likely present` | Probably present on indirect evidence; not acknowledged |
| `Likely present (but discussed)` | Probably present on indirect evidence; acknowledged |
| `Partly present` | Affects part of the methodology; not acknowledged |
| `Partly present (but discussed)` | Affects part of the methodology; acknowledged |
| `Present` | Present throughout; not acknowledged |
| `Present (but discussed)` | Present throughout; acknowledged |

**"Unclear from text" vs "Likely present".** These are different. Use `Unclear from text` when you
genuinely cannot tell in either direction. Use `Likely present` when the absence of information is
itself evidence — the setup makes the pitfall probable even though it is not stated outright.

---

## The nine pitfalls

### P1 — Data Poisoning via Internet Scraping

Applies if a dataset used to train a model is collected from the internet without strategies to
verify the integrity and trustworthiness of the data (e.g., to check for poisoned examples).

- Applies **even if** the data is taken from a dataset published by a different paper, if no
  verification was performed there either.
- Applies **only if there is training or fine-tuning** in the paper. It does not apply otherwise.
- If data is collected by scraping third-party websites like GitHub or Stack Overflow without manual
  verification, the pitfall is **likely present**.

### P2 — LLM-generated Label Inaccuracy

Applies when LLMs are used to annotate data with labels via classification or "LLM-as-a-judge"
procedures, without further validation of correctness.

- Check whether the paper verifies the correctness of these labels or applies mitigation strategies.
- If LLM-as-a-judge is used to evaluate jailbreaks or attacks without further validation, this
  pitfall applies.

### P3 — Data Leakage

Refers to situations where information unavailable in real-world deployment is inadvertently
included in the training data. Examples include:

- An LLM pre-trained or fine-tuned on data containing labels, metadata, or content from the test
  phase.
- The model has access to future data during training that would not be available at inference time.

Grading:

- If only **part** of the training datasets is affected, the pitfall is **partly present**.
- The model must be **directly affected** by the data contamination. Otherwise it is **not present**.
- Since it is often hard to verify what data is present in an LLM's pretraining, in most cases this
  pitfall will be **likely present**. For example, GPT-2 was trained on Wikipedia, but we have no
  detailed sources for GPT-4, which makes it *likely present*.
- If data is most likely present (such as Wikipedia), but there is no clear proof, yet it is widely
  assumed in the community, then the pitfall is **present**.

### P4 — Model Collapse via Synthetic Training Data

Applies if the model's weights are influenced in any way (e.g., through fine-tuning) by synthetic
data generated by an LLM.

- Also applies if external components, such as the tokenizer, are updated or trained with
  LLM-generated data.
- **In the case of in-context learning, the pitfall does not apply**, as there are no weight
  adjustments.
- "Synthetic data" means data produced as output by the same or a different LLM, then used for
  training or fine-tuning.

### P5 — Spurious Correlations / Unrelated Features

Applies when the LLM relies on spurious correlations or unrelated artifacts from the problem space,
instead of learning to generalize to the underlying task.

- Check whether the model is capable enough for the chosen task.
- Also applies if reported performance **varies considerably across models**, suggesting the proposed
  approach is only effective for the specific model used.
- Look for evidence of explainability or interpretability analysis to determine what features the
  model relies on.
- Also applies if the same performance could be achieved with much simpler features (e.g., based on
  variable names or code formatting instead of semantics). For example, a code vulnerability detector
  that performs well simply because vulnerable functions in the dataset tend to contain certain
  variable names, not because the model understands the logic.

### P6 — Context Truncation

Applies if the LLM's context size is not large enough for the evaluation or its intended task, such
that inputs need to be truncated.

- Check the model's maximum context length and the length of the inputs used.
- If input length is not disclosed, **estimate** it and convert to an approximate token count.
- If the evaluation is affected by truncation, this pitfall applies.

### P7 — Prompt Sensitivity

Applies if the prompt used to instruct the LLMs is either fixed across all models and experiments, or
lacks sufficient expressiveness for the specific task.

The pitfall is considered **present** if:

- The study uses only a single prompt configuration (e.g., one prompt applied uniformly across all
  models) without justification or variation.
- Models are tested for robustness against adversarial inputs but are instructed using only generic
  prompts such as "You are a helpful AI assistant."

Further grading:

- If the authors do not disclose how the prompting is designed and it appears generalized for all
  models, the pitfall is **likely present**.
- If prompt variation is mentioned, but mitigation is insufficient or superficial, then the pitfall
  is **present (but discussed)**.
- **Does not apply** to standard machine-learning classification tasks (e.g., mapping code to labels
  using BERT) where prompts are not part of the input pipeline.

### P8 — Proxy / Surrogate Fallacy

Applies when findings using specific LLMs are inappropriately generalized to other, sometimes larger,
models, or even to entire classes of language models, without sufficient empirical validation.

- Includes generalizing to different, untested **quantization methods** or different **access
  methods** (API vs. web), if evaluations were only done on one.
- **The authors must make explicit claims for this pitfall to apply.** Do not mark as present for
  vague implications.
- Example that applies: an attack is tested on a small open-source Llama-8b model, but the paper
  claims applicability to much larger or proprietary models like GPT.
- If claims are very vague (e.g., "LLMs cannot decipher obfuscated code from this framework"), mark
  as **partly present**.

### P9 — Model Ambiguity

Applies when model details are insufficient for precise identification, preventing reproducibility.
Missing details can include:

- **Model IDs** (e.g., mentioning only "GPT-4" instead of a specific version like
  `gpt-4o-2024-11-20`)
- **Snapshots** (for hosted models)
- **Commit IDs** (for local models, e.g., on Hugging Face or Ollama)
- **Quantization level**

Grading:

- **If even one of these is missing, the pitfall is `Present`** — not *partly present*.
- **`Partly present` is used only** if the information is missing for some but not all of the models
  used (e.g., 1 out of 3).
- If using a hosted model rather than an open-source one, the paper must specify whether experiments
  were conducted via the **API or the web interface**, as these may differ in content moderation,
  system prompts, or other hidden context.
- If it is possible to reproduce the model version unambiguously (e.g., only one commit existed at
  the time of publication), then this pitfall **does not apply**.

---

## Report format

Return only a Markdown report, with no surrounding commentary or code fence. Use this structure:

```markdown
# Pitfall review: <paper identifier or title>

## Summary

| Pitfall | Finding | Confidence |
|---|---|---|
| P1 — Data Poisoning via Internet Scraping | <exact label> | High, Medium, or Low |
| ... | ... | ... |
| P9 — Model Ambiguity | <exact label> | High, Medium, or Low |

## P1 — Data Poisoning via Internet Scraping

**Finding:** <exact label>

**Confidence:** High, Medium, or Low

**Evidence**

> <verbatim quotation from the paper>

— <section, table, figure, or page>

**Reasoning**

<2–4 sentences tracing the decision tree to the finding>
```

Repeat the detailed section for `P1` through `P9`, in order, using each pitfall's full title.

Rules:

- Use one of the nine label strings **exactly**, including capitalization and parentheses, for every
  `Finding`. The summary and detailed section must agree.
- Include at least one verbatim paper quotation for every finding except `Does not apply` and
  `Unclear from text`. For either exception, write `No quotation required for this finding.` under
  `Evidence` when no useful quotation exists.
- Put every quotation in a Markdown blockquote and identify its section, table, figure, or page
  immediately below it. Copy quotations exactly; never paraphrase or invent them.
- Quote only from the paper's running text, whose word order is the paper's own. Cite something you
  read off a table or a figure by its number instead (`— Table 4`, `— Figure 3, page 5`), and quote
  the caption or the prose that introduces it.
- Explain each finding in 2–4 sentences by tracing the decision tree. When the label ends in
  `(but discussed)`, identify how the paper acknowledges, contextualizes, or addresses the issue.
