# Day 3 — Research Question

**Topic:** Preference training generalization — memorization vs. genuine learning  
**Questioner:** Samuel Lachisa  
**Explainer:** Bethelhem  
**Date:** 2026-05-07

---

## Final Sharpened Question

> "My SimPO adapter was trained on 200 slot-filled pairs where every chosen response follows an escalation pattern and every rejected response follows a hard-commitment pattern. It achieved 82% capacity_honesty on an eval set drawn from the same template distribution. What observable differences in training dynamics and eval results would distinguish a model that internalized the rule 'do not over-commit capacity' from one that learned 'penalize the word immediately'? And which single test — paraphrase eval, out-of-distribution eval, loss curve divergence, or token-level confidence calibration — is the highest-signal diagnostic for this specific setup?"

---

## Why This Question Is Sharp

It has three interlocking requirements that each force precision:

1. **Empirical grounding** — the 82% figure is a real measurement from a real training run. The question is not asking about memorization in the abstract; it demands an answer that explains whether that specific number is trustworthy.
2. **Causal specificity** — "learned the rule vs. learned the word" is not the same as "generalized vs. overfit." Getting the causal mechanism right changes which diagnostic test matters most.
3. **Forced ranking** — asking for the *highest-signal* diagnostic prevents a list-of-things answer. The question can only be answered by understanding which signal is least confounded by the template structure of the training data.

A surface-level answer ("run a generalization eval") does not tell you which eval to run first or why it is more diagnostic than the others. The question forces the answer down to a specific test and a specific reason.

---

## Context: Week 11 System

- **Method:** SimPO (Simple Preference Optimization with a reference-free reward)
- **Training data:** 200 preference pairs generated from slot-filled templates
- **Chosen pattern:** Escalation responses (e.g., "Our enterprise director will be in touch")
- **Rejected pattern:** Hard-commitment responses (e.g., "We can start immediately")
- **Eval metric:** capacity_honesty — 82% on the eval set
- **Gap:** The eval set was drawn from the same template distribution as the training data, so 82% may reflect template recognition, not rule internalization
