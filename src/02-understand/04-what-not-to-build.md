# Chapter 9: Don't Build What You Don't Need

> Save yourself months of wasted effort. Sometimes a simple prompt or existing tool is enough.

---

## TL;DR

- Most coding tasks don't need custom agents
- Simple prompts work for one-off code generation
- Existing tools (Aider, Continue, OpenCode) solve 90% of use cases
- Multi-agent is almost never the answer for beginners
- Start simple, add complexity only when you hit a wall

---

## Prerequisites

- Completed [Chapters 1-8](../01-beginner/01-write-code.md) through [Chapter 8](./03-context.md)
- Have used agents for coding
- Ready to learn when NOT to build custom solutions

---

## The Complexity Ladder

**Start at the bottom. Climb only when necessary.**

```
┌─────────────────────────────────────────────────────────────┐
│              Complexity Ladder                               │
│                                                              │
│  5. Multi-Agent Systems         ← Almost never needed       │
│     ├─ Specialized agents (coder, reviewer, tester)         │
│     └─ Complex orchestration                                │
│                                                              │
│  4. Custom Agent Framework      ← Only if you need          │
│     ├─ LangGraph, CrewAI, AutoGen                           │
│     └─ Full control over loops                              │
│                                                              │
│  3. Custom Agent Loop           ← When existing tools       │
│     ├─ Your own Plan-Code-Verify loop                       │
│     └─ Using llama.cpp/Ollama directly                      │
│                                                              │
│  2. Existing Agent Tools        ← Start here for most work  │
│     ├─ Aider, Continue, OpenCode                            │
│     └─ Pre-built, well-tested                               │
│                                                              │
│  1. Simple Prompts              ← For one-off tasks         │
│     ├─ Ollama chat, web UIs                                 │
│     └─ No setup, just ask                                   │
│                                                              │
│  0. Manual Coding               ← Sometimes still best      │
│     └─ For tiny, well-understood tasks                      │
└─────────────────────────────────────────────────────────────┘
```

**Rule:** Start at level 0-1. Only climb when you hit a wall.

---

## When a Simple Prompt Is Enough

### Use Simple Prompts For:

✅ **One-off code generation**
```
Write a Python function that parses a CSV file and returns a list of dictionaries.
```

✅ **Quick debugging**
```
This code gives IndexError: list index out of range. How do I fix it?

def get_first(items):
    return items[0]
```

✅ **Learning concepts**
```
Explain how decorators work in Python with examples.
```

✅ **Code review (quick)**
```
Review this code for bugs:
{paste code}
```

✅ **Test generation**
```
Write pytest tests for this function:
{paste code}
```

### Don't Build an Agent For:

❌ **Generating a single function** → Use a prompt
❌ **Fixing one bug** → Use a prompt
❌ **Learning a concept** → Use a prompt or documentation
❌ **Occasional code review** → Use a prompt

---

## When Existing Tools Solve Your Problem

### Use Aider For:

✅ **Terminal-based coding with AI**
```bash
aider src/my_script.py --message "Add error handling"
```

✅ **Git-aware editing**
- Automatically commits changes
- Shows diffs before applying
- Maintains commit history

✅ **Multi-file edits**
```bash
aider src/ --message "Refactor to use dataclasses"
```

**Setup time:** 5 minutes
**Build time:** 0 minutes (it exists)

---

### Use Continue For:

✅ **VS Code integration**
- Install extension
- Select code → Right-click → "Edit with AI"
- Inline editing, no terminal needed

✅ **IDE-aware context**
- Understands your workspace
- Uses LSP for symbol lookup
- Respects project structure

✅ **Chat + editing**
- Chat panel for questions
- Inline editing for changes
- Both in one interface

**Setup time:** 10 minutes
**Build time:** 0 minutes (it exists)

---

### Use OpenCode For:

✅ **Interactive terminal agent**
```bash
opencode "Add search feature to my CLI tool"
```

✅ **Tool use with approval**
- Shows what it will do
- Asks before dangerous operations
- Lets you intervene

✅ **Session persistence**
- Remembers across turns
- Maintains project context
- Good for longer tasks

**Setup time:** 10 minutes
**Build time:** 0 minutes (it exists)

---

### Comparison: Existing Tools

| Tool | Best For | Setup | Learning Curve |
|---|---|---|---|
| **Aider** | Terminal workflow | ⭐ Easy | Low |
| **Continue** | VS Code users | ⭐ Easy | Low |
| **OpenCode** | Interactive agent | ⭐ Easy | Low |
| **Cursor** | Full IDE replacement | ⭐⭐ Mod | Medium |

---

## When Multi-Agent Is Almost Never the Answer

### The Multi-Agent Hype

**Marketing says:**
> "Use specialized agents: one for coding, one for reviewing, one for testing. They collaborate for perfect results!"

**Reality:**
- More complexity = more things to break
- Single competent model beats multiple weak ones
- Coordination overhead eats benefits
- Hard to debug when things go wrong

### When Multi-Agent Actually Makes Sense

✅ **You have a specific workflow that requires it**
- Example: Automated PR review pipeline
- Example: Continuous integration with multiple checks

✅ **You're building a product, not just coding**
- Example: SaaS platform with AI agents
- Example: Internal tool for your team

✅ **You've hit limits with single-agent**
- Example: Single agent can't handle the complexity
- Example: You need parallel processing

### When Multi-Agent Is Overkill

❌ **Just learning about agents** → Use single agent
❌ **Coding personal projects** → Use single agent
❌ **One-off tasks** → Use simple prompts
❌ **Most beginner/intermediate use cases** → Use single agent

