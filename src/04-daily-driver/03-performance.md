# Chapter 18: Squeezing Performance

> Memory management, speed optimization, CPU-only survival, and when to upgrade hardware vs optimize software.

---

## TL;DR

- **Memory management:** KV cache, layer offloading, GPU/CPU hybrid
- **Speed optimization:** Thread count, prefix caching, context tuning
- **CPU-only viable:** Yes, with small models (1.5B-7B)
- **Upgrade decision:** Know when software optimization hits limits

---

## Why Performance Matters

Slow AI = Frustrated developer.

**Acceptable speeds:**
- ✅ 20+ tokens/sec: Feels instant, great for chat
- ✅ 10-20 tokens/sec: Good for coding assistance
- ✅ 5-10 tokens/sec: Usable, acceptable for most tasks
- ⚠️ 2-5 tokens/sec: Slow but workable for complex tasks
- ❌ <2 tokens/sec: Painful, consider upgrades

**Key insight:** You can often get 2-3x performance improvements with proper tuning, without buying new hardware.

---

## Memory Management

### Understanding Memory Usage

When you load a model, it uses memory for:

1. **Model weights:** The actual model parameters
2. **KV cache:** Key-value cache for attention (grows with context)
3. **Activation memory:** Temporary computation buffers
4. **Overhead:** Framework, buffers, etc.

**Total memory = Model weights + KV cache + Activations + Overhead**

### Model Size vs Memory

| Model | Q4 Quantized | VRAM Needed | RAM Needed (CPU) |
|---|---|---|---|
| 1.5B | ~1 GB | 2 GB | 2 GB |
| 3B | ~2 GB | 4 GB | 4 GB |
| 7B | ~4 GB | 8 GB | 8 GB |
| 14B | ~9 GB | 16 GB | 16 GB |
| 32B | ~20 GB | 24 GB | 24 GB |
| 70B | ~40 GB | 48 GB | 48 GB |

**Rule of thumb:** Add 20-30% for KV cache and overhead.

### KV Cache Optimization

**What is KV cache?**

The KV (Key-Value) cache stores attention computations so the model doesn't recompute them for each token. Larger context = larger KV cache.

**Control KV cache size:**

```bash
# Ollama: Reduce context window
OLLAMA_NUM_CTX=2048 ollama run qwen2.5-coder:7b

# llama.cpp: Context size
./main -m model.gguf -c 2048
```

**Trade-off:**
- Smaller context → Less memory, faster
- Larger context → More context, more memory

**Recommendation:**
- **8GB RAM:** Context 2048-4096
- **16GB RAM:** Context 4096-8192
- **32GB+ RAM:** Context 8192+

### Layer Offloading (GPU/CPU Hybrid)

**Concept:** Split model layers between GPU and CPU.

```
GPU: Layers 0-35 (fast)
CPU: Layers 36-79 (slower but works)
```

**Why it works:**
- GPU handles what fits in VRAM
- CPU handles the rest
- You can run larger models than your VRAM allows

**llama.cpp example:**
```bash
# Offload 35 layers to GPU, rest on CPU
./main -m model.gguf -ngl 35 -t 8
```

**Ollama example:**
```bash
# Ollama automatically does hybrid offloading
# Control with environment variables
OLLAMA_NUM_GPU_LAYERS=35 ollama run qwen2.5-coder:14b
```

**Performance impact:**
- All GPU: 50-100 tokens/sec
- Hybrid (35 GPU layers): 10-30 tokens/sec
- All CPU: 2-8 tokens/sec

### Memory Mapping

**Concept:** Load model from disk on-demand instead of all at once.

```bash
# llama.cpp: Use memory mapping
./main -m model.gguf -mmap
```

**Benefits:**
- Lower initial memory usage
- Can load models larger than RAM (slowly)

**Drawbacks:**
- Slower initial load
- Disk I/O bottleneck

**Use when:**
- Model barely fits in RAM
- You're willing to wait for initial load

---

## Speed Optimization

### Thread Count Tuning

**Critical insight:** Use **physical cores**, not logical threads.

