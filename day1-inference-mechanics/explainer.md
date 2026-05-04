# Prefill vs. Decode: Where Your 200ms Actually Goes

*Written for Kidane. Published at [Medium](https://medium.com/@abay.betty.21/prefill-vs-decode-where-your-inference-latency-actually-goes-a796c3495afa).*

---

My Week 11 ORPO judge reports ~200ms per inference call on a T4 GPU. The model is Qwen 2.5 7B, 4-bit quantized, processing 500–1000 token prompts and returning 50–100 tokens of structured JSON. I want to know where that 200ms lives — so I know whether to compress the prompt or shorten the output.

## Two Phases, Two Bottlenecks

Every transformer inference call has two mechanically distinct phases.

**Prefill** is a single forward pass over the entire input prompt. All tokens are processed in parallel. The GPU computes attention over the full sequence simultaneously, writes the resulting key-value tensors into cache, and produces hidden states for each input token. Because the computation is batched across the sequence dimension, prefill is limited by the GPU's **compute throughput** (FLOP/s). A longer prompt means more work, but the GPU can parallelize across positions.

**Decode** is a sequential loop. The model generates one new token per step. At each step, only the new token's query is computed; the keys and values for all prior tokens are read from the KV cache. Because each step loads the full model weights from memory to compute a single token, decode is limited by **memory bandwidth** (GB/s) — not compute. The GPU is mostly idle waiting for weights to arrive from VRAM.

These two bottlenecks — compute-bound vs. memory-bandwidth-bound — are why prompt length and output length have fundamentally different cost structures.

## The Latency Formula

Total inference time can be modeled as:

```
latency ≈ a × prompt_tokens + b × output_tokens + c
```

Where:
- `a` = amortized prefill cost per prompt token (lower, because parallel)
- `b` = decode cost per output token (higher, because sequential and memory-bound)
- `c` = fixed overhead: CUDA kernel launch, tokenization, sampling, Python call stack

On a T4 (320 GB/s memory bandwidth), the per-token decode cost for Qwen 2.5 7B at 4-bit quantization is bounded by how fast the GPU can read the model weights. At ~3.5 GB of quantized weights, each decode step reads the entire model once:

```
decode_step_floor = 3.5 GB / 320 GB/s ≈ 11 ms per token
```

For 50 output tokens, that alone is ~550ms in the worst case — already more than the 200ms I reported. This tells me one of three things: my decode steps are interleaved with KV cache reads that overlap with compute, the quantized kernel is faster than the naive bandwidth estimate, or my actual output is shorter than I assumed. The experiment below will tell me which.

## Why Prefill May Still Dominate at This Token Ratio

The bandwidth-bound argument above applies to single-sample decode. But prefill has its own cost that scales with sequence length. For a transformer with `L` layers, hidden dimension `d`, and `n` input tokens, the attention computation is O(n² × d × L) in FLOPs. With FlashAttention, this is computed in fused tiled kernels, but the quadratic scaling with sequence length still exists in the worst case.

At 500–1000 prompt tokens and only 50 output tokens, the **token ratio** is 10:1 to 20:1. Even if per-token decode cost `b` is several times larger than per-token prefill cost `a`, the asymmetry in token counts can tip the balance toward prefill dominating total latency.

As a concrete prior: for tasks where `output_tokens / prompt_tokens < 0.2` (my case: 50–100 / 500–1000), prefill is likely to be the larger contributor to total time on compute-capable hardware like the T4. This is especially true for short JSON outputs where sampling terminates quickly.

**Implication:** shortening the prompt is probably the higher-leverage optimization. Reducing prompt tokens from 800 to 400 cuts prefill time roughly in half. Reducing output tokens from 75 to 50 cuts decode time by a third — but if decode is only 30% of total time, the net win is smaller.

## Experiment Design

To verify this, run a sweep across prompt lengths with output length held fixed, then run another sweep across output lengths with prompt length held fixed:

```python
import time
import torch
from transformers import AutoTokenizer, AutoModelForCausalLM, BitsAndBytesConfig

model_id = "Qwen/Qwen2.5-7B-Instruct"
bnb_config = BitsAndBytesConfig(load_in_4bit=True, bnb_4bit_compute_dtype=torch.float16)

tokenizer = AutoTokenizer.from_pretrained(model_id)
model = AutoModelForCausalLM.from_pretrained(
    model_id, quantization_config=bnb_config, device_map="auto"
)

def timed_generate(prompt: str, max_new_tokens: int) -> dict:
    inputs = tokenizer(prompt, return_tensors="pt").to(model.device)
    prompt_len = inputs["input_ids"].shape[-1]

    # Warm up CUDA
    with torch.no_grad():
        _ = model.generate(**inputs, max_new_tokens=5)

    torch.cuda.synchronize()
    t0 = time.perf_counter()

    with torch.no_grad():
        outputs = model.generate(**inputs, max_new_tokens=max_new_tokens)

    torch.cuda.synchronize()
    t1 = time.perf_counter()

    output_len = outputs.shape[-1] - prompt_len
    return {
        "prompt_tokens": prompt_len,
        "output_tokens": output_len,
        "total_ms": (t1 - t0) * 1000,
    }

# Sweep 1: vary prompt length, fix output length
results = []
base_prompt = "Evaluate the following email: " + ("x " * 100)
for multiplier in [1, 2, 4, 6, 8]:
    prompt = base_prompt * multiplier
    r = timed_generate(prompt, max_new_tokens=60)
    results.append(r)
    print(r)
```

Fit `a` and `b` by running both sweeps and solving:

```
latency_1 = a × n1 + b × m + c
latency_2 = a × n2 + b × m + c
```

Subtracting: `a = (latency_1 - latency_2) / (n1 - n2)`. Do the same for `b`. This gives you empirical coefficients that are specific to your GPU, quantization scheme, and batch size.

## Separating Prefill Time from Decode Time

To measure the phases independently without a custom CUDA profiler:

```python
torch.cuda.synchronize()
t_prefill_start = time.perf_counter()

# Run only the prefill: forward pass with no generation
with torch.no_grad():
    _ = model(**inputs)  # prefill only, no .generate()

torch.cuda.synchronize()
t_prefill_end = time.perf_counter()
prefill_ms = (t_prefill_end - t_prefill_start) * 1000

# Then measure full generate
torch.cuda.synchronize()
t_gen_start = time.perf_counter()

with torch.no_grad():
    outputs = model.generate(**inputs, max_new_tokens=max_new_tokens)

torch.cuda.synchronize()
t_gen_end = time.perf_counter()
total_ms = (t_gen_end - t_gen_start) * 1000
decode_ms = total_ms - prefill_ms
```

`torch.cuda.synchronize()` before each `perf_counter()` call is essential — without it, the CPU timer returns before the GPU has finished, giving you measurement artifacts.

## Conclusion: Compress the Prompt First

For the Week 11 ORPO judge at 500–1000 prompt tokens and 50–100 output tokens, the prior is that prefill dominates total latency. The 10:1+ token ratio and the compute-heavy nature of processing long sequences on the T4 both point in this direction. The bandwidth-bound argument for decode dominating applies more strongly when output lengths are hundreds of tokens — not 50.

The recommended optimization order:
1. **Compress the prompt** — remove redundant instructions, abbreviate the rubric, truncate candidate emails to the most distinctive sections.
2. **Measure before cutting outputs** — the JSON response is already short; cutting it further risks accuracy loss without meaningful latency gain.
3. **Run the sweep** — confirm with empirical coefficients before committing to either change.

If prompt compression brings the average prompt from 750 tokens to 400 tokens, the model predicts a latency reduction proportional to `a × 350`. If `a` is around 0.2ms/token, that's a 70ms reduction — roughly 35% of the reported 200ms.

---

*Sources in [sources.md](sources.md). Thread: [X](https://x.com/carinobetty22/status/2051376092359832010).*
