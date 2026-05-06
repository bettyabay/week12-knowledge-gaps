# Question Sharpening Notes — Day 2

**Date:** 2026-05-05  
**Researcher:** Bethelhem (explainer), Mistire (questioner)  
**Format:** Live discussion

---

## Starting Question

"How does tool selection work in my agent? The model never seems to generate a tool call."

---

## Sharpening Round 1 — Separate the two problems

The opening question mixed a theoretical understanding problem with an architectural design decision. I flagged this directly:

> "Your question mixes two separate problems — which one are you actually asking about? You described a system where Python makes all routing decisions before the LLM runs (lines 259–270). That means the model never had the chance to call a tool in the first place."

The split:
- **A** — How does token-level tool selection work mechanically?
- **B** — Should I refactor `agent.py` to let the model drive routing — and if so, what would I actually change?

I noted that A doesn't automatically imply B. Pre-LLM Python gating can be faster, cheaper, and more reliable than model-driven tool selection for deterministic rules like `is_booking_intent()`. If the Python conditional is 100% accurate, replacing it with function calling introduces latency, token cost, and potential model error for no gain.

**Mistire chose A.**

---

## Sharpening Round 2 — Pin to one layer of the stack

Mechanism still spanned two different places:

> "You have no observed example of what tool-call tokens actually look like in your system — you're asking how something works that you've never seen fire. So when you say 'what is the model generating at the token level' — are you asking about A1 (raw tokens before API interception) or A2 (what the API does after generation to enforce structural validity)?"

- **A1** — what tokens or token sequences does the model produce in raw output: does it emit something like `<tool_call>` or `{"name": "book", "arguments": ...}` as raw text?
- **A2** — what does the API layer do afterward: schema validation, logit masking, grammar-guided sampling?

A1 is about the model's vocabulary and training. A2 is about the API wrapper. They're different places in the stack with different answers.

**Mistire chose A1.**

---

## Agreed Scope for the Explainer

Answer exactly this: at the token level, before any API interception, what does a model actually generate when it executes a tool call? Cover both GPT-4 (OpenAI tools API) and Qwen 2.5 — since Mistire's agent uses a model in the Qwen family and that's also what I used in Week 11.

One constraint Mistire set: tie the answer back to his own code at the end. The explanation should land on something actionable about `is_booking_intent()` and when you'd actually want to replace Python routing with model-driven tool calls.
