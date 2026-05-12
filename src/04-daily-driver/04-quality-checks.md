# Chapter 19: Quality Checks

> Simple heuristics, the 5-minute test, when to upgrade model vs fix prompts, and regression detection for your personal workflow.

---

## TL;DR

- **Simple heuristics:** Does it run? Do tests pass? Does it meet requirements?
- **The 5-minute test:** Give agent same task 3 times, compare consistency
- **Upgrade vs fix prompts:** Know which lever to pull
- **Red flags:** When your setup is fundamentally broken
- **Regression detection:** Catch quality degradation early

---

## Why Quality Checks Matter

Without quality checks, you're flying blind:

**Bad setup, no checks:**
- Agent produces broken code daily
- You spend hours debugging
- You blame yourself, not the agent
- You quit thinking "AI coding doesn't work"

**Good setup, with checks:**
- You catch quality issues immediately
- You know when to fix prompts vs upgrade model
- You maintain confidence in your workflow
- You continuously improve

**Key insight:** Quality checks save you from wasting days on a broken setup.

---

## Simple Heuristics: The 4-Question Test

After every significant agent interaction, ask:

### Question 1: Does It Run?

```bash
# Can you execute the code without errors?
python my_script.py

# If it crashes → Quality issue
# If it runs → Continue checking
```

**Pass:** Code executes without errors
**Fail:** Syntax errors, import errors, runtime crashes

### Question 2: Do Tests Pass?

```bash
# Run the test suite
pytest

# If tests fail → Quality issue
# If tests pass → Continue checking
```

**Pass:** All tests pass (or no tests exist yet)
**Fail:** Tests fail, or agent removed tests

### Question 3: Does It Meet Requirements?

```
Original requirement: "Parse CSV and return list of dicts"
Agent output: [code that does exactly that]

✓ Meets requirements
```

**Pass:** Output matches what you asked for
**Fail:** Missing features, wrong behavior, incomplete

### Question 4: Is It Maintainable?

```
# Quick code review
- Are there type hints?
- Is there basic documentation?
- Is the code readable?
- Are there obvious anti-patterns?

Mostly yes → Maintainable
Mostly no → Quality issue
```

**Pass:** Code is readable and maintainable
**Fail:** Spaghetti code, no structure, impossible to modify

### The 4-Question Decision Tree

```
Does it run?
├── No → Fix: Regenerate with error feedback
│
└── Yes → Do tests pass?
    ├── No → Fix: Ask agent to fix tests or regenerate
    │
    └── Yes → Meets requirements?
        ├── No → Fix: Clarify requirements, regenerate
        │
        └── Yes → Maintainable?
            ├── No → Fix: Ask for cleaner code
            │
            └── Yes → Quality acceptable ✓
```

---

## The 5-Minute Test: Consistency Check

### The Problem

You ask the agent to do something:
```
"Write a function to parse CSV"
```

First try: Works perfectly.
Second try: Hallucinates APIs.
Third try: Missing error handling.

**Inconsistent quality = Broken setup**

### The 5-Minute Test

**Run this test weekly (or when you suspect quality issues):**

**Step 1: Pick a Standard Task**

Choose something representative of your work:
```
"Write a function to parse CSV and return list of dicts"
"Create a Flask endpoint that returns JSON"
"Write pytest tests for this function"
```

**Step 2: Run 3 Times**

```
# Try 1
ollama run qwen2.5-coder:7b "Write a function to parse CSV..."

# Try 2
ollama run qwen2.5-coder:7b "Write a function to parse CSV..."

# Try 3
ollama run qwen2.5-coder:7b "Write a function to parse CSV..."
```

**Step 3: Compare Results**

Use the 4-Question Test on each:

| Try | Runs? | Tests Pass? | Meets Requirements? | Maintainable? |
|---|---|---|---|---|
| 1 | ✓ | ✓ | ✓ | ✓ |
| 2 | ✓ | ✗ | ✓ | ✗ |
| 3 | ✓ | ✓ | ✓ | ✓ |

### Interpreting Results

**Consistent quality (Good):**
```
Try 1: ✓✓✓✓
Try 2: ✓✓✓✓
Try 3: ✓✓✓✓

All 3 produce working, maintainable code.
→ Your setup is reliable.
```

**Mostly consistent (Acceptable):**
```
Try 1: ✓✓✓✓
Try 2: ✓✓✓✗ (minor style issues)
Try 3: ✓✓✓✓

2/3 perfect, 1/3 minor issues.
→ Your setup is mostly reliable.
→ Consider improving prompts for consistency.
```

**Inconsistent (Bad):**
```
Try 1: ✓✓✓✓
Try 2: ✗✗✗✗ (completely broken)
Try 3: ✓✓✗✗ (partial)

Only 1/3 works well.
→ Your setup is unreliable.
→ Fix prompts or upgrade model.
```

**Never works (Broken):**
```
Try 1: ✗✗✗✗
Try 2: ✗✗✗✗
Try 3: ✗✗✗✗

None work.
→ Your setup is fundamentally broken.
→ Revisit everything: model, prompts, hardware.
```

