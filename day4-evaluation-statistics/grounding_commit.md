# Grounding Commit — Day 4

**Purpose:** Apply today's research to a concrete edit in Week 11 work.

---

## File Being Edited

**Repo:** https://github.com/bettyabay/tenacious-bench
**File:** `publication/model_card.md`
**Section:** "Evaluation Results" — the 95% CI on held-out accuracy

---

## Original Claim

```
| ORPO judge (this model) | 52/61 | **85.2%** | **[0.77, 0.93]** |
```

The CI [0.77, 0.93] was computed using the Wald (normal approximation) formula:
`p ± 1.96 × √(p(1-p)/n)`

---

## What Today's Research Revealed

The Wald formula requires both `n×p ≥ 10` and `n×(1−p) ≥ 10`. With 52 correct and 9 incorrect from n=61, the second condition fails: `n×(1−p) = 9 < 10`. The distribution is left-skewed, meaning the lower bound is overconfident.

The correct formula for this case is Wilson:
- Wald: [0.763, 0.941] → rounds to [0.77, 0.93] ← what was reported
- Wilson: [0.743, 0.920] → rounds to [0.74, 0.92] ← correct

The lower bound shifts from 0.77 to 0.74. Anyone reading the model card and relying on "at least 77% accurate" is seeing an overconfident number.

---

## The Edit

**In the Evaluation Results table, change:**

```markdown
| ORPO judge (this model) | 52/61 | **85.2%** | **[0.77, 0.93]** |
```

**To:**

```markdown
| ORPO judge (this model) | 52/61 | **85.2%** | **[0.74, 0.92]** (Wilson) |
```

**Add a footnote below the table:**

```markdown
*CI computed using the Wilson interval (Brown, Cai & DasGupta, 2001), not the
Wald (normal approximation) formula. With only 9 failures from n=61, Wald
underestimates the lower bound by ~0.02. Wilson formula:
lower = (2np + z² − z√(z² + 4np(1−p))) / (2(n + z²))*
```

---

## Commit Reference

**Repo:** https://github.com/bettyabay/tenacious-bench
**Branch:** `main`
**Commit:** [COMMIT_HASH — fill in after push]
**Diff URL:** [DIFF_URL — fill in after push]
