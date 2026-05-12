# Chapter 4: Review and Improve Code

> Use AI to find bugs, security issues, performance problems, and style violations.

---

## TL;DR

- AI can catch bugs, security issues, and anti-patterns
- Categorize findings: critical, warning, suggestion
- Use AI for both initial review and iterative improvement
- Don't trust blindly—verify critical findings
- Combine with automated tools (linters, type checkers)

---

## Prerequisites

- Completed [Chapters 1-3](./01-write-code.md)
- Have code to review
- Basic familiarity with code quality concepts

---

## Why AI Code Review?

Human review is slow and inconsistent. AI review is:
- **Fast** — Instant feedback
- **Consistent** — Same standards every time
- **Comprehensive** — Checks many dimensions
- **Available** — Anytime, no scheduling needed

**But:** AI can miss context-specific issues and produce false positives. Use as a first pass, not final authority.

---

## The Code Review Loop

```
┌─────────────────────────────────────────────────────┐
│            AI Code Review Loop                       │
│                                                      │
│  1. SUBMIT: Send code to AI with review request      │
│     ↓                                                │
│  2. ANALYZE: AI checks for:                          │
│     - Bugs and logic errors                          │
│     - Security vulnerabilities                       │
│     - Performance anti-patterns                      │
│     - Style violations                               │
│     - Maintainability issues                         │
│     ↓                                                │
│  3. CATEGORIZE: Findings grouped by severity:        │
│     🔴 Critical (bugs, security)                     │
│     🟡 Warning (performance, maintainability)        │
│     🔵 Suggestion (style, best practices)            │
│     ↓                                                │
│  4. REVIEW: Human evaluates findings                 │
│     ↓                                                │
│  5. FIX: Apply fixes for accepted findings           │
│     ↓                                                │
│  6. RE-REVIEW: Verify fixes, iterate                 │
└─────────────────────────────────────────────────────┘
```

---

## The Review Prompt Pattern

### Basic Review

```
Review this code for issues. Categorize findings:

🔴 Critical: Bugs, security vulnerabilities, data loss
🟡 Warning: Performance issues, maintainability concerns
🔵 Suggestion: Style improvements, best practices

Code:
{paste code}

For each finding:
- Line number(s)
- Category (🔴/🟡/🔵)
- Description
- Suggested fix (code snippet)

If no issues: "No issues found."
```

### Example Review

**Prompt:**
```
Review this code for issues. Categorize findings:

🔴 Critical: Bugs, security vulnerabilities
🟡 Warning: Performance, maintainability
🔵 Suggestion: Style, best practices

Code:
def get_user(user_id):
    query = f"SELECT * FROM users WHERE id = {user_id}"
    cursor.execute(query)
    return cursor.fetchone()

def process_data(data):
    result = []
    for item in data:
        result.append(item * 2)
    return result

def read_large_file(filepath):
    with open(filepath) as f:
        content = f.read()
    return content.split('\n')
```

**Expected Output:**
```
🔴 Line 2-3: SQL injection vulnerability
The query concatenates user input directly into SQL.
Fix:
- query = f"SELECT * FROM users WHERE id = {user_id}"
+ query = "SELECT * FROM users WHERE id = %s"
+ cursor.execute(query, (user_id,))

🟡 Line 7-10: Inefficient list building
Using append in a loop is less efficient than list comprehension.
Fix:
- result = []
- for item in data:
-     result.append(item * 2)
+ result = [item * 2 for item in data]

🔵 Line 1: Missing type hints
Add type annotations for better code clarity.
Fix:
+ def get_user(user_id: int) -> dict | None:
```

---

## Review Dimensions

### 1. Bugs and Logic Errors

**What to look for:**
- Off-by-one errors
- Incorrect conditionals
- Missing error handling
- Wrong algorithm
- State management issues

**Example:**
```python
# Bug: Off-by-one error
for i in range(len(items) - 1):  # Misses last item
    process(items[i])

# Fix:
for i in range(len(items)):  # Or just: for item in items:
    process(items[i])
```

### 2. Security Vulnerabilities

**What to look for:**
- SQL injection
- Command injection
- Path traversal
- Hardcoded secrets
- Insecure random
- XSS vulnerabilities

**Example:**
```python
# Vulnerable: Command injection
os.system(f"echo {user_input}")

# Fix:
subprocess.run(["echo", user_input], check=True)
```

### 3. Performance Anti-Patterns

**What to look for:**
- O(n²) when O(n) possible
- Unnecessary copying
- Repeated computations
- Blocking I/O in loops
- Large memory allocations

**Example:**
```python
# Slow: O(n²) lookup
for item in large_list:
    if item in another_list:  # Linear search each time
        process(item)

# Fast: O(n) lookup
another_set = set(another_list)  # Pre-compute
for item in large_list:
    if item in another_set:  # Constant-time lookup
        process(item)
```

### 4. Maintainability Issues

**What to look for:**
- Long functions (>50 lines)
- Deep nesting (>3 levels)
- Duplicate code
- Magic numbers/strings
- Poor naming

