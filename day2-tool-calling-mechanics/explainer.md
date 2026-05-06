# What a Model Actually Generates When It "Calls a Tool"

*Written for Mistire. Published at [Medium](https://medium.com/@abay.betty.21/what-a-model-actually-generates-when-it-calls-a-tool-6c8c51efed73).*

---

When someone says a model "called a tool," it sounds like the model did something special — reached out, executed a function, triggered an API. It didn't. It generated text. Exactly the same next-token prediction it always does. Understanding this precisely changes how you think about every agentic system you build.

## There Are No Magic Tool-Call Tokens in the Base Model

A raw pretrained language model — before any fine-tuning — has no concept of tool calls. It predicts tokens based on training data. If you show it a function signature and ask it to use it, it might produce something that looks like a function call, or it might not, depending entirely on what patterns it saw during pretraining.

The structured, reliable tool-calling behavior you see in production models is not a base model feature. It is entirely a product of fine-tuning. The model was trained on millions of examples of correct tool invocation syntax until that syntax became the high-probability completion in the right context.

## What GPT-4 Actually Generates

When you pass a tools array to the OpenAI API and the model decides a tool is appropriate, the raw output it generates looks like this:

```json
{"name": "book_meeting", "arguments": {"time": "3pm", "prospect": "Acme Corp"}}
```

This is plain text. There are no special tokens marking it as a tool call in the base vocabulary — it is JSON that the model learned to produce through RLHF and tool-use fine-tuning on OpenAI's side. The training process rewarded the model for producing this exact structure when a function signature was in context and the user's intent matched.

The OpenAI API then intercepts this output before returning it to you. It detects the JSON pattern, parses it, and returns it through the `tool_calls` field of the response object rather than as `content`. From your code's perspective, you receive a structured object. But the model just wrote JSON.

## What Qwen 2.5 Actually Generates

Qwen 2.5 uses a different approach: explicit special tokens added to the model's vocabulary during instruction fine-tuning. When tool use is enabled and the model decides to invoke a function, it generates:

```
<tool_call>
{"name": "book_meeting", "arguments": {"time": "3pm", "prospect": "Acme Corp"}}
</tool_call>
```

The `<tool_call>` and `</tool_call>` markers are actual tokens in Qwen's vocabulary — token IDs that correspond to these strings, added specifically to signal tool invocation. They are not regular text that happens to look like XML. The model learned during fine-tuning that when a function signature is present in the system prompt and the context calls for it, the correct behavior is to open a `<tool_call>` block, write valid JSON matching the function signature, and close the block.

The chat template — applied by `tokenizer.apply_chat_template()` when you set `tools=` — injects the function schemas into the system prompt in a format the model was trained to recognize. This is why you must use the chat template correctly for tool calling to work: the model is pattern-matching against what it saw during fine-tuning, and the template produces that exact pattern.

## Why This Works at All: Fine-Tuning Does the Heavy Lifting

In both cases, the mechanism is the same:

1. The function schema is placed in context (system prompt or tools array)
2. The model predicts the highest-probability continuation given that context
3. Because of fine-tuning, the highest-probability continuation in a tool-relevant context is a correctly structured tool call

The "reliability" is a statistical property of the fine-tuned distribution. The model was shown so many correct examples that valid JSON with the right field names became the overwhelmingly likely output. It is not enforced at the generation level — that is the A2 layer, a separate mechanism involving logit masking or grammar-guided sampling.

This distinction matters: a fine-tuned model can still hallucinate tool names or argument types that don't match the schema. It just does so at a low rate because fine-tuning drove that probability down. The base probability is never zero.

## What This Means for Mistire's `agent.py`

Mistire's agent routes via Python conditionals at lines 259–270. Before the LLM ever runs, `is_booking_intent()` inspects the input and decides which branch to take. The model is never given a tools array, never generates a `<tool_call>` block, and never has the opportunity to select a function.

This is not a bug. For a deterministic rule like `is_booking_intent()`, Python routing has properties that model-driven tool selection cannot match:

- **Zero hallucination rate** — the conditional either fires or it doesn't, based on explicit logic
- **Zero latency overhead** — no LLM call needed to make the routing decision
- **Zero token cost** — the routing decision costs nothing in tokens
- **Deterministic** — same input, same routing decision, every time

Model-driven tool selection is the right architecture when the routing decision is *itself* a language understanding problem — when determining which tool to call requires reading ambiguous natural language, handling edge cases that don't fit explicit rules, or combining information across multiple fields that no deterministic function handles cleanly.

If `is_booking_intent()` is 100% accurate, replacing it with a `<tool_call>` prediction introduces latency, token cost, and a nonzero hallucination rate in exchange for no improvement in routing quality. The architectural question is not "should I use tool calling" but "which routing decisions are hard enough to need a language model?"

## The Short Version

A model "calling a tool" is next-token prediction that happens to produce structured JSON or special-token-wrapped JSON, because fine-tuning made that the high-probability output in tool-relevant contexts. No magic, no special mechanism at the generation level. The API layer intercepts the output and presents it to you as a structured tool call, but underneath it is just text the model wrote.

The reason Python routing can outperform model-driven tool selection for deterministic rules is exactly this: you are replacing a statistical process (high-probability token prediction) with a logical one (boolean evaluation). When the logical version is accurate, the statistical version is strictly worse.

---

*Sources in [sources.md](sources.md). Thread: [X](https://x.com/carinobetty22/status/2052080516875080068?s=46).*
