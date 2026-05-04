# Grounding Commit — Day 1

**Purpose:** Apply today's research to a concrete edit in Week 11 work — correcting or extending something that today's analysis revealed was imprecise.

---

## Week 11 File Being Edited

**File:** `week11-orpo-judge/model_card.md` (or equivalent memo/README)  
**Section:** "Performance — Inference Latency"

---

## Original Claim (Week 11)

> "The judge model runs at approximately 200ms per inference on a T4 GPU."

---

## What Today's Research Revealed

This claim is accurate but underspecified in a way that makes it misleading. The 200ms is an end-to-end wall-clock measurement that conflates two mechanically distinct phases — prefill (compute-bound) and decode (memory-bandwidth-bound) — which have different scaling properties with respect to prompt length and output length.

As written, the latency claim implies a single cost that scales uniformly with "input size." In reality:
- Shortening the prompt reduces prefill cost, which likely dominates for this task.
- Shortening the JSON output reduces decode cost, which is likely smaller.
- The two are not interchangeable optimization targets.

---

## The Edit

**Add the following paragraph to the latency section of the Week 11 model card:**

> The 200ms figure reflects total `model.generate()` time and includes both the prefill phase (one parallel forward pass over the full prompt) and the decode phase (sequential per-token generation). For this task — 500–1000 token prompts and 50–100 token JSON outputs — the prefill phase is expected to dominate latency due to the high prompt-to-output token ratio (~10:1). Empirical decomposition using `torch.cuda.synchronize()` timing around each phase is needed to confirm. The primary latency optimization lever is prompt compression, not output length reduction.

---

## Why Today's Research Prompted It

The prefill/decode phase analysis and the `latency ≈ a × prompt_tokens + b × output_tokens + c` formula make it clear that reporting a single latency number without specifying which phase it comes from — or which variable drives it — is insufficient for anyone trying to reproduce or optimize the system. The edit makes the latency claim actionable.

---

## Commit Reference

**Repo:** https://github.com/bettyabay/tenacious-bench  
**Branch:** `main`  
**Commit:** `b84eedb`  
**Diff URL:** https://github.com/bettyabay/tenacious-bench/commit/b84eedb

---

*Make the edit, commit it to the Week 11 repo, then paste the commit hash and diff URL above.*
