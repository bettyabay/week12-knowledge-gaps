# Grounding Commit — Day 3

**Purpose:** Apply today's research to a concrete edit in prior work — correcting or extending something that today's analysis revealed was imprecise.

---

## Week 11 File Being Edited

**Repo:** https://github.com/bettyabay/tenacious-bench  
**File:** `evaluation/eval_capacity_honesty.py`  
**Section:** Eval script — the section that loads and scores the eval set

---

## Original Code in Eval Script

```python
# Load eval pairs and score
results = []
for pair in eval_pairs:
    score = judge_model(pair["prompt"], pair["chosen"], pair["rejected"])
    results.append(score)

accuracy = sum(results) / len(results)
print(f"capacity_honesty: {accuracy:.2%}")
```

---

## What Today's Research Revealed

The eval script loads `eval_pairs` from the same JSONL file that shares the template distribution of the training data. The 82% capacity_honesty figure is computed entirely on in-distribution examples — the eval set uses the same escalation and commitment phrasing as the training set.

Today's analysis shows this is the wrong eval to trust if you want to know whether the model learned the rule. The eval is measuring template-match accuracy, not semantic preference generalization. There is no paraphrase eval, no OOD eval, and no comment explaining this limitation.

---

## The Edit

**Add the following comment block and a paraphrase eval stub below the existing accuracy calculation:**

```python
# NOTE: This eval uses the same template distribution as training data.
# A high score here does not confirm the model learned the semantic rule
# ("do not over-commit capacity") — it may reflect vocabulary memorization
# (associating escalation phrases with chosen, commitment phrases with rejected).
# To test generalization, run eval_capacity_honesty_paraphrase.py, which
# uses synonym-substituted versions of these same pairs.
accuracy = sum(results) / len(results)
print(f"capacity_honesty (template eval): {accuracy:.2%}")
print("WARNING: Run paraphrase eval to validate generalization.")
```

---

## Why Today's Research Prompted It

Understanding that SimPO loss does not distinguish semantic learning from vocabulary memorization makes it clear that the eval script is measuring the wrong thing — or rather, not the full thing. Adding the warning and the pointer to a paraphrase eval makes the limitation explicit to anyone who reads the script, so they do not treat the 82% as unconditional evidence of a well-generalized preference.

The warning also creates a forcing function: if a paraphrase eval script does not exist yet, seeing the pointer in the template eval output is a reminder to build it.

---

## Commit Reference

**Repo:** https://github.com/bettyabay/tenacious-bench  
**Branch:** `main`  
**Commit:** [COMMIT_HASH — fill in after push]  
**Diff URL:** [DIFF_URL — fill in after push]
