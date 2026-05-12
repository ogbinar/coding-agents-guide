# Chapter 5: The Full Loop

> Put it all together: Plan → Code → Verify → Fix → Commit

---

## TL;DR

- The complete agent workflow: Plan → Code → Verify → Fix → Commit
- Each chapter taught one piece; now combine them
- Know when to stop iterating
- Git hygiene: meaningful commits, one change per commit
- Build confidence through repeated successful loops

---

## Prerequisites

- Completed [Chapters 1-4](./01-write-code.md)
- Familiar with code generation, debugging, testing, and review
- Have a small project to work on

---

## The Complete Agent Loop

You've learned each piece separately. Now let's put them together.

```
┌─────────────────────────────────────────────────────────────┐
│                  Complete Agent Loop                         │
│                                                              │
│   1. PLAN: Define what to build                             │
│      ↓                                                       │
│   2. CODE: Generate implementation                          │
│      ↓                                                       │
│   3. TEST: Generate and run tests                           │
│      ↓                                                       │
│   4. REVIEW: Check for bugs, security, style               │
│      ↓                                                       │
│   5. FIX: Address issues from review or failing tests      │
│      ↓                                                       │
│   6. VERIFY: All tests pass, linter clean                  │
│      ↓                                                       │
│   7. COMMIT: Git commit with meaningful message             │
│      ↓                                                       │
│   8. DECIDE: More changes needed? → Loop back to PLAN      │
│      Done → Move to next feature                            │
└─────────────────────────────────────────────────────────────┘
```

---

## Step-by-Step Walkthrough

### Step 1: PLAN

**Goal:** Define what to build before writing code.

**Prompt:**
```
I need to build {feature/task}.

Context:
- This is part of {project description}
- Existing code: {brief description}
- Constraints: {language, dependencies, etc.}

Before writing code, outline your approach:
1. What files need to be created/modified?
2. What functions/classes are needed?
3. What are the key design decisions?
4. What are the potential pitfalls?

Don't write code yet—just outline the approach.
```

**Example:**
```
I need to add a "search" feature to my CLI log parser.

Context:
- This is a Python CLI tool that parses log files
- Existing: parser.py (parsing), filters.py (filtering)
- Constraints: Python standard library only

Before writing code, outline your approach.
```

**Expected Output:**
```
Approach:

1. Files to create/modify:
   - src/logparser/search.py (new) - Search logic
   - src/logparser/cli.py (modify) - Add search command
   - tests/test_search.py (new) - Tests

2. Functions needed:
   - search_logs(logs: List[LogEntry], query: str) -> List[LogEntry]
   - parse_search_query(query: str) -> SearchCriteria

3. Design decisions:
   - Case-insensitive search by default
   - Support simple boolean operators (AND, OR)
   - Return matching log entries with context

4. Potential pitfalls:
   - Special characters in search query
   - Very large log files (memory)
   - Unicode handling
```

---

### Step 2: CODE

**Goal:** Generate the implementation based on the plan.

**Prompt:**
```
Based on this plan:
{paste plan}

Implement the feature. Create/modify these files:
- src/logparser/search.py
- src/logparser/cli.py

Requirements:
- Type hints on all functions
- Docstrings with examples
- Handle edge cases
- Follow existing code style

Return each file's content with its path.
```

**Expected Output:**
```
File: src/logparser/search.py

import re
from typing import List, Dict


def search_logs(logs: List[str], query: str) -> List[str]:
    """
    Search logs for entries matching the query.
    
    Args:
        logs: List of log entries
        query: Search query (supports AND, OR)
        
    Returns:
        Matching log entries
        
    Examples:
        >>> search_logs(["ERROR: failed", "INFO: ok"], "ERROR")
        ['ERROR: failed']
    """
    # Implementation...


File: src/logparser/cli.py

# ... existing code ...

def add_search_command(parser):
    """Add search subcommand to CLI."""
    search_parser = parser.add_parser("search")
    search_parser.add_argument("query", help="Search query")
    search_parser.add_argument("logfile", help="Log file to search")
    search_parser.set_defaults(func=handle_search)
```

---

### Step 3: TEST

**Goal:** Generate and run tests to verify the implementation.

**Prompt:**
```
Write pytest tests for this code:

{paste new code}

Requirements:
- Cover happy path
- Cover edge cases: empty logs, special characters, unicode
- Cover error cases: invalid query
- Use descriptive test names
```

