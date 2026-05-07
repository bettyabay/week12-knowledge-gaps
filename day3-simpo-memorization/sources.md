# Sources — Day 3: SimPO Memorization vs. Genuine Learning

---

## Papers

**SimPO: Simple Preference Optimization with a Reference-Free Reward** — Meng et al. (2024)  
The foundational paper for the training method used in Week 11. Section 3 describes the SimPO objective: a margin-based reward on the average log-probability of chosen vs. rejected completions, without a reference model. The key property relevant to the memorization question: the gradient signal does not distinguish between a model that learned a semantic rule and one that learned a vocabulary distribution — both can reduce the SimPO loss. The paper does not address generalization to paraphrased inputs, which is the gap this day's question probes.  
https://arxiv.org/abs/2405.14734

**Direct Preference Optimization: Your Language Model is Secretly a Reward Model** — Rafailov et al. (2023)  
The predecessor to SimPO. DPO Section 4.1 discusses the relationship between the preference optimization objective and the underlying reward model. Useful for understanding why preference training on low-diversity data can produce a model that fits the training distribution without learning the intended reward function — the DPO loss, like the SimPO loss, is minimized by maximizing the probability ratio of chosen vs. rejected, which can be achieved by vocabulary-level pattern matching.  
https://arxiv.org/abs/2305.18290

**A General Language Assistant as a Laboratory for Alignment** — Askell et al. (Anthropic, 2021)  
Section 5 discusses the gap between training performance and behavioral generalization in RLHF-trained models. The core observation: a model that scores well on a held-out set drawn from the same distribution as training data may fail on semantically equivalent inputs with different surface forms. This is the structural problem behind the partner's 82% figure. The paper argues for diverse, adversarial evaluation as a necessary complement to held-out accuracy.  
https://arxiv.org/abs/2112.00861

**Shortcut Learning in Deep Neural Networks** — Geirhos et al. (2020)  
The clearest treatment of the general phenomenon: neural networks preferentially learn the shortest path to a correct answer, which is often a spurious feature rather than the intended rule. In the context of preference training, the "shortcut" is the vocabulary distribution of chosen vs. rejected responses. The paper's framework — shortcut vs. core feature — maps directly onto vocabulary pattern vs. semantic rule in this context.  
https://arxiv.org/abs/2004.07780

---

## Documentation / Code

**SimPO Training Script — TRL Library**  
The `CPOTrainer` in HuggingFace TRL implements SimPO when `loss_type="simpo"` is set. The loss function directly shows why template diversity matters: the gradient updates are computed over token-level log-probabilities of chosen and rejected completions. If all chosen completions share the same surface vocabulary, the gradient will concentrate on those token probabilities rather than distributing across semantically equivalent phrasings.  
https://huggingface.co/docs/trl/cpo_trainer

---

## Concept Reference

**Paraphrase Evaluation as a Generalization Test**  
The standard methodology for paraphrase eval in NLP: take a test set, rewrite each example to preserve meaning while changing vocabulary and surface form, then measure accuracy delta. A model that learned a semantic rule should show near-zero accuracy delta. A model that learned surface patterns will show a significant drop — the exact magnitude depends on how much the paraphrase diverges from training vocabulary. For template-trained models, even moderate paraphrasing (synonym substitution, sentence reordering) is sufficient to expose vocabulary memorization.
