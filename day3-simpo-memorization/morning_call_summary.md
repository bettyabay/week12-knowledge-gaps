# Question Sharpening Notes — Day 3

**Date:** 2026-05-07  
**Researcher:** Bethelhem (explainer), Samuel Lachisa (questioner)  
**Format:** Live discussion

---

## Starting Question

"When a model is preference-trained on a small dataset built from templates rather than real diverse conversations, how do you know whether it learned the underlying behavioral preference or just the surface-form vocabulary of the training data?"

---

## Sharpening Round 1 — Separate the theoretical from the diagnostic

The opening question spans two different problems. I flagged this:

> "You're asking two things at once. The first is theoretical: what is the conceptual difference between learning a rule and memorizing a surface pattern? The second is empirical: what would you actually look at in your training logs and eval results to tell them apart? Which one is your real confusion?"

The split:
- **A** — What mechanistically distinguishes a model that has learned a preference from one that has memorized surface vocabulary?
- **B** — Given your specific setup (200 template pairs, 82% eval accuracy), what is the highest-signal test you could run right now to diagnose which case you are in?

I noted that A is answerable from first principles but B is more useful — it requires A as a foundation but forces the answer to be actionable.

**Partner chose B, with A as necessary background.**

---

## Sharpening Round 2 — Pin the diagnosis to a specific confound

With scope fixed to the diagnostic question, it still spanned four different tests (loss curve, paraphrase eval, OOD eval, token confidence). I pressed:

> "All four tests are valid, but they are not equally diagnostic for your setup. Your specific confound is that your eval set was drawn from the same template distribution as your training data. Which of the four tests most directly breaks that confound — and which one would give you a false negative even if the model had memorized?"

The key distinction:
- **Loss curve divergence** — detects overfitting to the training set but does not tell you *what* was memorized (the rule or the vocabulary)
- **Paraphrase eval** — directly tests whether the model transfers meaning across vocabulary, which is exactly the confound at issue
- **OOD eval** — tests template generalization but conflates vocabulary and structure changes
- **Token confidence calibration** — tests whether the model is overconfident, which is a symptom of memorization but not a direct test of it

Paraphrase eval is the highest-signal diagnostic because it isolates the vocabulary variable: same rule, different words.

**Partner agreed to narrow the question to: which test is highest-signal, and why.**

---

## Agreed Scope for the Explainer

Explain the mechanistic difference between genuine learning and memorization in SimPO, then argue specifically why paraphrase eval is the highest-signal diagnostic for a template-trained adapter — and what the result of that test actually tells you about the training process.
