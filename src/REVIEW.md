# Agentic Guide: Review Rubric and Recommendations

> **Reviewer Lens:** PhD AI Engineer · Veteran Software Engineer · Vibe Coder · Education Expert · Book Publisher
> **Date:** 2026-05-11
> **Scope:** PLAN.md + all existing research documents
> **Goal:** Improve coherence, learnability, and utility for a beginner coder who wants AI-assisted coding on low resources with open source tools.

---

## Executive Summary

The research content is **excellent** — technically deep, well-organized, and genuinely useful. The plan's structure is logically sound. But there is a **fundamental tension** between the plan's stated audience ("beginner vibe coders") and the actual learning path it prescribes. The plan teaches you to _build_ agents before teaching you to _use_ them effectively. It puts heavy conceptual middleware (agent loops, tool design, context management) between the reader and their first useful outcome.

**The plan reads like a course for someone who wants to build agent frameworks, not a book for someone who wants AI to help them code on a cheap laptop.**

Below is a structured rubric, diagnosis, and actionable recommendations.

---

## Part 1: Rubric — 8 Evaluation Dimensions

Each dimension is scored 1–5 (5 = excellent fit for target audience).

### 1. Learning Progression (Score: 3/5)

**What it measures:** Does the book take the reader from zero to competence along the path of least cognitive resistance?

**Finding:** The progression is _technically_ correct (setup → concepts → workflows → control → optimization → patterns → eval) but psychologically backwards for the audience. A beginner "vibe coder" wants to _see magic happen_ in Chapter 1, not reason about VRAM decision trees. The current Part 1 is infrastructure-heavy before any AI-assisted coding happens. The "first agent" doesn't arrive until Chapter 4, after three chapters of setup theory.

**Evidence:**
- Ch 1–3 are hardware/engine/model selection. No code generation.
- Ch 5 ("The Agent Loop") requires understanding Think→Act→Observe before the reader has felt the value of AI coding help.
- Part 4 ("Getting Better Output") arrives at Week 5 — a beginner will have abandoned long before understanding temperature and sampling.

### 2. Audience Fit (Score: 2/5)

**What it measures:** Does the content, tone, and pacing match the stated primary audience?

**Finding:** The plan says "assume curiosity, not expertise" but the chapter content assumes the reader understands what a KV cache is, what quantization means, what ReAct is, and what an AST is. The research documents confirm this: `agents.md` discusses "mixture of experts" and "speculative decoding"; `constrained-systems.md` dives into layer offloading and ring attention. These are real topics but they're not beginner-friendly.

**Evidence:**
- "Quantization demystified" (Ch 2) is listed as Week 1 content. A beginner doesn't need to understand Q4 vs Q5 in Week 1; they need to know which button to click to get started.
- Ch 7 ("Context Is Everything") talks about "token budgets" and "progressive disclosure" — these are agent _builder_ concepts, not agent _user_ concepts.
- The 8 essential tools (Ch 6) are framed as "design tools small models can use" — the audience wants to _use_ tools, not _design_ them.

### 3. Cognitive Load Management (Score: 2/5)

**What it measures:** Does the book avoid overwhelming the reader with too many new concepts at once?

**Finding:** The book introduces too many abstraction layers simultaneously. In Part 2 alone, the reader must grasp: agent loops, tool schemas, context windows, token budgets, KV caches, failure modes, AND recovery strategies. That's 7 new mental models in one part. A beginner needs ~1–2 new models per chapter maximum.

**Evidence:**
- Part 2 (4 chapters) introduces the core mechanics of agents. Each chapter is dense.
- Part 4 (5 chapters) on "control" is essentially a prompt engineering course nested inside an agent book.
- Part 5 (5 chapters) on "constrained systems" is a hardware optimization course nested inside the same book.
- The book is trying to be three books: (a) an AI coding guide, (b) a prompt engineering textbook, and (c) a local LLM hardware manual.

### 4. Practical Utility (Score: 4/5)

**What it measures:** Will the reader be able to do useful things after reading?

**Finding:** The workflows section (Part 3) is the strongest part. Chapters 9–14 map directly to real coding tasks. The exercise-driven structure is correct. The "copy-paste ready" principle is right. Where it loses points: the exercises in Parts 1–2 are _building_ exercises (build your agent loop, add a tool) rather than _coding_ exercises (use AI to build a web page, debug a script). The audience wants to code _with_ AI, not build AI.

