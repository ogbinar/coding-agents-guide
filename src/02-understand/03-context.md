# Chapter 8: Context — Why Your Agent Forgets Things

> Understand context windows, token budgets, and how to manage what your agent remembers.

---

## TL;DR

- Context window = the model's "working memory" (it's RAM for your LLM)
- Token budgets determine how much code you can load
- Loading entire codebases blows the context
- Progressive disclosure: load what you need, when you need it
- "On Your Hardware": context window size vs available RAM

---

## Prerequisites

- Completed [Chapters 6-7](./01-how-agents-think.md)
- Understand agent loops and tools
- Noticed that agents sometimes "forget" earlier conversation

---

## The Context Window Explained

### What Is a Context Window?

**Think of it as:** The model's working memory.

**Technical definition:** The maximum amount of text (in tokens) the model can consider at once.

**Includes:**
- Your prompt
- The model's previous responses
- Files you've loaded
- Tool outputs
- Conversation history

### Context Window Sizes

| Model | Context Window | Approximate Content |
|---|---|---|
| **Qwen2.5-Coder-7B** | 4K-8K tokens | ~3000-6000 lines of code |
| **Qwen2.5-Coder-14B** | 8K-32K tokens | ~6000-24000 lines |
| **Qwen2.5-Coder-32B** | 32K tokens | ~24000 lines |
| **Claude 3.5** | 200K tokens | ~150000 lines |
| **GPT-4o** | 128K tokens | ~96000 lines |

**Reality check:** Most local models have 4K-8K context. That's not much.

---

## Token Budgets: Where Your Context Goes

### The Budget Breakdown

```
Total Context: 8000 tokens

┌─────────────────────────────────────────────────────────┐
│ Conversation History: 1500 tokens (19%)                 │
│ ├─ Your prompts                                         │
│ └─ Model responses                                      │
├─────────────────────────────────────────────────────────┤
│ Loaded Files: 5000 tokens (63%)                         │
│ ├─ src/parser.py: 1200 tokens                           │
│ ├─ src/cli.py: 800 tokens                               │
│ ├─ src/search.py: 1500 tokens                           │
│ └─ tests/test_search.py: 1500 tokens                    │
├─────────────────────────────────────────────────────────┤
│ Tool Outputs: 1000 tokens (12%)                         │
│ ├─ Test results: 500 tokens                             │
│ └─ Linter output: 500 tokens                            │
├─────────────────────────────────────────────────────────┤
│ Buffer: 500 tokens (6%)                                 │
│ └─ Space for new responses                              │
└─────────────────────────────────────────────────────────┘
```

**Problem:** Once you hit the limit, old content gets cut off.

### What Gets Cut Off?

**First to go:** The oldest content (FIFO - first in, first out)

**Example:**
```
Session starts:
- You: "Add search feature"
- Agent: "Here's my plan..."
- You: "Looks good"
- Agent: "Generating code..."

After 8000 tokens:
[Oldest content removed to make room]
- You: "Fix the bug"  ← Recent, stays
- Agent: "Here's the fix"  ← Recent, stays
```

**Result:** Agent forgets the original plan.

---

## Loading Code Efficiently

### Don't Load the Entire Codebase

**Bad:**
```
Load all files in src/
- parser.py (1200 tokens)
- cli.py (800 tokens)
- search.py (1500 tokens)
- counter.py (1000 tokens)
- filters.py (900 tokens)
- utils.py (600 tokens)
- __init__.py (200 tokens)
Total: 6200 tokens (77% of context!)
```

**Good:**
```
Load only relevant files:
- search.py (1500 tokens)  ← What we're modifying
- cli.py (800 tokens)  ← What we're modifying
Total: 2300 tokens (29% of context)
```

### How to Load Only What You Need

#### 1. Be Specific in Prompts

**Bad:**
```
Here's my codebase:
{paste entire src/ directory}
```

**Good:**
```
I'm working on the search feature.
Read these files:
- src/logparser/search.py
- src/logparser/cli.py
```

#### 2. Use Search First

```
Before loading files, search:
search_files("search_logs", path="src/")

Result: Found in src/logparser/search.py:15

Now load only that file:
read_file("src/logparser/search.py")
```

#### 3. Load by Sections

```
# Don't do this:
read_file("src/large_module.py")  # 2000 tokens

# Do this:
read_file("src/large_module.py", start_line=1, end_line=100)  # First 100 lines
read_file("src/large_module.py", start_line=200, end_line=300)  # Just the function we need
```

---

## Progressive Disclosure

### The Strategy

**Load code in stages, not all at once.**

**Stage 1: High-level understanding**
```
Agent: "What's the project structure?"
Tool: list_directory("src/", recursive=True)
Result: ["parser.py", "cli.py", "search.py", ...]

Agent: "What does search.py do?"
Tool: read_file("src/logparser/search.py", start_line=1, end_line=50)
Result: First 50 lines (function signatures, docstrings)
```

**Stage 2: Deep dive into relevant code**
```
Agent: "I need to modify search_logs()"
Tool: read_file("src/logparser/search.py", start_line=15, end_line=80)
Result: Just the function implementation
```

