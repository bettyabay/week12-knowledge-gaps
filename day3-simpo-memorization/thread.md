# Tweet Thread — Day 3: SimPO Memorization vs. Genuine Learning

*Published at [X — link pending]*

---

**Tweet 1/5**

Your SimPO adapter hit 82% on the eval set.

But every training pair came from the same template. Every chosen response used the same escalation phrasing. Every rejected response used the same commitment phrasing.

Did it learn the rule — or just the words? 🧵

---

**Tweet 2/5**

There are two very different things a preference-trained model can learn from template data:

**The rule:** Don't commit capacity you can't deliver. Escalate instead.

**The vocabulary:** The word "immediately" is in the rejected column.

82% accuracy on a template eval cannot tell you which one happened.

---

**Tweet 3/5**

Here's why. SimPO maximizes the log-probability ratio of chosen over rejected completions.

If chosen responses always use the same escalation phrases and rejected responses always use the same commitment phrases — the optimizer can reduce loss by learning the vocabulary distribution, not the semantic rule.

Loss goes down either way.

---

**Tweet 4/5**

There are four diagnostic tests: loss curve divergence, token confidence calibration, OOD eval, and paraphrase eval.

The highest-signal one for template-trained data is **paraphrase eval**.

Same meaning. Different words.

→ "Available to start immediately" → "Can join your team right away"
→ "Enterprise director will be in touch" → "A senior account manager will follow up"

If accuracy holds: the model learned the rule.
If accuracy drops: the model memorized the vocabulary.

---

**Tweet 5/5**

If the paraphrase eval fails, it's not a SimPO problem. It's a data coverage problem.

The fix: augment your 200 training pairs with paraphrased variants of the same preference. Let the model see the same rule in multiple surface forms.

82% on templates is a start. Paraphrase eval tells you if it's real.

Full writeup: [Medium — link pending]