**Evidence:**
- Ch 4 exercise: "build a simple code generation agent" — the reader wants to _use_ code generation, not implement the framework.
- Ch 6 exercise: "add a new tool to your agent" — again, builder work.
- Ch 9 exercise: "generate a complete module from a spec" — this is the right kind. More of this, earlier.

### 5. Structural Coherence (Score: 3/5)

**What it measures:** Do the parts fit together as a unified narrative, or do they feel like separate documents bolted together?

**Finding:** There is real content bleeding between parts. "Quantization" appears in Ch 2 (Week 1), then again as its own chapter in Ch 20 (Week 6). "Context management" is Ch 7 (Part 2) but is also central to Part 5. "Self-correction" is Ch 28 (Part 6) but is needed for Ch 10's debugging workflow. The reader will experience "I already read about this" and "I need to know this before I get there."

**Evidence:**
- Ch 2 mentions quantization; Ch 20 is the quantization deep dive. The plan says Ch 2 should "demystify" it, but a beginner reading Ch 2 will either skip it (too early) or be confused by the shallow treatment.
- Ch 25 (Plan-Code-Verify) in Part 6 is the actual core loop for coding agents, but it arrives at Week 7 — six weeks after the reader has been using a generic ReAct loop.
- Ch 27 (Greenfield vs Brownfield) in Part 6 fundamentally changes how you approach Ch 9 (Generate Code) in Part 3, but the reader encounters it months later.

### 6. Open Source / Low-Resource Commitment (Score: 4/5)

**What it measures:** Does the book genuinely serve the low-resource, open-source constraint, or does it mention it as window dressing?

**Finding:** This is genuinely strong. The research documents have real hardware tiers, real quantization guidance, real CPU-only paths, and real budget build recommendations. The model recommendations are specific and versioned. This is where the project has a real moat — most guides assume cloud access.

**Evidence:**
- `constrained-systems.md` has specific VRAM budgets per model, engine comparisons per GPU tier, and CPU-only survival strategies.
- Budget build table ($0–$2000) is practical and honest.
- Cloud alternatives section acknowledges when local truly isn't enough.

### 7. Differentiation / Market Position (Score: 5/5)

**What it measures:** Is this book filling a real gap, or is it another LLM tutorial?

**Finding:** This is the strongest dimension. There is no book that teaches local-first, constrained-hardware, coding-agent workflows for beginners. The closest things are scattered blog posts and framework documentation that assume cloud access. This book, if executed well, would be the definitive reference.

### 8. Completeness (Score: 3/5)

**What it measures:** Does the book cover what the audience actually needs, and leave out what they don't?

