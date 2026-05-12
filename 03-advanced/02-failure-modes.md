# Chapter 11: When Things Go Wrong

> The 10 most common failure modes, how to recognize them, and recovery strategies.

---

## TL;DR

- Learn the 10 most common failure modes
- Recognize each by its symptoms
- Apply specific recovery strategies
- Know when to blame prompt vs model vs hardware
- Stop loops before they waste hours

---

## Prerequisites

- Completed [Part 1-2](../01-beginner/01-write-code.md) through [Chapter 10](./01-better-prompts.md)
- Have used agents for coding
- Encountered failures and wondered why

---

## The 10 Most Common Failure Modes

### 1. Hallucinated APIs

**Symptom:** Agent uses libraries or functions that don't exist

**Example:**
```python
import pandas as pd
df = pd.read_csv("data.csv")
df.transform_columns()  # This method doesn't exist
```

**Cause:** Model trained on data that included fake examples or made assumptions

**Fix:**
```
Add constraint: "Use ONLY Python standard library. If unsure about a function, use basic Python."
```

**Prevention:**
- Specify allowed libraries explicitly
- Use coding-specific models (Qwen-Coder, DeepSeek-Coder)
- Provide few-shot examples of correct API usage

---

### 2. Infinite Fix Loops

**Symptom:** Agent keeps "fixing" the same issue but makes it worse

**Example:**
```
Iteration 1: Test fails with TypeError
Iteration 2: Fixed TypeError, now NameError
Iteration 3: Fixed NameError, now SyntaxError
Iteration 4: Fixed SyntaxError, now TypeError (back to start)
```

**Cause:** Agent doesn't understand root cause, just patches symptoms

**Fix:**
```
Stop the loop. Ask:
"I've tried fixing this for 4 iterations. Let's step back.
What's the ROOT CAUSE of all these errors?"
```

**Prevention:**
- Set max iterations (5-10)
- Ask for root cause analysis before fixing
- Review errors manually if loop continues

---

### 3. Context Overflow

**Symptom:** Agent forgets earlier decisions or code

**Example:**
```
You: "Add search feature"
Agent: "Here's my plan..."
[10 turns later]
You: "Now add tests"
Agent: "What feature are we adding tests for?"
```

**Cause:** Context window exceeded, old content dropped

**Fix:**
```
Summarize what we've done:
- Added search_logs() function
- Modified CLI to add search command
- All tests passing

Now add tests for the search feature.
```

**Prevention:**
- Summarize every 10-15 turns
- Start new sessions for new features
- Load only relevant files

---

### 4. Format Violations

**Symptom:** Agent ignores output format constraints

**Example:**
```
You: "Return ONLY the code. No explanation."
Agent: "Here's the code you requested:
def foo():
    pass
I hope this helps!"
```

**Cause:** Small models struggle with strict format constraints

**Fix:**
```
Use XML tags instead:
"Wrap your answer in <code> tags."

Or strengthen constraint:
"IMPORTANT: Return ONLY the code. If you include any text, I will reject it."
```

**Prevention:**
- Use XML tags for structured output
- Lower temperature (0.1)
- Repeat critical constraints

---

### 5. Silent Failures

**Symptom:** Code looks correct but doesn't work

**Example:**
```python
def parse_csv(filepath):
    with open(filepath) as f:
        return f.readlines()  # Returns raw lines, not parsed CSV
```

**Cause:** Agent didn't actually implement the logic, just structure

**Fix:**
```
Test the code immediately. If it doesn't work:
"This code doesn't actually parse CSV. It just reads lines.
Implement proper CSV parsing using the csv module."
```

**Prevention:**
- Always test generated code
- Ask for self-review: "Verify the code actually works"
- Use smaller, testable chunks

---

### 6. Over-Engineering

**Symptom:** Agent creates complex solution for simple problem

**Example:**
```python
# You asked for: "A function to add two numbers"
# Agent wrote:
from abc import ABC, abstractmethod
from typing import Protocol, List

class AdditionStrategy(ABC):
    @abstractmethod
    def add(self, a: int, b: int) -> int:
        pass

class IntegerAdder(AdditionStrategy):
    def add(self, a: int, b: int) -> int:
        return a + b

# ... 50 more lines of boilerplate
```

**Cause:** Agent assumes you want "production-ready" code

**Fix:**
```
"Keep it simple. Just write:
def add(a: int, b: int) -> int:
    return a + b

No classes, no strategies, just the function."
```

