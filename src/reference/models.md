# Reference: Models and Engines

> Quick lookup for model recommendations, engine comparisons, and installation commands.

---

## Model Recommendations by Hardware

### Coding-Specific Models (Prioritize These)

| Model | Size | Q4 Size | VRAM Needed | Coding Strength | Best For |
|---|---|---|---|---|---|
| **Qwen2.5-Coder-1.5B** | 1.5B | ~1 GB | 2 GB | ★★★ | 4–8 GB RAM, CPU-only |
| **Qwen2.5-Coder-3B** | 3B | ~2 GB | 4 GB | ★★★★ | 8 GB RAM, tight VRAM |
| **Qwen2.5-Coder-7B** | 7B | ~4 GB | 8 GB | ★★★★★ | 16 GB RAM (sweet spot) |
| **Qwen2.5-Coder-14B** | 14B | ~9 GB | 16 GB | ★★★★★ | 32 GB RAM |
| **Qwen2.5-Coder-32B** | 32B | ~20 GB | 24 GB | ★★★★★ | 48 GB RAM |
| **DeepSeek-Coder-V2-Lite** | 16B | ~10 GB | 16 GB | ★★★★ | Multi-language, long context |

### Generalist Models (Decent Coding)

| Model | Size | Q4 Size | VRAM Needed | Coding Strength | Notes |
|---|---|---|---|---|---|
| **Phi-3.5-mini** | 3.8B | ~2.5 GB | 4 GB | ★★★ | Great instruction following |
| **Llama-3.1-8B** | 8B | ~5 GB | 8 GB | ★★★ | Good generalist |
| **Mistral-7B-v0.3** | 7B | ~4 GB | 8 GB | ★★★ | Good reasoning |
| **Gemma-2-2B** | 2B | ~1.5 GB | 4 GB | ★★ | Simple tasks only |

### Where to Download

- **Ollama:** `ollama pull <model-name>` (easiest)
- **Hugging Face GGUF:** Search for `model-name GGUF`
- **Bartowski:** Fresh, well-organized quantizations
- **TheBloke:** Extensive library (older but reliable)

---

## Inference Engines Comparison

### GPU (NVIDIA)

| Engine | Best For | Setup Difficulty | Speed | Notes |
|---|---|---|---|---|
| **Ollama** | Ease of use | ⭐ Easy | Good | Best for beginners |
| **llama.cpp** | Portability | ⭐⭐ Moderate | Good | GGUF format, CPU+GPU |
| **ExLlamaV2** | Speed | ⭐⭐⭐ Hard | Fastest | Consumer GPUs only |
| **vLLM** | Throughput | ⭐⭐⭐ Hard | Fastest | Server deployments |

### GPU (AMD/ROCm)

| Engine | Best For | Setup Difficulty | Speed | Notes |
|---|---|---|---|---|
| **llama.cpp** | General purpose | ⭐⭐ Moderate | Good | ROCm support improving |
| **vLLM** | Throughput | ⭐⭐⭐ Hard | Fast | Check version compatibility |

### CPU-Only

| Engine | Best For | Setup Difficulty | Speed | Notes |
|---|---|---|---|---|
| **llama.cpp** | Best CPU inference | ⭐⭐ Moderate | 1–10 tok/s | AVX2/AVX-512 |
| **MLC LLM** | Cross-platform | ⭐⭐⭐ Hard | 1–5 tok/s | Web, mobile, desktop |
| **Ollama** | Ease of use | ⭐ Easy | 1–8 tok/s | Metal support on Mac |

### Apple Silicon

| Engine | Best For | Setup Difficulty | Speed | Notes |
|---|---|---|---|---|
| **llama.cpp** | General purpose | ⭐⭐ Moderate | 10–30 tok/s | Metal backend |
| **MLX** | Apple-native | ⭐⭐⭐ Hard | 15–40 tok/s | Optimized for Apple |
| **Ollama** | Ease of use | ⭐ Easy | 10–25 tok/s | Metal support built-in |

---

## Install Commands by Platform

### macOS

```bash
# Ollama (easiest)
curl -fsSL https://ollama.com/install.sh | sh

# llama.cpp
git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp && mkdir build && cd build
cmake .. -DGGML_METAL=ON
make -j$(sysctl -n hw.ncpu)
```

### Linux (NVIDIA GPU)

```bash
# Ollama
curl -fsSL https://ollama.com/install.sh | sh

# llama.cpp with CUDA
git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp && mkdir build && cd build
cmake .. -DGGML_CUDA=ON
make -j$(nproc)
```

### Linux (AMD GPU)

```bash
# llama.cpp with ROCm
git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp && mkdir build && cd build
cmake .. -DGGML_HIPBLAS=ON
make -j$(nproc)
```

### Windows

