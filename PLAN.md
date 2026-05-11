# Agentic Guide: Site Plan and Information Architecture

> Plan to transform this research into a mini-book / website for beginner
> "vibe coders" who want to use coding agents on low-resource machines.

---

## Audience

**Primary:** Beginner-to-intermediate developers who:
- Want to use AI coding agents but don't have cloud GPU budgets
- Are comfortable coding but new to LLMs and agents
- Run on consumer hardware (laptop, used GPU, CPU-only)
- Think in terms of "what can I build?" not "what's the architecture?"
- May be hobbyists, students, indie devs, or developers in regions with
  limited access to cloud AI services

**Secondary:** Experienced devs exploring local agents for privacy, cost, or
offline work.

**Tone:** Practical, opinionated, hands-on. Less academic paper, more
"here's what actually works and here's why." Assume curiosity, not expertise.

---

## Site Structure

```
agentic-guide/
├── README.md                  # Landing page / home
├── PLAN.md                    # This file
├── WORKS.md                   # Quick-start: "Make something work today"
│
├── 01-setup/                  # Part 1: Get Running
│   ├── 01-what-you-need.md
│   ├── 02-choose-a-model.md
│   ├── 03-install-an-engine.md
│   └── 04-your-first-agent.md
│
├── 02-core/                   # Part 2: How Agents Think
│   ├── 01-the-agent-loop.md
│   ├── 02-tools-and-actions.md
│   ├── 03-context-is-everything.md
│   └── 04-when-agents-fail.md
│
├── 03-workflows/              # Part 3: What Agents Can Do
│   ├── 01-generate-code.md
│   ├── 02-debug-and-fix.md
│   ├── 03-write-tests.md
│   ├── 04-review-and-improve.md
│   ├── 05-refactor-and-migrate.md
│   └── 06-the-full-loop.md
│
├── 04-control/                # Part 4: Getting Better Output
│   ├── 01-system-prompts.md
│   ├── 02-few-shot-examples.md
│   ├── 03-structured-output.md
│   ├── 04-temperature-and-sampling.md
│   └── 05-guardrails.md
│
├── 05-constrained/            # Part 5: Making It Work on Less
│   ├── 01-quantization.md
│   ├── 02-memory-management.md
│   ├── 03-speed-optimization.md
│   ├── 04-cpu-only-survival.md
│   └── 05-budget-builds.md
│
├── 06-patterns/               # Part 6: Patterns That Work
│   ├── 01-plan-code-verify.md
│   ├── 02-fake-multi-agent.md
│   ├── 03-greenfield-vs-brownfield.md
│   ├── 04-self-correction.md
│   ├── 05-session-memory.md
│   └── 06-human-in-the-loop.md
│
├── 07-eval/                   # Part 7: Knowing It's Good
│   ├── 01-measure-what-matters.md
│   ├── 02-test-your-agent.md
│   ├── 03-debugging-failures.md
│   └── 04-improvement-loop.md
│
├── 08-reference/              # Part 8: Quick Reference
│   ├── models.md
│   ├── engines.md
│   ├── tools-catalog.md
│   ├── prompt-templates.md
│   ├── troubleshooting.md
│   └── glossary.md
│
└── workflows.md               # Master workflow map (existing)
```

---

## Table of Contents (Reader-Facing)

### Front Matter
- **Introduction** — Why local coding agents, who this is for, what you'll learn
- **Before You Start** — Hardware self-assessment, setting expectations

---

### Part 1: Get Running (Week 1)

> Goal: Have a working coding agent on your machine by the end of this part.

**Chapter 1: What You Actually Need**
- Hardware reality check: what runs on what
- The VRAM/RAM/cores decision tree
- "I only have a laptop" — yes, that's fine
- Software prerequisites (keep it minimal)

**Chapter 2: Choose a Model**
- Model size vs capability explained simply
- Coding models vs general models — why it matters
- Quantization demystified: what does "Q4" actually mean for your code?
- Recommendation matrix by your hardware
- Where to download models (trusted sources)

