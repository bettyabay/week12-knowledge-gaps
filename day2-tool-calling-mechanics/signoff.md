# Mistire's Signoff — Day 2

**Questioner:** Mistire  
**Explainer:** Bethelhem  
**Date:** 2026-05-05

---

## Gap Closure Judgment

| Sub-question | Status | Notes |
|---|---|---|
| What does the model generate at the token level during a tool call? | ✅ Closed | GPT-4 emits plain JSON; Qwen emits `<tool_call>` special tokens — the concrete examples landed |
| Is this enforced at generation time or from fine-tuning? | ✅ Closed | Fine-tuning only — generation-time enforcement is the separate A2 layer |
| Why does Python routing outperform model-driven tool calls for deterministic rules? | ✅ Closed | Replacing a logical process with a statistical one is a downgrade when the logical one is accurate |
| When *should* I switch to model-driven tool selection? | ✅ Closed | When the routing decision requires language understanding — ambiguous intent, multi-field inference, edge cases that don't fit explicit rules |

---

## What I Now Understand

Before this session, I assumed tool calling was enforced by the API — that something in the infrastructure guaranteed structured output. The explainer showed me this is backwards: the model produces the structure because fine-tuning made it the high-probability completion, and the API layer intercepts and parses what the model already wrote. The enforcement (logit masking, schema validation) is optional and sits on top.

The `is_booking_intent()` insight was the most directly actionable thing. I had been thinking of Python routing as a workaround until I "properly" implemented tool calling. Now I see it as the correct architecture for that routing decision — and I have a clear criterion for when to switch.

---

## What Was Still Fuzzy

The hallucination rate question is not fully closed. I know it's model- and schema-dependent, but I don't have an intuition for what "low" means quantitatively for Qwen 2.5 7B on my specific schema. That needs a shadow-mode measurement.

---

## One Follow-up Question This Raised

> "If I do eventually want the model to drive routing for ambiguous cases, how do I run both `is_booking_intent()` and model tool selection in parallel to compare them without doubling my inference cost?"

---

## Overall Verdict

- [x] Gap closed — I can now explain what the model generates during a tool call and I understand why my current architecture is correct for deterministic rules.
