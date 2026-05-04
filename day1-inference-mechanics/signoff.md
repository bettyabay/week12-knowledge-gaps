# Kidane's Signoff — Day 1

**Questioner:** Kidane  
**Explainer:** Bethelhem  
**Date:** 2026-05-04

---

## Gap Closure Judgment

| Sub-question | Status | Notes |
|---|---|---|
| Mechanical explanation of prefill vs. decode | ✅ Closed | Compute-bound vs. memory-bandwidth-bound framing made it concrete |
| Which phase dominates for short-output tasks | ✅ Closed | Token ratio argument was convincing; gave a clear prior even before measuring |
| Mathematical latency model | ✅ Closed | The `a × n + b × m + c` formula is something I can apply directly |
| How to measure the breakdown empirically | ✅ Closed | The sweep experiment and `synchronize()` pattern are immediately usable |

---

## What I Now Understand

Before this session, my intuition was that decode is always the bottleneck — "token generation is slow." The explainer showed me why that intuition is specific to long-output tasks. For my ORPO judge (50–100 token JSON outputs, 500–1000 token prompts), the 10:1 token ratio means prefill likely dominates, which flips the optimization priority: compress the prompt, not the output.

I also learned that timing GPU operations without `torch.cuda.synchronize()` gives you meaningless numbers — the CPU sees the kernel launch, not the kernel completion. That's the kind of subtle mistake that produces misleading benchmarks.

---

## What Was Still Fuzzy

The bandwidth floor calculation (3.5 GB / 320 GB/s ≈ 11ms per decode step) gave a theoretical minimum that's lower than what I'd expect from the 200ms total. The explainer acknowledged this but didn't fully explain the gap between the theoretical floor and the actual per-token decode time on a real system (cache misses, dequantization overhead, CUDA kernel scheduling). That would be worth a follow-up.

---

## One Follow-up Question This Raised

> "If I compress the prompt from 750 to 400 tokens by truncating the candidate emails, how do I know whether I've removed the tokens that contain the decision-relevant signal? Is there a way to measure information loss from truncation?"

---

## Overall Verdict

- [x] Gap closed — I can explain this myself without looking it up and I have a concrete experiment to run against my Week 11 setup.
