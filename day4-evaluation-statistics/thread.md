# Tweet Thread — Day 4: Composite p-Values and Per-Dimension Significance

*Published at [THREAD_URL]*

---

**Tweet 1/5**

Your model gets a composite p=0.003 across 7 rubric dimensions. Significant! 🎉

But one dimension improved by only +0.09 while another improved by +0.31.

Does the composite result cover the weak dimension — or is it just being carried by the strong ones? 🧵

---

**Tweet 2/5**

The composite bootstrap tests exactly one thing:

H0: mean(delta across ALL dimensions) = 0

A p=0.003 means the average improvement is real. It says nothing about whether any single dimension's improvement is individually real.

Strong dimensions (+0.31) can and do carry weak ones (+0.09) in the average.

---

**Tweet 3/5**

If you run 7 individual tests at α=0.05, you have a 30% chance of at least one false positive:

1 − (1 − 0.05)^7 = 0.302

The fix: Bonferroni correction.
Adjusted threshold = 0.05 / 7 ≈ 0.0071

Any dimension with p > 0.0071 cannot be claimed as individually significant.

---

**Tweet 4/5**

The code is simple — run your existing paired_bootstrap() once per dimension:

```python
BONFERRONI_ALPHA = 0.05 / 7  # ≈ 0.0071

for dim in dimensions:
    baseline = [r["baseline"][dim] for r in results]
    finetuned = [r["finetuned"][dim] for r in results]
    delta, p_val = paired_bootstrap(baseline, finetuned)
    flag = "✅" if p_val < BONFERRONI_ALPHA else "⚠️ not individually significant"
    print(f"{dim}: Δ={delta:+.3f}, p={p_val:.4f} {flag}")
```

The scores are already in probe_results.json. No new data needed.

---

**Tweet 5/5**

Why it matters for model cards:

If a client deploys your judge specifically to enforce word count, they're relying on a per-dimension claim your composite test never actually made.

Rule: report composite significance for overall model quality. Report per-dimension significance (Bonferroni-corrected) for any dimension-specific deployment claim.

Full writeup: [BLOG_URL]
