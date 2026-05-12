# Chapter 6: How Your Agent Thinks

> Understand the Plan-Code-Verify loop that powers coding agents.

---

## TL;DR

- Coding agents use Plan-Code-Verify, not ReAct
- Plan: Analyze task, read files, outline approach
- Code: Generate implementation (files, diffs)
- Verify: Run tests, linter, build
- Fix: Iterate until green
- Max iterations: 10-20 for most tasks
- Early exit: Stop when tests pass or you hit a wall

---

## Prerequisites

- Completed [Part 1: Beginner](../01-beginner/01-write-code.md) through [Chapter 5](../01-beginner/05-full-loop.md)
- Have used agents for coding tasks
- Ready to understand what's happening under the hood

---

## The Agent Loop You've Been Using

You've been using agents for coding. Now let's name what you've been doing.

### Plan-Code-Verify (The Coding Loop)

```
┌─────────────────────────────────────────────────────────────┐
│              Plan-Code-Verify Loop                           │
│                                                              │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐                 │
│  │  PLAN   │ →  │  CODE   │ →  │ VERIFY  │                 │
│  └─────────┘    └─────────┘    └─────────┘                 │
│       ↑                               │                     │
│       │         ┌─────────┐           │                     │
│       └──────── │  FIX    │ ←─────────┘                     │
│                 └─────────┘                                 │
│                                                              │
│  1. PLAN:  Analyze task, read files, outline approach       │
│  2. CODE:  Generate implementation (files, diffs)           │
│  3. VERIFY: Run tests → linter → build                      │
│  4. FIX:   Diagnose errors, generate fixes                  │
│  5. Loop back to CODE until verification passes             │
│  6. COMMIT: When green, generate commit message             │
└─────────────────────────────────────────────────────────────┘
```

---

## Why Plan-Code-Verify Beats ReAct for Coding

### ReAct (Reason → Act → Observe)

**Designed for:** General agents, search, decision-making

```
Reason: I need to find information about X
Act: Search the web for "X"
Observe: Got search results
Reason: I need more details
Act: Click on link Y
Observe: Got article content
...
```

**Problem for coding:**
- No clear termination condition
- Subjective "reasoning" steps
- No objective verification
- Prone to infinite loops

### Plan-Code-Verify (The Coding Loop)

**Designed for:** Coding tasks

```
Plan: I'll add a search feature
  - Create search.py
  - Modify cli.py
  - Add tests

Code: Generate the files

Verify: Run tests
  ✓ All tests pass → Done
  ✗ Tests fail → Fix

Fix: Address test failures

Verify: Run tests again
  ✓ All tests pass → Done
```

**Why it works:**
- **Objective verification** — Tests pass/fail, not subjective
- **Concrete artifacts** — Code files can be reviewed
- **External termination** — Green build, not internal confidence
- **Known failure state** — Uncommitted changes can be reverted

---

## The Loop in Detail

### Step 1: PLAN

**What happens:**
1. Agent reads the task description
2. Agent reads relevant files (if any)
3. Agent outlines the approach
4. Agent identifies potential pitfalls

**What the agent thinks:**
```
Task: Add search feature to log parser

Files to read:
- src/logparser/parser.py (understand LogEntry structure)
- src/logparser/cli.py (understand CLI structure)

Approach:
1. Create search.py with search_logs() function
2. Modify cli.py to add search subcommand
3. Write tests

Pitfalls:
- Special characters in search query
- Case sensitivity
- Performance on large files
```

**Your control:**
- Provide clear task descriptions
- Point to relevant files
- Specify constraints upfront

---

### Step 2: CODE

**What happens:**
1. Agent generates code based on plan
2. Agent writes files (new or modified)
3. Agent outputs diffs or full file contents

**What the agent thinks:**
```
Generating search.py:
- Import necessary modules
- Define search_logs() function
- Add type hints and docstrings
- Handle edge cases

Generating cli.py modifications:
- Add search subcommand parser
- Connect to handle_search() function
```

**Output formats:**

**Full file:**
```
File: src/logparser/search.py

import re
from typing import List

def search_logs(logs: List[str], query: str) -> List[str]:
    """Search logs for matching entries."""
    # Implementation...
```