### Exercise: Run the 5-Minute Test

**Task:** Test your agent's consistency.

**Steps:**
1. Pick a standard task (e.g., "Write a function to parse CSV")
2. Run it 3 times with the same prompt
3. Apply the 4-Question Test to each
4. Compare results

**Timebox:** 10 minutes

**Success criteria:** You know if your setup is reliable or broken.

---

## When to Upgrade Model vs Fix Prompts

### The Decision Matrix

When quality is poor, you have two levers:
1. **Fix prompts** (free, immediate)
2. **Upgrade model** (costs money, may not help)

**Most of the time, fix prompts first.**

### Fix Prompts If

| Symptom | Likely Cause | Fix |
|---|---|---|
| **Agent ignores constraints** | Prompt unclear | Repeat constraints in system + user prompt |
| **Output format wrong** | Format not specified | Use XML tags or JSON schema |
| **Missing edge cases** | Not mentioned | Explicitly list edge cases |
| **Inconsistent quality** | Prompt too vague | Add examples (few-shot) |
| **Hallucinates APIs** | No boundaries | Add "Use only standard library" |
| **Code doesn't run** | No verification step | Add "Test and fix errors" |
| **Style inconsistent** | No style guide | Reference style guide in prompt |

**Prompt fix template:**
```
# Before (vague)
"Write a function to parse CSV."

# After (specific)
"Write a Python function that parses CSV files.

Requirements:
- Use the csv module from standard library
- Return a list of dicts (one dict per row)
- Handle missing files gracefully (return empty list)
- Handle empty files (return empty list)
- Include type hints
- Include docstring
- Handle edge cases: empty rows, extra columns

Return ONLY the code."
```

### Upgrade Model If

| Symptom | Likely Cause | Fix |
|---|---|---|
| **Can't handle complex reasoning** | Model too small | Upgrade to larger model |
| **Consistently hallucinates** | Model lacks knowledge | Upgrade to better-trained model |
| **Can't follow simple instructions** | Model too small | Upgrade to larger model |
| **Quality ceiling reached** | Model capability limit | Upgrade to larger/better model |
| **Poor at your domain** | Model not specialized | Try domain-specific model |

**Upgrade path:**
```
1.5B → 3B → 7B → 14B → 32B → 70B

Each step up:
- Better reasoning
- Better instruction following
- Better code quality
- More memory required
- Slower inference
```

### The Fix-First Rule

**Always try these before upgrading:**

1. **Clarify the prompt** (be more specific)
2. **Add examples** (few-shot learning)
3. **Use structured output** (XML, JSON)
4. **Add verification steps** (test, fix errors)
5. **Repeat constraints** (system + user prompt)

**Only upgrade if:**
- You've tried all 5 fixes
- Quality is still poor
- You have the hardware budget

### Example: Fix vs Upgrade Decision

**Problem:** Agent keeps hallucinating non-existent Python libraries.

**Attempt 1: Clarify prompt**
```
"Write a function using only Python standard library."
```
**Result:** Still hallucinates.

**Attempt 2: Add examples**
```
"Write a function. Use only standard library like:
- csv (for CSV parsing)
- json (for JSON parsing)
- os (for file operations)
- pathlib (for paths)

Do NOT use: pandas, numpy, requests (unless I ask)"
```
**Result:** Better, but still occasional hallucinations.

**Attempt 3: Structured output**
```
"Return your answer in this format:
<imports>
# List imports here
</imports>

<code>
# Code here
</code>

Only use standard library. List all imports in <imports> section."
```
**Result:** Much better, no hallucinations.

**Decision:** Prompt fix worked. No need to upgrade model.

---

## Red Flags: When Your Setup Is Broken

### Critical Red Flags

**If you see these, your setup is fundamentally broken:**

❌ **Code never runs without errors**
- Every piece of generated code has bugs
- You spend more time fixing than coding
- **Fix:** Try different model, simplify prompts

❌ **Tests always fail**
- Agent generates code that breaks tests
- Agent removes tests instead of fixing code
- **Fix:** Add "tests must pass" to every prompt

❌ **Agent can't follow basic instructions**
- You say "use csv module", agent uses pandas
- You say "return list", agent returns dict
- **Fix:** Use structured output, add examples

❌ **Same prompt gives different results each time**
- Try 1: Works perfectly
- Try 2: Completely broken
- Try 3: Partially works
- **Fix:** Run 5-minute test, fix prompts or upgrade

❌ **Agent "forgets" after 2-3 turns**
- Conversation starts well
- After 3 messages, agent loses context
- **Fix:** Reduce context, use sessions, upgrade model

❌ **Consistently produces insecure code**
- SQL injection vulnerabilities
- Hardcoded secrets
- No input validation
- **Fix:** Add security requirements to prompts, consider larger model

### Warning Signs

**These aren't critical, but indicate room for improvement:**

⚠️ **Slow speeds (<5 tokens/sec)**
- **Fix:** Tune threads, reduce context, upgrade hardware

⚠️ **Occasional hallucinations**
- **Fix:** Add "use only known libraries" to prompts

