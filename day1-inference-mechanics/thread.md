# Tweet Thread — Day 1: Prefill vs. Decode

*Published at [X](https://x.com/carinobetty22/status/2051376092359832010)*

---

**Tweet 1/5**

My Week 11 ORPO judge runs at ~200ms per inference on a T4 GPU. 500–1000 token prompts, 50–100 token JSON output.

I wanted to know: where does that 200ms actually live? The answer changes which optimization is worth your time 🧵

---

**Tweet 2/5**

LLM inference has two phases with completely different bottlenecks:

**Prefill** — one parallel forward pass over the full prompt. Compute-bound. Scales with prompt length.

**Decode** — generates one token at a time, reading all model weights from memory each step. Memory-bandwidth-bound. Scales with output length.

Same model, two different GPU bottlenecks.

---

**Tweet 3/5**

The latency formula:

```
latency ≈ a × prompt_tokens + b × output_tokens + c
```

`a` (prefill cost/token) is lower because the GPU parallelizes across the sequence.

`b` (decode cost/token) is higher because each token step is sequential and memory-bound.

On a T4, reading 3.5 GB of 4-bit Qwen 2.5 7B weights takes ~11ms per decode step. For 50 tokens: ~550ms theoretical floor — but real-world numbers are lower due to overlap and quantized kernels.

---

**Tweet 4/5**

For my use case: 750 avg prompt tokens vs. 75 avg output tokens. That's a 10:1 ratio.

Even if per-token decode cost `b` > per-token prefill cost `a`, the token count asymmetry tips the balance toward **prefill dominating total latency**.

The intuition: "generation is always the slow part" breaks down when your output is short and your prompt is long.

---

**Tweet 5/5**

Experiment to verify (run this sweep):

```python
torch.cuda.synchronize()
t0 = time.perf_counter()
_ = model(**inputs)          # prefill only
torch.cuda.synchronize()
prefill_ms = (time.perf_counter() - t0) * 1000

t1 = time.perf_counter()
model.generate(**inputs, max_new_tokens=75)
torch.cuda.synchronize()
total_ms = (time.perf_counter() - t1) * 1000
```

The `synchronize()` calls matter — without them you're timing the CPU, not the GPU.

My next move: compress prompts from ~750 to ~400 tokens and re-measure. If prefill dominates, that's a ~35% latency reduction.

Full writeup: https://medium.com/@abay.betty.21/prefill-vs-decode-where-your-inference-latency-actually-goes-a796c3495afa
