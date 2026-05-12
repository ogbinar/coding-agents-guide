# Chapter 10: Better Prompts

> System prompts, few-shot examples, structured output, and temperature settings that actually work.

---

## TL;DR

- System prompts define the agent's role and constraints
- Few-shot examples beat abstract instructions for small models
- Use XML tags instead of JSON for reliable structured output
- Temperature 0.1 for code, 0.7 for creative tasks
- Repeat critical constraints in both system and user prompts

---

## Prerequisites

- Completed [Part 1-2](../01-beginner/01-write-code.md) through [Chapter 9](../02-understand/04-what-not-to-build.md)
- Have used agents for coding
- Noticed that sometimes the output quality varies

---

## Why Prompts Matter

**Same model, different prompts:**

**Bad prompt:**
```
Write some code to parse CSV.
```

**Result:**
```python
def parse_csv(f):  # No type hints, no error handling
    lines = open(f).readlines()
    return lines  # Doesn't even parse CSV properly
```

**Good prompt:**
```
Write a Python function that parses a CSV file.

Requirements:
- Type hints on all parameters and return value
- Docstring with description, params, returns, examples
- Handle edge cases: empty file, missing columns
- Use only standard library
- Follow PEP 8 style

Return ONLY the code.
```

**Result:**
```python
import csv
from typing import List, Dict

def parse_csv(filepath: str, delimiter: str = ',') -> List[Dict[str, str]]:
    """
    Parse a CSV file and return a list of dictionaries.
    
    Args:
        filepath: Path to the CSV file
        delimiter: Field delimiter (default: comma)
        
    Returns:
        List of dictionaries, one per row
        
    Examples:
        >>> parse_csv("data.csv")
        [{'name': 'Alice', 'age': '30'}]
    """
    if not filepath:
        raise ValueError("Filepath cannot be empty")
    
    try:
        with open(filepath, 'r', newline='', encoding='utf-8') as f:
            reader = csv.DictReader(f, delimiter=delimiter)
            return list(reader)
    except FileNotFoundError:
        raise FileNotFoundError(f"File not found: {filepath}")
    except csv.Error as e:
        raise ValueError(f"CSV parsing error: {e}")
```

**Same model (qwen2.5-coder:7b). Completely different results.**

---

## System Prompts: Anatomy of a Great One

### System Prompt Structure

```
ROLE: {Who the model is}

TASK: {What it should do}

CONSTRAINTS:
- {Constraint 1}
- {Constraint 2}
- {What NOT to do}

OUTPUT FORMAT:
{How to structure output}

EXAMPLES:
{2-4 few-shot examples}
```

### Example: Coding Agent System Prompt

```
You are an expert Python developer. Write clean, production-ready code.

TASK:
Generate Python code based on user requirements. Always produce working, tested code.

CONSTRAINTS:
- Use only Python standard library unless specified
- Follow PEP 8 style guidelines
- Include type hints on ALL parameters and return values
- Include docstrings with description, params, returns, examples
- Handle edge cases explicitly
- Write defensive code (validate inputs, handle errors)

DO NOT:
- Use external packages unless explicitly requested
- Leave TODOs or stubs
- Return incomplete code
- Include markdown fences around code

OUTPUT FORMAT:
Return ONLY the code. No explanation, no commentary.

EXAMPLES:
User: Write a function to calculate factorial
Assistant:
def factorial(n: int) -> int:
    """
    Calculate the factorial of n.
    
    Args:
        n: Non-negative integer
        
    Returns:
        n! (factorial of n)
        
    Examples:
        >>> factorial(5)
        120
        >>> factorial(0)
        1
    """
    if n < 0:
        raise ValueError("Factorial not defined for negative numbers")
    
    result = 1
    for i in range(2, n + 1):
        result *= i
    return result
```

### Why This Works

1. **Clear role** — "Expert Python developer"
2. **Specific task** — "Generate Python code"
3. **Explicit constraints** — What to do and what NOT to do
4. **Output format** — "ONLY the code"
5. **Examples** — Shows exactly what you want

---

## Few-Shot Examples: Why Examples Beat Instructions

### The Problem with Abstract Instructions

**Prompt:**
```
Write clean, production-ready code with proper error handling.
```

