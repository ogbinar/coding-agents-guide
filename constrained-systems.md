# Constrained Systems: Running Agents on Limited Hardware

> Practical techniques for running LLM agents when you don't have an A100.

## 1. Understanding Your Constraints

Before optimizing, know your limits:

```
Resource          | Typical Constrained System
------------------|----------------------------------
GPU VRAM          | 4–24 GB (consumer GPU)
System RAM        | 8–32 GB
CPU cores         | 4–16
Disk              | SSD preferred
Network           | Variable (may be offline)
Power             | Laptop battery or limited
```

## 2. Inference Engines

The right engine can give you 2–5x speedup over naive implementations.

### GPU (NVIDIA)

| Engine | Best For | Notes |
|---|---|---|
| **llama.cpp** | General purpose, CPU+GPU | Most portable, GGUF format |
| **vLLM** | Throughput, PagedAttention | Best for server deployments |
| **ExLlamaV2** | Speed on consumer GPUs | Excellent for 24GB cards |
| **TensorRT-LLM** | Maximum performance | NVIDIA-only, complex setup |
| **Ollama** | Ease of use | Great for quick prototyping |

### GPU (AMD/ROCm)

| Engine | Best For | Notes |
|---|---|---|
| **llama.cpp** | General purpose | ROCm support improving |
| **vLLM** | Throughput | ROCm support (check version) |
| **Triton** | Custom kernels | AMD support growing |

### CPU-Only

| Engine | Best For | Notes |
|---|---|---|
| **llama.cpp** | Best CPU inference | AVX2/AVX-512 acceleration |
| **MLC LLM** | Cross-platform | Web, mobile, desktop |
| **ONNX Runtime** | Optimized CPU | Good for quantized models |

### Apple Silicon

| Engine | Best For | Notes |
|---|---|---|
| **llama.cpp** | General purpose | Metal backend, excellent performance |
| **MLX** | Apple-native | Optimized for Apple Silicon |
| **Ollama** | Ease of use | Metal support built-in |

## 3. Quantization Deep Dive

Quantization is your primary tool for fitting models into constrained systems.

### Quantization Formats

| Format | Engine | Size Reduction | Quality Loss |
|---|---|---|---|
| **GGUF Q4_K_M** | llama.cpp | ~75% | Minimal for most tasks |
| **GGUF Q5_K_M** | llama.cpp | ~70% | Very minimal |
| **GGUF Q8_0** | llama.cpp | ~50% | Negligible |
| **GGUF Q2_K** | llama.cpp | ~85% | Significant for agents |
| **AWQ 4-bit** | AutoAWQ | ~75% | Good for specific models |
| **GPTQ 4-bit** | GPTQ models | ~75% | Good, but less portable |

### Quantization and Agent Capabilities

Different agent capabilities degrade at different rates:

```
Capability preserved best at low bits:
  ┌──────────────────────────────────────────┐
  │ Q2    ████████░░░░░░░░░░░░░░░░░░░░░░░░  │ Text completion
  │ Q3    ████████████░░░░░░░░░░░░░░░░░░░░  │ Summarization
  │ Q4    ████████████████░░░░░░░░░░░░░░░░  │ Instruction following
  │ Q5    ████████████████████░░░░░░░░░░░░  │ Tool calling/formatting
  │ Q6+   ████████████████████████░░░░░░░░  │ Complex reasoning
  │ Q8    ████████████████████████████░░░░  │ Nuanced understanding
  └──────────────────────────────────────────┘
```

**For agents, Q4_K_M is the practical minimum.** Below that, tool calling and instruction following become unreliable.

### Where to Get Quantized Models

- **Hugging Face:** Search for `GGUF` + model name
- **TheBloke** collections — Extensive GGUF library
- **Bartowski** — Fresh, well-organized quantizations
- **Quantize yourself:** Use `llama.cpp`'s `quantize` tool for custom levels

## 4. Memory Optimization Techniques

### KV Cache Management

The KV cache is often the biggest memory consumer after model weights.

```
Memory usage ≈ Model weights + KV cache + overhead

For a 7B Q4 model with 8K context:
  Model weights:  ~4 GB
  KV cache (8K):  ~1 GB
  Overhead:       ~0.5 GB
  Total:          ~5.5 GB
```

**Optimization strategies:**

1. **Reduce context window:** Use 4K instead of 8K if your tasks allow it
2. **KV cache quantization:** Store KV cache in 8-bit or 4-bit instead of FP16
3. **Sliding window attention:** Only attend to recent tokens (supported by some engines)
4. **Ring attention:** For very long contexts, process in chunks

### Offloading Strategies

```
Option A: Full GPU (fastest, needs enough VRAM)
  Model: GPU | KV Cache: GPU | Computation: GPU

Option B: Hybrid (most common for constrained)
  Model: GPU (partial) + CPU | KV Cache: GPU | Computation: Mixed

Option C: CPU-heavy (slowest, most compatible)
  Model: CPU | KV Cache: CPU | Computation: CPU
```

