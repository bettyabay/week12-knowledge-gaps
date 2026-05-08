# Day 4 — Research Questions

**Topic:** Evaluation and Statistics
**Date:** 2026-05-08
**Partners:** Bethelhem Abay & Rahel Samson
**Format:** Mutual exchange — each partner holds a question, each writes an explainer for the other

---

## Bethelhem's Question (answered by Rahel)

> "My `ablation_results.json` reports 85.2% held-out accuracy (52/61 correct) with 95% CI [0.77, 0.93], calculated using the normal approximation formula: p ± 1.96 × √(p(1-p)/n). But n=61 is small, the distribution might not be normal, and my labels are binary (correct/incorrect). Is the normal approximation valid here, or would a bootstrap confidence interval give a more accurate picture of my true uncertainty? And if I switched to bootstrap, would the interval widen, narrow, or stay the same — and why?"

**Artifact:** `ablation_results.json` — 52 correct / 9 incorrect / n=61, CI [0.77, 0.93]

**Why it matters:** If the Wald CI is overconfident, the lower bound of 0.77 in the model card overstates certainty. A reviewer or deployer relying on "at least 77% accurate" is making decisions on a number that is wrong in a known direction.

**Sharpening path:** The initial draft asked whether the formula was "right in general." After interrogation (specifically: what is n(1-p)? which direction would the error go? which bound is the overconfident one?), the question narrowed to: is it the lower or the upper bound that the Wald formula gets wrong, and why?

---

## Rahel's Question (answered by Bethelhem)

> "My `model_card.md` reports Delta A = +0.332 with p=0.003. But the bootstrap only tests the overall composite score. Is that enough, or should I test each rubric dimension separately? Specifically: does a significant composite p-value guarantee that each dimension's improvement is individually distinguishable from noise — or is the +0.09 on `word_count_violation` only surviving because the stronger dimensions are carrying the average?"

**Artifact:** `model_card.md` per-dimension table · `ablations/statistical_test.py` `paired_bootstrap()`

**Why it matters:** A client deploying the judge filter specifically to enforce word limits is relying on a per-dimension claim the current statistical test never actually made. If `word_count_violation` (+0.09) is non-significant at the adjusted threshold, the model card overstates the result for that dimension.

**Sharpening path:** The initial draft was a general methodological worry — "is one p-value enough?" After interrogation (which dimension is weakest? what would you change in the model card if some were non-significant?), the question landed on a specific failure case: +0.09 on `word_count_violation` versus +0.31 on `bench_over_commitment`, and whether the weak dimension survives only because the strong ones raise the composite.
