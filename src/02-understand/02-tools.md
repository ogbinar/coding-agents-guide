# Chapter 7: Tools — How Your Agent Reads Files and Runs Commands

> Understand the 8 essential tools every coding agent uses.

---

## TL;DR

- Tools let agents read files, run commands, and interact with the system
- The 8 essential tools: read_file, write_file, edit_file, list_directory, run_command, search_files, git_status, git_diff
- Tool output management is critical (don't blow your context)
- Existing tools (Aider, Continue) handle this under the hood
- You don't need to design tools—just understand what your agent is doing

---

## Prerequisites

- Completed [Chapter 6](./01-how-agents-think.md)
- Understand the Plan-Code-Verify loop
- Ready to understand how agents interact with the system

---

## What Are Tools?

**Tools** are functions that an agent can call to interact with the outside world.

**Without tools:** Agent can only read your prompt and generate text.

**With tools:** Agent can:
- Read files from your project
- Run commands (tests, linters, builds)
- Search for code
- Make git changes
- And more

### The Agent-Tool Relationship

```
┌─────────────────────────────────────────────────────────────┐
│                    Agent + Tools                             │
│                                                              │
│  ┌──────────┐                                               │
│  │   LLM    │  →  Decides which tool to use                 │
│  └──────────┘                                               │
│       ↓                                                     │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐              │
│  │read_file │    │run_cmd   │    │ git_diff │  ...         │
│  └──────────┘    └──────────┘    └──────────┘              │
│       ↓                 ↓                 ↓                 │
│  ┌─────────────────────────────────────────────┐           │
│  │           File System / Shell / Git         │           │
│  └─────────────────────────────────────────────┘           │
│       ↓                 ↓                 ↓                 │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐              │
│  │  File    │    │  Output  │    │  Diff    │  ← Results  │
│  └──────────┘    └──────────┘    └──────────┘              │
│       ↓                 ↓                 ↓                 │
│  ┌─────────────────────────────────────────────┐           │
│  │         Results sent back to LLM            │           │
│  └─────────────────────────────────────────────┘           │
└─────────────────────────────────────────────────────────────┘
```

---

## The 8 Essential Tools

Every coding agent needs these tools:

### 1. read_file

**Purpose:** Read contents of a file

**Signature:**
```python
read_file(path: str, start_line: int = None, end_line: int = None) -> str
```

**Example:**
```
Agent: I need to understand the existing code.
Tool call: read_file("src/logparser/parser.py")
Result: "import re...\ndef parse_log..."
```

**Why it matters:** Agent can't write code without understanding existing code.

---

### 2. write_file

**Purpose:** Create or overwrite a file

**Signature:**
```python
write_file(path: str, content: str) -> None
```

**Example:**
```
Agent: I'll create the new search module.
Tool call: write_file("src/logparser/search.py", "def search_logs...")
Result: File created
```

**Why it matters:** Agent needs to create new files for new features.

---

### 3. edit_file

**Purpose:** Make targeted edits to a file

**Signature:**
```python
edit_file(path: str, old_text: str, new_text: str) -> None
```

**Example:**
```
Agent: I need to add the search command to the CLI.
Tool call: edit_file("src/logparser/cli.py", 
  old_text="# Existing commands...",
  new_text="# Existing commands...\nsearch_parser = ...")
Result: File updated
```

**Why it matters:** More efficient than rewriting entire files.

---

### 4. list_directory

**Purpose:** List files in a directory

**Signature:**
```python
list_directory(path: str, recursive: bool = False) -> List[str]
```

**Example:**
```
Agent: What files exist in the project?
Tool call: list_directory("src/", recursive=True)
Result: ["parser.py", "cli.py", "filters.py", ...]
```

**Why it matters:** Agent needs to understand project structure.

---

### 5. run_command

**Purpose:** Execute a shell command

**Signature:**
```python
run_command(command: str, timeout: int = 60, cwd: str = None) -> str
```

**Example:**
```
Agent: I need to run tests to verify my changes.
Tool call: run_command("pytest tests/test_search.py -v", timeout=30)
Result: "test_search_basic PASSED\n..."
```

**Why it matters:** Agent needs to verify code works.

---

### 6. search_files

**Purpose:** Search for text in files

**Signature:**
```python
search_files(pattern: str, path: str = ".", extensions: List[str] = None) -> List[Match]
```

**Example:**
```
Agent: Where is the parse_log function defined?
Tool call: search_files("def parse_log", path="src/", extensions=[".py"])
Result: [{"file": "parser.py", "line": 15, "match": "def parse_log..."}]
```

**Why it matters:** Agent needs to find code quickly.

---

### 7. git_status

**Purpose:** Show git status

**Signature:**
```python
git_status() -> str
```

**Example:**
```
Agent: What changes have I made?
Tool call: git_status()
Result: "modified: src/logparser/cli.py\nnew file: search.py"
```

**Why it matters:** Agent needs to track changes.

---

### 8. git_diff

**Purpose:** Show changes

**Signature:**
```python
git_diff(path: str = None, staged: bool = False) -> str
```

**Example:**
```
Agent: What did I change in cli.py?
Tool call: git_diff("src/logparser/cli.py")
Result: "--- a/src/logparser/cli.py\n+++ b/src/logparser/cli.py\n..."
```

**Why it matters:** Agent needs to review its own changes.

---

## Tool Output Management

### The Problem

Tool outputs can be huge:
- Reading a 1000-line file = 1000 tokens
- Running tests = 500+ tokens
- Searching codebase = 1000+ tokens

**Context window fills up fast.** See [Chapter 8: Context](./03-context.md) for strategies to manage context limits effectively.

### Solutions

#### 1. Read Only Relevant Lines

**Bad:**
```
read_file("src/large_module.py")  # Reads entire 2000-line file
```

**Good:**
```
read_file("src/large_module.py", start_line=100, end_line=150)  # Just the function
```

#### 2. Summarize Tool Output

**Bad:**
```
Agent reads 10 files, each 500 lines = 5000 tokens
```

**Good:**
```
Agent reads summary: "parser.py has parse_log() and validate(). 
cli.py has create_parser() and handle_search()."
```

#### 3. Progressive Disclosure

**Bad:**
```
Load entire codebase at once
```

**Good:**
```
Load only files needed for current task
Load more files if needed
```

---

## How Existing Tools Handle This

### Aider

**Approach:** Git-aware, diff-based editing

- Reads only files mentioned in conversation
- Uses diffs instead of full file writes
- Maintains context through git history
- Automatically manages tool calls

**You do:**
```bash
aider src/logparser/search.py --message "Add search feature"
```

**Aider handles:**
- Reading relevant files
- Generating diffs
- Running tests
- Managing context

---

### Continue

**Approach:** IDE integration, context-aware

- Reads files from IDE workspace
- Uses LSP for symbol search
- Manages context through IDE state
- Provides inline editing

**You do:**
```
Select code → Right-click → "Edit with AI"
```

**Continue handles:**
- Loading relevant context
- Understanding file relationships
- Applying edits safely
- Running linters/tests

---

### OpenCode

**Approach:** Terminal-based, interactive

- Reads files on demand
- Runs commands in terminal
- Shows tool outputs inline
- Lets you approve dangerous operations

**You do:**
```bash
opencode "Add search feature to log parser"
```

**OpenCode handles:**
- Tool selection
- Output formatting
- Safety checks
- Interactive approval

---

## Tool Design (For Reference)

You don't need to design tools, but understanding helps.

### Tool Schema

```python
from typing import List, Optional

class Tool:
    name: str
    description: str
    parameters: dict
    
# Example: read_file tool
read_file_tool = Tool(
    name="read_file",
    description="Read contents of a file",
    parameters={
        "type": "object",
        "properties": {
            "path": {
                "type": "string",
                "description": "Path to the file"
            },
            "start_line": {
                "type": "integer",
                "description": "Start line (optional)"
            },
            "end_line": {
                "type": "integer", 
                "description": "End line (optional)"
            }
        },
        "required": ["path"]
    }
)
```

### Tool Calling Pattern

```
1. Agent decides it needs to read a file
2. Agent outputs tool call:
   {"tool": "read_file", "args": {"path": "src/parser.py"}}
3. System executes tool
4. System sends result back to agent
5. Agent continues with new information
```

### Tool Security

**Dangerous operations require approval:**
- `write_file` — Can overwrite important files
- `run_command` — Can execute arbitrary code
- `delete_file` — Can permanently remove data
- `git_commit` — Can create bad commits

**Safe operations:**
- `read_file` — Read-only
- `search_files` — Read-only
- `list_directory` — Read-only

---

## Exercise: Observe Tool Usage

**Task:** Watch how your agent uses tools.

**Steps:**
1. Start a new agent session
2. Give it a task: "Add a count command to the log parser"
3. Watch the tool calls:
   - What files does it read first?
   - When does it run commands?
   - How does it handle errors?
4. Note the pattern

**Reflection:**
- Did the agent read relevant files first?
- Did it verify changes with tests?
- Did it manage context well?
- Where did it struggle?

---

## On Your Hardware

### Small Models (1.5B-3B)

- May call wrong tools
- Struggle with tool schemas
- Need simpler tool sets (max 5 tools)
- Prefer explicit tool calls over auto-detection

### Medium Models (7B-14B)

- Good tool selection
- Understand tool schemas
- Handle 8-12 tools well
- Can use auto-detection

### Large Models (32B+)

- Excellent tool selection
- Can handle complex tool schemas
- Manage 20+ tools
- Can design new tools if needed

---

## Common Patterns

### Pattern 1: Read-Modify-Write

```
1. read_file("src/parser.py")  # Understand existing code
2. [Agent plans changes]
3. edit_file("src/parser.py", old, new)  # Make changes
4. run_command("pytest")  # Verify
5. [If fails, read error and fix]
```

### Pattern 2: Search-Understand-Implement

```
1. search_files("parse_log")  # Find where function is
2. read_file("src/parser.py")  # Read the file
3. [Agent understands context]
4. write_file("src/new_parser.py")  # Create new implementation
```

### Pattern 3: Test-Driven

```
1. write_file("tests/test_new.py")  # Write tests first
2. run_command("pytest tests/test_new.py")  # Should fail
3. write_file("src/new.py")  # Write implementation
4. run_command("pytest tests/test_new.py")  # Should pass
```

---

## Tool Catalog

See [Reference: Tools Catalog](../reference/tools-catalog.md) for the complete list of 24 essential tools.

---

## Next Steps

- [Chapter 8: Context](./03-context.md) — Why agents forget things
- [Chapter 9: What Not to Build](./04-what-not-to-build.md) — When to use existing tools

---

## Further Reading

- [Reference: Tools Catalog](../reference/tools-catalog.md) — Complete tool list
- [Reference: Prompt Templates](../reference/prompt-templates.md) — Tool usage prompts
- [WORKS.md](../WORKS.md) — Manual tool usage patterns
