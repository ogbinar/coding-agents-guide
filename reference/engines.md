# Reference: Inference Engines

> Install commands and setup guides for every platform.

---

## Quick Install: Ollama (Recommended)

Ollama is the easiest way to get started with local LLMs.

### All Platforms

```bash
# Install
curl -fsSL https://ollama.com/install.sh | sh

# Pull a coding model
ollama pull qwen2.5-coder:7b

# Run it
ollama run qwen2.5-coder:7b
```

### Windows

Download installer from [ollama.com](https://ollama.com)

```powershell
# After installation
ollama pull qwen2.5-coder:7b
ollama run qwen2.5-coder:7b
```

---

## llama.cpp (Advanced)

Best for: Customization, CPU-only, AMD GPUs

### macOS (Metal)

```bash
git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp
mkdir build && cd build
cmake .. -DGGML_METAL=ON
make -j$(sysctl -n hw.ncpu)

# Run
./main -m /path/to/model.gguf -n 512
```

### Linux (CUDA)

```bash
git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp
mkdir build && cd build
cmake .. -DGGML_CUDA=ON
make -j$(nproc)

# Run
./main -m /path/to/model.gguf -n 512
```

### Linux (ROCm/AMD)

```bash
git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp
mkdir build && cd build
cmake .. -DGGML_HIPBLAS=ON
make -j$(nproc)

# Run
./main -m /path/to/model.gguf -n 512
```

### CPU-Only (Any Platform)

```bash
git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp
mkdir build && cd build
cmake ..  # No GPU flags
make -j$(nproc)

# Run
./main -m /path/to/model.gguf -n 512 -t 8
```

---

## MLX (Apple Silicon Only)

Best for: Maximum performance on Mac

```bash
# Install
pip install mlx-lm

# Download and run a model
mlx_lm.generate --model mlx-community/qwen2.5-coder-7b-instruct-4bit \
  --prompt "Write a Python function to sort a list" \
  --max-tokens 512
```

---

## vLLM (Server Deployments)

Best for: High throughput, multi-user

```bash
# Install
pip install vllm

# Run a server
python -m vllm.entrypoints.api_server \
  --model qwen/qwen2.5-coder-7b-instruct \
  --port 8000

# Query the server
curl http://localhost:8000/generate \
  -d '{
    "prompt": "Write a Python function",
    "max_tokens": 512
  }'
```

---

## ExLlamaV2 (Consumer GPUs)

Best for: Fastest inference on NVIDIA GPUs

```bash
# Install
git clone https://github.com/turboderp/exllamav2
cd exllamav2
pip install -e .

# Run
python example_api.py \
  --model /path/to/model \
  --prompt "Write a Python function"
```

---

## Ollama Configuration

### Environment Variables

```bash
# Context window size (default: 2048)
export OLLAMA_NUM_CTX=4096

# Number of threads (default: physical cores)
export OLLAMA_NUM_THREAD=8

# Maximum models in memory (GPU only)
export OLLAMA_MAX_LOADED=2

# GPU layers (0 = CPU only)
export OLLAMA_NUM_GPU_LAYERS=999
```

### Modelfile (Custom Models)

```dockerfile
FROM qwen2.5-coder:7b

SYSTEM """
You are an expert Python developer. Write clean, production-ready code.
"""

PARAMETER temperature 0.1
PARAMETER num_ctx 4096
```

Create and use:
```bash
ollama create my-coder -f Modelfile
ollama run my-coder
```

---

## Engine Comparison Cheat Sheet

| Engine | Best For | Setup | Speed | Platform |
|---|---|---|---|---|
| **Ollama** | Beginners | ⭐ Easy | Good | All |
| **llama.cpp** | Portability | ⭐⭐ Mod | Good | All |
| **MLX** | Apple Silicon | ⭐⭐ Mod | Excellent | Mac |
| **vLLM** | Servers | ⭐⭐⭐ Hard | Fastest | Linux |
| **ExLlamaV2** | NVIDIA GPUs | ⭐⭐⭐ Hard | Fastest | Linux/Win |

---

## Troubleshooting

### "CUDA not found"
```bash
# Check NVIDIA drivers
nvidia-smi

# Install CUDA toolkit if needed
# See: https://developer.nvidia.com/cuda-toolkit
```

### "ROCm not available"
```bash
# Check AMD GPU
rocm-smi

# Install ROCm drivers
# See: https://rocm.docs.amd.com
```

### "Metal not available on Mac"
```bash
# Rebuild llama.cpp with Metal
cd llama.cpp/build
cmake .. -DGGML_METAL=ON
make clean && make
```

---

*See also: [Models](./models.md), [Troubleshooting](./troubleshooting.md)*
