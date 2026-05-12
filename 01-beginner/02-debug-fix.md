# Chapter 2: Debug and Fix Bugs

> Use AI to diagnose errors, find root causes, and implement minimal fixes.

---

## TL;DR

- Feed error messages and stack traces directly to the AI
- Distinguish root cause from symptom
- Ask for minimal fixes, not refactors
- Iterate: error → fix → verify → repeat

---

## Prerequisites

- Completed Chapter 1
- Have code that produces errors

---

## The Debugging Loop

```
1. Receive error message
2. Feed it to AI with the code
3. Get analysis and fix
4. Apply fix and re-run
5. Repeat until green
```

---

## Feeding Error Messages

### The Prompt Pattern

```
The following code produces this error:

Code:
{paste code}

Error:
{paste full error message and stack trace}

Analyze the error:
1. What is the direct cause of this error?
2. What is the root cause in the code?
3. What is the minimal fix?

Return your analysis and the fixed code as a diff.
```

---

## Root Cause vs Symptom

**Symptom:** The error message you see
**Root cause:** The actual bug in the code

Example:
```
Error: IndexError: list index out of range (symptom)
Root cause: Function doesn't check if list is empty before accessing [0] (root cause)
```

Ask the AI to distinguish these:
```
Distinguish between:
- The symptom (what the error says)
- The root cause (why the error happens)
```

---

## Exercise: Debug a Planted Bug

**Task:** Fix a known bug using AI.

**Code with bug:**
```python
def divide_numbers(a, b):
    return a / b

result = divide_numbers(10, 0)
print(result)
```

**Prompt:**
```
The following code produces this error:

Code:
def divide_numbers(a, b):
    return a / b

result = divide_numbers(10, 0)
print(result)

Error:
ZeroDivisionError: division by zero

Analyze and provide the minimal fix.
```

**Expected fix:**
```python
def divide_numbers(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b
```

---

## When the Agent Makes Things Worse

### Recognizing Harmful Fixes

The AI might "fix" the wrong thing, introducing new bugs. Watch for:

- **Symptom fix:** The error message changes but the underlying bug remains
- **Over-correction:** The fix breaks previously working code
- **Scope creep:** The fix refactors unrelated code

**What to do:**
```
That fix changed the error from ZeroDivisionError to TypeError.
The original issue isn't resolved. Let me show you the new error:

{paste new error}

Let's step back and reconsider the root cause.
```

### Rolling Back Bad Fixes

**Always commit before asking AI to fix something.** Then you can revert:

```bash
git diff          # See what changed
git checkout -- . # Revert all changes
# Or revert specific file:
git checkout -- src/broken_file.py
```

---

## Common Pitfalls

### Pitfall 1: Fixing the Symptom, Not the Cause

**Bad:**
```
Error: NameError: name 'data' is not defined
Fix: Add data = [] at the top of the function
```

**Good:**
```
Error: NameError: name 'data' is not defined
Root cause: 'data' parameter was renamed to 'items' in function signature
Fix: Update all references from 'data' to 'items'
```

### Pitfall 2: Trusting the First Fix

**Bad:**
```
AI provides fix → Apply fix → Done
```

**Good:**
```
AI provides fix → Apply fix → Run tests → Verify all pass → Done
```

### Pitfall 3: Feeding Incomplete Error Messages

**Bad:**
```
I get an error on line 5
```

**Good:**
```
Traceback (most recent call last):
  File "app.py", line 5, in <module>
    result = divide(10, 0)
  File "app.py", line 2, in divide
    return a / b
ZeroDivisionError: division by zero
```

---

## On Your Hardware

### Small Models (1.5B-3B)

- May misdiagnose complex errors
- Good at fixing obvious syntax errors
- Provide the exact line number and error message
- Expect 2-3 iterations for non-trivial bugs

### Medium Models (7B-14B)

- Handle most debugging tasks well
- Can trace through multi-function call stacks
- Good at distinguishing root cause from symptom

### Large Models (32B+)

- Excellent at complex debugging
- Can analyze architectural issues causing bugs
- Often find the root cause in one pass

---

## Next Steps

- [Chapter 3: Write Tests](./03-write-tests.md)
- [Chapter 6: How Your Agent Thinks](../02-understand/01-how-agents-think.md)
