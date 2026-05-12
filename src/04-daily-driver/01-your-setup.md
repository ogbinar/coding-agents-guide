# Chapter 16: Your Setup

> Build your daily driver: the tools, workflows, and prompt library that become your permanent AI coding environment.

---

## TL;DR

- Choose your daily driver tool (Aider, Continue, OpenCode, or custom)
- Set up VS Code integration or terminal workflow
- Build your prompt library and example collection
- Define your model lineup: which model for which task
- Create your permanent workspace with session persistence

---

## Prerequisites

- Completed [Part 1-3](../01-beginner/01-write-code.md) through [Chapter 15](../03-advanced/06-long-term.md)
- Have used agents for various tasks
- Ready to build a permanent, optimized setup

---

## Choosing Your Daily Driver

### Tool Comparison

| Tool | Best For | Setup | Integration | Learning Curve |
|---|---|---|---|---|
| **Aider** | Terminal workflow | ⭐ Easy | Git-aware | Low |
| **Continue** | VS Code users | ⭐ Easy | IDE-native | Low |
| **OpenCode** | Interactive agent | ⭐ Easy | Terminal | Low |
| **Cursor** | Full IDE replacement | ⭐⭐ Mod | Built-in | Medium |
| **Custom** | Specific workflows | ⭐⭐⭐ Hard | Your choice | High |

### Decision Matrix

**You're a VS Code user who wants inline editing:**
→ **Continue**

**You prefer terminal and want git integration:**
→ **Aider**

**You want an interactive agent that asks before acting:**
→ **OpenCode**

**You want a full IDE with AI built-in:**
→ **Cursor**

**You need custom tool integration or specific workflow:**
→ **Custom** (build your own)

---

## Setting Up Aider

### Installation

```bash
pip install aider-chat
```

### Basic Usage

```bash
# Edit a single file
aider src/my_script.py --message "Add error handling"

# Edit multiple files
aider src/ --message "Refactor to use dataclasses"

# Interactive mode
aider src/
# Then type your requests in the prompt
```

### Configuration

**~/.aider.conf.yml:**
```yaml
# Default model
model: ollama/qwen2.5-coder:7b

# Default files to edit
files:
  - src/
  - tests/

# Auto-commit changes
auto-commits: true

# Show diffs before applying
show-diffs: true

# Voice input (optional)
voice-input: false
```

### Best Practices

- **One feature per session** — Start fresh for each task
- **Commit frequently** — Let aider handle git
- **Review diffs** — Always check before applying
- **Use `@file` to reference code** — `@src/parser.py Add error handling`

---

## Setting Up Continue

### Installation

**VS Code Extension:**
1. Open Extensions (Ctrl+Shift+X)
2. Search for "Continue"
3. Install

### Configuration

**.continue/config.json:**
```json
{
  "models": [
    {
      "title": "Qwen Coder",
      "provider": "ollama",
      "model": "qwen2.5-coder:7b"
    }
  ],
  "tabAutocompleteModel": {
    "title": "Qwen Small",
    "provider": "ollama",
    "model": "qwen2.5-coder:1.5b"
  },
  "systemMessage": "You are an expert Python developer. Write clean, production-ready code."
}
```

### Usage

**Inline editing:**
1. Select code
2. Right-click → "Continue: Edit with AI"
3. Type your request
4. Accept/reject changes

**Chat panel:**
1. Open Continue panel (Ctrl+L)
2. Ask questions or give tasks
3. Apply suggestions

### Best Practices

- **Use tab autocomplete** — Fills in as you type
- **Reference files with `/`** — `/code src/parser.py`
- **Use shortcuts** — Ctrl+L for chat, Ctrl+I for inline
- **Keep context small** — Edit one file at a time

---

## Setting Up OpenCode

### Installation

```bash
pip install opencode
```

### Basic Usage

```bash
# Start interactive session
opencode

# Give it a task
"Add search feature to my log parser"

# It will ask before making changes
# Review and approve each step
```

### Configuration

**~/.opencode/config.yaml:**
```yaml
model: qwen2.5-coder:7b
provider: ollama

# Safety settings
require_approval:
  - write_file
  - run_command
  - git_commit

# Session persistence
session_dir: ~/.opencode/sessions/
```

### Best Practices

- **Review before approving** — Don't auto-approve everything
- **Use sessions** — Keep work organized by project
- **Interrupt when stuck** — Guide the agent if it loops
- **Save good prompts** — Build your prompt library

---

## Building Your Prompt Library

### Why You Need One

**Without a library:**
- Re-invent prompts every time
- Inconsistent quality
- Forget what worked before

**With a library:**
- Copy-paste proven prompts
- Consistent results
- Continuous improvement

### Structure

