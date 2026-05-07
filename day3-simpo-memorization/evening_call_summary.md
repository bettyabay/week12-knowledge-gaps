# Evening Call Summary — Day 3

**Date:** 2026-05-07  
**Questioner:** Samuel Lachisa  
**Explainer:** Bethelhem  
**Format:** Stress-test of the explainer's answer

---

## What Was Stress-Tested

### Challenge 1: Why is paraphrase eval higher-signal than OOD eval?

**Partner's challenge:** "OOD eval also changes the vocabulary. Why is paraphrase eval specifically better — aren't they the same thing?"

**Response:** They are not the same because they change different variables. OOD eval changes vocabulary *and* sentence structure *and* domain simultaneously. If the model fails on OOD examples, you cannot determine which change caused the failure. Paraphrase eval holds everything constant except the specific words — same meaning, same task, same domain, different surface form. It is a controlled experiment where OOD eval is not.

**Partner's verdict:** Accepted.

---

### Challenge 2: Could the loss curve tell you the same thing?

**Partner's challenge:** "If training loss and validation loss both track closely, doesn't that already tell you the model generalized?"

**Response:** Only if the validation set contains meaningfully different examples from the training set. In this setup, both training and validation data came from the same template distribution. A loss curve that tracks closely means the model generalized to the *template distribution* — which is exactly the thing you cannot trust. Paraphrase eval is the test that gets you *outside* that distribution.

**Partner's verdict:** Accepted — this was the key point they had not considered.

---

### Challenge 3: What does a "significant" accuracy drop look like?

**Partner's challenge:** "How much accuracy drop on the paraphrase eval counts as evidence of memorization? Is 70% still acceptable?"

**Response:** The right comparison is not to a fixed threshold but to the baseline. If the model scores 82% on the template eval and 80% on the paraphrase eval, the drop is within noise — likely genuine learning. If it drops to 60% or below, that is a large drop for a synonym substitution, suggesting the model was relying on specific lexical cues. The exact threshold depends on paraphrase difficulty: near-synonym substitution should show near-zero drop; structural rewriting might show a 5–10% drop even for a well-generalized model.

**Partner's verdict:** Partially accepted — wanted a cleaner rule of thumb. Agreed on: "a drop larger than 10 percentage points on minimal paraphrase (synonym-only) is a red flag."

---

## What Remained Open

**The data augmentation question:** If the paraphrase eval fails, what is the minimum number of additional pairs needed to fix the coverage problem? The explainer argued this depends on the variance of the vocabulary in the target domain — no clean answer without empirical measurement.

**The interaction with model size:** Larger base models may generalize from fewer template examples because they bring more pretraining knowledge about synonym relationships. The 7B scale used in Week 11 may behave differently from a 1B or 70B model on the same template data.

---

## Overall Assessment

The core claim — that paraphrase eval is the highest-signal diagnostic for vocabulary memorization in template-trained preference data — held up to stress-testing. The mechanism (SimPO loss does not distinguish semantic from lexical learning) is correct. The ranking of tests (paraphrase > OOD > confidence calibration > loss curve for this specific confound) was defended and accepted.