```powershell
# Ollama (download installer from ollama.com)
# Then in PowerShell:
ollama pull qwen2.5-coder:7b

# llama.cpp (pre-built binaries)
# Download from: https://github.com/ggml-org/llama.cpp/releases
```

---

## Quantization Guide

### Quantization Formats

| Format | Engine | Size Reduction | Quality Loss | Best For |
|---|---|---|---|---|
| **GGUF Q4_K_M** | llama.cpp, Ollama | ~75% | Minimal | Most tasks |
| **GGUF Q5_K_M** | llama.cpp, Ollama | ~70% | Very minimal | Better quality |
| **GGUF Q8_0** | llama.cpp, Ollama | ~50% | Negligible | When you have space |
| **GGUF Q2_K** | llama.cpp, Ollama | ~85% | Significant | Last resort |
| **AWQ 4-bit** | AutoAWQ | ~75% | Good | Specific models |
| **GPTQ 4-bit** | GPTQ | ~75% | Good | Less portable |

### Quantization and Agent Capabilities

```
Capability preserved at low bits:

Q2    ████████░░░░░░░░░░░░░░░░░░░░░░░░  Text completion
Q3    ████████████░░░░░░░░░░░░░░░░░░░░  Summarization
Q4    ████████████████░░░░░░░░░░░░░░░░  Instruction following ← MINIMUM FOR AGENTS
Q5    ████████████████████░░░░░░░░░░░░  Tool calling/formatting
Q6+   ████████████████████████░░░░░░░░  Complex reasoning
Q8    ████████████████████████████░░░░  Nuanced understanding
```

**For coding agents, Q4_K_M is the practical minimum.** Below that, tool calling and instruction following become unreliable.

---

## Model Selection Decision Tree

```
Do you have a GPU?
├─ No → Use CPU-optimized models
│   ├─ < 8 GB RAM → qwen2.5-coder:1.5b or phi3:mini
│   └─ 8+ GB RAM → qwen2.5-coder:7b (slower but works)
│
└─ Yes → What VRAM do you have?
    ├─ 4–6 GB → qwen2.5-coder:3B Q4
    ├─ 8–12 GB → qwen2.5-coder:7B Q4 (recommended)
    ├─ 16–24 GB → qwen2.5-coder:14B Q4
    └─ 24+ GB → qwen2.5-coder:32B Q4 or 70B Q4
```

---

## Performance Benchmarks (Approximate)

### Token Speed by Hardware

| Hardware | Model | Tokens/Second | Usable? |
|---|---|---|---|
| **CPU (8 cores)** | 1.5B Q4 | 15–25 | ✓ Very fast |
| **CPU (8 cores)** | 7B Q4 | 3–8 | ✓ Usable |
| **CPU (8 cores)** | 14B Q4 | 1–3 | ⚠ Slow but works |
| **RTX 3060 (12GB)** | 7B Q4 | 20–40 | ✓ Excellent |
| **RTX 3060 (12GB)** | 14B Q4 | 10–20 | ✓ Good |
| **RTX 3090 (24GB)** | 14B Q4 | 30–60 | ✓ Excellent |
| **RTX 3090 (24GB)** | 32B Q4 | 15–30 | ✓ Good |
| **M1 MacBook Air** | 7B Q4 | 10–20 | ✓ Good |
| **M2 MacBook Pro** | 14B Q4 | 20–35 | ✓ Excellent |

---

## Troubleshooting

### "Out of memory"

- Use a smaller model
- Reduce context window: `OLLAMA_NUM_CTX=2048`
- Enable layer offloading (llama.cpp: `-ngl 35` for partial GPU)

### "Too slow"

- Use a smaller model
- Enable GPU acceleration
- Reduce thread count (too many threads = overhead)
- Use Q4 instead of Q8

### "Model hallucinates APIs"

- Use a coding-specific model (Qwen-Coder, DeepSeek-Coder)
- Add "Use only standard library" to your prompt
- Provide few-shot examples of correct API usage

### "Invalid JSON output"

- Lower temperature: `temperature=0.1`
- Use XML tags instead of JSON: `<answer>...</answer>`
- Enable JSON mode if your engine supports it

---

## Quick Reference: Copy-Paste Commands

```bash
# Install Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Pull coding models
ollama pull qwen2.5-coder:1.5b   # For 4–8 GB RAM
ollama pull qwen2.5-coder:7b     # For 8–16 GB RAM (recommended)
ollama pull qwen2.5-coder:14b    # For 16–32 GB RAM
ollama pull qwen2.5-coder:32b    # For 32+ GB RAM

# Run a model
ollama run qwen2.5-coder:7b

# Check installed models
ollama list

# Remove a model
ollama rm qwen2.5-coder:14b
```

---

*See also: [Troubleshooting](./troubleshooting.md), [Prompt Templates](./prompt-templates.md)*
