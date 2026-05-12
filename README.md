# Agentic Guide: AI Coding on Constrained Systems

> A practical guide for using AI coding agents on your laptop, consumer GPU, or CPU-only machine.

---

## Who This Is For

You're a developer who:
- Wants AI to help you code, but doesn't have cloud GPU budgets
- Runs on consumer hardware (laptop, used GPU, CPU-only)
- Is comfortable coding but new to local LLMs and agents
- Thinks in terms of "what can I build?" not "what's the architecture?"

**This guide gets you coding with AI in 30 minutes, not 3 weeks.**

---

## Read Online

The full book is published as a styled, navigable site:

👉 **[Read the book online](https://ogbinar.github.io/coding-agents-guide/)**

---

## Quick Start (30 Minutes)

Don't read the whole book. Just do this:

1. **Install Ollama** — `curl -fsSL https://ollama.com/install.sh | sh`
2. **Pull a model** — `ollama pull qwen2.5-coder:7b` (or smaller if you have <8GB RAM)
3. **Run your first AI-coded function** — See [Quick Start](./src/00-quick-start/01-first-function.md)

That's it. You'll have AI writing code before you've read a single chapter of theory.

---

## Run Locally

The book source lives in the `src/` directory and is built with [mdBook](https://rust-lang.github.io/mdBook/).

### Build and serve

```bash
cargo install mdbook
mdbook serve --open
```

### Build only

```bash
mdbook build
```

The built site is generated in `./book/` and can be opened in a browser.

---

## The Guide

### Quick Start (30 min)
Get AI writing code immediately. No theory.

- [Chapter 0: Your First AI-Coded Function](src/00-quick-start/01-first-function.md)

### Part 1: Beginner — Do Things
Use AI to do real coding tasks. Learn by doing.

- [Chapter 1: Write New Code with AI](src/01-beginner/01-write-code.md)
- [Chapter 2: Debug and Fix Bugs](src/01-beginner/02-debug-fix.md)
- [Chapter 3: Write Tests](src/01-beginner/03-write-tests.md)
- [Chapter 4: Review and Improve Code](src/01-beginner/04-review-improve.md)
- [Chapter 5: The Full Loop](src/01-beginner/05-full-loop.md)

### Part 2: Understand — How It Works
Now that you've seen it work, understand what's happening.

- [Chapter 6: How Your Agent Thinks](src/02-understand/01-how-agents-think.md)
- [Chapter 7: Tools — How Your Agent Reads Files](src/02-understand/02-tools.md)
- [Chapter 8: Context — Why Your Agent Forgets](src/02-understand/03-context.md)
- [Chapter 9: Don't Build What You Don't Need](src/02-understand/04-what-not-to-build.md)

### Part 3: Advanced — Better Results
You've hit quality walls. Learn the patterns that separate good from great.

- [Chapter 10: Better Prompts](src/03-advanced/01-better-prompts.md)
- [Chapter 11: When Things Go Wrong](src/03-advanced/02-failure-modes.md)
- [Chapter 12: Greenfield vs Brownfield](src/03-advanced/03-greenfield-brownfield.md)
- [Chapter 13: Refactoring and Migration](src/03-advanced/04-refactor-migrate.md)
- [Chapter 14: Patterns That Work](src/03-advanced/05-patterns.md)
- [Chapter 15: Working Long-Term](src/03-advanced/06-long-term.md)

### Part 4: Daily Driver — Make It Yours
Build the personal setup that becomes your daily workflow.

- [Chapter 16: Your Setup](src/04-daily-driver/01-your-setup.md)
- [Chapter 17: Models and Quantization](src/04-daily-driver/02-models-quantization.md)
- [Chapter 18: Squeezing Performance](src/04-daily-driver/03-performance.md)
- [Chapter 19: Quality Checks](src/04-daily-driver/04-quality-checks.md)
- [Chapter 20: Trust and Safety](src/04-daily-driver/05-trust-and-safety.md)

### Reference
Lookup, not read linearly.

- [Models](src/reference/models.md) — Recommendation matrix by hardware
- [Engines](src/reference/engines.md) — Install commands for every platform
- [Tools Catalog](src/reference/tools-catalog.md) — The 24 essential coding agent tools
- [Prompt Templates](src/reference/prompt-templates.md) — Copy-paste system prompts
- [Troubleshooting](src/reference/troubleshooting.md) — Symptom → cause → fix
- [Glossary](src/reference/glossary.md) — Terms explained simply

---

## Running Projects

This guide uses **2 running projects** so you build a cumulative portfolio:

- **Project A (CLI Tool):** A small command-line utility you build from scratch (greenfield)
- **Project B (Open-Source):** An existing project you contribute to (brownfield)

Each chapter's exercise advances one of these projects.

---

## Why Local?

Most AI coding guides assume cloud GPUs and expensive APIs. This guide focuses on:

- **Consumer GPUs** (8–24 GB VRAM)
- **CPU-only systems** (yes, it works)
- **Quantized models** (4-bit, 3-bit, even 2-bit)
- **Offline work** (privacy, cost, no internet required)

The goal: **high-quality AI-assisted coding without expensive infrastructure.**

---

## What Makes This Different

| Other Guides | This Guide |
|---|---|
| Cloud-first, assume A100 access | Local-first, assume laptop |
| Teach you to build agents | Teach you to use agents |
| Theory before practice | Practice in 30 minutes |
| Generic examples | Coding-specific workflows |
| 34 chapters of theory | 20 chapters + reference |
| Isolated exercises | Running projects A/B/C |

---

## Deployment

This site is built with mdBook and deployed to GitHub Pages via GitHub Actions.

1. Push to the `master` branch
2. `.github/workflows/deploy.yml` triggers the build
3. mdBook compiles `src/` into static HTML in `./book/`
4. The `actions/deploy-pages` action publishes to GitHub Pages

---

## Contributing

This is a living document. Add sections, refactor, and expand as the landscape evolves.

---

## Archive

Original research documents that informed this guide are archived in [`archive/`](./archive/):

- `agents.md` — Agent architectures and loops
- `constrained-systems.md` — Low-resource hardware considerations
- `evaluation.md` — Evaluation strategies and metrics
- `prompt-patterns.md` — Prompt engineering patterns
- `references.md` — References and resources
- `tool-use.md` — Tool use and function calling
- `workflows.md` — Coding workflows and patterns

These are kept for reference but the content has been integrated into the main guide.

---

## License

This guide is open source. Feel free to use, share, and adapt.
