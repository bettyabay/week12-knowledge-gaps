# Day 1 — Research Question

**Topic:** Inference-time mechanics  
**Researcher:** Bethelhem (solo)  
**Date:** 2026-05-04

---

## Final Sharpened Question

> "In my Week 11 ORPO judge, I report ~200ms inference latency per email on a T4 GPU. I understand that LLM inference has two phases: prefill (processing the prompt) and decode (generating tokens). For my use case — processing a 500–1000 token prompt and generating a short 50–100 token JSON response — which phase dominates the 200ms? What is the mathematical relationship between prompt length, response length, and total latency? Knowing this would help me decide whether to optimize by shortening prompts or by reducing response length."

---

## Why This Question Is Sharp

It has three interlocking requirements that each force precision:

1. **Empirical grounding** — the 200ms number is a real measurement from Week 11. The question is not abstract; it demands an answer that matches observed latency.
2. **Causal direction** — "which phase dominates" is not the same as "what are the two phases." Getting the causal direction wrong leads to the wrong optimization target.
3. **Actionable output** — the latency model `latency ≈ a × prompt_tokens + b × output_tokens + c` needs to be derivable from first principles so I can plug in my actual token counts and predict which variable to compress.

A surface-level answer ("prefill processes the prompt, decode generates tokens") does not tell me where to spend engineering effort. The question forces the answer down to coefficients and experimental design.

---

## Context: Week 11 System

- **Model:** Qwen 2.5 7B, 4-bit quantized (bitsandbytes NF4)
- **Task:** Binary preference judge for ORPO training data
- **Hardware:** T4 GPU (16 GB GDDR6, 320 GB/s memory bandwidth)
- **Prompt structure:** System instruction + two candidate emails + scoring rubric
- **Output structure:** JSON with `winner`, `score_a`, `score_b`, `reasoning`
- **Reported latency:** ~200ms per inference call
