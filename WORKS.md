# WORKS.md: Make Something Work Today

> **Skip the theory. Just get it working.**

---

## The 5-Minute Setup

```bash
# 1. Install Ollama
curl -fsSL https://ollama.com/install.sh | sh

# 2. Pull a coding model
ollama pull qwen2.5-coder:7b

# 3. Start chatting
ollama run qwen2.5-coder:7b
```

---

## Copy-Paste Prompts That Work

### Generate a Function

```
Write a Python function that {description}.

Requirements:
- Type hints on all parameters and return value
- Docstring with description, params, returns
- Handle edge cases: {list}
- No external dependencies

Return ONLY the code.
```

### Debug an Error

```
The following code produces this error:

Code:
{paste code}

Error:
{paste error message}

Analyze the error and provide the minimal fix as a diff.
```

### Write Tests

```
Write pytest tests for this code:

{paste code}

Requirements:
- Cover happy path, edge cases, error cases
- Use descriptive test names
- Each test should be independent

Return ONLY the test code.
```

### Code Review

```
Review this code for issues. Categorize:

🔴 Critical: Bugs, security vulnerabilities
🟡 Warning: Performance, maintainability
🔵 Suggestion: Style, best practices

Code:
{paste code}

For each finding: line number, category, description, suggested fix.
```

---

## Model Selection Cheat Sheet

| Your Hardware | Model | Command |
|---|---|---|
| **4–8 GB RAM** | Qwen2.5-Coder 1.5B | `ollama pull qwen2.5-coder:1.5b` |
| **8–16 GB RAM** | Qwen2.5-Coder 7B | `ollama pull qwen2.5-coder:7b` |
| **16–32 GB RAM** | Qwen2.5-Coder 14B | `ollama pull qwen2.5-coder:14b` |
| **32+ GB RAM** | Qwen2.5-Coder 32B | `ollama pull qwen2.5-coder:32b` |
| **Any (CPU-only)** | Phi-3 Mini | `ollama pull phi3:mini` |

---

## When Things Go Wrong

| Problem | Quick Fix |
|---|---|
| "Out of memory" | Use smaller model |
| "Too slow" | Use smaller model or enable GPU |
| "Hallucinates APIs" | Add "Use only standard library" to prompt |
| "Invalid JSON" | Use XML tags instead: `<answer>...</answer>` |
| "Ignores constraints" | Repeat constraints in both system and user prompt |
| "Infinite loop" | Press Ctrl+C, reduce context, try again |

---

## The Plan-Code-Verify Loop (Manual Version)

When using the chat interface directly:

```
1. PLAN: Ask the AI to outline its approach
   "Before writing code, outline your approach for {task}"

2. CODE: Ask for the implementation
   "Now implement it. Return ONLY the code."

3. VERIFY: Ask for tests
   "Write pytest tests for this code."

4. FIX: Run the tests, feed errors back
   "These tests failed. Fix the code: {test output}"
```

---

## Common Tasks, Copy-Paste Prompts

### Scaffold a New Project

```
Scaffold a new Python CLI tool called {name} that {description}.

Requirements:
- Use argparse for CLI
- Include pyproject.toml with dependencies
- Include README.md with usage examples
- Include tests/ directory with one test
- Follow PEP 8 style

Return the file structure first, then the content of each file.
```

### Refactor Code

```
Refactor this code to {goal}:

{paste code}

Requirements:
- Preserve all existing behavior
- Improve {specific aspect: readability, performance, etc.}
- Return as a unified diff

Show only the changes, not the entire file.
```

### Migrate Code

```
Migrate this JavaScript code to TypeScript:

{paste code}

Requirements:
- Add proper type annotations
- Use TypeScript best practices
- Include any necessary tsconfig.json changes
- Return the migrated code

Preserve all existing behavior.
```

---

## Hardware-Specific Tips

### 8GB RAM or Less
- Use `qwen2.5-coder:1.5b` or `phi3:mini`
- Keep prompts short
- Don't load entire codebases into context
- Accept slower speeds (5–10 tokens/sec)

### 16GB RAM
- Use `qwen2.5-coder:7b` (sweet spot)
- Can handle moderate context (5–10 files)
- Good balance of speed and quality

### 32GB+ RAM
- Use `qwen2.5-coder:14b` or `32b`
- Can handle large context (20+ files)
- Better at complex reasoning tasks

---

## When to Use Existing Tools Instead

Don't build your own agent if:

- **You just need code generation** → Use Aider or Continue
- **You need IDE integration** → Use Continue (VS Code extension)
- **You want terminal-based coding** → Use OpenCode
- **You need multi-file edits** → Use Aider (git-aware, diff-based)

Build your own agent when:
- You need custom tool integration
- You want full control over the loop
- You're building a specific workflow
- You're learning how agents work

---

## Quick Reference

### Ollama Commands

```bash
ollama list              # Show installed models
ollama pull <model>      # Download a model
ollama run <model>       # Start chatting
ollama stop <model>      # Stop a running model
ollama rm <model>        # Remove a model
```

### Environment Variables

```bash
OLLAMA_NUM_CTX=4096      # Context window size (default: 2048)
OLLAMA_NUM_THREAD=8      # CPU threads (default: physical cores)
OLLAMA_MAX_LOADED=2      # Max models in memory (GPU only)
```

### Model Sizes (Q4 Quantized)

| Model | Size | VRAM Needed |
|---|---|---|
| 1.5B | ~1 GB | 2 GB |
| 3B | ~2 GB | 4 GB |
| 7B | ~4 GB | 8 GB |
| 14B | ~9 GB | 16 GB |
| 32B | ~20 GB | 24 GB |
| 70B | ~40 GB | 48 GB |

---

## Next Steps

1. **Try the prompts above** — Copy-paste, adapt for your needs
2. **Read Chapter 0** — [Quick Start Guide](./00-quick-start/01-first-function.md)
3. **Move to Part 1** — Learn the systematic approach

---

*This is a living cheat sheet. Add your own prompts and tips.*