**Finding:** The book covers too much of what _builders_ need and too little of what _users_ need. Missing: how to work with existing AI coding tools (Aider, Continue, Cursor's open-source alternatives), how to integrate agents into your daily workflow (VS Code, terminal), how to handle the social/trust aspects of AI-assisted coding, how to decide what tasks to give AI vs do yourself. The evaluation section (Part 7) is too advanced for the primary audience — a beginner doesn't need to build golden test datasets; they need to know "is my agent doing a good job?"

---

## Part 2: Structural Diagnosis

### The Core Problem: Builder vs User

The plan is structured as a course for **agent builders** — people who want to understand and construct agentic systems. But the stated audience is **agent users** — people who want AI to help them code.

These two audiences need fundamentally different books:

| | Agent Builder Course | Agent User Guide (Stated Audience) |
|---|---|---|
| Primary question | "How do agents work?" | "How do I get AI to help me code?" |
| First outcome | A working agent loop | A piece of code the AI wrote |
| Key concepts | ReAct, tool schemas, context windows | Prompts, workflows, when to trust the AI |
| Hardware chapter | Quantization, KV cache, offloading | "Here's what runs on your laptop" |
| Tone | Technical, architectural | Pragmatic, workflow-oriented |

**The plan is writing the left column for the right audience.**

### The Secondary Problem: Three Books in One

The plan attempts to cover:
1. **Local LLM infrastructure** (Parts 1, 5) — hardware, engines, quantization, optimization
2. **Agent mechanics** (Parts 2, 4, 6) — loops, tools, prompts, patterns
3. **Coding workflows** (Part 3) — generate, debug, test, review, refactor

These are three valid books. Combining them creates a 34-chapter behemoth that no beginner will finish. The coding workflows (Part 3) are what the audience actually wants. The infrastructure and mechanics are enablers that should be _just-in-time_, not _just-in-case_.

### The Tertiary Problem: Week Numbers Are Fiction

The plan assigns weeks (Week 1–8) but the volume of content per week is unrealistic:
- Week 1: 4 chapters on hardware, models, engines, and building your first agent
- Week 6: 5 chapters on quantization, memory, speed, CPU, and budget builds
- Week 7: 6 chapters on advanced patterns

No beginner reads 5 chapters per week of technical material with exercises. This is more like a 6-month commitment. The week framing creates false expectations.

---

## Part 3: Recommendations

### Recommendation 1: Invert the Learning Path — Magic First, Mechanics Later

**Problem:** The reader doesn't experience value until Chapter 4.
**Fix:** Start with immediate utility, then explain the mechanics.

**Proposed new flow:**

```
PART 0: Quick Start (30 minutes)
  → Install one tool, run one model, get AI to write code
  → Reader has a "wow" moment before they've read a chapter

PART 1: Your First Projects (Week 1-2)
  → Use your agent to: generate a script, debug a bug, write tests
  → Learning by doing. No theory yet.

PART 2: How It Works (Week 3)
  → Now that they've seen it work, explain: what is a model,
    what is an agent loop, what are tools, why did it fail that time

PART 3: Getting Better Results (Week 4-5)
  → Prompts, examples, structured output, temperature
  → Now they care because they've hit quality walls

PART 4: Making It Work on Your Hardware (Week 6)
  → Quantization, memory, speed — only now do they need to optimize

PART 5: Advanced Patterns (Week 7-8)
  → Plan-Code-Verify, fake multi-agent, brownfield strategies

PART 6: Reference
  → Quick lookup, glossary, troubleshooting
```

This follows the pedagogical principle of **concrete before abstract**. The reader first _experiences_ the thing, then _understands_ it, then _optimizes_ it.

### Recommendation 2: Collapse from 34 Chapters to ~18

**Problem:** 34 chapters is a textbook, not a book a beginner will read.
**Fix:** Merge overlapping topics and cut builder-centric content.

**Merges:**
- Ch 1 + Ch 3 → "Get Setup" (one chapter, pick Ollama, done)
- Ch 2 + Ch 20 → "Models and Quantization" (move to Part 2, after they've used one)
- Ch 5 + Ch 25 → "How Agents Think" (teach Plan-Code-Verify from the start, not ReAct then swap)
- Ch 6 + Ch 2 (tool sections) → "Tools" (one chapter)
- Ch 7 + Ch 21 → "Context and Memory" (one chapter)
- Ch 8 + Ch 33 → "When Things Go Wrong" (one troubleshooting chapter)
- Ch 15 + Ch 16 + Ch 18 → "Better Prompts" (one chapter on prompt quality)
- Ch 17 + Ch 19 → "Reliable Output and Safety" (one chapter)
- Ch 22 + Ch 23 + Ch 24 → "Squeezing Performance" (one chapter)
- Ch 26 + Ch 28 → "Patterns: Multi-Agent and Self-Correction" (one chapter)
- Ch 29 + Ch 30 → "Working Long-Term" (one chapter)
- Ch 31 + Ch 32 + Ch 34 → "Is Your Agent Good?" (one lightweight chapter)
- Ch 4 (first agent) moves to Part 0 (Quick Start)

**Cuts for primary audience:**
- Ring attention, speculative decoding (move to appendix/online)
- Detailed engine comparisons (recommend one, mention alternatives in reference)
- CI/CD integration for agent evaluation (too advanced)
- LLM-as-judge methodology (too academic)
- Session memory architecture (move to reference)

### Recommendation 3: Teach Plan-Code-Verify From Day One, Not ReAct

**Problem:** The plan teaches ReAct in Part 2, then introduces Plan-Code-Verify in Part 6 (Week 7). For coding agents, Plan-Code-Verify is the right loop from the start.

**Fix:** The agent loop used throughout the book should be Plan-Code-Verify. ReAct becomes a footnote ("this is what general agents do; for coding, we use Plan-Code-Verify because..."). This eliminates a major restructuring moment in Week 7 and makes every workflow chapter more coherent.

### Recommendation 4: Make the "Constrained" Thread Continuous, Not a Separate Part

**Problem:** Quantization, memory management, and speed optimization are siloed in Part 5 (Week 6). But these constraints affect every decision from Chapter 1 onward.

**Fix:** Weave hardware-awareness into every chapter as margin notes or "On Your Hardware" callout boxes. Keep Part 5 as the deep dive for readers who want to optimize, but don't make them wait 6 weeks to know that their 8GB laptop can run a 7B model at Q4.

**Example:** Ch 2 (Choose a Model) should immediately say "If you have 8GB RAM, download this file." Not "here's how quantization works."

### Recommendation 5: Add a "What Not to Build" Chapter

**Problem:** The complexity ladder in `agents.md` is correct but buried. Beginners will over-engineer.

**Fix:** Add an early chapter (end of Part 1 or start of Part 2) titled "Don't Build What You Don't Need" that covers:
- When a simple prompt is enough (no agent needed)
- When an existing tool (Aider, Continue) solves your problem
- Why multi-agent is almost never the answer for beginners
- The "good enough" agent vs the "perfect" agent

This saves readers months of wasted effort.

### Recommendation 6: Ground Every Concept in a Coding Scenario

**Problem:** The research documents are strong on general agent concepts but the _book_ plan sometimes drifts into generic agent territory.

**Fix:** Every chapter's examples should be coding scenarios. Not "search for weather data" but "find all usages of a function before renaming it." Not "classify this text" but "is this error a bug or a feature request?"

The existing research already does this well in `workflows.md` and `prompt-patterns.md`'s coding section. The book plan should follow that lead more consistently.

### Recommendation 7: Add Real-World Project Threads

**Problem:** The exercises are isolated tasks. The reader doesn't build a coherent skill set.

**Fix:** Thread 2–3 running projects through the book:
- **Project A:** A small CLI tool (greenfield, follows reader from Part 0 through Part 3)
- **Project B:** An existing open-source project the reader contributes to (brownfield, introduced in Part 3)
- **Project C:** A migration task (e.g., JS to TS, introduced in Part 5)

Each chapter's exercise advances one of these projects. By the end, the reader has a portfolio of real work done with AI assistance.

### Recommendation 8: Re-Position Part 7 (Evaluation) as Lightweight Self-Check

**Problem:** Part 7 teaches the reader to build evaluation datasets, run regression suites, and set up CI/CD for agent testing. This is a PhD-level concern for a beginner audience.

**Fix:** Replace with a single chapter: "Is Your Agent Doing a Good Job?" that covers:
- Simple heuristics: does the code run? do the tests pass?
- The "5-minute test": give your agent the same task 3 times, compare
- When to upgrade your model vs fix your prompts
- Red flags that mean your setup is fundamentally broken

Move the rigorous evaluation methodology to the companion website as advanced reading.

---

## Part 4: Proposed Revised Structure

```
AGENTIC GUIDE: AI Coding on a Budget
(Revised — ~18 chapters, 2 running projects)

FRONT MATTER
  Introduction: Why Local, Why Now, Who This Is For
  Before You Start: What's on Your Machine?

QUICK START (30 min)
  Chapter 0: Your First AI-Coded Function
  → Install Ollama, pull a model, write a function, run it

PART 1: DO THINGS (Chapters 1–4)
  Chapter 1: Write New Code with AI
  Chapter 2: Debug and Fix Bugs
  Chapter 3: Write Tests
  Chapter 4: Review and Improve Code

PART 2: UNDERSTAND WHAT'S HAPPENING (Chapters 5–7)
  Chapter 5: How Your Agent Thinks (Plan-Code-Verify)
  Chapter 6: Tools — How Your Agent Reads Files and Runs Commands
  Chapter 7: Context — Why Your Agent Forgets Things

PART 3: GET BETTER RESULTS (Chapters 8–10)
  Chapter 8: Better Prompts (system, few-shot, structured output)
  Chapter 9: When Things Go Wrong (10 failure modes + fixes)
  Chapter 10: Don't Build What You Don't Need (complexity ladder)

PART 4: WORK LIKE A PRO (Chapters 11–14)
  Chapter 11: Greenfield vs Brownfield
  Chapter 12: Refactoring and Migration
  Chapter 13: Patterns That Work (fake multi-agent, self-correction)
  Chapter 14: Working Long-Term (sessions, git, human-in-the-loop)

PART 5: MAKE IT WORK ON YOUR HARDWARE (Chapters 15–16)
  Chapter 15: Models and Quantization (which model for your machine)
  Chapter 16: Squeezing Performance (memory, speed, CPU survival)

PART 6: IS YOUR AGENT GOOD? (Chapter 17)
  Chapter 17: Quick Quality Checks

REFERENCE (Chapter 18)
  Chapter 18: Quick Lookup (models, engines, prompts, troubleshooting, glossary)
```

**Changes from original:**
- 34 chapters → 18 chapters (47% reduction)
- Quick Start moves value experience to minute 30, not day 5
- Plan-Code-Verify taught from Chapter 5, not Week 7
- Hardware content moved later but threaded throughout via callouts
- Evaluation reduced from 4 chapters to 1
- Builder-centric content (tool design, KV cache architecture) moved to reference or cut
- Running project threads added

---

## Part 5: Content Quality Notes on Existing Research

The research documents are strong. Specific notes:

| Document | Strength | Gap |
|---|---|---|
| `agents.md` | Excellent complexity ladder, Plan-Code-Verify, fake multi-agent | Too much framework comparison; tool calling section generic |
| `constrained-systems.md` | Best-in-class hardware guidance, quantization chart, budget builds | KV cache section too technical for target audience |
| `prompt-patterns.md` | Excellent coding-specific templates, brownfield onboarding prompt | CoT/ToT section is too academic; self-consistency is expensive for constrained systems |
| `tool-use.md` | Tool catalog is excellent, security section practical | Three calling patterns create choice overload; just recommend one |
| `workflows.md` | Most comprehensive coding workflow mapping I've seen | 16 workflows is too many for a beginner book; prioritize 6 |
| `evaluation.md` | Executable verification is the right approach for code | Golden dataset section is too advanced; LLM-as-judge is wrong for constrained |
| `references.md` | Well-curated, up-to-date | Add Aider, Continue, OpenCode as "start here" tools |

---

## Part 6: Publishing Strategy

### Format
- **Primary:** Static website (MkDocs Material is the right call — fast, sidebar nav, search, PDF export)
- **Secondary:** PDF for offline reading (important for the target audience who may have limited connectivity)
- **Companion:** GitHub repo with working code for every exercise

### Versioning
- Release v0.1 with Parts 0–2 (Quick Start + Do Things + Understand) — this is the minimum viable book
- Iterate based on reader feedback before writing Parts 3–6
- This avoids the classic mistake of writing 34 chapters nobody reads

### Monetization / Distribution (if relevant)
- Open source the book (matches the open source ethos of the audience)
- Paid tier: companion repo with pre-built agents, video walkthroughs, community
- Sponsorship: local LLM tool companies (Ollama, LM Studio, etc.) would naturally sponsor

---

## Summary Scorecard

| Dimension | Score | Verdict |
|---|---|---|
| Learning Progression | 3/5 | Invert: magic before mechanics |
| Audience Fit | 2/5 | Too builder-centric, needs user-first reframing |
| Cognitive Load | 2/5 | Too many concepts per chapter; collapse and defer |
| Practical Utility | 4/5 | Workflows are strong; exercises need to be user tasks, not builder tasks |
| Structural Coherence | 3/5 | Content bleeding between parts; merge and reorder |
| Low-Resource Commitment | 4/5 | Genuine strength; weave throughout, don't silo |
| Market Differentiation | 5/5 | Real gap, real need, real moat |
| Completeness | 3/5 | Too much builder content, missing user workflow content |
| **Overall** | **3.4/5** | **Strong foundation. Needs structural inversion, not content rewriting.** |

---

## Bottom Line

The research is publication-ready quality. The plan needs a **structural inversion**, not a content rewrite. The fix is:

1. **Flip the path:** value first, understanding second, optimization third
2. **Cut in half:** 34 chapters → 18 chapters by merging and deferring
3. **Teach Plan-Code-Verify from the start:** it's the right loop for coding, not a Week 7 upgrade
4. **Thread hardware awareness throughout:** don't make beginners wait 6 weeks to know what runs on their machine
5. **Add running projects:** isolated exercises → cumulative portfolio
6. **Release early:** ship the first 4 chapters as v0.1, iterate based on real readers

The book, as planned, would be a reference manual that experts consult. With these changes, it becomes a _book that beginners actually read and finish_.