**Model interpretation:**
- "Clean" = ?
- "Production-ready" = ?
- "Proper error handling" = ?

**Result:** Inconsistent, often wrong

### The Power of Examples

**Prompt:**
```
Write clean, production-ready code with proper error handling.

Example 1:
User: Write a function to divide numbers
Assistant:
def divide(a: float, b: float) -> float:
    """Divide a by b."""
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b

Example 2:
User: Write a function to parse JSON
Assistant:
def parse_json(text: str) -> dict:
    """Parse JSON string."""
    if not text or not text.strip():
        raise ValueError("Input cannot be empty")
    try:
        return json.loads(text)
    except json.JSONDecodeError as e:
        raise ValueError(f"Invalid JSON: {e}")

Now write: A function to parse CSV
```

**Result:** Follows the pattern (validation, error handling, docstrings)

### How Many Examples?

| Model Size | Examples Needed |
|---|---|
| **1.5B-3B** | 3-5 examples |
| **7B-14B** | 2-3 examples |
| **32B+** | 1-2 examples |

**Rule:** Smaller models need more examples.

---

## Structured Output: Getting Reliable Formats

### The JSON Problem

**Prompt:**
```
Return your answer as JSON:
{"function": "name", "code": "implementation"}
```

**Small model output:**
```json
{"function": "parse_csv", "code": "def parse_csv..."}  ← Good
```

**Or:**
```
Here's the JSON you asked for:
{"function": "parse_csv", "code": "def parse_csv..."}  ← Extra text breaks parsing
```

**Or:**
```
{function: "parse_csv", code: "def parse_csv..."}  ← Invalid JSON (missing quotes)
```

### The XML Solution

**Prompt:**
```
Return your answer using XML tags:
<function>name</function>
<code>implementation</code>
```

**Small model output:**
```xml
<function>parse_csv</function>
<code>def parse_csv(filepath: str) -> List[Dict]:
    """Parse CSV file."""
    ...
</code>
```

**Much more reliable.**

### XML Tag Pattern

```
Wrap your answer in these tags:

<plan>
Your approach and reasoning
</plan>

<code>
The actual code implementation
</code>

<tests>
Test cases
</tests>
```

**Extraction:**
```python
import re

def extract_tags(text: str) -> dict:
    """Extract content from XML tags."""
    result = {}
    for tag in ['plan', 'code', 'tests']:
        match = re.search(f'<{tag}>(.*?)</{tag}>', text, re.DOTALL)
        if match:
            result[tag] = match.group(1).strip()
    return result
```

---

## Temperature and Sampling

### What Temperature Does

**Temperature controls randomness:**

```
Temperature = 0.0
████████████████████ Deterministic, always same output

Temperature = 0.1
████████████████░░░░ Very focused, minimal variation

Temperature = 0.5
██████████░░░░░░░░░░ Balanced, some creativity

Temperature = 0.7
██████░░░░░░░░░░░░░░ Creative, more variation

Temperature = 1.0
██░░░░░░░░░░░░░░░░░░ Highly creative, random
```

### Temperature for Coding

| Task | Temperature | Why |
|---|---|---|
| **Code generation** | 0.1-0.2 | Deterministic, reproducible |
| **Debugging** | 0.1 | Precise fixes |
| **Test generation** | 0.2 | Some variety in test cases |
| **Code review** | 0.3 | Consider multiple perspectives |
| **Architecture design** | 0.5 | Creative solutions |
| **Refactoring ideas** | 0.5 | Explore options |

### Setting Temperature

**Ollama:**
```bash
ollama run qwen2.5-coder:7b --temperature 0.1
```

**Programmatic:**
```python
from ollama import chat

response = chat(
    model='qwen2.5-coder:7b',
    messages=[...],
    options={'temperature': 0.1}
)
```

---

## Prompt Patterns That Work

### Pattern 1: Constraints First

```
CONSTRAINTS:
- Use only standard library
- Type hints on ALL parameters
- Handle these edge cases: empty, None, negative

Write a function that {task}.
```

**Why:** Constraints upfront set expectations before the task.

### Pattern 2: Repeat Critical Rules

```
System: "Use only standard library. No external packages."

User: "Write a function... Remember, use ONLY standard library."
```

