# Chapter 14: Patterns That Work

> Fake multi-agent, self-correction, multi-pass refinement, and when real multi-agent makes sense.

---

## TL;DR

- **Fake multi-agent:** Role-switching with a single model (better than real multi-agent for most cases)
- **Self-correction:** Compile-and-fix loops that iterate until green
- **Multi-pass refinement:** Working → Quality → Optimization passes
- **Real multi-agent:** Rarely needed for beginners, high coordination overhead

---

## Why Patterns Matter

Random prompting gives random results. **Patterns give consistent results.**

These patterns have been proven across thousands of coding sessions. They work because they:
1. **Structure the interaction** (clear phases, clear goals)
2. **Leverage model strengths** (iteration, self-correction)
3. **Avoid common pitfalls** (hallucination, incomplete code)

**Key insight:** You don't need to invent new patterns. Use these proven ones.

---

## Pattern 1: Fake Multi-Agent (Role-Switching)

### The Problem with Real Multi-Agent

**Real multi-agent:** Multiple AI agents working together.

```
Agent A (Coder) → writes code → Agent B (Reviewer) → reviews → Agent C (Revisor) → fixes
```

**Problems:**
- **Coordination overhead:** Agents need to communicate
- **Context loss:** Each agent sees different context
- **Slower:** More turns, more latency
- **Complex:** Hard to debug when things go wrong
- **Expensive:** More API calls, more tokens

**For 99% of coding tasks, you don't need real multi-agent.**

### The Solution: Fake Multi-Agent

**Single model, multiple roles.**

You manually switch the model's "persona" within a single conversation:

```
# Phase 1: Coder
You are an expert Python developer. Write a function that parses CSV files.
Requirements: {list}
Return only the code.

# Phase 2: Reviewer  
You are a senior code reviewer. Review this code for:
- Bugs
- Security issues
- Performance problems
- Style violations

Code: {paste code from Phase 1}

Return findings in this format:
🔴 Critical: {issues}
🟡 Warning: {issues}
🔵 Suggestion: {issues}

# Phase 3: Revisor
You are an expert developer. Fix the issues found in the review.

Review findings: {paste review from Phase 2}
Original code: {paste code from Phase 1}

Return only the corrected code.
```

### Why Fake Multi-Agent Works Better

| Aspect | Real Multi-Agent | Fake Multi-Agent |
|---|---|---|
| **Context** | Split across agents | Maintained throughout |
| **Speed** | Slower (more turns) | Faster (same conversation) |
| **Cost** | Higher (more calls) | Lower (same call) |
| **Debugging** | Hard (which agent failed?) | Easy (you control the flow) |
| **Quality** | Variable | Consistent |

### Fake Multi-Agent Prompt Template

```
# ROLE: Coder
You are an expert {language} developer.

Task: {describe task}

Requirements:
- {requirement 1}
- {requirement 2}
- {requirement 3}

Return ONLY the code. No explanations.

---

# ROLE: Reviewer
You are a senior {language} code reviewer.

Review this code for:
- Bugs and logic errors
- Security vulnerabilities
- Performance issues
- Style violations
- Edge cases not handled

Code:
{paste code from Coder phase}

Return findings in this format:
🔴 Critical: {line number} - {description} - {fix}
🟡 Warning: {line number} - {description} - {fix}
🔵 Suggestion: {line number} - {description} - {fix}

If no issues, say "No issues found."

---

# ROLE: Revisor
You are an expert {language} developer.

Fix all issues found in the review.

Original code:
{paste code from Coder phase}

Review findings:
{paste review from Reviewer phase}

Return ONLY the corrected code. No explanations.
```

### Example: Fake Multi-Agent in Action

**Task:** Write a function to validate email addresses.

**Phase 1 - Coder:**
```
You are an expert Python developer. Write a function that validates email addresses.

Requirements:
- Use regex for validation
- Handle edge cases (missing @, invalid domains, etc.)
- Return True/False
- Include docstring

Return ONLY the code.
```

**Agent outputs:**
```python
import re

def validate_email(email):
    """Validate email address using regex."""
    pattern = r'^[a-zA-Z0-9]+@[a-zA-Z0-9]+\.[a-zA-Z]+$'
    return bool(re.match(pattern, email))
```

