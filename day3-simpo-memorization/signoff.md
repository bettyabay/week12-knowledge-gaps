# Samuel Lachisa's Signoff — Day 3

**Questioner:** Samuel Lachisa  
**Explainer:** Bethelhem  
**Date:** 2026-05-07

---

## Gap Closure Judgment

| Sub-question | Status | Notes |
|---|---|---|
| What mechanistically distinguishes genuine learning from surface memorization in SimPO? | ✅ Closed | SimPO loss is minimized by vocabulary distribution matching as readily as semantic rule learning — the gradient signal does not distinguish them |
| Why can't 82% accuracy on the template eval answer this question? | ✅ Closed | The eval set came from the same template distribution as training data — in-distribution accuracy cannot detect vocabulary memorization |
| Which diagnostic test is highest-signal for this specific setup? | ✅ Closed | Paraphrase eval — it isolates vocabulary as the only variable, holding meaning constant |
| What does a failing paraphrase eval imply about the training process? | ✅ Closed | Data coverage problem, not a SimPO method failure — fix by augmenting training pairs with paraphrased variants |
| What counts as a significant accuracy drop on the paraphrase eval? | ⚠️ Partially closed | Agreed on >10 percentage points for minimal (synonym-only) paraphrase as a red flag; exact threshold is context-dependent |

---

## What I Now Understand

Before this session, I assumed that good validation loss and high eval accuracy were sufficient evidence that my model had learned the preference. The explainer showed me this assumption fails when the validation set shares the same template structure as training data. The model can achieve low loss and high accuracy by memorizing vocabulary distributions — and nothing in the standard training loop exposes this.

The paraphrase eval insight was the most directly actionable thing. I now have a specific test I can run: take 20 eval examples, rewrite them with synonym substitution preserving meaning, and measure the accuracy delta. If the delta is small, the 82% number is trustworthy. If it is large, I have a data coverage problem that augmentation can fix.

---

## What Was Still Fuzzy

The interaction between model size and template generalization is not fully closed. A larger base model might generalize from the same 200 pairs because it already has rich representations of synonym relationships from pretraining. I do not know how much of the 82% comes from the fine-tuning and how much comes from the base model's pretraining — and the paraphrase eval does not separate them.

---

## One Follow-up Question This Raised

> "If I add paraphrased variants to my training set, how do I know when I have enough variety — is there a way to measure vocabulary diversity in a preference dataset before training?"

---

## Overall Verdict

- [x] Gap closed — I can now explain the mechanistic difference between genuine preference learning and surface memorization in SimPO, and I have a specific test to run on my existing adapter to determine which case I am in.
