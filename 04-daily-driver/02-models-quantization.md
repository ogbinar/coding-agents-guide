# Chapter 17: Models and Quantization

> Model size vs capability, what quantization actually means, and which model to run on your hardware.

---

## TL;DR

- Model size determines capability: larger = better at complex tasks
- Quantization reduces memory at cost of some quality
- Q4 (4-bit) is the sweet spot for coding agents
- Recommendation matrix by hardware tier included
- Download from trusted sources (Bartowski, TheBloke)

---

## Prerequisites

- Completed [Part 1-3](../01-beginner/01-write-code.md) through [Chapter 16](./01-your-setup.md)
- Have used agents for coding
- Ready to optimize your model selection

---

## Model Size vs Capability

### The Hierarchy

```
Capability by model size:

1.5B    ████████░░░░░░░░░░░░░░░░░░░░░░░░  Basic code completion
3B      ████████████░░░░░░░░░░░░░░░░░░░░  Simple functions
7B      ████████████████░░░░░░░░░░░░░░░░  Good coding (sweet spot)
14B     ████████████████████░░░░░░░░░░░░  Complex features
32B     ████████████████████████░░░░░░░░  Architecture, design
70B     ████████████████████████████░░░░  Near-cloud quality
```

### What Each Size Can Do

#### 1.5B-3B Models

**Can do:**
- Simple function generation
- Code completion
- Basic debugging
- Test generation (simple cases)

**Struggles with:**
- Complex reasoning
- Multi-file changes
- Architecture decisions
- Subtle bug detection

**Best for:** Autocomplete, quick snippets, CPU-only systems

#### 7B Models (Sweet Spot)

**Can do:**
- Full function generation
- Multi-step debugging
- Test generation (comprehensive)
- Code review (common issues)
- Refactoring (straightforward)

**Struggles with:**
- Very complex architecture
- Subtle security issues
- Large-scale migrations

**Best for:** Daily coding, most tasks, 16GB RAM systems

#### 14B-32B Models

**Can do:**
- Complex feature implementation
- Architecture design
- Code review (subtle issues)
- Large refactoring
- Framework migrations

**Struggles with:**
- Extremely complex systems
- Cutting-edge frameworks

**Best for:** Complex projects, 32GB+ RAM systems

#### 70B+ Models

**Can do:**
- Near-cloud-quality code
- Complex architecture
- Security analysis
- Full system design

**Best for:** Critical projects, 48GB+ VRAM systems

---

## Quantization Without the Math

### What Is Quantization?

**Original model:** 16-bit floating point numbers
- Each weight uses 16 bits
- High precision
- Large file size

**Quantized model:** 4-bit integers
- Each weight uses 4 bits
- Lower precision
- 4x smaller file size

### The Trade-off

```
Precision vs Size:

FP16 (16-bit)  ████████████████████  100% size, 100% quality
Q8 (8-bit)     ████████████████░░░░   50% size, 99% quality
Q6 (6-bit)     ██████████████░░░░░░   38% size, 98% quality
Q5 (5-bit)     ████████████░░░░░░░░   31% size, 97% quality
Q4 (4-bit)     ███████████░░░░░░░░░   25% size, 95% quality ← SWEET SPOT
Q3 (3-bit)     █████████░░░░░░░░░░░   19% size, 90% quality
Q2 (2-bit)     ███████░░░░░░░░░░░░░   13% size, 80% quality ← Minimum
```

### Quantization Formats

| Format | Engine | Quality | Portability |
|---|---|---|---|
| **GGUF Q4_K_M** | llama.cpp, Ollama | Excellent | High |
| **GGUF Q5_K_M** | llama.cpp, Ollama | Better | High |
| **GGUF Q8_0** | llama.cpp, Ollama | Best | High |
| **AWQ 4-bit** | AutoAWQ | Good | Medium |
| **GPTQ 4-bit** | GPTQ | Good | Low |

**Recommendation:** Use GGUF Q4_K_M for most tasks.

---

## Coding Capability Degradation Curve

### How Quantization Affects Coding

```
Code quality by quantization:

Q8    ████████████████████  Perfect code, handles all edge cases
Q6    ██████████████████░░  Excellent code, rare edge case misses
Q5    █████████████████░░░  Very good code, occasional fixes needed
Q4    ████████████████░░░░  Good code, regular iteration needed ← MINIMUM
Q3    ██████████████░░░░░░  Decent code, frequent fixes needed
Q2    ████████████░░░░░░░░  Poor code, often broken
```

### Minimum Quantization for Agents

**Q4_K_M is the practical minimum for coding agents.**

