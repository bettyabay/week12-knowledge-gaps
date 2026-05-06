# Week 12 — Knowledge Gaps (TRP1 Challenge)

Daily paired research on AI system internals. Each day I work with a different partner: one person asks a sharp question, the other writes an explainer and tweet thread. We close with an evening call to stress-test the answer.

## Structure

```
week12-knowledge-gaps/
├── day1-inference-mechanics/
│   ├── question.md
│   ├── morning_call_summary.md
│   ├── explainer.md
│   ├── thread.md
│   ├── evening_call_summary.md
│   ├── signoff.md
│   ├── grounding_commit.md
│   └── sources.md
└── ...
```

## Table of Contents

| Day | Topic | Partner | My Role | Blog | Thread | Status |
|-----|-------|---------|---------|------|--------|--------|
| [Day 1](day1-inference-mechanics/) | Inference-time mechanics — Prefill vs. Decode | Kidane | Explainer | [Medium](https://medium.com/@abay.betty.21/prefill-vs-decode-where-your-inference-latency-actually-goes-a796c3495afa) | [X](https://x.com/carinobetty22/status/2051376092359832010) | ✅ |
| [Day 2](day2-tool-calling-mechanics/) | Tool calling mechanics — Token-level tool selection | Mistire | Explainer | [BLOG_URL] | [THREAD_URL] | ⏳ |
| Day 3 | TBD | TBD | TBD | — | — | ⏳ |
| Day 4 | TBD | TBD | TBD | — | — | ⏳ |
| Day 5 | TBD | TBD | TBD | — | — | ⏳ |
| Day 6 | TBD | TBD | TBD | — | — | ⏳ |
| Day 7 | TBD | TBD | TBD | — | — | ⏳ |

## Key Insights

**Day 1:** For tasks with an output/prompt token ratio below 0.2, **prefill dominates latency**. Optimize by compressing prompts, not output length.

**Day 2:** A model "calling a tool" is just next-token prediction producing structured JSON or special-token-wrapped JSON — because fine-tuning made that the high-probability output in tool-relevant contexts. Python routing beats model-driven tool selection for deterministic rules.

## Roles

**Explainer** — takes the partner's question, writes a 600–1000 word blog post and a tweet thread, then defends the answer on the evening call.

**Questioner** — formulates and sharpens a genuine knowledge gap, stress-tests the explainer's answer, and signs off on whether the gap is closed.

## Week Goal

Surface real confusion about how LLM systems work under the hood — not surface-level definitions, but the kind of question that breaks your mental model if you can't answer it precisely.