### llama.cpp Layer Offloading

```bash
# Offload all layers to GPU (if VRAM allows)
./main -m model.gguf -ngl 99

# Offload specific layers (tune for your VRAM)
./main -m model.gguf -ngl 35  # ~half on GPU, half on CPU

# CPU-only
./main -m model.gguf -ngl 0
```

## 5. Speed Optimization

### Batch Processing

For agents that make multiple independent calls:

```python
# Instead of sequential calls:
results = [model.chat(prompt) for prompt in prompts]  # Slow

# Use batching if your engine supports it:
results = model.batch_chat(prompts)  # Faster
```

### Speculative Decoding

Use a small "draft" model to propose tokens, validated by the main model:

- **Speedup:** 1.5–2x for coherent text
- **Overhead:** Need two models loaded
- **Best for:** Agents that produce long outputs
- **Tools:** llama.cpp supports speculative decoding with draft models

### Prompt Caching / Prefix Caching

When your system prompt stays the same across calls:

```python
# llama.cpp caches the KV for repeated prefixes
# Only the new tokens need computation
result = model.chat(system_prompt + user_input)  # system_prompt cached
```

### Thread Count Tuning

```bash
# Match threads to your CPU cores (llama.cpp)
./main -m model.gguf -t 8  # 8 threads

# Too many threads → context switching overhead
# Too few threads → underutilized CPU
# Rule of thumb: threads = physical cores (not hyperthreads)
```

## 6. Docker and Containerization

For reproducible constrained deployments:

```dockerfile
FROM nvidia/cuda:12.4-runtime-ubuntu22.04

# Install llama.cpp
RUN apt-get update && apt-get install -y \
    build-essential git cmake \
    && rm -rf /var/lib/apt/lists/*

RUN git clone https://github.com/ggml-org/llama.cpp \
    && cd llama.cpp \
    && mkdir build && cd build \
    && cmake .. -DGGML_CUDA=ON \
    && make -j$(nproc)

WORKDIR /app
COPY . .

CMD ["./llama.cpp/build/bin/main", "-m", "/models/model.gguf", "-ngl", "99"]
```

## 7. Edge and Mobile Considerations

### Mobile (iOS/Android)

- **MLC LLM:** Best cross-platform support
- **MediaPipe:** Good for on-device inference
- **Target model size:** < 2B parameters for real-time, < 4B for batch
- **Quantization:** INT4 or INT8, avoid FP16

### Embedded / IoT

- **TinyLLM / Phi-mini:** Sub-3B models designed for edge
- **ONNX Runtime:** Good hardware acceleration on edge chips
- **NPU utilization:** Check if your device has a neural processing unit

### Raspberry Pi / SBCs

- **llama.cpp** with CPU-only mode
- **Target:** 1–3B parameter models, Q4 or Q5
- **Expect:** 1–5 tokens/second (batch/async use only)

## 8. Benchmarking Your Setup

Know your baseline:

```bash
# llama.cpp benchmark
./llama-bench -m model.gguf -ngl 99 -p 512 -n 256 -t 8

# Key metrics to track:
# - t/s (tokens per second) — throughput
# - tmt (time to first token) — latency
# - model load time — startup cost
```

### Agent-Specific Benchmarks

Beyond raw token speed, measure:

1. **End-to-end task latency** — Time from input to final answer
2. **Memory peak** — Maximum RAM/VRAM during agent execution
3. **Success rate at temperature** — How often does the agent produce valid output?
4. **Iterations to completion** — Average loop count per task

## 9. Practical Hardware Recommendations

### Budget Builds for Local Agents

| Budget | GPU | VRAM | Models You Can Run |
|---|---|---|---|
| $0 (existing) | Integrated / CPU | N/A | 1–3B Q4 (CPU, slow) |
| $200–300 | RTX 3060 | 12 GB | 7B Q4, 3B Q8 |
| $300–400 | RTX 4060 Ti | 16 GB | 14B Q4, 7B Q8 |
| $500–700 | RTX 3090 (used) | 24 GB | 34B Q4, 14B Q8, 70B Q4 (tight) |
| $800–1000 | RTX 4090 | 24 GB | 34B Q5, 70B Q4 |
| $2000+ | RTX 6000 Ada | 48 GB | 70B Q5, Mixtral Q4 |

### Cloud Alternatives for Testing

When local hardware is insufficient for development:

- **RunPod / Vast.ai:** Rent GPUs by the hour ($0.20–0.50/hr for 24GB)
- **Google Colab Pro:** A100 access for experimentation
- **Kaggle Notebooks:** Free T4 GPU (16GB), 30 hrs/week

---

*See also: [agents.md](./agents.md) for core agent principles, [tool-use.md](./tool-use.md) for efficient tool calling.*