⚠️ **Inconsistent style**
- **Fix:** Add style guide reference to prompts

⚠️ **Missing edge cases**
- **Fix:** Explicitly list edge cases in prompts

### The Red Flag Checklist

Run this checklist weekly:

```
[ ] Code runs without errors (most of the time)
[ ] Tests pass (when they exist)
[ ] Agent follows basic instructions
[ ] Results are consistent across tries
[ ] Agent maintains context through conversations
[ ] Speed is acceptable (>5 tokens/sec)
[ ] No security vulnerabilities in generated code

If 0-2 boxes checked → Your setup is broken. Fix immediately.
If 3-4 boxes checked → Your setup is okay. Improve prompts.
If 5-6 boxes checked → Your setup is good. Minor tweaks.
```

---

## Regression Detection

### The Problem

Your setup works great for weeks. Then:
- Quality slowly degrades
- You don't notice until it's bad
- You waste days before realizing

**Solution:** Regular regression checks.

### Weekly Regression Check

**Every week, run this 10-minute check:**

**Step 1: Run Standard Tasks**

Pick 3 tasks you do regularly:
```
Task 1: "Write a function to parse CSV"
Task 2: "Write pytest tests for this function"
Task 3: "Review this code for bugs"
```

**Step 2: Score Each Task**

Use the 4-Question Test:
```
Task 1: ✓✓✓✓ (4/4)
Task 2: ✓✓✓✗ (3/4)
Task 3: ✓✓✓✓ (4/4)

Total: 11/12 = 92%
```

**Step 3: Compare to Baseline**

Last week's score: 95%
This week's score: 92%

**Change:** -3%

**Step 4: Decide Action**

| Change | Action |
|---|---|
| **0-5%** | Normal variation, no action |
| **5-10%** | Investigate, adjust prompts |
| **10-20%** | Significant regression, fix prompts |
| **20%+** | Major regression, try different model |

### Automated Regression Testing

**Create a test suite:**

```python
# test_agent_quality.py
import subprocess

def run_agent(prompt):
    """Run agent with prompt, return output."""
    result = subprocess.run(
        ["ollama", "run", "qwen2.5-coder:7b"],
        input=prompt,
        capture_output=True,
        text=True
    )
    return result.stdout

def test_csv_parsing():
    """Test agent can write CSV parser."""
    prompt = "Write a function to parse CSV..."
    code = run_agent(prompt)
    
    # Check code runs
    assert "import csv" in code
    assert "def " in code
    # Add more checks

def test_test_generation():
    """Test agent can write tests."""
    prompt = "Write pytest tests for this function..."
    tests = run_agent(prompt)
    
    # Check tests are valid
    assert "def test_" in tests
    assert "assert " in tests

if __name__ == "__main__":
    test_csv_parsing()
    test_test_generation()
    print("All quality checks passed!")
```

**Run weekly:**
```bash
python test_agent_quality.py
```

---

## Exercise: Quality Check Routine

### Exercise 1: Run the 5-Minute Test (10 min)

**Task:** Test consistency on a Project A task.

**Steps:**
1. Pick a standard Project A task (e.g., "add a filter by date to the log parser")
2. Run 3 times with different models or same model
3. Score each with 4-Question Test
4. Determine if setup is reliable

### Exercise 2: Fix a Prompt Problem (15 min)

**Task:** Improve a prompt that's not working for Project A/B/C.

**Steps:**
1. Identify a failing pattern from your Project A/B/C work
2. Apply fix-first rules (clarify, add examples, structure)
3. Test the improved prompt
4. Compare before/after

### Exercise 3: Run Regression Check (10 min)

**Task:** Check for quality degradation on your Project A/B/C tasks.

**Steps:**
1. Run 3 standard tasks from your projects
2. Score each with 4-Question Test
3. Compare to last week's score
4. Take action if needed

**Total time:** 35 minutes

---

## Quality Checklist

### Daily

```
[ ] Code runs without errors
[ ] Tests pass
[ ] Meets requirements
```

### Weekly

```
[ ] Run 5-minute test (consistency check)
[ ] Run regression check
[ ] Update prompt library with improvements
[ ] Note any red flags
```

### Monthly

```
[ ] Review overall quality trends
[ ] Decide if model upgrade needed
[ ] Update evaluation criteria
[ ] Document new failure modes and fixes
```

---

## On Your Hardware

### Small Models (1.5B-3B)

- 5-minute test is essential — inconsistency is common
- Expect 40-60% consistency score (lower than larger models)
- Focus on prompt improvement, not model upgrades
- If code doesn't run consistently, simplify your prompts

### Medium Models (7B-14B)

- Expect 60-80% consistency score
- Good balance of quality and speed
- Upgrade model only if consistency <60%

### Large Models (32B+)

- Expect 80-95% consistency score
- Quality issues usually prompt-related, not model-related
- Focus on prompt library refinement

---

## Next Steps

- [Chapter 20: Trust and Safety](./05-trust-and-safety.md) - Learn to use agents safely long-term
- [Chapter 19 Reference](../reference/troubleshooting.md) - Symptom → cause → fix lookup
