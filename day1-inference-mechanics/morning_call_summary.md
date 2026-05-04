# Question Sharpening Notes — Day 1

**Date:** 2026-05-04  
**Researcher:** Bethelhem (solo)  
**Format:** Independent question development (no morning call)

---

## Starting Point

The initial question was loose: "Which phase is slower — prefill or decode — for my judge model?" That framing had no quantitative anchor and wouldn't produce an actionable answer.

---

## Sharpening Process

**Round 1 — Ground it in Week 11 data:**  
The question became useful only when tied to a real number. The Week 11 ORPO judge reports ~200ms per inference, so the question became: where does that 200ms live? This forces the answer to be falsifiable — I can run a measurement and check it.

**Round 2 — Distinguish the phases precisely:**  
"Prefill" and "decode" are sometimes used loosely to mean "input processing" and "output generation." Defining them mechanically — prefill as a single parallel forward pass, decode as a sequential per-token loop — was necessary because those definitions have direct consequences for which variable (prompt length or output length) drives latency.

**Round 3 — Add the mathematical structure:**  
A yes/no answer to "which phase dominates" isn't enough. Adding the formula `latency ≈ a × prompt_tokens + b × output_tokens + c` made the question testable with a sweep and gave a clear path from the answer to an optimization decision.

**Round 4 — Scope the optimization goal:**  
The reason I care about the breakdown is to decide between two options: compress the prompt or shorten the output. Including that decision in the question kept the explainer focused on producing a recommendation, not just a description.

---

## Final Question

> "In my Week 11 ORPO judge, I report ~200ms inference latency per email on a T4 GPU. For my use case — 500–1000 token prompts and 50–100 token JSON responses — which phase dominates the 200ms? What is the mathematical relationship between prompt length, response length, and total latency? Knowing this would help me decide whether to optimize by shortening prompts or by reducing response length."
