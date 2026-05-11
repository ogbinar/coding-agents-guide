# Agents: Core Principles for Local Agentic Systems

> Best practices for building LLM agents that work well on local hardware.

## 1. Start Small, Then Scale

The biggest mistake with local agents is reaching for the most complex pattern first.

### The Complexity Ladder

```
Level 1: Single-shot prompt → Direct answer
Level 2: Chain → Sequential prompts with passing context
Level 3: Router → Conditional branching based on classification
Level 4: ReAct → Reason + Act loop with tool use
Level 5: Multi-agent → Multiple specialized agents coordinating
```

**Rule of thumb:** Use the lowest level that solves your problem. Each level adds:
- More token consumption (latency + memory)
- More failure modes (compounding errors)
- More debugging complexity

### When to Move Up

| From | To | When |
|---|---|---|
| Single-shot | Chain | When you need structured multi-step reasoning |
| Chain | Router | When different inputs need different processing paths |
| Router | ReAct | When the agent needs to interact with external tools/data |
| ReAct | Multi-agent | When tasks require distinct expertise or parallel execution |

## 2. Model Selection for Agents

Not all small models are equal at agentic tasks. Key capabilities matter more than raw benchmark scores.

### Critical Capabilities for Agents

1. **Instruction following** — Does the model respect system prompts and constraints?
2. **Tool calling** — Can it reliably format function calls and parse outputs?
3. **Reasoning depth** — Can it maintain a chain of thought without collapsing?
4. **Context window** — Can it hold the conversation history + tool outputs?
5. **Output consistency** — Does it produce parseable, structured output reliably?

### Model Recommendations by Tier

**< 8 GB VRAM (or CPU):**
- Phi-3-mini (3.8B) — Surprisingly strong instruction following
- Qwen2.5-3B — Good tool calling support
- Gemma-2-2B — Strong for simple chains

**8–16 GB VRAM:**
- Qwen2.5-7B / Qwen3-8B — Best all-around for agents in this range
- Llama-3.1-8B — Solid, widely supported
- Mistral-7B-v0.3 — Good reasoning, smaller context needs

**16–24 GB VRAM:**
- Qwen2.5-14B / Qwen3-14B — Strong reasoning and tool use
- Llama-3.1-70B (4-bit quantized) — If you can fit it, the quality jump is real
- Mixtral-8x7B (4-bit) — Mixture of experts, good for diverse tasks

### Quantization Notes

- **Q4_K_M** (4-bit) is the sweet spot for most agent workloads
- Avoid Q2 for agents — instruction following degrades significantly below Q3
- For tool calling, Q5 or higher preserves formatting reliability
- Test your specific model+quant combo on your actual prompts, not benchmarks

## 3. The Agent Loop

A well-structured agent loop is more important than model size.

### Minimal ReAct Loop

```
┌─────────────┐
│  User Input  │
└──────┬──────┘
       ▼
┌─────────────┐     ┌──────────┐
│  Think/Plan  │────▶│ Use Tool?  │
└──────┬──────┘     └────┬─────┘
       │                 │ Yes
       │ No              ▼
       │          ┌────────────┐
       │          │ Execute    │
       │          │ Tool       │
       │          └─────┬──────┘
       │                ▼
       │          ┌────────────┐
       │          │ Observe    │
       │          │ Result     │
       │          └─────┬──────┘
       │                │
       │                ▼
       │          ┌─────────────┐
       │          │  Think/Plan  │  (loop back)
       │          └──────┬──────┘
       │                 │
       ▼                 ▼
┌──────────────────────────┐
│    Final Answer          │
└──────────────────────────┘
```

### Key Design Decisions

1. **Max iterations:** Always set a hard limit (3–5 for small models, 5–10 for larger)
2. **Early exit:** If the model says "I have enough information," don't force more loops
3. **State management:** Keep a running summary, not the full history (saves context tokens)
4. **Tool output limits:** Truncate tool outputs before feeding back to the model

## 4. Error Handling and Recovery

Local agents fail more often than cloud agents. Build resilience in.

### Common Failure Modes

| Failure | Detection | Recovery |
|---|---|---|
| Tool call malformed | JSON parse error | Retry with corrected schema in prompt |
| Model ignores constraint | Output doesn't match format | Re-prompt with stronger constraints |
| Infinite loop | Iteration count exceeded | Return best partial result + error note |
| Context overflow | Token count near limit | Summarize history, continue with summary |
| Tool execution error | Exception/timeout | Feed error message back, let model decide |

### Retry Patterns