**Then run:**
```bash
pytest tests/test_search.py -v
```

**If tests fail:**
```
These tests failed:

{paste test output}

Fix the code to pass all tests.
```

---

### Step 4: REVIEW

**Goal:** Check for bugs, security issues, and style problems.

**Prompt:**
```
Review this code for issues. Categorize:

🔴 Critical: Bugs, security vulnerabilities
🟡 Warning: Performance, maintainability
🔵 Suggestion: Style, best practices

Code:
{paste code}
```

**Then fix critical issues:**
```
Fix these critical issues:
{paste critical findings}

Code:
{paste code}
```

---

### Step 5: FIX

**Goal:** Address all issues from tests and review.

Iterate between fixing and re-running tests/review until:
- All tests pass
- No critical review findings
- No warnings you want to address

---

### Step 6: VERIFY

**Goal:** Final quality check before committing.

**Run:**
```bash
# Tests
pytest tests/ -v

# Coverage (optional)
pytest --cov=src/logparser tests/

# Linting
ruff check src/

# Type checking
mypy src/
```

**All should pass/clean.**

---

### Step 7: COMMIT

**Goal:** Create a meaningful git commit.

**Prompt:**
```
Generate a git commit message for these changes:

{paste git diff}

Format:
- Subject line (max 50 chars)
- Blank line
- Bullet points of key changes

Follow conventional commits format.
```

**Expected Output:**
```
feat: Add search functionality to log parser

- Add search_logs() function with AND/OR support
- Add CLI search subcommand
- Add comprehensive test suite (8 tests)
- Add search documentation
```

**Then commit:**
```bash
git add src/logparser/search.py src/logparser/cli.py tests/test_search.py
git commit -m "feat: Add search functionality to log parser

- Add search_logs() function with AND/OR support
- Add CLI search subcommand
- Add comprehensive test suite (8 tests)
- Add search documentation"
```

---

### Step 8: DECIDE

**Goal:** Determine if more work is needed.

**Ask yourself:**
- Are all planned features complete?
- Are there known bugs to fix?
- Is the code quality acceptable?
- Do I want to add more tests?

**If yes to any:** Loop back to PLAN for the next iteration.

**If no:** Move to the next feature or wrap up.

---

## Complete Example: Full Loop in Action

Let's walk through a complete example from start to finish.

### Task: Add a "count" command to the log parser

**Step 1: PLAN**

```
I need to add a "count" command that counts log entries by level.

Context:
- Python CLI tool for log parsing
- Existing: parser.py, filters.py, search.py
- Constraints: Python standard library only

Outline your approach.
```

**AI Response:**
```
Approach:

1. Files to create/modify:
   - src/logparser/counter.py (new) - Counting logic
   - src/logparser/cli.py (modify) - Add count command

2. Functions needed:
   - count_by_level(logs: List[LogEntry]) -> Dict[str, int]
   - format_count_output(counts: Dict[str, int]) -> str

3. Design decisions:
   - Count by log level (ERROR, WARN, INFO, DEBUG)
   - Show total count
   - Optional: filter by level before counting

4. Potential pitfalls:
   - Unknown log levels
   - Empty log files
   - Case sensitivity (ERROR vs error)
```

**Step 2: CODE**

```
Implement based on this plan. Create src/logparser/counter.py and modify cli.py.
```

**Step 3: TEST**

```
Write pytest tests for counter.py.
```

```bash
pytest tests/test_counter.py -v
# All pass ✓
```

**Step 4: REVIEW**

```
Review this code:
{paste counter.py}
```

**AI Response:**
```
🟡 Line 15: Consider using collections.Counter for cleaner code
🔵 Line 1: Missing type hint for return type
```

**Step 5: FIX**

```
Fix these issues:
{paste findings}
```

**Step 6: VERIFY**

```bash
pytest tests/ -v          # All pass ✓
ruff check src/           # Clean ✓
mypy src/                 # Clean ✓
```

**Step 7: COMMIT**

```bash
git add src/logparser/counter.py src/logparser/cli.py tests/test_counter.py
git commit -m "feat: Add count command to log parser

- Add count_by_level() function
- Add CLI count subcommand
- Add test suite (5 tests)"
```

**Step 8: DECIDE**

```
What's next?
- [ ] Add export command (CSV/JSON output)
- [ ] Add timestamp filtering
- [ ] Add colorized output
- [x] Done for now
```

---

## When to Stop Iterating

### Signs You're Done

