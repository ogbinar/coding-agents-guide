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

## Core Design Decisions

- **Value first, mechanics later.** Quick Start delivers a "wow" moment in 30 minutes. Theory comes after the reader has experienced utility.
- **Agent user, not agent builder.** Exercises are "use AI to write a script," not "build an agent loop from scratch."
- **Plan-Code-Verify from Day 1.** Not ReAct-then-swap. The right loop for coding is taught upfront.
- **Hardware awareness woven throughout.** "On Your Hardware" callout boxes in every chapter. Deep dive comes later.
- **20 chapters, not 34.** Merged overlapping topics, cut builder-centric content, deferred advanced material to reference.
- **2 running projects.** A greenfield CLI tool and a brownfield contribution, threaded across chapters so the reader builds a cumulative portfolio.

---

## Site Structure

```
agentic-guide/
├── README.md                    # Landing page / home
├── PLAN.md                      # This file
├── WORKS.md                     # Quick-start: "Make something work today"
│
├── 00-quick-start/              # Quick Start (30 min)
│   └── 01-first-function.md     # Chapter 0
│
├── 01-beginner/                 # Part 1: Beginner — Do Things
│   ├── 01-write-code.md         # Chapter 1
│   ├── 02-debug-fix.md          # Chapter 2
│   ├── 03-write-tests.md        # Chapter 3
│   ├── 04-review-improve.md     # Chapter 4
│   └── 05-full-loop.md          # Chapter 5
│
├── 02-understand/               # Part 2: Understand — How It Works
│   ├── 01-how-agents-think.md   # Chapter 6
│   ├── 02-tools.md              # Chapter 7
│   ├── 03-context.md            # Chapter 8
│   └── 04-what-not-to-build.md  # Chapter 9
│
├── 03-advanced/                 # Part 3: Advanced — Better Results
│   ├── 01-better-prompts.md     # Chapter 10
│   ├── 02-failure-modes.md      # Chapter 11
│   ├── 03-greenfield-brownfield.md  # Chapter 12
│   ├── 04-refactor-migrate.md   # Chapter 13
│   ├── 05-patterns.md           # Chapter 14
│   └── 06-long-term.md          # Chapter 15
│
├── 04-daily-driver/             # Part 4: Daily Driver — Make It Yours
│   ├── 01-your-setup.md         # Chapter 16
│   ├── 02-models-quantization.md # Chapter 17
│   ├── 03-performance.md        # Chapter 18
│   ├── 04-quality-checks.md     # Chapter 19
│   └── 05-trust-and-safety.md   # Chapter 20
│
└── reference/                   # Reference (lookup, not read linearly)
    ├── models.md
    ├── engines.md
    ├── tools-catalog.md
    ├── prompt-templates.md
    ├── troubleshooting.md
    └── glossary.md
```

---

## Table of Contents (Reader-Facing)

### Front Matter
- **Introduction** — Why local coding agents, who this is for, what you'll learn
- **Before You Start** — Hardware self-assessment, setting expectations

---

### Quick Start (30 Minutes)

> Goal: AI writes you a function that runs, before you've read a chapter.

**Chapter 0: Your First AI-Coded Function**
- Install one tool (Ollama, opinionated default)
- Pull one model (recommendation based on hardware self-assessment)
- Write one function with AI help, run it
- "On Your Hardware" callout: what to download if you have 8GB RAM, 16GB RAM, GPU, CPU-only
- No theory, no explanation of how it works yet
- Project A seed: start the CLI tool project with its first function

---

### Part 1: Beginner — Do Things (Chapters 1–5)

> Goal: Use AI to do real coding tasks. Learn by doing. Theory comes later.

**Chapter 1: Write New Code with AI**
- From description to working code
- How to ask for what you want (coding-specific prompt patterns)
- Greenfield scaffolding: starting a new project
- Specifying constraints that actually work
- Self-review before you even look at the output
- Project A: scaffold the CLI tool project structure
- "On Your Hardware" callout: model size vs code quality tradeoffs