**Diff:**
```
--- a/src/logparser/cli.py
+++ b/src/logparser/cli.py
@@ -10,6 +10,12 @@ def create_parser():
     subparsers = parser.add_subparsers()
     
     # Existing commands...
+    
+    # Search command
+    search_parser = subparsers.add_parser("search")
+    search_parser.add_argument("query")
+    search_parser.add_argument("logfile")
+    search_parser.set_defaults(func=handle_search)
     
     return parser
```

**Your control:**
- Specify output format (full file vs diff)
- Request specific file paths
- Ask for incremental vs bulk changes

---

### Step 3: VERIFY

**What happens:**
1. Agent runs tests
2. Agent runs linter
3. Agent runs build (if applicable)
4. Agent collects results

**What the agent thinks:**
```
Running tests:
  pytest tests/test_search.py -v
  
  test_search_basic PASSED
  test_search_empty PASSED
  test_search_special_chars FAILED
  
  2 passed, 1 failed

Running linter:
  ruff check src/logparser/search.py
  All checks passed!
```

**Verification commands:**
```bash
# Tests
pytest tests/ -v

# Coverage
pytest --cov=src tests/

# Linting
ruff check src/

# Type checking
mypy src/

# Build
python -m build
```

**Your control:**
- Specify which tests to run
- Set coverage targets
- Choose linters and formatters

---

### Step 4: FIX

**What happens:**
1. Agent analyzes failures
2. Agent generates fixes
3. Agent applies changes
4. Agent loops back to VERIFY

**What the agent thinks:**
```
Test failure:
  test_search_special_chars FAILED
  AssertionError: assert ['log1'] == ['log1', 'log2']
  
Root cause:
  Search is case-sensitive, but test expects case-insensitive
  
Fix:
  Convert both query and log entries to lowercase before comparing
```

**Fix prompt:**
```
This test is failing:

Code:
def search_logs(logs, query):
    return [log for log in logs if query in log]

Test:
def test_search_case_insensitive():
    assert search_logs(["ERROR: failed", "error: ok"], "error") == \
           ["ERROR: failed", "error: ok"]

Error:
AssertionError: assert [] == ["ERROR: failed", "error: ok"]

Fix the implementation.
```

**Your control:**
- Provide clear error messages
- Point to specific failures
- Suggest fix direction if needed

---

## Max Iterations

### What Are Iterations?

Each loop through Plan-Code-Verify-Fix counts as one iteration.

**Example:**
```
Iteration 1:
  Plan: Add search feature
  Code: Generate search.py
  Verify: 2/3 tests pass
  Fix: Fix case sensitivity

Iteration 2:
  Code: Update search.py
  Verify: 3/3 tests pass ✓
  Done!
```

**Total: 2 iterations**

### Recommended Limits

| Task Complexity | Max Iterations | Timebox |
|---|---|---|
| **Simple function** | 3-5 | 10-15 min |
| **New feature** | 5-10 | 30-60 min |
| **Complex feature** | 10-15 | 1-2 hours |
| **Refactoring** | 10-20 | 2-3 hours |

### When to Stop

**Stop and reassess when:**
- You hit max iterations
- Same bug persists for 3+ iterations
- Tests keep failing for different reasons
- Code quality is degrading

**Prompt:**
```
I've tried fixing this for 5 iterations and it's still failing.

Current issue: {describe issue}

Let's step back and reconsider the approach.
What's a different way to solve this?
```

---

## Early Exit Conditions

### Success (Stop Early)

**Conditions:**
- ✅ All tests pass
- ✅ Linter clean
- ✅ No critical review findings
- ✅ Meets requirements

**Action:** Commit and move on.

### Failure (Stop and Reassess)

**Conditions:**
- ❌ Same bug for 3+ iterations
- ❌ Tests failing for different reasons each time
- ❌ Code quality degrading
- ❌ Agent seems confused

**Action:**
1. Stop the loop
2. Review what's happening
3. Try a different approach
4. Or do it manually

**Prompt:**
```
This isn't working. Let's try a different approach.

Current approach: {describe}
Problem: {describe}

Alternative: {suggest different approach}

What do you think?
```

---

## State Management

### What the Agent Remembers

**Within a session:**
- Conversation history (all prompts and responses)
- Files that have been read/modified
- Test results and errors
- Previous fix attempts