**Find your physical cores:**
```bash
# Linux
lscpu | grep "Core(s) per socket"

# macOS
sysctl -n hw.physicalcpu

# Windows (PowerShell)
Get-WmiObject Win32_Processor | Select-Object NumberOfCores
```

**Set thread count:**
```bash
# Match physical cores
OLLAMA_NUM_THREAD=8 ollama run qwen2.5-coder:7b

# llama.cpp
./main -m model.gguf -t 8
```

**Why not hyperthreads?**
- LLM inference is memory-bound, not CPU-bound
- Hyperthreads share memory bandwidth
- Using hyperthreads often **slows down** inference

**Rule:** Use physical cores, not logical threads.

### Batch Size Optimization

**What is batch size?**

Number of tokens processed in parallel during prompt encoding.

```bash
# Ollama
OLLAMA_NUM_BATCH=4096 ollama run qwen2.5-coder:7b

# llama.cpp
./main -m model.gguf -b 4096
```

**Trade-offs:**
- Larger batch → Faster prompt encoding, more memory
- Smaller batch → Slower encoding, less memory

**Recommendation:**
- **8GB RAM:** Batch 1024-2048
- **16GB RAM:** Batch 2048-4096
- **32GB+ RAM:** Batch 4096-8192

### Prefix Caching

**Concept:** Cache computations for repeated prompt prefixes.

**How it works:**
```
Prompt 1: "You are a helpful coding assistant. [system prompt] User: Write a function..."
Prompt 2: "You are a helpful coding assistant. [system prompt] User: Write another function..."

↑ Same prefix → Cached computation ↓
```

**Enable prefix caching:**
```bash
# llama.cpp
./main -m model.gguf -fa  # Flash attention (includes prefix caching)
```

**Ollama:** Automatic with same system prompt.

**Benefits:**
- 20-50% faster for repeated prompts
- No memory overhead (cache is small)

**Strategy:**
- Keep system prompt constant
- Only change user messages
- Use templates for common prompts

### Quantization Trade-offs

**Quantization:** Reduce precision to save memory and speed up inference.

| Quantization | Size (7B model) | Speed | Quality |
|---|---|---|---|
| FP16 | 14 GB | Fastest | Best |
| Q8 (8-bit) | 8 GB | Fast | Very good |
| Q6 | 6 GB | Fast | Good |
| Q4 | 4 GB | Fastest | Good |
| Q3 | 3 GB | Fastest | Fair |
| Q2 | 2 GB | Fastest | Poor |

**For coding:**
- **Q4 or Q6:** Sweet spot (good quality, small size)
- **Q8:** If you have memory to spare
- **Q3 or lower:** Only if absolutely necessary

**Test quality:**
```bash
# Compare Q4 vs Q6 on your tasks
ollama run qwen2.5-coder:7b-q4
ollama run qwen2.5-coder:7b-q6

# Same prompt, compare output quality
```

---

## CPU-Only Survival Guide

### Yes, CPU-Only Works

**Reality check:** You don't need a GPU to use coding agents.

**Speed expectations:**

| Model | 4-core CPU | 8-core CPU | 16-core CPU |
|---|---|---|---|
| 1.5B | 10-15 tok/s | 20-30 tok/s | 40-60 tok/s |
| 3B | 5-8 tok/s | 10-15 tok/s | 20-30 tok/s |
| 7B | 2-4 tok/s | 5-8 tok/s | 10-15 tok/s |
| 14B | 1-2 tok/s | 2-4 tok/s | 5-8 tok/s |

**Usable thresholds:**
- ✅ 1.5B-3B on 8-core: Very usable
- ✅ 7B on 8-core: Acceptable
- ⚠️ 14B on 8-core: Slow but works
- ❌ 32B+ on CPU: Too slow

### CPU Optimization Tips

**1. Use AVX2/AVX-512**

Modern CPUs have vector instructions that speed up inference.

```bash
# llama.cpp: Enable AVX2
./main -m model.gguf -fa

# Check if your CPU supports AVX2
lscpu | grep avx
```

**2. Optimize memory bandwidth**