**Prevention:**
- Specify complexity level: "Simple script, not production code"
- Set constraints: "No classes, no design patterns"
- Use smaller models (they're less prone to over-engineering)

---

### 7. Under-Engineering

**Symptom:** Agent writes code that's too simple, missing error handling

**Example:**
```python
def read_file(filepath):
    with open(filepath) as f:
        return f.read()
# No error handling for missing file, permissions, etc.
```

**Cause:** Agent didn't consider edge cases

**Fix:**
```
"This code will crash if the file doesn't exist.
Add error handling for:
- File not found
- Permission denied
- Empty file
```

**Prevention:**
- Specify edge cases explicitly
- Ask for defensive coding
- Require error handling in constraints

---

### 8. Stale Context

**Symptom:** Agent references code that was changed or deleted

**Example:**
```
You: "Update the parse function to handle UTF-8"
Agent: "I'll modify parse_csv()..."
[But you renamed it to parse_data.py]
```

**Cause:** Agent working from old context

**Fix:**
```
"Before making changes, check what files exist:
list_directory('src/')"
```

**Prevention:**
- Refresh context before major changes
- Use `git status` to see what changed
- Start new session for new features

---

### 9. Constraint Drift

**Symptom:** Agent starts following constraints, then gradually ignores them

**Example:**
```
Turn 1: "Use only standard library" ✓
Turn 5: "Use only standard library" ✓
Turn 10: Uses pandas (violates constraint)
```

**Cause:** Agent prioritizes "helpfulness" over constraints

**Fix:**
```
"Stop. You're using pandas, but I said ONLY standard library.
Rewrite using only Python built-ins."
```

**Prevention:**
- Repeat critical constraints every few turns
- Put constraints in system prompt
- Check output before accepting

---

### 10. Model Limitations

**Symptom:** Agent consistently fails at certain types of tasks

**Example:**
```
- Can't handle regex patterns
- Can't do complex algorithmic logic
- Can't understand multi-file architecture
```

**Cause:** Model is too small for the task

**Fix:**
```
"This task requires more reasoning than this model can handle.
Switch to a larger model (14B or 32B)."
```

**Prevention:**
- Match model size to task complexity
- Use 7B for simple tasks, 14B+ for complex
- Know your model's limits

---

## Blame Assignment Guide

When things go wrong, ask:

### Is it the Prompt?

**Signs:**
- Vague instructions
- Missing constraints
- No examples
- Contradictory requirements

**Fix:** Improve the prompt

---

### Is it the Model?

**Signs:**
- Consistent failures on specific task types
- Hallucinations despite constraints
- Can't follow simple instructions

**Fix:** Use a larger model or different model family

---

### Is it the Hardware?

**Signs:**
- Slow inference causing timeouts
- Out of memory errors
- Context truncated

**Fix:** Use smaller model, reduce context, or upgrade hardware

---

### Is it the Tools?

**Signs:**
- Tool calls fail
- File operations don't work
- Commands produce unexpected output

**Fix:** Check tool implementation, permissions, paths

---

## Recovery Strategies

### Strategy 1: The Hard Reset

**When:** Loop stuck for 5+ iterations

**Action:**
```
1. Stop the agent
2. Start fresh session
3. Provide summary of what worked
4. State the problem clearly
5. Try different approach
```

---

### Strategy 2: The Manual Intervention

**When:** Agent going in wrong direction

**Action:**
```
1. Interrupt the agent
2. Show correct solution direction
3. Ask agent to follow that approach
4. Monitor closely
```

---

### Strategy 3: The Model Switch

**When:** Consistent failures despite good prompts

**Action:**
```
1. Switch to larger model
2. Or switch model family (Qwen → DeepSeek)
3. Test with same prompt
4. Compare results
```

---

### Strategy 4: The Task Decomposition

**When:** Task too complex for one prompt

**Action:**
```
1. Break task into smaller steps
2. Complete each step separately
3. Verify each step before proceeding
4. Combine results at the end
```

---

## Exercise: Diagnose Failures

**Task:** Intentionally trigger failures and practice recovery.

**Steps:**
1. Use a bad prompt, observe failure
2. Use a good prompt with small model, observe limitations
3. Create context overflow, practice summarization
4. Trigger infinite loop, practice hard reset
5. Document what you learned

**Reflection:**
- Which failure modes did you see?
- Which recovery strategies worked?
- What will you do differently next time?

---

## On Your Hardware

### Small Models (1.5B-3B)

**Most common failures:**
- Hallucinated APIs
- Format violations
- Model limitations

**Strategy:** Use more examples, simpler tasks, accept more iteration

### Medium Models (7B-14B)

**Most common failures:**
- Constraint drift
- Silent failures
- Over-engineering

**Strategy:** Repeat constraints, test code, specify complexity

### Large Models (32B+)

**Most common failures:**
- Context overflow
- Stale context
- Rare: model limitations

**Strategy:** Manage context, refresh frequently

---

## Quick Reference: Failure Mode Checklist

When something goes wrong:

- [ ] Is the prompt clear and specific?
- [ ] Are constraints explicit and repeated?
- [ ] Are there examples for small models?
- [ ] Is the model size appropriate?
- [ ] Is context still valid?
- [ ] Are tools working correctly?
- [ ] Have I hit max iterations?
- [ ] Should I try a different approach?

---

## Next Steps

- [Chapter 12: Greenfield vs Brownfield](./03-greenfield-brownfield.md) — Different contexts
- [Chapter 11 Reference](../reference/troubleshooting.md) — Detailed troubleshooting

---

## Further Reading

- [Reference: Troubleshooting](../reference/troubleshooting.md) — Symptom → fix lookup
- [Reference: Models](../reference/models.md) — Model capabilities
- [WORKS.md](../WORKS.md) — Quick fixes