```python
# Smart retry: don't just repeat, adapt
def agent_retry(task, max_retries=3):
    for attempt in range(max_retries):
        result = agent.run(task)
        if validate(result):
            return result
        # Feed the failure back as context
        task = adapt_prompt(task, last_result=result, error=validation_error)
    return fallback(task)  # Graceful degradation
```

## 5. Context Management

Context is your most expensive resource on constrained systems.

### Strategies

1. **Rolling window:** Keep last N exchanges, discard older ones
2. **Summarization:** Periodically summarize conversation history into a compact form
3. **Selective retention:** Keep only tool calls and their results, discard intermediate reasoning
4. **External memory:** Store facts in a vector DB or key-value store, retrieve as needed
5. **Prompt templating:** Use variable substitution instead of repeating instructions

### Token Budget Example (8K context window)

```
System prompt:        ~500 tokens
Task description:     ~200 tokens
Conversation history: ~4000 tokens (managed)
Tool definitions:     ~800 tokens
Current input:        ~500 tokens
Model output:         ~2000 tokens (reserved)
─────────────────────────────────────
Total:                ~8000 tokens
```

## 6. When to Use Multi-Agent vs Single-Agent

Multi-agent patterns are tempting but expensive.

### Single-Agent Wins When

- Tasks are sequential or simple
- You need minimal latency
- Context window is tight
- Debugging simplicity matters

### Multi-Agent Makes Sense When

- Tasks require genuinely different expertise (code review + writing + testing)
- Parallel execution saves meaningful time
- You need isolation (one agent's failures shouldn't corrupt another's context)
- You have enough VRAM/memory for concurrent model instances

### The "Fake Multi-Agent" Pattern

For constrained systems, simulate multi-agent behavior with a single model:

```python
# Instead of spawning 3 model instances, use prompt switching
def multi_role_agent(task):
    coder_response = model.chat(
        system="You are an expert Python developer.",
        user=task
    )
    reviewer_response = model.chat(
        system="You are a code reviewer. Critique this code:",
        user=coder_response
    )
    final = model.chat(
        system="You are a developer. Revise your code based on feedback:",
        user=f"Original: {coder_response}\nFeedback: {reviewer_response}"
    )
    return final
```

Same sequential latency, but only one model loaded. Quality is often comparable for constrained systems.

## 7. Framework Choices

| Framework | Best For | Local-Friendly? |
|---|---|---|
| **LangGraph** | Complex workflows, state machines | Yes, lightweight |
| **CrewAI** | Multi-agent teams | Heavy, but works |
| **AutoGen** | Conversational agents | Moderate |
| **Semantic Kernel** | Enterprise integration | Yes |
| **Custom (no framework)** | Maximum control, minimum overhead | Best for constrained |
| **LlamaIndex** | RAG + agent combos | Yes |

**Recommendation:** Start with custom loops. Add a framework only when you need features you can't easily build (checkpoints, persistence, complex routing).

## 8. Monitoring and Observability

You can't improve what you can't measure.

### Essential Metrics

- **Token usage per step** — Where is context going?
- **Latency per iteration** — Is the loop too slow?
- **Success rate** — What % of runs produce valid output?
- **Tool call accuracy** — Are tools being called correctly?
- **Cost per task** — Even local has costs (electricity, time)

### Simple Logging Pattern

```python
import json, time

class AgentLogger:
    def __init__(self, log_file="agent.log"):
        self.log_file = log_file
        self.steps = []

    def log_step(self, step, role, content, tokens=None, duration=None):
        self.steps.append({
            "step": step,
            "role": role,
            "tokens": tokens,
            "duration_ms": duration,
            "timestamp": time.time()
        })

    def save(self):
        with open(self.log_file, "a") as f:
            json.dump(self.steps, f)
            f.write("\n")
```

## 9. Security Considerations for Local Agents

Local agents have unique security advantages and risks.

### Advantages
- Data never leaves your machine
- No API key exposure
- Full control over tool execution environment

### Risks
- **Tool execution:** Agents running shell commands, file writes, or network requests on your machine
- **Prompt injection:** Malicious input can still manipulate the agent
- **Sandboxing:** Always run agent tool execution in a constrained environment

### Best Practices
- Never give agents unrestricted shell access
- Use allowlists for file paths and commands
- Validate all tool inputs before execution
- Log all tool calls for auditability

---

*See also: [constrained-systems.md](./constrained-systems.md) for hardware-specific optimization, [prompt-patterns.md](./prompt-patterns.md) for controlling output quality.*