**Rule of thumb:** If you can't explain why you need multi-agent in one sentence, you don't need it.

---

## The "Good Enough" Agent vs the "Perfect" Agent

### The Perfect Agent Trap

**You:** "I'll build a custom agent that:"
- Reads the entire codebase
- Understands architecture
- Plans features intelligently
- Writes perfect code
- Reviews its own work
- Runs comprehensive tests
- Handles edge cases
- Self-improves over time

**Time to build:** 3-6 months
**Result:** Still not as good as existing tools

### The Good Enough Agent

**You:** "I'll use Aider to:"
- Edit files with AI
- Run tests automatically
- Commit changes to git

**Time to set up:** 10 minutes
**Result:** 80% of what you wanted, today

### The 80/20 Rule

| Effort | Result |
|---|---|
| **Simple prompt** | 20% of value, 1% effort |
| **Existing tool** | 80% of value, 5% effort |
| **Custom agent** | 90% of value, 50% effort |
| **Perfect agent** | 95% of value, 200% effort |

**Sweet spot:** Existing tools (80% value, 5% effort)

---

## Decision Tree: What Should You Use?

```
What are you trying to do?
│
├─ Generate one function / fix one bug?
│   └─ Use a simple prompt (Ollama chat, web UI)
│
├─ Code regularly but don't want to build tools?
│   ├─ Terminal workflow? → Use Aider
│   ├─ VS Code user? → Use Continue
│   └─ Want interactive agent? → Use OpenCode
│
├─ Need custom workflow that existing tools don't support?
│   ├─ Simple automation? → Build a custom loop (Chapter 6)
│   └─ Complex orchestration? → Consider framework (LangGraph)
│
├─ Building a product with AI agents?
│   └─ Build custom agent (you need full control)
│
└─ Just learning?
    └─ Start with existing tools, build later if needed
```

---

## Exercise: Evaluate Your Setup

**Task:** Assess whether you're using the right level of complexity.

**Steps:**
1. List your current AI coding workflows
2. For each workflow, ask:
   - "Am I using a simple prompt?"
   - "Could an existing tool do this?"
   - "Am I building something custom?"
   - "Why am I building instead of using?"
3. Identify opportunities to simplify
4. Try an existing tool for one week

**Reflection:**
- What are you building that already exists?
- What could be simpler?
- Where does custom actually make sense?
- What will you stop building?

---

## On Your Hardware

### Small Models (1.5B-3B)

**Best approach:** Simple prompts + existing tools
- Small models struggle with complex agent loops
- Existing tools handle context management
- Don't try to build custom agents

### Medium Models (7B-14B)

**Best approach:** Existing tools + occasional custom loops
- Can handle simple custom loops
- Existing tools still recommended for most work
- Build custom only for specific needs

### Large Models (32B+)

**Best approach:** Any approach works
- Can handle complex agent loops
- Still, existing tools are often easier
- Build custom when you need specific features

---

## Common Pitfalls

### Pitfall 1: Building Before Evaluating

**Bad:**
```
"I need an agent for code review"
→ Immediately start building
```

**Good:**
```
"I need an agent for code review"
→ Try Aider's review feature
→ Try Continue's review feature
→ Only build if neither works
```

### Pitfall 2: Over-Engineering

**Bad:**
```
Build multi-agent system for:
- Coder agent
- Reviewer agent  
- Tester agent
- Orchestrator agent

Total: 4 agents, 2000 lines of code
```

**Good:**
```
Use single agent with Plan-Code-Verify loop
Total: 1 agent, 200 lines of code
```

### Pitfall 3: Ignoring Existing Solutions

**Bad:**
```
"None of the existing tools do exactly what I want"
→ Build from scratch
```

**Good:**
```
"None of the existing tools do exactly what I want"
→ Can I adapt my workflow to match existing tools?
→ Can I combine multiple tools?
→ Only build if truly necessary
```

---

## When to Actually Build

Build your own agent when:

✅ **You need custom tool integration**
- Example: Your company has internal APIs
- Example: Specialized testing framework

✅ **You want full control over the loop**
- Example: Specific verification steps
- Example: Custom state management

✅ **You're building a specific workflow**
- Example: Automated documentation generator
- Example: Code migration tool

✅ **You're learning how agents work**
- Example: Educational project
- Example: Research

✅ **Existing tools don't meet your needs**
- Example: Specific constraints
- Example: Unique requirements

---

## Summary: The Simplicity Principle

**Start simple. Add complexity only when you hit a wall.**

1. **Try a simple prompt first** — 50% of tasks done here
2. **Use existing tools** — 40% of tasks done here
3. **Build custom loops** — 9% of tasks here
4. **Build complex systems** — 1% of tasks here

**Your goal:** Spend 90% of your time coding, not building agents.

---

## Next Steps

You've completed the **Understand** section. You now know:

- ✅ How agents think (Plan-Code-Verify)
- ✅ How agents use tools
- ✅ How context works
- ✅ When NOT to build

### Continue to Part 3: Advanced

- [Chapter 10: Better Prompts](../03-advanced/01-better-prompts.md) — Improve your results
- [Chapter 11: When Things Go Wrong](../03-advanced/02-failure-modes.md) — Handle failures
- [Chapter 12: Greenfield vs Brownfield](../03-advanced/03-greenfield-brownfield.md) — Different contexts

---

## Further Reading

- [Reference: Tools Catalog](../reference/tools-catalog.md) — What tools exist
- [Reference: Prompt Templates](../reference/prompt-templates.md) — Ready-to-use prompts
- [WORKS.md](../WORKS.md) — Quick reference for existing tools
