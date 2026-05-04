# Sources — Day 1: Inference-time Mechanics / Prefill vs. Decode

---

## Papers

**Efficient Streaming Language Models with Attention Sinks** — Xiao et al. (2023)  
Directly relevant to understanding how KV cache grows during decode and the memory pressure it creates. Section 2 explains the prefill/decode distinction and why the KV cache from prefill persists across all decode steps. The attention sink finding is a bonus but the latency analysis is the main reference.  
https://arxiv.org/abs/2309.17453

**FlashAttention-2: Faster Attention with Better Parallelism and Work Partitioning** — Dao (2023)  
Explains why prefill computation is GPU-compute-bound rather than memory-bound at typical sequence lengths, and how tiled computation enables the O(n²) attention to run efficiently. The discussion of FLOP utilization vs. memory bandwidth in Section 3 directly supports the prefill/decode bottleneck distinction used in the explainer.  
https://arxiv.org/abs/2307.08691

---

## Tools

**PyTorch Profiler** (`torch.profiler`)  
The recommended tool for decomposing GPU time between prefill and decode beyond what `perf_counter` can show. The profiler can attribute CUDA kernel time to individual operations, which allows you to see exactly how much time is spent in attention computation (prefill) vs. linear layers during decode steps.  
Documentation: https://pytorch.org/docs/stable/profiler.html  
Usage for inference timing:
```python
with torch.profiler.profile(
    activities=[torch.profiler.ProfilerActivity.CUDA],
    record_shapes=True,
) as prof:
    outputs = model.generate(**inputs, max_new_tokens=75)

print(prof.key_averages().table(sort_by="cuda_time_total", row_limit=20))
```

---

## Secondary References

**Hugging Face — `use_cache` and `past_key_values`**  
Documents how the transformers library implements KV cache across decode steps. Useful for understanding what `model.generate()` is doing internally at the framework level.  
https://huggingface.co/docs/transformers/main/en/perf_infer_gpu_one

**NVIDIA T4 GPU Datasheet**  
Source for the 320 GB/s memory bandwidth figure used in the decode step floor calculation.  
https://www.nvidia.com/en-us/data-center/tesla-t4/

**BitsAndBytes NF4 Quantization**  
Source for the 4-bit = 0.5 bytes/parameter accounting and the effective model size calculation (~3.5 GB for Qwen 2.5 7B in NF4).  
https://huggingface.co/blog/4bit-transformers-bitsandbytes
