# Grounding Commit — Day 2

**Purpose:** Apply today's research to a concrete edit in prior work — correcting or extending something that today's analysis revealed was imprecise.

---

## Week 11 File Being Edited

**Repo:** https://github.com/bettyabay/tenacious-bench  
**File:** `publication/model_card.md`  
**Section:** "How to Use" — the `model.generate()` call

---

## Original Code in Model Card

```python
outputs = model.generate(inputs, max_new_tokens=64, temperature=0.0)
decision = tokenizer.decode(outputs[0][inputs.shape[1]:], skip_special_tokens=True)
```

---

## What Today's Research Revealed

The model card shows `model.generate()` being called directly without `apply_chat_template(tools=...)`. This is correct for the current judge design — the model outputs a plain text verdict (`SUPPRESS`, `PASS`, etc.), not a structured tool call.

But the "How to Use" section gives no explanation for *why* the call looks this way. A reader familiar with tool-calling patterns might wonder why there is no tools array, no `tool_calls` parsing, and no `<tool_call>` token handling. Today's analysis provides the answer: the judge is a text classifier, not a tool-calling agent. It generates a verdict token followed by a rationale. Using tool calling would require fine-tuning the model to emit `<tool_call>` JSON instead of plain verdict text — an unnecessary complexity for a task with a fixed output space.

---

## The Edit

**Add the following comment block above the `model.generate()` call in the model card:**

```python
# Architecture note: this judge generates plain text verdicts (SUPPRESS, PASS, etc.),
# not tool calls. Tool calling would require the model to emit <tool_call> JSON tokens,
# which in turn requires tool-use fine-tuning and a tools= argument in apply_chat_template.
# For a fixed-output-space classifier like this judge, plain text generation is simpler,
# faster, and equally accurate. See: Week 12 Day 2 explainer on tool calling mechanics.
outputs = model.generate(inputs, max_new_tokens=64, temperature=0.0)
```

---

## Why Today's Research Prompted It

Understanding that `<tool_call>` tokens come from fine-tuning — not from the base model — makes it clear why the judge doesn't and shouldn't use them. The model card previously gave no reason for the design choice. The comment makes the decision explicit and points to the Day 2 research for anyone who wants the full explanation.

---

## Commit Reference

**Repo:** https://github.com/bettyabay/tenacious-bench  
**Branch:** `main`  
**Commit:** [COMMIT_HASH — fill in after push]  
**Diff URL:** [DIFF_URL — fill in after push]
