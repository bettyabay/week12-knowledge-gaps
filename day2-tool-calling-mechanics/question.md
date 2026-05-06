# Day 2 — Research Question

**Topic:** Tool calling mechanics  
**Questioner:** Mistire  
**Explainer:** Bethelhem  
**Date:** 2026-05-05

---

## Final Sharpened Question

> "When a model 'calls a tool,' what is it actually generating at the token level — before any API interception? Does the model emit special tokens, raw JSON, or something else? And is this enforced at generation time, or does it come entirely from fine-tuning?"

---

## Sharpening Path

Mistire's original question mixed two separate problems that required separating before either could be answered cleanly.

**Original framing:**
"How does tool selection work in my agent? The model never seems to generate a tool call."

**First split — mechanism vs. design decision:**
The original question conflated understanding *how* tool calling works mechanically (A) with whether Mistire should refactor his agent to use it (B). Mistire's `agent.py` routes via Python conditionals at lines 259–270 before the LLM ever runs — so the model never had the opportunity to call a tool. That makes the question ambiguous: is the confusion about the mechanism, or about whether the architecture should change?

Mistire chose **A — the mechanism**.

**Second split — token level vs. API layer:**
With the scope fixed to mechanism, the question still spanned two different places in the stack:

- **A1** — what does the model generate in raw tokens before the API intercepts? (model vocabulary + fine-tuning)
- **A2** — what does the API do *after* generation — schema validation, logit masking, grammar-guided sampling — to guarantee structural validity?

Mistire chose **A1**.

---

## Why This Question Is Sharp

Mistire's agent never fires a tool call, which means he has no observed example of what tool-call tokens look like in his own system. The question is about a mechanism he has never seen execute. Pinning it to A1 forces the answer to be about the model's vocabulary and fine-tuning specifically — not about the API wrapper, not about agent architecture — which is one answerable thing.
