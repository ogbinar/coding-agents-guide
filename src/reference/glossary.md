# Reference: Glossary

> Terms explained simply.

---

## A

**Agent** — A program that uses an LLM to make decisions, take actions, and achieve goals. For coding agents, this means AI that can write code, run tests, and fix bugs.

**AST (Abstract Syntax Tree)** — A tree representation of code structure. Used to understand code without loading the entire file into context.

---

## B

**Benchmark** — A standardized test to measure model performance. Common coding benchmarks:
- **HumanEval** — Can the model write passing functions?
- **MBPP** — Can the model solve programming problems?
- **SWE-bench** — Can the model fix real GitHub issues?

**Brownfield** — Working with an existing codebase. The agent must understand and respect existing patterns, conventions, and architecture.

---

## C

**Context Window** — The amount of text (in tokens) the model can consider at once. Think of it as the model's "working memory." For coding, this includes the code you're modifying, related files, and conversation history.

**Coding Agent** — An LLM agent specifically designed for coding tasks: generating code, debugging, testing, reviewing, and refactoring.

**CPU-Only** — Running inference without a GPU. Slower (1–10 tokens/sec) but works on any machine.

---

## D

**Diff** — A representation of changes between two versions of a file. Coding agents often output diffs instead of full files to minimize changes and preserve existing code.

---

## F

**Few-Shot Prompting** — Providing examples in the prompt to show the model what you want. Critical for small models that don't infer intent well.

**Framework** — A library or toolkit for building agents. Examples: LangGraph, CrewAI, AutoGen. For constrained systems, custom loops often work better than frameworks.

---

## G

**GGUF** — A file format for quantized models used by llama.cpp and Ollama. The standard format for local LLMs.

**Greenfield** — Building a new project from scratch. The agent can make architectural decisions and define conventions.

---

## H

**Hallucination** — When the model makes up facts, APIs, or code that doesn't exist. More common in smaller models and when the model lacks grounding in tool outputs.

---

## I

**Inference** — The process of running a model to generate predictions. For LLMs, this means generating text or code from a prompt.

**Inference Engine** — Software that runs LLMs locally. Examples: llama.cpp, Ollama, vLLM, ExLlamaV2.

---

## K

**KV Cache** — Key-Value cache used during inference to speed up generation. Consumes memory proportional to context window size.

---

## L

**Layer Offloading** — Splitting a model between GPU and CPU. Some layers run on GPU (fast), others on CPU (slow). Allows running larger models on limited VRAM.

**LLM (Large Language Model)** — A neural network trained on text (and often code). The "brain" of an agent.

**Local LLM** — Running an LLM on your own machine instead of using a cloud API. Privacy, cost, and offline advantages.

---

## M

**Model** — The trained neural network weights. For coding, common models include Qwen-Coder, DeepSeek-Coder, Llama, and Phi.

**Quantization** — Reducing the precision of model weights (from 16-bit to 4-bit, for example) to save memory. Q4 (4-bit) is the sweet spot for most agent workloads.

---

## O

**Ollama** — An easy-to-use tool for running local LLMs. Recommended for beginners.

**Open Source** — Software with publicly available source code. This guide focuses on open-source models and tools.

---

## P

**Prompt** — The input you give to a model. Includes system prompts (instructions), user prompts (tasks), and few-shot examples.

**Prompt Engineering** — Designing effective prompts to get reliable output. For coding agents, this means clear constraints, examples, and output formats.

---

## Q

**Quantization** — See under Q.

---

## R

**ReAct** — A reasoning pattern: **Re**ason → **Act** → Observe → Repeat. Common for general agents, but Plan-Code-Verify is better for coding.

**RAG (Retrieval-Augmented Generation)** — Retrieving relevant information before answering. Useful for coding agents to load relevant code files.

---

## S

**System Prompt** — The initial instructions given to a model that define its role, constraints, and behavior. The most important control mechanism for agents.

---

## T

**Temperature** — A parameter that controls randomness. Low temperature (0.1) for deterministic output (code), higher temperature (0.7) for creative tasks.

**Token** — The basic unit of text for LLMs. Roughly 4 characters or 0.75 words. Code is more token-dense than natural language.

**Tool Use** — Allowing the model to call functions or APIs. For coding agents, tools include file operations, code execution, git commands, etc.

---

## V

**VRAM (Video RAM)** — Memory on a GPU. Determines what size models you can run. Consumer GPUs have 4–24 GB VRAM.

---

## Common Acronyms

| Acronym | Meaning |
|---|---|
| **API** | Application Programming Interface |
| **AST** | Abstract Syntax Tree |
| **CI/CD** | Continuous Integration/Continuous Deployment |
| **CPU** | Central Processing Unit |
| **GPU** | Graphics Processing Unit |
| **JSON** | JavaScript Object Notation |
| **LSP** | Language Server Protocol |
| **LLM** | Large Language Model |
| **Q4/Q5/Q8** | Quantization levels (4-bit, 5-bit, 8-bit) |
| **RAG** | Retrieval-Augmented Generation |
| **SDK** | Software Development Kit |
| **VRAM** | Video Random Access Memory |

---

## Related Concepts

- **Tokens → Context Window → Memory**
- **Quantization → Model Size → VRAM Requirements**
- **Temperature → Randomness → Output Quality**
- **System Prompt → Constraints → Output Format**

---

*See also: [Models](./models.md), [Troubleshooting](./troubleshooting.md)*