**Chapter 3: Install an Inference Engine**
- Engine comparison: llama.cpp, Ollama, LM Studio — which one for you?
- Step-by-step install for your platform (Linux, macOS, Windows)
- GPU setup (NVIDIA, AMD, Apple Silicon)
- CPU-only setup
- Smoke test: run your first prompt

**Chapter 4: Your First Coding Agent**
- From zero to a working agent in under an hour
- The minimal agent loop (code you can copy-paste)
- Your first task: "Write a Python function that..."
- Your first tool: file reading
- Your first verification: does the code actually run?
- Exercise: build a simple code generation agent

---

### Part 2: How Agents Think (Week 2)

> Goal: Understand what's happening inside so you can debug and improve.

**Chapter 5: The Agent Loop**
- Think → Act → Observe → Repeat (with diagrams)
- Why the loop matters more than the model
- Max iterations and early exit
- State management: what the agent remembers
- Exercise: trace an agent loop by hand

**Chapter 6: Tools and Actions**
- What tools are and why agents need them
- The 8 essential tools every coding agent needs
- Designing tools small models can actually use
- Tool output management (don't blow your context)
- Exercise: add a new tool to your agent

**Chapter 7: Context Is Everything**
- The context window explained (it's RAM for your model)
- Token budgets: where your context goes
- Loading code efficiently (don't load the whole repo)
- Progressive disclosure: load what you need, when you need it
- Exercise: optimize context usage for a real task

**Chapter 8: When Agents Fail**
- The 10 most common failure modes (with real examples)
- How to recognize each failure mode
- Recovery strategies for each
- When to blame the prompt vs the model vs the tools
- Exercise: diagnose and fix a broken agent run

---

### Part 3: What Agents Can Do (Weeks 3–4)

> Goal: Use your agent for real coding workflows.

**Chapter 9: Generate Code**
- From description to working code
- Greenfield: scaffolding a new project
- Specifying constraints that actually work
- Self-review before you even look
- Exercise: generate a complete module from a spec

**Chapter 10: Debug and Fix**
- Feeding error messages to your agent
- Root cause vs symptom (teaching the agent the difference)
- The debugging loop: error → hypothesis → fix → verify
- When the agent makes things worse
- Exercise: fix a planted bug using only your agent

**Chapter 11: Write Tests**
- Generating tests from existing code
- Happy path, edge cases, error cases
- Running and fixing failing tests
- Coverage targets for agent-generated tests
- Exercise: achieve 80% coverage on a module

**Chapter 12: Review and Improve**
- Code review prompts that catch real issues
- Categorizing findings (critical / warning / suggestion)
- Style enforcement vs substance
- Performance anti-pattern detection
- Exercise: review and improve a piece of your own code

**Chapter 13: Refactor and Migrate**
- Safe refactoring with diffs
- Framework migrations (with real examples)
- Incremental migration strategies
- Preserving behavior during transformation
- Exercise: migrate a small module to a new pattern

**Chapter 14: The Full Loop**
- Plan → Code → Verify → Fix → Commit
- Putting all workflows together
- When to stop iterating
- Git integration and commit messages
- Exercise: complete a full feature from spec to commit

---

### Part 4: Getting Better Output (Week 5)

> Goal: Systematically improve the quality of your agent's output.

**Chapter 15: System Prompts**
- Anatomy of a great system prompt
- Coding-specific system prompts (copy-paste ready)
- Greenfield vs brownfield prompt differences
- Iterating on your system prompt
- Exercise: write and test a system prompt from scratch

**Chapter 16: Few-Shot Examples**
- Why examples beat instructions for small models
- Writing effective few-shot examples for code
- Matching examples to your actual use cases
- Maintaining an example library
- Exercise: improve a failing prompt with examples

**Chapter 17: Structured Output**
- JSON vs XML vs diffs — choosing the right format
- Getting small models to produce parseable output
- Constrained decoding (grammar-based output)
- Post-processing and recovery
- Exercise: build a reliable structured output pipeline

**Chapter 18: Temperature and Sampling**
- What temperature actually does (with code examples)
- Temperature guide by task type
- When to use randomness vs determinism
- Reproducibility and seeding
- Exercise: find the sweet spot for your model

**Chapter 19: Guardrails**
- Input sanitization (prevent prompt injection)
- Output validation (catch bad output before it's used)
- Tool execution safety (sandboxing)
- When to add a human approval gate
- Exercise: add guardrails to your agent

---

### Part 5: Making It Work on Less (Week 6)

> Goal: Squeeze maximum quality from minimum hardware.

**Chapter 20: Quantization**
- Quantization explained without the math
- Q4 vs Q5 vs Q8 — what you actually lose
- Coding capability degradation curve
- Finding and testing quantized models
- Exercise: compare Q4, Q5, Q8 on your actual tasks

**Chapter 21: Memory Management**
- KV cache explained simply
- Layer offloading (GPU + CPU hybrid)
- Context window strategies for code
- When to upgrade hardware vs optimize software
- Exercise: fit a bigger model into your VRAM

**Chapter 22: Speed Optimization**
- Tokens per second: what's acceptable?
- Prefix caching and prompt reuse
- Speculative decoding (draft model + main model)
- Thread count tuning
- Exercise: double your generation speed

**Chapter 23: CPU-Only Survival**
- Yes, you can do this on CPU
- Best models and engines for CPU
- Expectation setting (speed, quality, what's practical)
- Async and batch patterns for slow inference
- Exercise: build a usable agent on CPU-only

**Chapter 24: Budget Builds**
- GPU buying guide for local agents (2025)
- Used market picks (RTX 3090, etc.)
- Cloud rental for when you need a boost
- Raspberry Pi and SBC experiments
- Exercise: plan your hardware upgrade path

---

### Part 6: Patterns That Work (Week 7)

> Goal: Learn the patterns that separate good agents from great ones.

**Chapter 25: Plan-Code-Verify**
- Why this loop beats ReAct for coding
- Implementation with real code
- Convergence detection (when to stop)
- Exercise: implement Plan-Code-Verify

**Chapter 26: Fake Multi-Agent**
- Role-switching with a single model
- Coder → Reviewer → Revisor pattern
- When real multi-agent makes sense
- Exercise: build a fake multi-agent code reviewer

**Chapter 27: Greenfield vs Brownfield**
- How context changes everything
- Codebase onboarding strategies
- Convention detection and adherence
- Legacy code handling
- Exercise: adapt your agent for a new codebase

**Chapter 28: Self-Correction**
- Compile-and-fix loops
- Test-driven generation
- Multi-pass refinement
- Cost of verification vs benefit
- Exercise: add self-correction to your agent

**Chapter 29: Session Memory**
- Resuming work across sessions
- Change tracking and undo
- Branch-per-task workflows
- Exercise: build session persistence

**Chapter 30: Human-in-the-Loop**
- Approval gates for risky operations
- Pair programming mode vs autonomous mode
- Trust calibration
- Exercise: add human approval to your agent

---

### Part 7: Knowing It's Good (Week 8)

> Goal: Measure, debug, and continuously improve your agent.

**Chapter 31: Measure What Matters**
- Coding-specific metrics (syntax validity, diff accuracy, etc.)
- Setting up basic monitoring
- Token budgets and cost tracking
- Exercise: instrument your agent

**Chapter 32: Test Your Agent**
- Building a golden test dataset
- Workflow-specific test scenarios
- Automated evaluation with executable verification
- Exercise: write 10 evaluation tests

**Chapter 33: Debugging Failures**
- Failure analysis template
- Common patterns and their fixes
- When to change the prompt vs the model vs the tools
- Exercise: fix 3 failing test cases

**Chapter 34: The Improvement Loop**
- Evaluate → Analyze → Hypothesize → Implement → Re-evaluate
- Regression detection
- Documenting what works and why
- Exercise: complete one improvement cycle

---

### Part 8: Quick Reference

> Tear-out pages. Copy-paste ready.

**Models** — Recommendation matrix by hardware tier, with direct download links
**Engines** — Install commands for every platform
**Tools Catalog** — The 24 essential coding agent tools, with signatures
**Prompt Templates** — Copy-paste system prompts for every workflow
**Troubleshooting** — Symptom → cause → fix lookup table
**Glossary** — Terms explained simply (tokens, quantization, KV cache, etc.)

---

## Content Principles

### Writing Style
- **Show, don't tell:** Every concept accompanied by a code example or diagram
- **Opinionated defaults:** "Use X unless you have reason Y" not "here are 7 options"
- **Progressive disclosure:** Simple answer first, deep dive in expandable sections
- **Real failures:** Show what actually breaks, not just happy paths
- **Copy-paste friendly:** Code blocks should work as-is

### Each Chapter Should Have
1. **TL;DR** — 3 bullet points, what you'll learn
2. **Prerequisites** — What chapters to read first (if any)
3. **Core content** — Concepts, examples, diagrams
4. **Common pitfalls** — What goes wrong and how to fix it
5. **Exercise** — Hands-on task with expected outcome
6. **Further reading** — Links to references.md entries

### Diagrams
- ASCII diagrams for loops and architectures (render in terminal and browser)
- Mermaid.js for flowcharts (renders in GitHub, Obsidian, most MD viewers)
- Keep diagrams simple — complexity is the enemy

### Code Examples
- Python-first (most accessible, most common for agent development)
- Each example should be minimal and runnable
- Provide both the "simple" version and the "production" version
- Link to a companion repo with working examples

---

## Website Structure (When Ready)

```
Site: agentic-guide.local (or agentic-guide.dev)

/                     → Landing page (README.md adapted)
/setup/               → Part 1 chapters
/core/                → Part 2 chapters
/workflows/           → Part 3 chapters
/control/             → Part 4 chapters
/constrained/         → Part 5 chapters
/patterns/            → Part 6 chapters
/eval/                → Part 7 chapters
/reference/           → Part 8 quick reference

Static site generator: MkDocs Material or Astro
- Sidebar navigation with collapsible parts
- Search (essential for reference lookups)
- Code syntax highlighting
- Print/PDF export
- Dark mode (it's a coding guide, obviously)
```

---

## Production Plan

### Phase 1: Core Content (Weeks 1–3)
Write Parts 1–3 (chapters 1–14). This gets a reader from zero to
a working agent that can generate, debug, and test code.

### Phase 2: Quality and Control (Weeks 4–5)
Write Parts 4–5 (chapters 15–24). This teaches how to improve output
and make it work on constrained hardware.

### Phase 3: Advanced Patterns (Weeks 6–7)
Write Parts 6–7 (chapters 25–34). Patterns, evaluation, improvement.

### Phase 4: Reference and Polish (Week 8)
Write Part 8 quick reference. Cross-link everything. Add exercises.
Set up the website.

### Phase 5: Companion Repo
Create a companion GitHub repo with:
- Working agent implementations for each pattern
- Evaluation test suites
- Model comparison scripts
- Ready-to-use prompt templates

---

## Existing Content Mapping

Current files map to chapters as follows:

| Current File | Maps To Chapters |
|---|---|
| `agents.md` | Ch 5, 8, 25, 26 |
| `constrained-systems.md` | Ch 1, 3, 20–24 |
| `prompt-patterns.md` | Ch 15–19 |
| `tool-use.md` | Ch 6, Ref: Tools Catalog |
| `evaluation.md` | Ch 31–34 |
| `workflows.md` | Ch 9–14, 27 |
| `references.md` | Ref: Models, Engines, Glossary |

Each chapter will pull relevant content from these files and expand it
with beginner-friendly explanations, exercises, and real examples.

---

*This plan is a living document. Adjust scope, reorder chapters, and
split/merge as writing reveals the natural structure.*