- ✅ All planned features implemented
- ✅ All tests passing
- ✅ No critical review findings
- ✅ Code quality acceptable
- ✅ Diminishing returns on further refinement

### Signs You Should Continue

- ❌ Known bugs unaddressed
- ❌ Critical security issues
- ❌ Tests failing
- ❌ Major code smells
- ❌ Missing essential functionality

### The "Good Enough" Principle

**Perfection is the enemy of shipped.**

Ask:
- "Does this meet the requirements?"
- "Are there critical issues?"
- "Is the quality acceptable for this context?"

If yes → Ship it. You can always improve later.

---

## Git Hygiene

### Commit Message Best Practices

**Good:**
```
feat: Add search functionality

- Add search_logs() with AND/OR support
- Add CLI search subcommand
- Add 8 tests with 85% coverage
```

**Bad:**
```
fixed stuff
```

**Bad:**
```
Added the search feature that lets users search through their log files using various methods including AND and OR operators and also added tests and fixed some bugs and improved performance and...
```

### One Change Per Commit

**Good:**
```bash
git commit -m "feat: Add search functionality"
git commit -m "feat: Add count functionality"
git commit -m "docs: Update README with examples"
```

**Bad:**
```bash
git commit -m "all the changes"  # Everything in one commit
```

### Commit Before Big Changes

```bash
# Before refactoring
git commit -m "feat: Add current feature"

# Then refactor
git commit -m "refactor: Extract search logic to separate module"
```

---

## Exercise: Complete a Full Loop

**Task:** Add a new feature to your project using the full loop.

**Suggested features:**
- Add a "filter by date" command
- Add JSON output format
- Add a "recent" command (last N entries)
- Add colorized output

**Steps:**
1. PLAN: Outline your approach
2. CODE: Generate implementation
3. TEST: Write and run tests
4. REVIEW: Get AI review
5. FIX: Address issues
6. VERIFY: All checks pass
7. COMMIT: Git commit with message
8. DECIDE: What's next?

**Timebox:** 1-2 hours max

---

## On Your Hardware

### Small Models (1.5B-3B)

- May need more iteration
- Break loop into smaller steps
- Verify each step manually

### Medium Models (7B-14B)

- Can handle full loop smoothly
- Good balance of autonomy and quality
- Recommended for most workflows

### Large Models (32B+)

- Can run full loop with minimal intervention
- Better at planning and design
- May suggest architectural improvements

---

## Building Confidence

### Track Your Success

Keep a log of completed loops:

```
Feature: Search
- Plan: ✓
- Code: ✓
- Test: ✓ (8 tests, 85% coverage)
- Review: ✓ (2 warnings fixed)
- Commit: ✓
- Time: 45 minutes

Feature: Count
- Plan: ✓
- Code: ✓
- Test: ✓ (5 tests, 90% coverage)
- Review: ✓ (no issues)
- Commit: ✓
- Time: 30 minutes
```

### Notice Patterns

- Which features take longer?
- What types of bugs appear most?
- When do you need more iteration?
- What prompts work best?

### Iterate on Your Process

After 5-10 loops, ask:
- "Can I streamline any steps?"
- "Are there patterns I can automate?"
- "What prompts should I save for reuse?"

---

## Next Steps

You've completed the **Beginner** section. You can now:

- ✅ Generate code from descriptions
- ✅ Debug and fix errors
- ✅ Write comprehensive tests
- ✅ Review and improve code
- ✅ Run the full agent loop

### Continue to Part 2: Understand

You've learned by doing. Now let's understand what made this work — why agents make certain mistakes, how context shapes their output, and how to work with the tools that make it all possible.

- [Chapter 6: How Your Agent Thinks](../02-understand/01-how-agents-think.md) — Understand the mechanics
- [Chapter 7: Tools](../02-understand/02-tools.md) — How agents read files and run commands
- [Chapter 8: Context](../02-understand/03-context.md) — Why agents forget things

### Or Jump to Advanced

- [Chapter 10: Better Prompts](../03-advanced/01-better-prompts.md) — Improve your results
- [Chapter 16: Your Setup](../04-daily-driver/01-your-setup.md) — Build your daily driver

---

## Further Reading

- [Prompt Templates](../reference/prompt-templates.md) — Save your best prompts
- [Troubleshooting](../reference/troubleshooting.md) — Common loop issues
- [WORKS.md](../WORKS.md) — Quick cheat sheet
