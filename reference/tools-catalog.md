# Reference: Tools Catalog

> The 24 essential tools every coding agent uses.

---

## File Operations (6 tools)

### 1. read_file
**Purpose:** Read contents of a file

**Signature:**
```
read_file(path: str, start_line: int = None, end_line: int = None) -> str
```

**Example:**
```
read_file("src/parser.py", start_line=10, end_line=50)
```

---

### 2. write_file
**Purpose:** Write content to a file (creates or overwrites)

**Signature:**
```
write_file(path: str, content: str) -> None
```

**Example:**
```
write_file("src/new_module.py", "def hello():\n    print('Hello')\n")
```

---

### 3. edit_file
**Purpose:** Make targeted edits to a file

**Signature:**
```
edit_file(path: str, old_text: str, new_text: str) -> None
```

**Example:**
```
edit_file("src/parser.py", 
  old_text="def parse(): pass",
  new_text="def parse(data): return data.split()\n")
```

---

### 4. list_directory
**Purpose:** List files in a directory

**Signature:**
```
list_directory(path: str, recursive: bool = False) -> List[str]
```

**Example:**
```
list_directory("src/", recursive=True)
```

---

### 5. create_directory
**Purpose:** Create a new directory

**Signature:**
```
create_directory(path: str, parents: bool = True) -> None
```

**Example:**
```
create_directory("tests/unit/")
```

---

### 6. delete_file
**Purpose:** Delete a file

**Signature:**
```
delete_file(path: str) -> None
```

**Example:**
```
delete_file("temp.txt")
```

---

## Code Execution (3 tools)

### 7. run_command
**Purpose:** Execute a shell command

**Signature:**
```
run_command(command: str, timeout: int = 60, cwd: str = None) -> str
```

**Example:**
```
run_command("python src/main.py", timeout=30)
```

---

### 8. run_tests
**Purpose:** Run test suite

**Signature:**
```
run_tests(pattern: str = None, cwd: str = None) -> str
```

**Example:**
```
run_tests("tests/test_parser.py")
```

---

### 9. lint_code
**Purpose:** Run linter on code

**Signature:**
```
lint_code(path: str, linter: str = "ruff") -> str
```

**Example:**
```
lint_code("src/parser.py", linter="ruff")
```

---

## Search and Discovery (4 tools)

### 10. search_files
**Purpose:** Search for text in files

**Signature:**
```
search_files(pattern: str, path: str = ".", extensions: List[str] = None) -> List[Match]
```

**Example:**
```
search_files("def parse", path="src/", extensions=[".py"])
```

---

### 11. search_symbols
**Purpose:** Find function/class definitions

**Signature:**
```
search_symbols(name: str, symbol_type: str = None) -> List[Definition]
```

**Example:**
```
search_symbols("parse", symbol_type="function")
```

---

### 12. find_imports
**Purpose:** Find all imports in a file

**Signature:**
```
find_imports(path: str) -> List[Import]
```

**Example:**
```
find_imports("src/main.py")
```

---

### 13. get_references
**Purpose:** Find all references to a symbol

**Signature:**
```
get_references(symbol: str, path: str = None) -> List[Location]
```

**Example:**
```
get_references("parse_function", path="src/")
```

---

## Version Control (3 tools)

### 14. git_status
**Purpose:** Show git status

**Signature:**
```
git_status() -> str
```

**Example:**
```
git_status()
```

---

### 15. git_diff
**Purpose:** Show changes

**Signature:**
```
git_diff(path: str = None, staged: bool = False) -> str
```

**Example:**
```
git_diff("src/parser.py", staged=False)
```

---

### 16. git_commit
**Purpose:** Commit changes

**Signature:**
```
git_commit(message: str, all: bool = False) -> str
```

**Example:**
```
git_commit("Fix parsing bug", all=True)
```

---

## Analysis (4 tools)

### 17. get_file_info
**Purpose:** Get file metadata