**Chapter 2: Debug and Fix Bugs**
- Feeding error messages to your agent
- The debugging loop: error → hypothesis → fix → verify
- Root cause vs symptom (teaching the agent the difference)
- When the agent makes things worse
- Project A: debug a planted bug in the CLI tool
- Project B introduced: pick an open-source project to contribute to

**Chapter 3: Write Tests**
- Generating tests from existing code
- Happy path, edge cases, error cases
- Running and fixing failing tests
- Coverage targets for agent-generated tests
- Project A: achieve 80% coverage on a module
- Project B: add tests to an existing function

**Chapter 4: Review and Improve Code**
- Code review prompts that catch real issues
- Categorizing findings (critical / warning / suggestion)
- Style enforcement vs substance
- Performance anti-pattern detection
- Project A: review and improve the CLI tool
- Project B: submit a review of an existing PR

**Chapter 5: The Full Loop**
- Plan → Code → Verify → Fix → Commit (the core loop, taught as practice)
- Putting all workflows together from Chapters 1–4
- When to stop iterating
- Git integration and commit messages
- Project A: complete a full feature from spec to commit
- Project B: complete a small contribution

---

### Part 2: Understand — How It Works (Chapters 6–9)

> Goal: Now that you've seen it work, understand what's happening underneath.