**Why:** Small models forget. Repetition helps.

### Pattern 3: Negative Constraints

```
DO NOT:
- Use external packages
- Leave TODOs
- Include markdown fences
- Add explanations
```

**Why:** Explicitly saying what NOT to do prevents common mistakes.

### Pattern 4: Step-by-Step

```
Before writing code:
1. Outline your approach
2. Identify edge cases
3. Plan error handling

Then write the code.
```

**Why:** Forces the model to think before acting.

### Pattern 5: Self-Review

```
Write a function that {task}.

Before returning the code, verify:
- All imports are valid
- Syntax is correct
- Type hints are complete
- Edge cases are handled

If you find issues, fix them first.
```

**Why:** Model catches its own mistakes.

---

## Exercise: Improve a Prompt

**Task:** Take a bad prompt and make it work.

**Bad prompt:**
```
Write some code to handle user authentication.
```

**Steps:**
1. Try the bad prompt, note the output
2. Add constraints (language, libraries, security)
3. Add output format requirements
4. Add examples
5. Try again, compare results

**Good prompt example:**
```
Write a Python function for user authentication.

Requirements:
- Use hashlib for password hashing (not MD5 or SHA1)
- Include type hints on all parameters
- Handle edge cases: empty password, None username
- Include docstring with security notes
- Use only standard library

Example:
def hash_password(password: str) -> str:
    """Hash a password using SHA-256."""
    import hashlib
    return hashlib.sha256(password.encode()).hexdigest()

Now write: A function to authenticate a user
```

---

## On Your Hardware

### Small Models (1.5B-3B)

- Need 3-5 examples
- Temperature 0.1-0.2
- Repeat constraints multiple times
- Use XML tags for structure
- Keep prompts short and focused

### Medium Models (7B-14B)

- Need 2-3 examples
- Temperature 0.1-0.3
- Repeat critical constraints
- Use XML or JSON for structure
- Can handle moderate prompt length

### Large Models (32B+)

- Need 1-2 examples
- Temperature 0.2-0.5
- Constraints mentioned once
- Any format works
- Can handle long, detailed prompts

---

## Common Pitfalls

### Pitfall 1: Vague Constraints

**Bad:**
```
Write clean code.
```

**Good:**
```
Write code that:
- Follows PEP 8 style
- Includes type hints on all functions
- Has docstrings with examples
- Handles edge cases: empty input, None values
```

### Pitfall 2: Too Many Constraints

**Bad:**
```
Write code that:
- Follows PEP 8
- Uses Google-style docstrings
- Has 100% test coverage
- Is O(n) complexity
- Uses dataclasses
- Includes logging
- Has type hints
- Handles 15 edge cases
- Is thread-safe
- Is async-compatible
...
```

**Good:**
```
Write code that:
- Follows PEP 8
- Includes type hints
- Handles these edge cases: {specific}
```

**Rule:** 3-5 constraints max. Be specific, not exhaustive.

### Pitfall 3: Contradictory Instructions

**Bad:**
```
Return ONLY the code.
Also, explain your approach.
```

**Good:**
```
Return ONLY the code. No explanation.
```

**Or:**
```
First explain your approach.
Then return ONLY the code.
```

---

## Quick Reference: Prompt Checklist

Before sending a prompt, check:

- [ ] Is the role clear? ("You are a Python expert")
- [ ] Is the task specific? ("Write a function that...")
- [ ] Are constraints explicit? ("Use only standard library")
- [ ] Is output format specified? ("Return ONLY the code")
- [ ] Are there examples? (2-3 for 7B models)
- [ ] Are critical rules repeated? (if small model)
- [ ] Is temperature appropriate? (0.1 for code)
- [ ] Are negative constraints included? ("DO NOT...")

---

## Next Steps

- [Chapter 11: When Things Go Wrong](./02-failure-modes.md) — Handle failures
- [Chapter 10 Reference](../reference/prompt-templates.md) — Copy-paste templates

---

## Further Reading

- [Reference: Prompt Templates](../reference/prompt-templates.md) — Ready-to-use prompts
- [Reference: Troubleshooting](../reference/troubleshooting.md) — Prompt issues
- [WORKS.md](../WORKS.md) — Quick prompt cheat sheet
