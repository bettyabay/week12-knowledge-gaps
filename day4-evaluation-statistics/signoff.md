# Day 4 Sign-offs

**Date:** 2026-05-08

---

## Rahel's Sign-off on Bethelhem's Explainer

**Question answered:** Does a significant composite p-value guarantee per-dimension significance?

| Sub-question | Status | Notes |
|---|---|---|
| What does the composite bootstrap actually test? | ✅ Closed | Tests mean(delta) = 0 — not any individual delta |
| Can strong dimensions carry weak ones in the aggregate? | ✅ Closed | Yes — the concrete illustration made this unambiguous |
| How to test per dimension? | ✅ Closed | `paired_bootstrap()` per dimension on `probe_results.json` — runnable from existing code |
| What Bonferroni threshold applies? | ✅ Closed | α = 0.05/7 ≈ 0.0071 |
| What to change in `model_card.md`? | ✅ Closed | Add per-dimension significance table; flag `word_count_violation` and `tone_violation` as ⚠️ pending re-run |

**What I now understand:** A composite p-value is a claim about the average, not a claim about each component. Before this, I was treating p=0.003 as evidence that all 7 dimensions improved significantly. It is evidence that the *mean* improvement is significant. I need to run the per-dimension test before I can say anything about `word_count_violation` specifically.

**One follow-up question this raised:**
> "If I re-run and `word_count_violation` fails the Bonferroni threshold, should I remove it from the training objective or just flag it in the model card?"

**Verdict:** ✅ Gap closed.

---

## Bethelhem's Sign-off on Rahel's Explainer

**Question answered:** Is the Wald CI valid for 85.2% accuracy on n=61, and which bound is wrong?

| Sub-question | Status | Notes |
|---|---|---|
| What condition does the Wald formula require? | ✅ Closed | Both n×p ≥ 10 AND n×(1−p) ≥ 10 — I fail the second (9 failures) |
| Which direction is the error? | ✅ Closed | Left-skewed distribution → lower bound is overconfident |
| Which bound specifically? | ✅ Closed | Lower bound (0.77 → corrected to ~0.74), not the upper |
| What is the correct CI? | ✅ Closed | Wilson: [0.743, 0.920] — replaces Wald [0.763, 0.941] |
| When to use Wilson vs bootstrap? | ✅ Closed | Wilson for single-proportion; bootstrap when comparing multiple metrics simultaneously |

**What I now understand:** My reported CI [0.77, 0.93] overstates the lower bound by about 0.02. Anyone reading "at least 77% accurate with 95% confidence" is relying on a number that is wrong in a known direction. The correct lower confidence limit is 0.74. The fix is one line of code.

**One follow-up question this raised:**
> "My n=61 is below the ~80–100 recommended minimum. If I evaluate on more held-out tasks from the full `tenacious_bench_v0.1`, what n would I need to get the Wilson CI below ±0.05 width?"

**Verdict:** ✅ Gap closed.