- Use DDR4/DDR5 RAM (not DDR3)
- Dual-channel memory configuration
- High RAM speed (3200MHz+)

**3. Reduce context**

```bash
# Smaller context = less memory bandwidth needed
OLLAMA_NUM_CTX=2048 ollama run qwen2.5-coder:7b
```

**4. Use smaller models**

```bash
# 1.5B or 3B models are fast on CPU
ollama pull qwen2.5-coder:1.5b
ollama pull qwen2.5-coder:3b
```

**5. Close other apps**

- Free up RAM
- Reduce CPU contention
- Disable background processes

**6. Context reduction strategy**

When CPU inference is slow, reduce context aggressively:
- Start with 2048 tokens for simple coding tasks
- Increase to 4096 only when you need longer context
- Use prefix caching to avoid re-encoding system prompts
```

### When CPU-Only Makes Sense

**Good for:**
- ✅ Learning and experimentation
- ✅ Simple coding tasks
- ✅ Privacy-sensitive work (no cloud)
- ✅ Offline work (travel, remote)
- ✅ Budget constraints

**Not good for:**
- ❌ Production workloads
- ❌ Complex reasoning tasks
- ❌ Large context requirements
- ❌ Time-sensitive work

---

## Budget Hardware Guide

### When to Upgrade

**Signs you need better hardware:**
- ❌ Consistently <2 tokens/sec
- ❌ Can't load models you need
- ❌ Out of memory errors
- ❌ Frustrated with speed

**Signs you should optimize first:**
- ⚠️ 3-5 tokens/sec (acceptable with tuning)
- ⚠️ Occasional OOM (reduce context)
- ⚠️ New to local LLMs (learn first)

### GPU Buying Guide

**Best value GPUs for LLMs:**

| GPU | VRAM | Price (used) | Model Size |
|---|---|---|---|
| RTX 3060 | 12GB | ~$200 | 7B (Q4-Q6) |
| RTX 3090 | 24GB | ~$700 | 32B (Q4) |
| RTX 4060 Ti | 16GB | ~$400 | 14B (Q4-Q6) |
| RTX 4090 | 24GB | ~$1600 | 32B (Q6-Q8) |
| RTX 3050 | 8GB | ~$150 | 7B (Q4) |

**Why VRAM matters more than raw speed:**
- Model must fit in VRAM first
- Speed comes second
- 24GB > fast 8GB for LLMs

**Best bang-for-buck:**
1. **RTX 3060 12GB** (~$200): Best entry-level
2. **RTX 3090 24GB** (~$700): Best value for large models
3. **RTX 4060 Ti 16GB** (~$400): Best new mid-range

### RAM Upgrades

**If staying CPU-only:**

| RAM | Model Size | Price |
|---|---|---|
| 16 GB | 3B (Q4) | ~$50 |
| 32 GB | 7B (Q4) | ~$100 |
| 64 GB | 14B (Q4) | ~$200 |
| 128 GB | 32B (Q4) | ~$400 |

**Recommendation:** 32GB is the sweet spot for CPU-only coding.

### Cloud Rental (Alternative to Buying)

**When you need occasional power:**

| Provider | A100 (80GB) | Price/hour |
|---|---|---|
| RunPod | A100 80GB | ~$1.50/hr |
| Vast.ai | A100 80GB | ~$1.20/hr |
| Lambda Labs | A100 80GB | ~$1.80/hr |

**Use case:**
- Run large models occasionally
- Test before buying
- Burst capacity

**Not cost-effective for:**
- Daily use (buy hardware instead)
- Small models (CPU works fine)

---

## Performance Tuning Workflow

### Step 1: Measure Baseline

```bash
# Measure current speed
time ollama run qwen2.5-coder:7b "Write a function to sort a list"

# Check memory usage
htop  # Linux
Activity Monitor  # macOS
Task Manager  # Windows
```

### Step 2: Tune Thread Count

```bash
# Find physical cores
lscpu | grep "Core(s)"