**~/.opencode/prompts/**
```
prompts/
├── code-generation.md
├── debugging.md
├── testing.md
├── code-review.md
├── refactoring.md
└── migration.md
```

### Example: code-generation.md

```markdown
# Code Generation Prompts

## Function Generation

```
Write a {language} function that {description}.

Requirements:
- Type hints on all parameters and return value
- Docstring with description, params, returns, examples
- Handle edge cases: {list}
- Use only {dependencies}

Return ONLY the code.
```

## Class Generation

```
Write a {language} class called {name} that {description}.

Attributes:
- {attr1}: {type} - {description}
- {attr2}: {type} - {description}

Methods:
- {method1}({params}) -> {return}: {description}
- {method2}({params}) -> {return}: {description}

Requirements:
- Type hints on all methods
- Complete docstrings
- Input validation
- {other constraints}

Return ONLY the code.
```
```

### Using Your Library

**Terminal:**
```bash
# Copy prompt
cat ~/.opencode/prompts/code-generation.md | pbcopy

# Paste and customize in your agent
```

**VS Code:**
- Use snippets extension
- Create custom snippets for each prompt type
- Trigger with shortcuts

---

## Your Model Lineup

### Which Model for Which Task

| Task | Model | Why |
|---|---|---|
| **Tab autocomplete** | 1.5B-3B | Fast, good for snippets |
| **Simple code generation** | 7B | Good balance |
| **Complex features** | 14B-32B | Better reasoning |
| **Code review** | 14B+ | Catch subtle issues |
| **Architecture design** | 32B+ | High-level thinking |
| **Debugging** | 7B-14B | Good at analysis |

### Recommended Setup

**For 16GB RAM:**
```bash
# Primary model (7B) - most tasks
ollama pull qwen2.5-coder:7b

# Small model (1.5B) - autocomplete
ollama pull qwen2.5-coder:1.5b
```

**For 32GB RAM:**
```bash
# Primary model (7B) - most tasks
ollama pull qwen2.5-coder:7b

# Large model (14B) - complex tasks
ollama pull qwen2.5-coder:14b

# Small model (1.5B) - autocomplete
ollama pull qwen2.5-coder:1.5b
```

### Switching Models

**Aider:**
```bash
aider --model ollama/qwen2.5-coder:14b
```

**Continue:**
- Select model from dropdown in chat panel

**OpenCode:**
```bash
opencode --model qwen2.5-coder:14b
```

---

## Building Your Permanent Workspace

### Directory Structure

```
~/projects/
├── current-project/       # What you're working on now
├── prompts/               # Your prompt library
├── examples/              # Code examples that work well
└── sessions/              # Saved agent sessions
```

### Session Management

**Start a session:**
```bash
# Aider
aider --edit-file-args src/main.py --session current-session.json

# OpenCode
opencode --session my-project
```

**Resume a session:**
```bash
# Aider
aider --session current-session.json

# OpenCode
opencode --resume my-project
```

### Git Integration

**Best practices:**
- **Branch per task** — One feature per branch
- **Commit messages from AI** — Let agent write them
- **Review before push** — Always check changes

**Aider auto-commits:**
```bash
aider --auto-commits --commit-prefix "feat:"
```

---

## Exercise: Set Up Your Daily Driver

**Task:** Configure your permanent AI coding environment for Project A/B/C.

**Steps:**
1. Choose your tool (Aider, Continue, or OpenCode)
2. Install and configure
3. Set up your model lineup
4. Create your prompt library (start with 3 prompts from this guide)
5. Set up your workspace directory structure:
   ```
   ~/projects/
   ├── project-a-logparser/     # Project A: CLI tool
   ├── project-b-contribution/  # Project B: open-source
   └── project-c-migration/     # Project C: migration task
   ```
6. Run a full coding task on Project A with your new setup

**Timebox:** 1-2 hours

**Deliverables:**
- Working tool installation
- Configured with your preferred model
- Prompt library with at least 3 prompts
- Organized workspace with Project A/B/C directories

---

## On Your Hardware

### 8GB RAM or Less

**Setup:**
- Single model: qwen2.5-coder:1.5b or phi3:mini
- Terminal-based (less overhead)
- Simple prompts (smaller context)
- Aider or OpenCode (lighter than IDE)

### 16GB RAM

**Setup:**
- Primary: qwen2.5-coder:7b
- Optional: qwen2.5-coder:1.5b for autocomplete
- VS Code + Continue (good balance)
- Moderate context windows

### 32GB+ RAM

**Setup:**
- Primary: qwen2.5-coder:14b or 32b
- Secondary: qwen2.5-coder:7b for quick tasks
- Full IDE integration
- Large context windows
- Multiple models running

---

## Common Pitfalls

### Pitfall 1: Tool Hopping

**Bad:**
```
Week 1: Try Aider
Week 2: Switch to Continue
Week 3: Try OpenCode
Week 4: Build custom
```

**Good:**
```
Week 1: Set up Aider
Week 2-4: Use Aider daily, learn its strengths
Month 2: Evaluate if it meets needs
Month 3: Switch only if necessary
```

### Pitfall 2: No Prompt Library

**Bad:**
```
"Write code to..."  # Every time, from scratch
```

**Good:**
```
# Copy from library
cat ~/.prompts/code-generation.md
# Customize and use
```

### Pitfall 3: Wrong Model for Task

**Bad:**
```
Use 32B for everything (slow)
Use 1.5B for architecture (bad results)
```

**Good:**
```
1.5B for autocomplete (fast)
7B for code generation (balanced)
14B+ for complex reasoning (smart)
```

---

## Quick Reference: Daily Setup Checklist

- [ ] Chosen daily driver tool
- [ ] Installed and configured
- [ ] Model(s) downloaded
- [ ] Prompt library created (start with 3)
- [ ] Workspace directory set up
- [ ] Git integration configured
- [ ] Session persistence enabled
- [ ] Shortcuts/keybindings set up
- [ ] Tested with real coding task

---

## Next Steps

- [Chapter 17: Models and Quantization](./02-models-quantization.md) — Deep dive on model selection
- [Chapter 18: Squeezing Performance](./03-performance.md) — Optimize your setup
- [Chapter 19: Quality Checks](./04-quality-checks.md) — Verify your results

---

## Further Reading

- [Reference: Models](../reference/models.md) — Model recommendations
- [Reference: Engines](../reference/engines.md) — Installation guides
- [Reference: Prompt Templates](../reference/prompt-templates.md) — Ready-to-use prompts
