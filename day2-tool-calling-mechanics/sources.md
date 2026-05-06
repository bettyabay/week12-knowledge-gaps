# Sources — Day 2: Tool Calling Mechanics

---

## Papers

**Toolformer: Language Models Can Teach Themselves to Use Tools** — Schick et al. (2023)  
The foundational paper on teaching language models to invoke tools via fine-tuning. Section 3 is the most relevant: it describes how tool calls are represented as special token sequences in the training data, and how the model learns to emit them by predicting API call syntax as part of its next-token predictions. Directly supports the claim that tool-calling behavior is entirely a product of training, not a generation-time mechanism.  
https://arxiv.org/abs/2302.04761

**Gorilla: Large Language Model Connected with Massive APIs** — Patil et al. (2023)  
Examines how fine-tuning models on API documentation affects tool-call accuracy and hallucination rates. Useful for the hallucination-rate question Mistire raised in the evening call: shows that accuracy varies significantly with model size, schema complexity, and training data coverage. The "retrieval-aware training" section directly addresses the question of what happens when the tool schema is novel vs. seen during fine-tuning.  
https://arxiv.org/abs/2305.15334

---

## Documentation

**Qwen2.5 Function Calling — HuggingFace Chat Templates**  
Documents exactly what `tokenizer.apply_chat_template(tools=...)` injects into the system prompt for Qwen 2.5 models. Includes the full JSON schema format, the `<tool_call>` / `</tool_call>` special token IDs, and the expected response format. The primary reference for the Qwen-specific section of the explainer.  
https://huggingface.co/docs/transformers/main/en/chat_templating#tools

**OpenAI Function Calling Guide**  
Documents the tools array format, the `tool_calls` response field, and the JSON pattern the model emits. Confirms that the model output is intercepted and re-routed by the API layer — the model produces JSON text, the API surfaces it as a structured object.  
https://platform.openai.com/docs/guides/function-calling

---

## Tool

**Hugging Face `transformers` — `apply_chat_template` with `tools=`**  
The specific Python method used to inject tool schemas into the Qwen 2.5 system prompt. Passing `tools=[{...}]` triggers the tool-call template path, which includes the `<tool_call>` special token vocabulary and the JSON schema injection.  
```python
inputs = tokenizer.apply_chat_template(
    messages,
    tools=[{"name": "book_meeting", "parameters": {...}}],
    tokenize=True,
    add_generation_prompt=True,
    return_tensors="pt"
)
```
https://huggingface.co/docs/transformers/main/en/internal/tokenization_utils#transformers.PreTrainedTokenizerBase.apply_chat_template
