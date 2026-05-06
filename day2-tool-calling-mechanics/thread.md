# Tweet Thread — Day 2: Tool Calling at the Token Level

*Published at [X](https://x.com/carinobetty22/status/2052080516875080068?s=46)*

---

**Tweet 1/5**

"The model called a tool" — sounds like something special happened mechanically.

It didn't. The model generated text. Same next-token prediction as always.

Here's what actually happens at the token level when a model "calls a tool" 🧵

---

**Tweet 2/5**

There are no magic tool-call tokens in a base model.

The structured behavior you see is 100% fine-tuning. The model was trained on millions of examples of correct tool-invocation syntax until that pattern became the high-probability completion in the right context.

Different model families implement this differently:

---

**Tweet 3/5**

**GPT-4 (OpenAI tools API):**
The model emits plain JSON —

```
{"name": "book_meeting", "arguments": {"time": "3pm"}}
```

No special tokens. Just JSON the model learned to produce via RLHF + tool-use fine-tuning. The API intercepts it, detects the pattern, and routes it to `tool_calls` in the response.

**Qwen 2.5:**
Uses special tokens added during instruction fine-tuning —

```
<tool_call>
{"name": "book_meeting", "arguments": {...}}
</tool_call>
```

`<tool_call>` is an actual token ID in Qwen's vocabulary.

---

**Tweet 4/5**

The key insight: reliability comes from fine-tuning, not generation-time enforcement.

The model predicts valid JSON because it saw millions of correct examples. The base probability is never zero — it can still hallucinate tool names or wrong argument types. It just does so at a low rate.

That's why schema validation and logit masking exist as a separate layer on top.

---

**Tweet 5/5**

Practical implication: if your routing decision is a deterministic rule (like `is_booking_intent()`), Python beats model-driven tool selection on every dimension —

→ Zero hallucination rate
→ Zero latency overhead
→ Zero token cost
→ Deterministic

Model-driven tool calls are the right architecture when the routing decision *itself* requires language understanding. Not before.

Full writeup: https://medium.com/@abay.betty.21/what-a-model-actually-generates-when-it-calls-a-tool-6c8c51efed73