**Chapter 6: How Your Agent Thinks**
- The Plan-Code-Verify loop explained (the loop you've been using, now named)
- Why this loop beats ReAct for coding tasks
- Max iterations and early exit (when to stop the agent)
- State management: what the agent remembers between turns
- Trace an agent loop by hand on a real example from Project A

**Chapter 7: Tools — How Your Agent Reads Files and Runs Commands**
- What tools are and why agents need them
- The 8 essential tools every coding agent uses
- Tool output management (don't blow your context)
- How existing tools (Aider, Continue) handle this under the hood
- No tool design exercises — just understanding what your agent is doing

**Chapter 8: Context — Why Your Agent Forgets Things**
- The context window explained (it's RAM for your model)
- Token budgets: where your context goes
- Loading code efficiently (don't load the whole repo)
- Progressive disclosure: load what you need, when you need it
- "On Your Hardware" callout: context window size vs available RAM

**Chapter 9: Don't Build What You Don't Need**
- When a simple prompt is enough (no agent needed)
- When an existing tool (Aider, Continue, OpenCode) solves your problem
- Why multi-agent is almost never the answer for beginners
- The "good enough" agent vs the "perfect" agent
- Complexity ladder: start simple, add complexity only when you hit a wall
- Saves readers months of wasted effort

---

### Part 3: Advanced — Better Results (Chapters 10–15)

> Goal: You've hit quality walls. Now learn the patterns that separate good results from great ones.

**Chapter 10: Better Prompts**
- System prompts: anatomy of a great one, coding-specific templates
- Few-shot examples: why examples beat instructions for small models
- Structured output: JSON vs XML vs diffs, getting small models to comply
- Temperature and sampling: what actually matters for coding tasks
- Project A: improve a failing prompt with examples and structure

**Chapter 11: When Things Go Wrong**
- The 10 most common failure modes (with real examples from Projects A/B)
- How to recognize each failure mode
- Recovery strategies for each
- When to blame the prompt vs the model vs the tools vs the hardware
- Project B: diagnose and fix a broken agent run on the brownfield project

**Chapter 12: Greenfield vs Brownfield**
- How context changes everything between new and existing code
- Codebase onboarding strategies
- Convention detection and adherence
- Legacy code handling
- Project B: adapt your agent for the open-source codebase conventions

**Chapter 13: Refactoring and Migration**
- Safe refactoring with diffs
- Framework migrations (with real examples)
- Incremental migration strategies
- Preserving behavior during transformation
- Project C introduced: migrate a small module (e.g., JS to TS)

**Chapter 14: Patterns That Work**
- Fake multi-agent: role-switching with a single model (Coder → Reviewer → Revisor)
- Self-correction: compile-and-fix loops, test-driven generation
- Multi-pass refinement
- When real multi-agent actually makes sense
- Project C: apply self-correction to the migration

**Chapter 15: Working Long-Term**
- Resuming work across sessions
- Branch-per-task workflows
- Human-in-the-loop: approval gates for risky operations
- Pair programming mode vs autonomous mode
- Project A/B/C: tie up loose ends with session persistence and git hygiene

---

### Part 4: Daily Driver — Make It Yours (Chapters 16–20)

> Goal: Build the personal setup, workflows, and intuition that become your daily driver for AI-assisted coding.

**Chapter 16: Your Setup**
- Choosing your daily driver tool (Aider, Continue, OpenCode, or custom)
- VS Code integration, terminal workflows
- Building your prompt library and example collection
- Your model lineup: which model for which task
- Project A/B/C: set up your permanent workspace

**Chapter 17: Models and Quantization**
- Model size vs capability explained simply
- Quantization without the math: what does "Q4" actually mean for your code?
- Coding capability degradation curve
- Recommendation matrix by your hardware tier
- Where to download models (trusted sources)
- "On Your Hardware" deep dive: which model to run, when

**Chapter 18: Squeezing Performance**
- Memory management: KV cache, layer offloading, GPU + CPU hybrid
- Speed optimization: tokens/sec, prefix caching, thread count tuning
- CPU-only survival: yes, you can do this
- Budget builds: GPU buying guide, used market picks, cloud rental
- When to upgrade hardware vs optimize software

**Chapter 19: Quality Checks**
- Simple heuristics: does the code run? do the tests pass?
- The "5-minute test": give your agent the same task 3 times, compare
- When to upgrade your model vs fix your prompts
- Red flags that mean your setup is fundamentally broken
- Regression detection for your personal workflow

**Chapter 20: Trust and Safety**
- Input sanitization: prevent prompt injection in your code
- Output validation: catch bad output before it's used
- Tool execution safety: sandboxing, approval gates
- Trust calibration: when to let the agent run vs when to stay in the loop
- Social aspects: explaining AI-assisted work to collaborators

---

### Reference (Lookup, Not Read Linearly)

> Tear-out pages. Copy-paste ready.

**Models** — Recommendation matrix by hardware tier, with direct download links
**Engines** — Install commands for every platform
**Tools Catalog** — The 24 essential coding agent tools, with signatures
**Prompt Templates** — Copy-paste system prompts for every workflow
**Troubleshooting** — Symptom → cause → fix lookup table
**Glossary** — Terms explained simply (tokens, quantization, KV cache, etc.)

---

## Running Projects

### Project A: CLI Tool (Greenfield)
- A small command-line utility the reader builds from scratch
- Follows the reader from Quick Start through Part 3
- Each chapter's exercise advances this project
- By end of Part 1, the reader has a working tool with tests

### Project B: Open-Source Contribution (Brownfield)
- Reader picks an existing open-source project
- Introduced in Chapter 2 (Debug), deepens in Chapter 12 (Brownfield)
- Teaches convention detection, PR workflows, legacy code handling

### Project C: Migration Task (Introduced in Advanced)
- e.g., JavaScript to TypeScript, or framework migration
- Introduced in Chapter 13, uses self-correction and multi-pass patterns

---

## Content Principles

### Writing Style
- **Show, don't tell:** Every concept accompanied by a code example or diagram
- **Opinionated defaults:** "Use X unless you have reason Y" not "here are 7 options"
- **Progressive disclosure:** Simple answer first, deep dive in expandable sections
- **Real failures:** Show what actually breaks, not just happy paths
- **Copy-paste friendly:** Code blocks should work as-is
- **"On Your Hardware" callouts:** Every chapter includes a hardware-aware sidebar

### Each Chapter Should Have
1. **TL;DR** — 3 bullet points, what you'll learn
2. **Prerequisites** — What chapters to read first (if any)
3. **Core content** — Concepts, examples, diagrams
4. **Common pitfalls** — What goes wrong and how to fix it
5. **Exercise** — Hands-on task advancing a running project
6. **On Your Hardware** — Hardware-aware callout box
7. **Further reading** — Links to reference section

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
/quick-start/         → Chapter 0
/beginner/            → Part 1 chapters
/understand/          → Part 2 chapters
/advanced/            → Part 3 chapters
/daily-driver/        → Part 4 chapters
/reference/           → Quick reference

Static site generator: MkDocs Material or Astro
- Sidebar navigation with collapsible parts
- Search (essential for reference lookups)
- Code syntax highlighting
- Print/PDF export
- Dark mode (it's a coding guide, obviously)
```

---

## Production Plan

### Phase 1: Quick Start + Beginner (Weeks 1–2)
Write Quick Start + Part 1 (Chapters 0–5). This is the minimum viable
book — gets a reader from zero to a working agent that can generate,
debug, test, review, and commit code. Ship this as v0.1.

### Phase 2: Understand (Weeks 3–4)
Write Part 2 (Chapters 6–9). Explains the mechanics behind what the
reader has already been doing. Includes "Don't Build What You Don't Need."

### Phase 3: Advanced (Weeks 5–6)
Write Part 3 (Chapters 10–15). Better prompts, failure modes,
brownfield work, refactoring, patterns, long-term workflows.

### Phase 4: Daily Driver (Weeks 7–8)
Write Part 4 (Chapters 16–20). Personal setup, models, performance,
quality checks, trust and safety.

### Phase 5: Reference and Polish (Week 9)
Write reference section. Cross-link everything. Set up the website.

### Phase 6: Companion Repo
Create a companion GitHub repo with:
- Working agent implementations for each pattern
- Project A/B/C starter code and solutions
- Ready-to-use prompt templates

---

## Existing Content Mapping

Current files map to chapters as follows:

| Current File | Maps To Chapters |
|---|---|
| `agents.md` | Ch 6, 9, 14, 15 |
| `constrained-systems.md` | Ch 0 (callout), 17, 18 |
| `prompt-patterns.md` | Ch 1, 10, Ref: Prompt Templates |
| `tool-use.md` | Ch 7, Ref: Tools Catalog |
| `evaluation.md` | Ch 11, 19 |
| `workflows.md` | Ch 1–5, 12, 13 |
| `references.md` | Ref: Models, Engines, Glossary |

Each chapter will pull relevant content from these files and expand it
with beginner-friendly explanations, exercises tied to running projects,
and "On Your Hardware" callout boxes.

---

## Changes from Original Plan

| | Original | Revised |
|---|---|---|
| Chapters | 34 | 20 |
| Structure | 8 parts | Quick Start + 4 parts + reference |
| First value | Chapter 4 (Week 1) | Chapter 0 (30 min) |
| Agent loop | ReAct → Plan-Code-Verify at Week 7 | Plan-Code-Verify from Chapter 0 |
| Hardware | Siloed in Part 5 (Week 6) | Callouts everywhere + deep dive in Part 4 |
| Exercises | Isolated builder tasks | Running projects A/B/C |
| Evaluation | 4 chapters, PhD-level | 1 chapter, lightweight self-check |
| Focus | Agent builder | Agent user |
| New | — | Chapter 9: Don't Build What You Don't Need |
| New | — | Chapter 20: Trust and Safety |
| New | — | Part 4: Daily Driver (personalization, long-term use) |

---

*This plan is a living document. Adjust scope, reorder chapters, and
split/merge as writing reveals the natural structure.*
