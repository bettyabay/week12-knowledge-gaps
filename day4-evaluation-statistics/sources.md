# Sources — Day 4: Evaluation and Statistics

---

## Papers

**Brown, Cai & DasGupta (2001) — Interval Estimation for a Binomial Proportion**
*Statistical Science, 16(2), 101–117.*
The definitive paper establishing Wilson as the default CI method over Wald for small n and extreme p. Table 1 maps exactly when Wald coverage probability falls below the nominal 95% — directly applicable to the n=61, p=0.852 case. The paper also introduces Jeffreys and Agresti-Coull intervals as alternatives.
https://www.jstor.org/stable/2676784

**Efron & Hastie (2016) — Computer Age Statistical Inference**
*Cambridge University Press. Chapter 11: Bootstrap Confidence Intervals.*
Covers the percentile bootstrap, BCa (bias-corrected and accelerated), and studentized bootstrap for proportion CIs. The BCa method is more accurate than the percentile method for skewed distributions but requires jackknife estimates — relevant as a next step if Wilson alone is insufficient. Chapter 15 covers multiple testing and the family-wise error rate, directly relevant to the Bonferroni correction applied in the per-dimension significance analysis.
https://hastie.su.domains/CASI/

**Holm (1979) — A Simple Sequentially Rejective Multiple Test Procedure**
*Scandinavian Journal of Statistics, 6(2), 65–70.*
Introduces the Holm-Bonferroni step-down procedure, which is uniformly more powerful than the standard Bonferroni correction (same family-wise error rate control, but rejects more true discoveries). Worth using instead of plain Bonferroni when running 7 per-dimension tests — it will recover more significant dimensions without inflating false positives.
https://www.jstor.org/stable/4615733

---

## Tools

**Python `statsmodels` — `proportion_confint`**
Implements Wald, Wilson, Clopper-Pearson, and Agresti-Coull CIs for binary proportions in one call. Replaces the manual Wald formula in `ablation_results.json` reporting.
```python
from statsmodels.stats.proportion import proportion_confint

low, high = proportion_confint(52, 61, alpha=0.05, method='wilson')
print(f"Wilson CI: [{low:.3f}, {high:.3f}]")
# Wilson CI: [0.743, 0.920]
```
https://www.statsmodels.org/stable/generated/statsmodels.stats.proportion.proportion_confint.html

**Python `scipy.stats` — `binom_test` / `binomtest`**
For exact (Clopper-Pearson) CI when both n×p and n×(1−p) are below 5. Also useful for computing the exact p-value that the true accuracy exceeds a specific threshold (e.g., testing H0: p ≤ 0.75).
https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.binomtest.html

---

## Rule-of-Thumb Reference

| Condition | Recommended CI method |
|---|---|
| n×p ≥ 10 AND n×(1−p) ≥ 10 | Wald acceptable; Wilson still preferred |
| Either condition < 10 (Bethelhem's case) | Wilson |
| Either condition < 5 | Clopper-Pearson (exact) |
| Comparing multiple metrics simultaneously | Bootstrap (percentile or BCa) |
| 7 per-dimension tests (Rahel's case) | Per-dimension bootstrap + Bonferroni or Holm-Bonferroni |
