# Evening Call Summary — Day 2

**Date:** 2026-05-05  
**Participants:** Bethelhem (explainer), Mistire (questioner)  
**Duration:** ~25 min

---

## Mistire's Initial Reaction

Mistire said the fine-tuning framing clicked immediately — he had assumed tool calling was enforced at the API level by default and had not thought about the base distribution at all. The distinction between the fine-tuned probability distribution and generation-time enforcement was the thing that most shifted his mental model.

---

## Questions Mistire Raised

**Q1: "You say the model can still hallucinate tool names even after fine-tuning. How often does this happen in practice? Is there a known rate?"**

My answer: It depends heavily on the model, the complexity of the tool schema, and how well the context matches the training distribution. For well-supported models like GPT-4 with simple single-tool schemas, hallucination rates are very low — anecdotally under 1% for clear-intent inputs. For smaller models like Qwen 2.5 7B on complex multi-tool schemas, rates can be meaningfully higher. The honest answer is: you measure it for your specific setup. There is no published universal rate.

**Q2: "You mention `tokenizer.apply_chat_template()` with `tools=` — I've never passed tools there. What does the injected system prompt actually look like?"**

My answer: When you pass `tools=` to `apply_chat_template`, Qwen's template serializes your function schemas into a structured block at the top of the system prompt — something like a JSON list of `{"name": ..., "description": ..., "parameters": ...}` objects. The model was fine-tuned to read this exact format and use it as the reference for what tool names and argument structures are valid. If you don't pass `tools=`, the model has no schema in context and will either produce nothing or hallucinate structure freely.

**Q3: "Your conclusion says Python gating is strictly better for deterministic rules. But what if my `is_booking_intent()` function is wrong sometimes — doesn't that change the calculus?"**

My answer: Yes, exactly — this is the threshold question. If `is_booking_intent()` has a nonzero error rate, then the comparison is between two imperfect systems, and model-driven routing might win if the model's fine-tuned judgment is better calibrated on ambiguous cases than your Python heuristic. The practical decision point is: run both in shadow mode, measure disagreement rate, and check which one is right on the disagreements.

---

## Where the Explainer Was Strong

- The GPT-4 vs. Qwen 2.5 comparison — Mistire said having two concrete examples made the mechanism tangible in a way a generic description wouldn't have.
- Tying the conclusion back to `is_booking_intent()` specifically — he said it was the first time an explanation of tool calling felt directly relevant to his own code.

---

## Where I Got Pushed

- I understated the hallucination risk in the main body — Mistire's Q1 revealed I had glossed over it. The "low rate" claim needed more qualification.
- I didn't explain what `apply_chat_template(tools=...)` actually injects — the explainer assumed the reader knew this.

---

## Revisions Made After Call

- [x] Added a note on hallucination rate variability and the need to measure for your specific setup
- [x] Added a one-sentence description of what `apply_chat_template(tools=...)` injects into the system prompt
- [x] Clarified the Python-vs-model comparison as conditional on `is_booking_intent()` accuracy
