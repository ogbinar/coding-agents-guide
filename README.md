# Agentic Guide: Local LLM Agents on Constrained Systems

A living guide collecting best practices for building, running, and controlling
LLM-based agents on local and resource-constrained systems.

## Why This Guide

Most agentic frameworks and tutorials assume cloud GPUs and large models (70B+).
This guide focuses on what works when you're running on:

- Consumer GPUs (8–24 GB VRAM)
- CPU-only systems
- Edge devices and laptops
- Quantized models (4-bit, 3-bit, even 2-bit)

The goal: **high-quality, controlled agent behavior without expensive infrastructure.**

## Focus: Coding Agents

While the principles apply broadly, this guide is optimized for **coding agents** —
LLM agents that understand, generate, modify, debug, test, and review code.

Coding agents have unique challenges:
- **Structured output is non-negotiable** — code must be syntactically valid
- **Context is code, not text** — loading the right files matters more than loading more text
- **Verification is executable** — you can compile, run tests, and lint to check quality automatically
- **Project context changes everything** — greenfield scaffolding vs brownfield refactoring are fundamentally different tasks

The [workflows.md](./workflows.md) document maps 16 end-to-end coding workflows across
10 different project contexts (greenfield, brownfield, monorepo, microservice, etc.).

## Guide Structure

| Document | What It Covers |
|---|---|
| [agents.md](./agents.md) | Core principles of local agentic systems |
| [constrained-systems.md](./constrained-systems.md) | Hardware constraints, quantization, inference optimization |
| [prompt-patterns.md](./prompt-patterns.md) | Prompt techniques for quality and control |
| [tool-use.md](./tool-use.md) | Tool/function calling strategies for small models |
| [workflows.md](./workflows.md) | End-to-end coding agent workflows and topic roadmap |
| [evaluation.md](./evaluation.md) | Measuring and improving agent quality |
| [references.md](./references.md) | Papers, tools, frameworks, and further reading |

## Scope

- **In scope:** Techniques, patterns, and practices for running agents locally
- **In scope:** Trade-offs between model size, quality, and speed
- **In scope:** Framework-agnostic advice (works with LangChain, LlamaIndex, CrewAI, custom, etc.)
- **In scope:** Coding-specific workflows (generation, debugging, testing, migration, etc.)
- **Out of scope:** Cloud deployment, training/fine-tuning deep dives, RAG architecture (unless relevant to agents)

## Contributing

This is a living document. Add sections, refactor, and expand as the landscape evolves.