# Test with different thread counts
OLLAMA_NUM_THREAD=4 ollama run qwen2.5-coder:7b
OLLAMA_NUM_THREAD=8 ollama run qwen2.5-coder:7b
OLLAMA_NUM_THREAD=12 ollama run qwen2.5-coder:7b

# Pick fastest
```

### Step 3: Tune Context

```bash
# Test different context sizes
OLLAMA_NUM_CTX=2048 ollama run qwen2.5-coder:7b
OLLAMA_NUM_CTX=4096 ollama run qwen2.5-coder:7b

# Balance speed vs context needs
```

### Step 4: Try Different Quantizations

```bash
# Compare Q4 vs Q6
ollama pull qwen2.5-coder:7b-q4
ollama pull qwen2.5-coder:7b-q6

# Test both on your tasks
# Pick best quality/speed balance
```

### Step 5: Enable GPU Offloading (if available)

```bash
# Check GPU detection
ollama list

# Force GPU layers
OLLAMA_NUM_GPU_LAYERS=35 ollama run qwen2.5-coder:14b
```

### Step 6: Re-measure

```bash
# Compare before/after
time ollama run qwen2.5-coder:7b "Write a function to sort a list"

# Calculate improvement
```

---

## Exercise: Performance Tuning

**Task:** Optimize your setup for best performance on Project A tasks.

**Steps:**

1. **Measure baseline** (5 min)
    ```bash
    time ollama run qwen2.5-coder:7b "Write a search function for the log parser"
    ```

2. **Tune threads** (10 min)
    - Find physical cores
    - Test 3 different thread counts
    - Pick fastest

3. **Tune context** (10 min)
    - Test 2048, 4096, 8192
    - Pick best balance for your Project A codebase size

4. **Try quantizations** (10 min)
    - Compare Q4 vs Q6 on a Project A task
    - Pick best quality/speed

5. **Measure improvement** (5 min)
    - Compare before/after on the same Project A prompt
    - Calculate speedup

**Timebox:** 40 minutes

**Success criteria:** 20-50% speed improvement or better quality at same speed.

---

## On Your Hardware

### 4-8 GB RAM

- CPU-only with 1.5B model: Fast, usable
- Use context 2048 max
- Physical cores: Use all available
- Quantization: Q4 minimum (Q3 if desperate)

### 8-16 GB RAM

- CPU-only with 3B-7B model: Usable
- GPU with 7B model: Fast
- Use context 4096
- Quantization: Q4-Q6

### 16-32 GB RAM

- CPU-only with 7B-14B: Acceptable
- GPU with 14B-32B: Good
- Use context 8192
- Quantization: Q4-Q6

### 32 GB+ RAM

- CPU-only with 14B-32B: Usable
- GPU with 32B: Excellent
- Use context 16384+
- Quantization: Q4-Q8

---

## Performance Checklist

### Daily Driver Setup

```
[ ] Thread count matches physical cores
[ ] Context size appropriate for tasks
[ ] Quantization balanced (Q4-Q6 for coding)
[ ] GPU layers optimized (if GPU available)
[ ] Prefix caching enabled (constant system prompt)
[ ] Memory usage monitored (no swapping)
```

### Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| **Out of memory** | Model too large | Use smaller model or quantization |
| **Very slow** | Wrong thread count | Match physical cores |
| **Slow with long prompts** | Large context | Reduce context size |
| **Inconsistent speed** | Background processes | Close other apps |
| **GPU not used** | Driver issue | Check CUDA installation |

---

## When to Upgrade vs Optimize

### Optimize First If

- ✅ Speed is 3-10 tokens/sec (acceptable with tuning)
- ✅ Occasional OOM errors (reduce context)
- ✅ New to local LLMs (learn tuning first)
- ✅ Budget is tight (optimize before spending)

### Upgrade If

- ❌ Consistently <2 tokens/sec after tuning
- ❌ Can't run models you need (even smallest)
- ❌ Daily frustration with speed
- ❌ Professional use (time is money)

---

## Next Steps

- [Chapter 19: Quality Checks](./04-quality-checks.md) - Learn to evaluate your setup
- [Chapter 18 Reference](../reference/models.md) - Model recommendations by hardware