**Stage 3: Context as needed**
```
Agent: "I need to understand how CLI works"
Tool: read_file("src/logparser/cli.py", start_line=50, end_line=120)
Result: Just the CLI command section
```

### Benefits

- **Saves context** — Only load what you need
- **Faster** — Less data to process
- **Clearer** — Agent focuses on relevant code
- **Flexible** — Load more if needed

---

## Context Window Size vs Available RAM

### The Relationship

**Larger context = More RAM needed**

```
Context Window → KV Cache → RAM/VRAM Usage

8K context  → ~200MB KV cache  → 4GB+ RAM total
16K context → ~400MB KV cache  → 8GB+ RAM total
32K context → ~800MB KV cache  → 16GB+ RAM total
```

### On Your Hardware

#### 8GB RAM or Less

**Context limit:** 4K tokens (default)

**Strategy:**
- Load one file at a time
- Use short prompts
- Summarize conversation history
- Start new sessions frequently

**Command:**
```bash
# Use default context (4K)
ollama run qwen2.5-coder:7b
```

#### 16GB RAM

**Context limit:** 8K tokens (comfortable)

**Strategy:**
- Load 3-5 files at once
- Moderate prompt length
- Summarize every 10-15 turns

**Command:**
```bash
# Increase context to 8K
OLLAMA_NUM_CTX=8192 ollama run qwen2.5-coder:7b
```

#### 32GB+ RAM

**Context limit:** 16K-32K tokens (generous)

**Strategy:**
- Load entire modules
- Long prompts with examples
- Keep longer conversation history

**Command:**
```bash
# Increase context to 16K
OLLAMA_NUM_CTX=16384 ollama run qwen2.5-coder:7b
```

---

## Managing Conversation History

### The Problem

Long conversations fill context with old prompts.

**Example:**
```
Turn 1: "Add search feature" → 200 tokens
Turn 2: "Here's my plan" → 300 tokens
Turn 3: "Generate code" → 400 tokens
...
Turn 20: "Fix bug" → 500 tokens

Total: 6000 tokens (75% of 8K context)
```

**Result:** Not enough room for code.

### Solution 1: Summarize

**Prompt:**
```
Summarize what we've done so far:

- Created search.py with search_logs()
- Added CLI search subcommand
- Fixed case sensitivity bug
- All tests passing

Now let's {next task}.
```

**Saves:** 4000+ tokens of conversation history

### Solution 2: New Session

**When:** After completing a feature

**Prompt:**
```
Starting fresh session. Context:

Project: Log parser CLI tool
Completed: Search feature (search_logs(), CLI command, tests)
Next task: Add count feature
Key files: src/logparser/
```

**Saves:** All conversation history tokens

### Solution 3: Truncate Old Messages

**Manual approach:**
```
Delete first 10 messages from conversation
Keep only recent 10 messages
```

**Result:** Frees up 2000-3000 tokens

---

## Exercise: Monitor Context Usage

**Task:** Track how your context fills up.

**Steps:**
1. Start a new agent session
2. Give it a task: "Add count feature"
3. After each turn, estimate tokens used:
   - Your prompt: ~50-200 tokens
   - Agent response: ~100-500 tokens
   - File reads: ~500-1500 tokens each
   - Tool outputs: ~200-500 tokens
4. Note when agent starts "forgetting"
5. Try summarizing and see if it helps

**Reflection:**
- How many turns before context fills?
- What consumed the most tokens?
- Did summarizing help?
- When should you start a new session?

---

## Common Patterns

### Pattern 1: Context Overflow

**Symptom:** Agent forgets earlier decisions

**Cause:** Context window exceeded

**Fix:**
```
Summarize what we've done:
{paste summary}

Continue from here:
{next task}
```

### Pattern 2: Loading Too Much

**Symptom:** Agent slow to respond, runs out of room for code

**Cause:** Loaded entire codebase

**Fix:**
```
Don't load all files. Just read:
- src/logparser/search.py (what we're modifying)
- src/logparser/cli.py (where to add command)
```

### Pattern 3: Missing Context

**Symptom:** Agent makes wrong assumptions

**Cause:** Didn't load enough context

**Fix:**
```
Before writing code, read:
- src/logparser/parser.py (to understand LogEntry)
- src/logparser/search.py (to follow existing patterns)
```

---

## Quick Reference: Context Management

### When to Summarize

- After 10-15 conversation turns
- Before starting a new feature
- When agent seems confused about earlier decisions

### When to Start New Session

- Feature complete
- Context > 80% full
- Switching to different part of codebase

### When to Load More Files

- Agent makes assumptions about code structure
- Agent references files you haven't loaded
- Agent asks "what does X do?"

### When to Load Less Files

- Context filling up
- Agent responding slowly
- Working on isolated change

---

## Next Steps

- [Chapter 9: What Not to Build](./04-what-not-to-build.md) — When to use existing tools instead
- [Chapter 10: Better Prompts](../03-advanced/01-better-prompts.md) — Improve your results

---

## Further Reading

- [Reference: Models](../reference/models.md) — Context window sizes by model
- [Reference: Troubleshooting](../reference/troubleshooting.md) — Context issues
- [WORKS.md](../WORKS.md) — Context management tips