**Why:**
- Below Q4, tool calling becomes unreliable
- Instruction following degrades
- Output format compliance drops
- Code quality becomes inconsistent

**Exception:** If you have no choice (very limited RAM), Q3 can work for simple tasks.

---

## Recommendation Matrix by Hardware

### 4-8 GB RAM

**Recommended model:** `qwen2.5-coder:1.5b` or `phi3:mini`

**Command:**
```bash
ollama pull qwen2.5-coder:1.5b
ollama run qwen2.5-coder:1.5b
```

**What to expect:**
- Speed: 15-25 tokens/sec (CPU)
- Quality: Basic code generation
- Best for: Simple functions, autocomplete, learning

**Don't attempt:**
- Complex architecture
- Multi-file changes
- Large refactoring

---

### 8-16 GB RAM

**Recommended model:** `qwen2.5-coder:7b` (Q4)

**Command:**
```bash
ollama pull qwen2.5-coder:7b
ollama run qwen2.5-coder:7b
```

**What to expect:**
- Speed: 10-30 tokens/sec (with GPU), 3-8 tokens/sec (CPU)
- Quality: Good for most coding tasks
- Best for: Daily coding, features, debugging, testing

**Can handle:**
- Full feature implementation
- Multi-file changes
- Code review
- Test generation

---

### 16-32 GB RAM

**Recommended model:** `qwen2.5-coder:14b` (Q4)

**Command:**
```bash
ollama pull qwen2.5-coder:14b
ollama run qwen2.5-coder:14b
```

**What to expect:**
- Speed: 5-20 tokens/sec (with GPU), 1-3 tokens/sec (CPU)
- Quality: Very good, handles complex tasks
- Best for: Complex features, architecture, migrations

**Can handle:**
- Complex feature implementation
- Architecture design
- Large refactoring
- Framework migrations

---

### 32+ GB RAM

**Recommended model:** `qwen2.5-coder:32b` (Q4)

**Command:**
```bash
ollama pull qwen2.5-coder:32b
ollama run qwen2.5-coder:32b
```

**What to expect:**
- Speed: 3-15 tokens/sec (with GPU)
- Quality: Excellent, near-cloud quality
- Best for: Critical projects, complex systems

**Can handle:**
- Full system design
- Complex architecture
- Security analysis
- Large-scale refactoring

---

### 48+ GB VRAM

**Recommended model:** `qwen2.5-coder:70b` (Q4) or cloud API

**Command:**
```bash
ollama pull qwen2.5-coder:70b
ollama run qwen2.5-coder:70b
```

**What to expect:**
- Speed: 2-10 tokens/sec
- Quality: Near-perfect code
- Best for: Production-critical work

---

## Where to Download Models

### Trusted Sources

#### 1. Ollama (Easiest)

```bash
ollama pull qwen2.5-coder:7b
```

**Pros:**
- One command
- Automatically downloads correct quantization
- Well-maintained

**Cons:**
- Limited model selection
- Less control over quantization

---

#### 2. Hugging Face (GGUF)

**Search:** `model-name GGUF`

**Example:**
```
https://huggingface.co/Qwen/Qwen2.5-Coder-7B-Instruct-GGUF
```

**Download:**
```bash
# Get Q4_K_M version
wget https://huggingface.co/Qwen/Qwen2.5-Coder-7B-Instruct-GGUF/resolve/main/qwen2.5-coder-7b-instruct-q4_k_m.gguf
```

**Load with llama.cpp:**
```bash
./main -m qwen2.5-coder-7b-instruct-q4_k_m.gguf -p "Your prompt"
```

**Pros:**
- Full control over quantization
- Wide selection
- Free

**Cons:**
- Manual setup
- Need to choose correct file

---

#### 3. Bartowski (Recommended GGUF Provider)

**Profile:** https://huggingface.co/bartowski

**Why:**
- Fresh quantizations
- Well-organized
- Good documentation
- Consistent quality

**Example:**
```
bartowski/Qwen2.5-Coder-7B-Instruct-GGUF
├── qwen2.5-coder-7b-instruct-Q4_K_M.gguf
├── qwen2.5-coder-7b-instruct-Q5_K_M.gguf
└── qwen2.5-coder-7b-instruct-Q8_0.gguf
```

---

#### 4. TheBloke (Legacy)

**Profile:** https://huggingface.co/TheBloke

**Why:**
- Extensive library
- Well-tested
- Older quantizations

**Note:** Less active now, but still reliable for older models.

---

## Model Selection Decision Tree

