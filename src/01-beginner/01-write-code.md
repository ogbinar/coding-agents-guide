# Chapter 1: Write New Code with AI

> Generate code from descriptions, scaffold new projects, and specify constraints that actually work.

---

## TL;DR

- AI can write complete functions, classes, and even entire projects from descriptions
- Be specific about requirements, constraints, and output format
- Start with greenfield (new code) before tackling brownfield (existing code)
- Always verify generated code runs and passes tests

---

## Prerequisites

- Completed [Quick Start](../00-quick-start/01-first-function.md)
- Have Ollama and a model installed
- Basic familiarity with your programming language

---

## From Description to Working Code

The most basic coding agent workflow: describe what you want, get code back.

### The Prompt Pattern

```
Write a {language} function/class/module that {description}.

Requirements:
- {Constraint 1: e.g., Type hints}
- {Constraint 2: e.g., No external dependencies}
- {Constraint 3: e.g., Handle edge cases}
- {Constraint 4: e.g., Include docstrings}

Context:
- This is part of a {project type} project
- The project uses {conventions}
- Available dependencies: {list}

Return ONLY the code. No explanation.
```

### Example: Generate a CSV Parser

**Prompt:**
```
Write a Python function that parses a CSV file and returns a list of dictionaries.

Requirements:
- Type hints on all parameters and return value
- Docstring with description, params, returns, examples
- Handle edge cases: empty file, missing columns
- Use only standard library

Context:
- This is part of a data processing CLI tool
- Follow PEP 8 style
- Python 3.10+

Return ONLY the code.
```

**Expected Output:**
```python
import csv
from typing import List, Dict, Optional


def parse_csv(filepath: str, delimiter: str = ',') -> List[Dict[str, str]]:
    """
    Parse a CSV file and return a list of dictionaries.
    
    Args:
        filepath: Path to the CSV file
        delimiter: Field delimiter character (default: comma)
        
    Returns:
        List of dictionaries, one per row, with column headers as keys
        
    Examples:
        >>> parse_csv("data.csv")
        [{'name': 'Alice', 'age': '30'}, {'name': 'Bob', 'age': '25'}]
        >>> parse_csv("empty.csv")
        []
    """
    if not filepath:
        raise ValueError("Filepath cannot be empty")
    
    try:
        with open(filepath, 'r', newline='', encoding='utf-8') as f:
            reader = csv.DictReader(f, delimiter=delimiter)
            return list(reader)
    except FileNotFoundError:
        raise FileNotFoundError(f"CSV file not found: {filepath}")
    except csv.Error as e:
        raise ValueError(f"Error parsing CSV: {e}")
```

### Verify the Code

```python
# Save to csv_parser.py
# Test it
from csv_parser import parse_csv

# Create test file
with open("test.csv", "w") as f:
    f.write("name,age\n")
    f.write("Alice,30\n")
    f.write("Bob,25\n")

# Run
result = parse_csv("test.csv")
print(result)  # [{'name': 'Alice', 'age': '30'}, {'name': 'Bob', 'age': '25'}]
```

---

## Greenfield: Scaffolding New Projects

When starting a new project, ask the AI to create the entire structure.

### Prompt Pattern

```
Scaffold a new {language/framework} {project type} called {name} that {description}.

Requirements:
- Include {essential files: README, config, etc.}
- Set up {build system, linter, formatter}
- Include {test framework} with one example test
- Follow {style guide/conventions}
- Include {usage examples}

Return the file structure first, then the content of each file.
```

### Example: Scaffold a CLI Tool

**Prompt:**
```
Scaffold a new Python CLI tool called "logparser" that parses and filters log files.

Requirements:
- Include pyproject.toml with dependencies
- Include README.md with installation and usage
- Include src/logparser/ with main module
- Include tests/ with one example test
- Use argparse for CLI
- Follow PEP 8 style
- Include pre-commit hooks config

Return the file structure first, then the content of each file.
```

**Expected Structure:**
```
logparser/
├── pyproject.toml
├── README.md
├── .pre-commit-config.yaml
├── src/
│   └── logparser/
│       ├── __init__.py
│       ├── cli.py
│       ├── parser.py
│       └── filters.py
└── tests/
    ├── __init__.py
    └── test_parser.py
```