**Signature:**
```
get_file_info(path: str) -> FileInfo
```

**Returns:** Size, line count, language, etc.

---

### 18. count_tokens
**Purpose:** Estimate token count

**Signature:**
```
count_tokens(text: str, model: str = "qwen2.5-coder:7b") -> int
```

**Example:**
```
count_tokens(file_content)  # Returns ~1500
```

---

### 19. get_syntax_tree
**Purpose:** Parse code into AST

**Signature:**
```
get_syntax_tree(path: str) -> AST
```

**Example:**
```
tree = get_syntax_tree("src/parser.py")
```

---

### 20. find_complexity
**Purpose:** Calculate cyclomatic complexity

**Signature:**
```
find_complexity(path: str) -> Dict[str, int]
```

**Example:**
```
find_complexity("src/")  # Returns {function_name: complexity}
```

---

## Documentation (2 tools)

### 21. generate_docs
**Purpose:** Generate docstrings/documentation

**Signature:**
```
generate_docs(path: str, style: str = "google") -> str
```

**Example:**
```
generate_docs("src/parser.py", style="google")
```

---

### 22. find_examples
**Purpose:** Find usage examples in codebase

**Signature:**
```
find_examples(symbol: str, path: str = None) -> List[Example]
```

**Example:**
```
find_examples("parse_function", path="examples/")
```

---

## Utility (2 tools)

### 23. copy_file
**Purpose:** Copy a file

**Signature:**
```
copy_file(src: str, dst: str) -> None
```

**Example:**
```
copy_file("template.py", "new_module.py")
```

---

### 24. rename_file
**Purpose:** Rename/move a file

**Signature:**
```
rename_file(old_path: str, new_path: str) -> None
```

**Example:**
```
rename_file("old_name.py", "new_name.py")
```

---

## Tool Usage Patterns

### Pattern 1: Read-Modify-Write
```
1. read_file("src/parser.py")
2. [AI analyzes and proposes changes]
3. edit_file("src/parser.py", old_text, new_text)
4. run_command("pytest tests/")
5. [If fails, read error and fix]
```

### Pattern 2: Search-Understand-Implement
```
1. search_symbols("parse_function")
2. read_file("src/parser.py")  # Where it's defined
3. get_references("parse_function")  # Where it's used
4. [AI understands context]
5. write_file("src/new_parser.py")  # New implementation
```

### Pattern 3: Test-Driven Development
```
1. write_file("tests/test_new.py")  # AI writes tests first
2. run_tests("tests/test_new.py")  # Should fail
3. write_file("src/new.py")  # AI writes implementation
4. run_tests("tests/test_new.py")  # Should pass
5. git_commit("Add new feature")
```

---

## Tool Security

### Dangerous Operations (Require Approval)
- `delete_file` - Can permanently remove data
- `run_command` - Can execute arbitrary code
- `git_commit` - Can create bad commits
- `write_file` - Can overwrite important files

### Safe Operations
- `read_file` - Read-only
- `search_files` - Read-only
- `get_file_info` - Read-only
- `count_tokens` - Read-only

---

## Implementation Notes

### For Custom Agents

When implementing tools:
1. **Validate paths** - Prevent directory traversal attacks
2. **Set timeouts** - Prevent infinite loops
3. **Limit output size** - Prevent context overflow
4. **Log all operations** - For debugging and audit
5. **Require confirmation** - For dangerous operations

### Example Tool Wrapper

```python
def safe_read_file(path: str, max_lines: int = 1000):
    """Safely read a file with limits."""
    # Validate path
    if ".." in path or path.startswith("/"):
        raise ValueError("Invalid path")
    
    # Read with limit
    with open(path) as f:
        lines = f.readlines()[:max_lines]
    
    return "".join(lines)
```

---

*See also: [Prompt Templates](./prompt-templates.md), [Troubleshooting](./troubleshooting.md)*