```
Do you have a GPU?
│
├─ No → CPU-only system
│   ├─ < 8 GB RAM → qwen2.5-coder:1.5b or phi3:mini
│   │   └─ Expect: 15-25 tok/s, basic code
│   │
│   └─ 8+ GB RAM → qwen2.5-coder:7b
│       └─ Expect: 3-8 tok/s, good code
│
└─ Yes → What VRAM do you have?
    │
    ├─ 4-6 GB VRAM
    │   └─ qwen2.5-coder:3B Q4
    │       └─ Expect: 20-40 tok/s, decent code
    │
    ├─ 8-12 GB VRAM
    │   └─ qwen2.5-coder:7B Q4 ← RECOMMENDED
    │       └─ Expect: 20-40 tok/s, good code
    │
    ├─ 16-24 GB VRAM
    │   └─ qwen2.5-coder:14B Q4
    │       └─ Expect: 10-20 tok/s, very good code
    │
    └─ 24+ GB VRAM
        └─ qwen2.5-coder:32B Q4 or 70B Q4
            └─ Expect: 5-15 tok/s, excellent code
```

---

## Exercise: Test Different Models

**Task:** Compare model quality on your hardware using a Project A task.

**Steps:**
1. Download 2-3 models for your hardware tier
2. Run the same prompt on each (use a real Project A task):
    ```
    I'm building a CLI log parser (Project A). Write a function that counts
    log entries by severity level (ERROR, WARN, INFO, DEBUG).
    
    Requirements:
    - Type hints on all parameters and return value
    - Docstring with examples
    - Handle edge cases: empty file, unknown levels
    - Use only standard library
    
    Return ONLY the code.
    ```
3. Compare outputs:
    - Which produced working code?
    - Which had better error handling?
    - Which followed constraints?
4. Choose your daily driver model for Project A/B/C

**Timebox:** 1 hour

---

## On Your Hardware

### Quick Reference

| Your Hardware | Model | Quantization | Expected Quality |
|---|---|---|---|
| **4-8 GB RAM** | 1.5B | Q4 | Basic |
| **8-16 GB RAM** | 7B | Q4 | Good |
| **16-32 GB RAM** | 14B | Q4 | Very Good |
| **32+ GB RAM** | 32B | Q4 | Excellent |
| **48+ GB VRAM** | 70B | Q4 | Near-Perfect |

---

## Common Pitfalls

### Pitfall 1: Using Too Low Quantization

**Bad:**
```
Use Q2 quantization to save memory
```

**Result:**
- Unreliable tool calling
- Poor instruction following
- Broken code frequently

**Good:**
```
Use Q4 minimum for agents
If Q4 doesn't fit, use smaller model
```

### Pitfall 2: Chasing Bigger Models

**Bad:**
```
Always use the biggest model possible
```

**Result:**
- Slow inference
- Overkill for simple tasks
- Wasted resources

**Good:**
```
Use appropriate model for task:
- 1.5B for autocomplete
- 7B for daily coding
- 14B+ for complex work
```

### Pitfall 3: Downloading from Untrusted Sources

**Bad:**
```
Download random GGUF from unknown uploader
```

**Risk:**
- Corrupted files
- Poor quality quantization
- Security issues

**Good:**
```
Download from trusted sources:
- Ollama official
- Bartowski
- TheBloke
- Official model repos
```

---

## Quick Reference: Download Commands

### Ollama (Easiest)

```bash
# Small models
ollama pull qwen2.5-coder:1.5b
ollama pull phi3:mini

# Medium models
ollama pull qwen2.5-coder:7b
ollama pull llama3.1:8b

# Large models
ollama pull qwen2.5-coder:14b
ollama pull qwen2.5-coder:32b
```

### Manual GGUF Download

```bash
# From Bartowski
wget https://huggingface.co/bartowski/Qwen2.5-Coder-7B-Instruct-GGUF/resolve/main/qwen2.5-coder-7b-instruct-Q4_K_M.gguf

# Load with llama.cpp
./main -m qwen2.5-coder-7b-instruct-Q4_K_M.gguf -p "Your prompt"
```

---

## Next Steps

- [Chapter 18: Squeezing Performance](./03-performance.md) — Optimize speed and memory
- [Chapter 19: Quality Checks](./04-quality-checks.md) — Verify your results
- [Chapter 20: Trust and Safety](./05-trust-and-safety.md) — Long-term use

---

## Further Reading

- [Reference: Models](../reference/models.md) — Complete model list
- [Reference: Engines](../reference/engines.md) — Installation guides
- [Reference: Troubleshooting](../reference/troubleshooting.md) — Model issues
