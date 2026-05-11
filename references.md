# References: Tools, Papers, and Further Reading

> Curated resources for local agentic development.

## Inference Engines

| Tool | Description | Link |
|---|---|---|
| **llama.cpp** | C/C++ inference, GGUF format, CPU+GPU | [github.com/ggml-org/llama.cpp](https://github.com/ggml-org/llama.cpp) |
| **Ollama** | Easy local model serving | [ollama.com](https://ollama.com) |
| **vLLM** | High-throughput serving, PagedAttention | [vllm.ai](https://vllm.ai) |
| **ExLlamaV2** | Fast inference for consumer GPUs | [github.com/turboderp/exllamav2](https://github.com/turboderp/exllamav2) |
| **MLC LLM** | Cross-platform (web, mobile, desktop) | [mlc.ai](https://mlc.ai) |
| **LM Studio** | GUI for local LLM inference | [lmstudio.ai](https://lmstudio.ai) |
| **Text Generation WebUI** | Web UI with extensions | [github.com/oobabooga](https://github.com/oobabooga/text-generation-webui) |

## Agent Frameworks

| Framework | Description | Local-Friendly |
|---|---|---|
| **LangGraph** | State machine-based agent workflows | ✓ |
| **LlamaIndex** | RAG + agent framework | ✓ |
| **CrewAI** | Multi-agent orchestration | ⚠ Heavy |
| **AutoGen** | Conversational multi-agent | ⚠ Moderate |
| **Semantic Kernel** | Microsoft's agent SDK | ✓ |
| **PydanticAI** | Type-safe agent framework (new) | ✓ |
| **Instructor** | Structured outputs from LLMs | ✓ |

## Coding-Specific Tools

| Tool | Purpose |
|---|---|
| **Aider** | Pair programming with LLMs, git-aware, diff-based editing |
| **OpenCode** | Terminal-based coding agent with tool use |
| **Cursor** | IDE with built-in AI (cloud, but sets the standard) |
| **Continue** | Open-source AI coding assistant VS Code extension |
| **CodeBuddy** | Local-first coding agent |
| **Tree-sitter** | Fast, incremental parsing for code analysis |
| **Tree-sitter-wasm** | Run tree-sitter in browser/Node for AST extraction |
| **LSP (Language Server Protocol)** | Get symbols, definitions, references without loading files |
| **tree-sitter-cli** | Extract symbols and structure from any supported language |

## Model Sources

| Source | Description |
|---|---|
| **Hugging Face GGUF** | Quantized models in GGUF format |
| **Bartowski** | Fresh, well-organized GGUF quantizations |
| **TheBloke** | Extensive library of quantized models |
| **Unsloth** | Fine-tuning optimized for efficiency |
| **ollama.com/library** | Pre-packaged models for Ollama |

## Key Papers

### Agent Architectures
- **ReAct** — "ReAct: Synergizing Reasoning and Acting in Language Models" (Yao et al., 2022)
- **Reflexion** — "Reflexion: Language Agents with Verbal and Reflective Learning" (Shinn et al., 2023)
- **ToolLLM** — "ToolLLM: Facilitating Large Language Models to Master 16000+ Real-World APIs" (Qin et al., 2023)
- **Plan-and-Solve** — "Plan-and-Solve Prompting: Improving Zero-Shot Chain-of-Thought Reasoning" (Wang et al., 2023)
- **Self-Consistency** — "Self-Consistency Improves Chain of Thought Reasoning" (Wang et al., 2022)

### Coding Benchmarks
- **HumanEval** — "Evaluating Large Language Models Trained on Code" (Hendrycks et al., 2021)
- **MBPP** — "MBPP: A Problem Dataset for Formalizing Real-World Program Repair" (Austin et al., 2021)
- **SWE-bench** — "SWE-bench: Can Language Models Resolve Real-World GitHub Issues?" (Jimenez et al., 2023)
- **AgentBench** — Benchmark for agent capabilities including coding
- **FPBench** — Fine-grated programming benchmark

### Coding-Focused Models
- **Qwen2.5-Coder** — Purpose-built coding models (3B, 7B, 14B, 32B)
- **DeepSeek-Coder-V2** — Strong multi-language coding, long context
- **Starcoder2** — BigCode's coding model (3B, 7B, 15B)
- **CodeQwen1.5** — Earlier coding-focused Qwen variant
- **Phi-3.5-mini** — Small but surprisingly capable at coding

### Small Models
- **Phi-3** — "Phi-3 Technical Report: A Highly Capable Language Model Locally on Your Phone" (Microsoft, 2024)
- **Qwen2.5** — Technical reports on Qwen series capabilities
- **Gemma** — Google's open models, good for edge deployment

### Quantization
- **GGML/GGUF** — llama.cpp quantization formats
- **AWQ** — "Activation-aware Weight Quantization" (Liu et al., 2023)
- **GPTQ** — "GPTQ: Accurate Post-Training Quantization" (Frantar et al., 2022)

## Blogs and Tutorials

- **llama.cpp Wiki** — Quantization guides, benchmarking
- **Hugging Face Docs** — Model cards, quantization tutorials
- **Towards Data Science** — Various agent architecture tutorials
- **Simon Willison's Blog** — Practical LLM application patterns

## Communities

- **r/LocalLLaMA** — Reddit community for local LLMs
- **Hugging Face Discord** — Model discussions
- **llama.cpp Discord** — Engine-specific help
- **Ollama Discord** — Ollama-specific discussions

## Tools for Development

| Tool | Purpose |
|---|---|
| **promptfoo** | Prompt testing and evaluation |
| **DeepEval** | LLM evaluation framework |
| **LangSmith** | Agent debugging and tracing |
| **Weights & Biases** | Experiment tracking |
| **MLflow** | Model and experiment management |

---

*This file is a living reference. Add new resources as they become relevant.*