**What gets lost:**
- Between sessions (unless you save state)
- Very old conversation history (context limit)
- File system changes (unless committed)

### Managing Context

**Problem:** Context window fills up, agent forgets early decisions.

**Solution 1: Summarize**
```
Summary of what we've done:
- Created search.py with search_logs()
- Added CLI search subcommand
- Fixed case sensitivity bug
- All tests passing

Now let's add the next feature: {next task}
```

**Solution 2: New session**
```
Starting fresh session. Context:
- Project: Log parser CLI tool
- Just completed: Search feature
- Next task: Count feature
- Key files: src/logparser/
```

**Solution 3: Load only relevant files**
```
Don't load the entire codebase. Just read:
- src/logparser/counter.py (new file)
- src/logparser/cli.py (to add count command)
```

---

## Tracing a Loop by Hand

Let's trace through a real example.

### Task: Fix a bug in the search feature

**Iteration 1:**

**Plan:**
```
Agent: I'll fix the search bug.
Reading: src/logparser/search.py
Issue: Case-sensitive search when it should be case-insensitive
```

**Code:**
```
Agent: Generating fix...

File: src/logparser/search.py

def search_logs(logs: List[str], query: str) -> List[str]:
    """Search logs (case-insensitive)."""
    query_lower = query.lower()
    return [log for log in logs if query_lower in log.lower()]
```

**Verify:**
```
Agent: Running tests...
pytest tests/test_search.py -v

test_search_basic PASSED
test_search_case_insensitive PASSED ✓
test_search_special_chars PASSED

All tests pass!
```

**Fix:** Not needed (tests pass)

**Result:** Done in 1 iteration.

---

## Exercise: Trace a Loop

**Task:** Run a full loop and trace what happens.

**Steps:**
1. Pick a small feature to add
2. Run the agent
3. Document each iteration:
   - What did the agent plan?
   - What code did it generate?
   - What tests failed?
   - How many fixes were needed?
4. Count total iterations
5. Note where it got stuck (if anywhere)

**Timebox:** 30 minutes

**Reflection:**
- Did the agent follow Plan-Code-Verify?
- How many iterations did it take?
- Where did it struggle?
- What could you improve?

---

## On Your Hardware

### Small Models (1.5B-3B)

- May need more iterations
- Less consistent planning
- More prone to getting stuck
- Recommend: 5-10 iteration limit

### Medium Models (7B-14B)

- Good planning and execution
- Consistent loop behavior
- Recommended: 10-15 iteration limit
- Best balance of autonomy and quality

### Large Models (32B+)

- Excellent planning
- Fewer iterations needed
- Better at avoiding dead ends
- Recommended: 15-20 iteration limit

---

## Common Patterns

### Pattern 1: First Try Success

```
Plan: Clear approach
Code: Good first implementation
Verify: All tests pass ✓
Done: 1 iteration
```

**When:** Simple tasks, well-defined requirements

### Pattern 2: One Fix Needed

```
Plan: Good approach
Code: Minor bug
Verify: 1 test fails
Fix: Address bug
Verify: All tests pass ✓
Done: 2 iterations
```

**When:** Moderate complexity, agent is competent

### Pattern 3: Iterative Refinement

```
Plan: Good approach
Code: Works but not optimal
Verify: Tests pass, review finds issues
Fix: Address review findings
Verify: Clean ✓
Done: 3 iterations
```

**When:** Quality matters, review catches issues

### Pattern 4: Getting Stuck

```
Plan: Unclear approach
Code: Wrong direction
Verify: Tests fail
Fix: Partial fix
Verify: Different tests fail
Fix: Different partial fix
... (repeats)
```

**When:** Requirements unclear, task too complex
**Action:** Stop, reassess, try different approach

---

## Next Steps

Now that you understand how agents think:

- [Chapter 7: Tools](./02-tools.md) — How agents read files and run commands
- [Chapter 8: Context](./03-context.md) — Why agents forget things
- [Chapter 9: What Not to Build](./04-what-not-to-build.md) — When to use existing tools

---

## Further Reading

- [WORKS.md](../WORKS.md) — Manual Plan-Code-Verify loop
- [Prompt Templates](../reference/prompt-templates.md) — Loop prompts
- [Troubleshooting](../reference/troubleshooting.md) — Loop issues