**Phase 2 - Reviewer:**
```
You are a senior Python code reviewer.

Review this code for bugs, security, performance, style, edge cases.

Code:
import re

def validate_email(email):
    """Validate email address using regex."""
    pattern = r'^[a-zA-Z0-9]+@[a-zA-Z0-9]+\.[a-zA-Z]+$'
    return bool(re.match(pattern, email))

Return findings in format: 🔴 Critical, 🟡 Warning, 🔵 Suggestion
```

**Agent outputs:**
```
🔴 Critical: Line 4 - Regex doesn't handle quoted emails or subdomains - Use RFC 5322 pattern
🟡 Warning: Line 4 - Regex is too permissive (allows +++, etc.) - Add stricter validation
🟡 Warning: No input validation - Check if email is None or empty
🔵 Suggestion: Line 3 - Docstring should describe return value and exceptions
```

**Phase 3 - Revisor:**
```
You are an expert Python developer. Fix all issues found.

Original code:
{paste code}

Review findings:
{paste review}

Return ONLY the corrected code.
```

**Agent outputs:**
```python
import re

def validate_email(email: str) -> bool:
    """
    Validate email address using regex.
    
    Args:
        email: Email address to validate
        
    Returns:
        True if valid, False otherwise
        
    Raises:
        TypeError: If email is not a string
    """
    if not isinstance(email, str):
        raise TypeError("Email must be a string")
    
    if not email or email.strip() == "":
        return False
    
    # RFC 5322 simplified pattern
    pattern = r'^[a-zA-Z0-9.!#$%&\'*+/=?^_`{|}~-]+@[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?(?:\.[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)*$'
    return bool(re.match(pattern, email))
```

### When to Use Fake Multi-Agent

**Use when:**
- Code quality matters (production code, open-source)
- You've had issues with bugs before
- You're learning best practices
- The task is complex

**Don't use when:**
- Quick prototype or experiment
- Code is throwaway
- You're in a hurry
- Task is trivial

---

## Pattern 2: Self-Correction (Compile-and-Fix)

### The Concept

Instead of asking for perfect code on the first try, let the agent **iterate to correctness**:

```
Generate → Test → Fail → Fix → Test → Fail → Fix → ... → Pass
```

### Manual Self-Correction

**Step 1: Generate**
```
Write a Python function that {task}.
```

**Step 2: Test**
```
# Run the code
python test.py

# Get error
TypeError: unsupported operand type(s) for +: 'int' and 'str'
```

**Step 3: Feed Error Back**
```
This code produces an error:

Code:
{paste code}

Error:
TypeError: unsupported operand type(s) for +: 'int' and 'str'

Fix the bug and return only the corrected code.
```

**Step 4: Repeat Until Green**
```
# Run again
python test.py

# Still failing?
ValueError: invalid literal for int() with base 10: 'abc'

# Feed back again
The previous fix introduced a new error:
{paste error}

Fix this new error. Return only the corrected code.
```

### Automated Self-Correction (With Aider/Continue)

Tools like Aider and Continue can automate this:

```bash
# Aider with auto-fix
aider --auto-commits --dirty-commits src/

# Agent will:
# 1. Make changes
# 2. Run tests
# 3. Fix errors automatically
# 4. Repeat until tests pass
```

### Self-Correction Prompt Template

```
Write code for {task}, then:

1. Run the code and check for errors
2. If errors occur, fix them automatically
3. Run tests (if any) and fix failures
4. Repeat until the code works without errors
5. Return only the final, working code

Do not show intermediate versions. Only show the final working code.

Task: {describe task}
```

### Example: Self-Correction for Data Processing

**Prompt:**
```
Write a Python function that reads a CSV file and returns a list of dicts.
Then test it, fix any errors, and return only the final working code.

Requirements:
- Use csv module
- Handle missing files gracefully
- Handle empty files
- Return empty list on error
```

**Agent might iterate:**
1. **First attempt:** Missing error handling → FileNotFound
2. **Second attempt:** Added try/except → Now returns None instead of []
3. **Third attempt:** Fixed to return [] → Works!

**Final output:** The working version only.

### When to Use Self-Correction

**Use when:**
- Code must work on first run
- You don't want to debug yourself
- Task has edge cases
- You're using a smaller model (more error-prone)

**Don't use when:**
- You want to see the thought process
- You're learning how to debug
- Code is experimental
- You have limited context/tokens

---

## Pattern 3: Multi-Pass Refinement

### The Concept

Different passes for different goals:

```
Pass 1: Make it work
Pass 2: Make it right
Pass 3: Make it fast
```

### Pass 1: Working Code

**Goal:** Get something that works, ignore quality.

**Prompt:**
```
Write a function that {task}.

Focus on getting it working. Don't worry about:
- Perfect error handling
- Optimization
- Edge cases (yet)

Just make it work for the basic case.
```

### Pass 2: Quality Improvement

**Goal:** Add error handling, type hints, docstrings.

**Prompt:**
```
Improve this working code:

{paste working code}

Requirements:
- Add comprehensive error handling
- Add type hints to all functions
- Add docstrings with description, params, returns
- Handle edge cases: {list}
- Follow {style guide}

Return the improved code.
```

### Pass 3: Optimization

**Goal:** Improve performance, reduce complexity.

**Prompt:**
```
Optimize this code for performance:

{paste quality code}

Requirements:
- Improve time complexity if possible
- Reduce memory usage if possible
- Keep the same behavior
- Don't sacrifice readability

Return the optimized code.
```

### Example: Multi-Pass for Data Processing

**Pass 1 - Working:**
```python
def process_data(data):
    result = []
    for item in data:
        result.append(item * 2)
    return result
```

**Pass 2 - Quality:**
```python
from typing import List

def process_data(data: List[int]) -> List[int]:
    """
    Process a list of integers by doubling each value.
    
    Args:
        data: List of integers to process
        
    Returns:
        List of doubled integers
        
    Raises:
        TypeError: If input is not a list or contains non-integers
    """
    if not isinstance(data, list):
        raise TypeError("Input must be a list")
    
    result: List[int] = []
    for item in data:
        if not isinstance(item, int):
            raise TypeError(f"Expected int, got {type(item)}")
        result.append(item * 2)
    return result
```

**Pass 3 - Optimized:**
```python
from typing import List

def process_data(data: List[int]) -> List[int]:
    """
    Process a list of integers by doubling each value.
    
    Args:
        data: List of integers to process
        
    Returns:
        List of doubled integers
    """
    return [item * 2 for item in data]
```

### When to Use Multi-Pass

**Use when:**
- You need production-quality code
- You're learning best practices
- Code will be maintained long-term
- Performance matters

**Don't use when:**
- Quick prototype
- One-time script
- You're in a hurry
- Code will be thrown away

---

## Pattern 4: Progressive Disclosure

### The Concept

Start simple, add complexity gradually:

```
Simple version → Add features → Add error handling → Add optimization
```

### Why It Works

- **Reduces cognitive load:** Agent focuses on one thing at a time
- **Easier debugging:** If something breaks, you know which step
- **Better learning:** You understand each layer
- **More control:** You can stop at any layer

### Progressive Disclosure Prompt

```
# Layer 1: Core functionality
Write the simplest version that works for the basic case.
Ignore error handling, edge cases, optimization.

# Layer 2: Error handling
Add error handling to the Layer 1 code.
Handle: {list edge cases}

# Layer 3: Type hints and docs
Add type hints and docstrings to the Layer 2 code.

# Layer 4: Optimization
Optimize the Layer 3 code for performance.

Start with Layer 1 only. I'll ask for the next layers when ready.
```

### Example: Progressive API Client

**Layer 1 - Core:**
```python
def fetch_data(url):
    import requests
    response = requests.get(url)
    return response.json()
```

**Layer 2 - Error handling:**
```python
def fetch_data(url):
    import requests
    from requests.exceptions import RequestException
    
    try:
        response = requests.get(url, timeout=10)
        response.raise_for_status()
        return response.json()
    except RequestException as e:
        raise RuntimeError(f"Failed to fetch {url}: {e}")
```

**Layer 3 - Types and docs:**
```python
from typing import Dict, Any
import requests
from requests.exceptions import RequestException

def fetch_data(url: str) -> Dict[str, Any]:
    """
    Fetch JSON data from a URL.
    
    Args:
        url: URL to fetch data from
        
    Returns:
        Parsed JSON data as a dictionary
        
    Raises:
        RuntimeError: If fetch fails or response is not JSON
    """
    try:
        response = requests.get(url, timeout=10)
        response.raise_for_status()
        return response.json()
    except RequestException as e:
        raise RuntimeError(f"Failed to fetch {url}: {e}")
```

**Layer 4 - Retry logic:**
```python
from typing import Dict, Any
import requests
from requests.exceptions import RequestException
import time

def fetch_data(url: str, retries: int = 3) -> Dict[str, Any]:
    """
    Fetch JSON data from a URL with retry logic.
    
    Args:
        url: URL to fetch data from
        retries: Number of retry attempts (default: 3)
        
    Returns:
        Parsed JSON data as a dictionary
        
    Raises:
        RuntimeError: If all retry attempts fail
    """
    for attempt in range(retries):
        try:
            response = requests.get(url, timeout=10)
            response.raise_for_status()
            return response.json()
        except RequestException as e:
            if attempt == retries - 1:
                raise RuntimeError(f"Failed after {retries} attempts: {e}")
            time.sleep(2 ** attempt)  # Exponential backoff
```

---

## When Real Multi-Agent Makes Sense

Real multi-agent (multiple independent AI agents) is **rarely needed** for coding tasks.

### Only Consider Real Multi-Agent When

1. **Parallel processing needed**
   - Multiple independent tasks running simultaneously
   - Example: Scrape 100 websites, process each independently

2. **Specialized expertise required**
   - One agent for security, one for performance, one for functionality
   - Each has deep, specialized knowledge

3. **Long-running workflows**
   - Agent A sets up, Agent B monitors, Agent C handles errors
   - Different agents for different phases

4. **Human-like team dynamics**
   - You're building a team of AI developers
   - Each agent has a role (PM, developer, QA, etc.)

### Real Multi-Agent Pattern (If You Must)

```
# Agent A: Coder
Task: Write the core functionality.
Output: Working code

# Agent B: Security Reviewer  
Task: Review code for security issues.
Input: Code from Agent A
Output: Security findings

# Agent C: Performance Reviewer
Task: Review code for performance issues.
Input: Code from Agent A
Output: Performance findings

# Agent D: Integrator
Task: Fix all issues and produce final code.
Input: Code from A, findings from B and C
Output: Final working code
```

**But honestly:** Fake multi-agent (role-switching) does 95% of this with 5% of the complexity.

---

## Exercise: Apply Patterns

### Exercise 1: Fake Multi-Agent (30 min)

**Task:** Use fake multi-agent to write production-quality code.

**Steps:**
1. Pick a function from Project A
2. Run Coder → Reviewer → Revisor pattern
3. Compare to single-pass generation
4. Note quality differences

**Prompt:**
```
# ROLE: Coder
Write a function that validates and parses a configuration file.
Requirements: {list}
Return ONLY the code.

# ROLE: Reviewer
Review this code for bugs, security, performance, style.
Return findings in format: 🔴 Critical, 🟡 Warning, 🔵 Suggestion

# ROLE: Revisor
Fix all issues found. Return ONLY the corrected code.
```

### Exercise 2: Self-Correction (20 min)

**Task:** Use self-correction to generate working code.

**Steps:**
1. Ask AI to write code with self-correction
2. Let it iterate until tests pass
3. Compare to single-pass generation
4. Note how many iterations were needed

**Prompt:**
```
Write a function that {task}, then test it, fix errors automatically,
and return only the final working code.
```

### Exercise 3: Multi-Pass (45 min)

**Task:** Use multi-pass refinement for a complex feature.

**Steps:**
1. Pass 1: Get working code
2. Pass 2: Add error handling and docs
3. Pass 3: Optimize
4. Compare to single-pass

**Prompt:**
```
# Pass 1: Write working code (ignore quality)
# Pass 2: Add error handling, type hints, docs
# Pass 3: Optimize for performance
```

---

## Pattern Selection Guide

| Task Type | Best Pattern | Why |
|---|---|---|
| **Quick prototype** | Single pass | Speed over quality |
| **Production code** | Fake multi-agent | Quality + consistency |
| **Complex logic** | Self-correction | Iterate to correctness |
| **Learning** | Multi-pass | Understand each layer |
| **Large refactor** | Progressive disclosure | Control and safety |
| **One-time script** | Single pass | No need for polish |

---

## On Your Hardware

### Small Models (1.5B-3B)

- Fake multi-agent works but each role switch loses context
- Self-correction needs more iterations (5-10)
- Multi-pass is the safest pattern for small models
- Keep each pass focused on one goal

### Medium Models (7B-14B)

- All patterns work well
- Self-correction converges in 2-4 iterations
- Fake multi-agent maintains role consistency

### Large Models (32B+)

- Self-correction often converges in 1-2 iterations
- Can handle complex multi-agent simulations
- Best results from combining patterns

---

## Next Steps

- [Chapter 15: Long-Term](./06-long-term.md) - Learn to work with agents over time
- [Chapter 14 Reference](../reference/prompt-templates.md) - Copy-paste pattern templates