**Example:**
```python
# Hard to maintain
def process(data):
    if data['type'] == 'email':
        if data['priority'] == 'high':
            if data['verified']:
                send_email(data['to'], data['subject'], data['body'])
            else:
                log_warning("Unverified email")
        # ... 50 more lines

# Better: Extract functions
def process_notification(notification):
    if not is_verified(notification):
        log_warning("Unverified notification")
        return
    
    if notification['type'] == 'email':
        send_email_notification(notification)

def is_verified(notification):
    return notification.get('verified', False)
```

### 5. Style Violations

**What to look for:**
- Inconsistent naming
- Missing docstrings
- Wrong indentation
- Long lines
- Unused imports

**Example:**
```python
# Inconsistent style
def processData():  # camelCase
    pass

def process_data():  # snake_case
    pass

# Fix: Use consistent style (snake_case for Python)
def process_data():
    pass
```

---

## Review Prompt Variations

### Security-Focused Review

```
Perform a security review of this code. Look for:

1. Injection vulnerabilities (SQL, command, XSS)
2. Authentication/authorization issues
3. Sensitive data exposure
4. Insecure dependencies
5. Hardcoded secrets

Code:
{paste code}

For each finding:
- Severity (Critical/High/Medium/Low)
- Vulnerability type
- Location (line numbers)
- Exploitation scenario
- Fix recommendation
```

### Performance-Focused Review

```
Perform a performance review of this code. Look for:

1. Algorithmic complexity issues
2. Unnecessary allocations
3. I/O bottlenecks
4. Missing caching/memoization
5. Parallelization opportunities

Code:
{paste code}

For each finding:
- Current complexity
- Bottleneck description
- Expected improvement
- Refactored code
```

### Style-Only Review

```
Review this code for style and consistency. Check:

- Naming conventions (functions, classes, variables)
- Docstring completeness and format
- Import organization
- Line length and formatting
- Type hint completeness

Code:
{paste code}

Follow PEP 8 style guidelines.
```

---

## Integrating with Automated Tools

### Linters (ruff, pylint, flake8)

**Run before AI review:**
```bash
ruff check src/
```

**Then ask AI to fix:**
```
ruff found these issues:

{paste ruff output}

Fix all the issues in this code:
{paste code}
```

### Type Checkers (mypy)

**Run before AI review:**
```bash
mypy src/
```

**Then ask AI to fix:**
```
mypy found these type errors:

{paste mypy output}

Add proper type hints to fix these errors:
{paste code}
```

### Formatters (black, ruff format)

**Run before AI review:**
```bash
ruff format src/
```

**Or ask AI to format:**
```
Format this code according to PEP 8:
{paste code}
```

---

## Exercise: Review Your Project

**Task:** Review the CLI tool project from Chapter 1.

**Step 1:** Run automated tools
```bash
ruff check src/
mypy src/
```

**Step 2:** AI review
```
Review this code for bugs, security issues, and maintainability:

{paste your code}

Categorize findings:
🔴 Critical
🟡 Warning
🔵 Suggestion
```

**Step 3:** Fix critical issues
```
Fix these critical issues:
{paste critical findings}

Code:
{paste code}
```

**Step 4:** Re-review
```
Review the fixed code to verify issues are resolved:
{paste fixed code}
```

---

## On Your Hardware

### Small Models (1.5B-3B)

- May miss subtle issues
- Good for style checks
- Focus on obvious bugs

### Medium Models (7B-14B)

- Catch most common issues
- Good balance of speed and quality
- Can handle security reviews

### Large Models (32B+)

- Catch subtle bugs and anti-patterns
- Better at security analysis
- Can suggest architectural improvements

---

## Common Pitfalls

### Pitfall 1: Blind Trust

**Bad:**
```
AI says it's secure → Deploy immediately
```

**Fix:**
```
AI review → Human verification → Security scan → Deploy
```

### Pitfall 2: Reviewing Too Much at Once

**Bad:**
```
Review this 5000-line file
```

**Fix:**
```
Review this 50-line function
```

Break large reviews into smaller chunks.

### Pitfall 3: Ignoring Context

**Bad:**
```
AI: "This is inefficient"
You: Fix it immediately
```

**Fix:**
```
AI: "This is inefficient"
You: "Is this a bottleneck? Should I optimize now or later?"
```

Not all issues need immediate fixing.

---

## Review Checklist

Use this as a mental model when reviewing AI findings:

### Critical (Fix Immediately)
- [ ] SQL injection
- [ ] Command injection
- [ ] Authentication bypass
- [ ] Data corruption risk
- [ ] Crash on normal input

### Warning (Fix Soon)
- [ ] Performance bottleneck
- [ ] Memory leak
- [ ] Poor error handling
- [ ] Code duplication
- [ ] Tight coupling

### Suggestion (Fix When Time)
- [ ] Style inconsistency
- [ ] Missing docstrings
- [ ] Suboptimal naming
- [ ] Minor refactoring
- [ ] Documentation improvements

---

## Next Steps

- [Chapter 5: The Full Loop](./05-full-loop.md) — Combine all workflows
- [Chapter 10: Better Prompts](../03-advanced/01-better-prompts.md) — Improve review quality

---

## Further Reading

- [Prompt Templates](../reference/prompt-templates.md) — Review templates
- [Troubleshooting](../reference/troubleshooting.md) — Review issues