Each file would contain appropriate starter content.

---

## Specifying Constraints That Work

Vague constraints → vague code. Be specific.

### Good Constraints

```
Requirements:
- Type hints on ALL parameters and return values (not just some)
- Docstrings in Google style (not NumPy or reStructuredText)
- Handle these edge cases: empty input, None values, negative numbers
- Use ONLY Python standard library (no external packages)
- Follow PEP 8 with max line length 88 (matching Black formatter)
- Include at least 3 examples in docstrings
```

### Bad Constraints

```
Requirements:
- Make it clean
- Handle errors
- Use good practices
- Make it production-ready
```

These are too vague. The AI doesn't know what "clean" or "production-ready" means to you.

---

## Self-Review Before You Look

Ask the AI to review its own code before returning it.

### Prompt Pattern

```
Write a function that {description}.

Before returning the code, verify:
- All imports are valid and included
- Syntax is correct (no missing colons, parentheses, etc.)
- Type hints are complete
- Edge cases are handled
- Docstring includes examples

If you find any issues, fix them before returning the code.

Return ONLY the final, verified code.
```

---

## Common Pitfalls

### Hallucinated APIs

**Problem:** AI uses libraries or functions that don't exist.

**Fix:**
```
Use ONLY Python standard library. Do NOT use external packages like pandas, numpy, etc.
If you're unsure about a function, use basic Python instead.
```

### Incomplete Implementations

**Problem:** AI returns stub code or leaves TODOs.

**Fix:**
```
Return a COMPLETE, WORKING implementation. No TODOs, no stubs, no "implement later".
If a feature is complex, implement a simplified but working version.
```

### Wrong Conventions

**Problem:** AI uses different style than your project.

**Fix:**
```
This project uses:
- snake_case for functions and variables
- PascalCase for classes
- Google-style docstrings
- Type hints on all functions
- Max line length: 88

Follow these conventions exactly.
```

---

## Exercise: Generate a Module

**Task:** Generate a complete Python module for a simple data structure.

**Prompt:**
```
Write a Python class called "Task" that represents a todo task.

Requirements:
- Attributes: title (str), description (str), completed (bool), priority (int 1-5)
- Methods: 
  - __init__ with validation
  - __str__ for readable output
  - __eq__ for comparison
  - toggle() to flip completed status
  - validate_priority() that raises ValueError if priority not 1-5
- Type hints on all methods
- Complete docstrings with examples
- Handle edge cases: empty title, negative priority

Return ONLY the code.
```

**Verify:**
1. Save to `task.py`
2. Import and test:
   ```python
   from task import Task
   
   t = Task("Buy milk", "Get 2% milk", priority=3)
   print(t)
   t.toggle()
   print(t.completed)  # True
   ```
3. Try edge cases:
   ```python
   Task("", "Empty title")  # Should raise ValueError
   Task("Test", priority=10)  # Should raise ValueError
   ```

---

## On Your Hardware

### 8GB RAM or Less

- Use `qwen2.5-coder:1.5b` or `phi3:mini`
- Keep prompts short and focused
- Generate one function at a time, not entire modules
- Accept that you may need to iterate more

### 16GB RAM

- Use `qwen2.5-coder:7b` (recommended)
- Can handle moderate prompts (entire classes or small modules)
- Good balance of speed and quality

### 32GB+ RAM

- Use `qwen2.5-coder:14b` or `32b`
- Can handle complex prompts (entire project scaffolds)
- Better at following multiple constraints simultaneously

---

## Next Steps

- [Chapter 2: Debug and Fix Bugs](./02-debug-fix.md) — Learn to diagnose and fix errors
- [Chapter 3: Write Tests](./03-write-tests.md) — Generate comprehensive test suites

Or understand the mechanics:
- [Chapter 6: How Your Agent Thinks](../02-understand/01-how-agents-think.md) — The Plan-Code-Verify loop

---

## Further Reading

- [Prompt Templates](../reference/prompt-templates.md) — Copy-paste system prompts
- [Models](../reference/models.md) — Model recommendations by hardware
- [Troubleshooting](../reference/troubleshooting.md) — Common issues and fixes
