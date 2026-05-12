# Chapter 12: Greenfield vs Brownfield

> How context changes everything between new and existing code.

---

## TL;DR

- **Greenfield (new code):** Agent can make architectural decisions, less context needed
- **Brownfield (existing code):** Agent must respect conventions, requires onboarding
- **Brownfield success:** Codebase analysis → Pattern detection → Convention adherence
- **Different prompts, different strategies** for each context

---

## The Fundamental Difference

When you ask an AI agent to work on code, the **context changes everything**:

| Aspect | Greenfield | Brownfield |
|---|---|---|
| **Architectural freedom** | High | Low (must follow existing) |
| **Context needed** | Minimal | Extensive (codebase onboarding) |
| **Risk** | Low (nothing to break) | High (can break existing) |
| **Agent role** | Architect + Builder | Integrator + Extender |
| **Success metric** | Does it work? | Does it fit? |

**Key insight:** The same agent will produce vastly different results depending on whether it's working on new code or existing code. Understanding this difference prevents frustration and failed attempts.

---

## Greenfield: Building from Scratch

### When You Have Greenfield

You're in greenfield mode when:
- Starting a new project from zero
- Creating a new module with no dependencies
- Prototyping a concept
- Building a proof-of-concept

### Greenfield Advantages

**1. Maximum Agent Freedom**

The agent can:
- Choose the best architecture
- Select appropriate patterns
- Make naming decisions
- Decide on file structure

**2. Minimal Context Required**

```
# Greenfield prompt (minimal context)
Create a Python CLI tool that:
- Takes a log file as input
- Parses each line as JSON
- Filters by log level
- Outputs filtered results

Use argparse, include tests, follow PEP 8.
```

**3. Faster Iteration**

No need to:
- Read existing code
- Understand conventions
- Worry about breaking things
- Match existing patterns

### Greenfield Prompt Pattern

```
Scaffold a new {language} {project-type} called "{name}" that {purpose}.

Requirements:
- Use {framework/tool} for {specific need}
- Include {config files: pyproject.toml, package.json, etc.}
- Include {tests} with {testing framework}
- Follow {style guide: PEP 8, Airbnb, etc.}
- Include {README.md} with usage examples

Return:
1. File structure first
2. Then content of each file
3. Include commands to run/test
```

**Example - Greenfield CLI Tool:**

```
Scaffold a new Python CLI tool called "logparser" that parses log files and filters by level.

Requirements:
- Use argparse for CLI arguments
- Include pyproject.toml with dependencies
- Include tests/ with pytest tests
- Follow PEP 8 style
- Include README.md with installation and usage

Return the file structure first, then the content of each file.
```

**Expected output:**
```
logparser/
├── pyproject.toml
├── README.md
├── src/
│   └── logparser/
│       ├── __init__.py
│       ├── parser.py
│       └── cli.py
└── tests/
    ├── __init__.py
    └── test_parser.py
```

### Greenfield Exercise: Project A

**Task:** Scaffold your CLI tool from scratch.

**Prompt:**
```
Scaffold a new Python CLI tool called "todo" that manages a todo list.

Requirements:
- Use argparse for CLI (commands: add, list, complete, delete)
- Store todos in a JSON file
- Include pyproject.toml
- Include tests with pytest
- Follow PEP 8
- Include README.md

Return file structure and all file contents.
```

**Success criteria:**
- [ ] Project structure created
- [ ] Can run `python -m logparser --help`
- [ ] Tests pass
- [ ] README has clear usage instructions

---

## Brownfield: Working with Existing Code

### When You Have Brownfield

You're in brownfield mode when:
- Adding features to existing projects
- Fixing bugs in production code
- Refactoring legacy code
- Contributing to open-source
- Migrating frameworks

### Brownfield Challenges

**1. Must Respect Conventions**

Every project has:
- Naming conventions (snake_case vs camelCase)
- Architecture patterns (MVC, layered, etc.)
- Error handling style (exceptions, return codes)
- Testing patterns (fixtures, mocks, etc.)
- Documentation style

**2. Requires Codebase Onboarding**

The agent **must** understand:
- Project structure
- Key files and their purposes
- Dependencies and their versions
- Build and test processes
- Coding standards

**3. Higher Risk**

Mistakes can:
- Break existing functionality
- Introduce regressions
- Violate project conventions
- Create merge conflicts

### Brownfield Onboarding Strategy

#### Step 1: Codebase Analysis

**Prompt:**
```
Analyze this project to understand its structure and conventions.

Reference these files:
- {entry point: main.py, app.py, index.js, etc.}
- {config: pyproject.toml, package.json, etc.}
- {key modules: src/, lib/, app/}
- {tests: tests/, __tests__/}

Answer:
1. What language, framework, and key libraries?
2. What is the entry point and overall architecture?
3. What coding conventions do you see (naming, style)?
4. How are tests organized?
5. How is the project built and run?
6. What patterns stand out (factory, observer, etc.)?

Be specific - reference actual file names and code snippets.
```

**Example output:**
```
1. Language: Python 3.11, Framework: FastAPI, Libraries: pydantic, sqlalchemy
2. Entry: main.py creates FastAPI app, architecture: layered (api/, core/, models/)
3. Conventions: snake_case, Google-style docstrings, type hints required
4. Tests: tests/ with pytest, one file per module, fixtures in conftest.py
5. Build: uv sync, run: uv run uvicorn main:app
6. Patterns: Dependency injection via FastAPI Depends, repository pattern for DB
```

#### Step 2: Pattern Detection

**Prompt:**
```
Look at these representative files:
- src/module_a.py
- src/module_b.py  
- tests/test_module_a.py

Identify patterns:
1. How are functions named and organized?
2. How is error handling done?
3. How are tests structured (fixtures, parametrization)?
4. What does a typical class look like?
5. How are imports organized?

Show me 2-3 line examples from actual code.
```

#### Step 3: Convention Confirmation

**Before writing code, have the agent confirm:**

```
Before I write the search feature, confirm you'll follow:
- snake_case for function and variable names
- Google-style docstrings with params and returns
- Type hints on all parameters and return values
- Custom exceptions in core/exceptions.py
- Tests in tests/ with pytest fixtures
- Import order: stdlib, third-party, local

Is this correct? What am I missing?
```

### Brownfield Prompt Pattern

```
Add {feature} to this existing {project-type}.

FIRST, understand the codebase:
- Read {file1} to understand {aspect}
- Read {file2} to understand {aspect}
- Look at {tests} to understand testing patterns

THEN, implement {feature} following existing patterns:
- Match the coding style
- Use the same patterns
- Add tests following test conventions
- Update {relevant docs}

Return only the new/changed files with diffs.
```

**Example - Brownfield Feature Addition:**

```
Add a search feature to this existing log parser.

FIRST, understand the codebase:
- Read src/logparser/parser.py to understand LogEntry structure
- Read src/logparser/cli.py to understand CLI argument patterns
- Look at tests/test_parser.py to understand testing patterns

THEN, add search functionality:
- Add a SearchFilter class following existing Filter patterns
- Add --search argument to CLI following argparse patterns
- Add tests in tests/test_search.py following pytest patterns
- Update README.md with search usage

Return only the new/changed files.
```

### Brownfield Exercise: Project B

**Task:** Contribute to an existing open-source project.

**Steps:**

1. **Pick a project**
   - Look for "good first issue" labels
   - Choose something small (bug fix or minor feature)
   - Ensure it uses a language you know

2. **Run onboarding analysis**
   ```
   Analyze this project: [repo URL]
   [Use Step 1 prompt from above]
   ```

3. **Find a good-first-issue**
   - Look for issues tagged "good first issue" or "help wanted"
   - Pick something that matches your skill level
   - Read the issue comments for context

4. **Implement following conventions**
   ```
   I want to fix issue #[number]: {issue description}
   
   FIRST, read:
   - {relevant files}
   
   THEN, implement the fix following existing patterns.
   Add tests if tests exist in this project.
   ```

5. **Submit PR**
   - Follow the project's CONTRIBUTING.md
   - Write a clear PR description
   - Respond to review feedback

**Timebox:** 2-3 hours

**Success criteria:**
- [ ] PR submitted (accepted or not, the attempt counts)
- [ ] Followed project conventions
- [ ] Tests pass (if project has tests)
- [ ] Code review feedback addressed

---

## When Context Blurs

Sometimes you're not purely greenfield or brownfield:

### Greenfield in Brownfield

**Scenario:** Adding a completely new module to an existing project.

**Strategy:**
1. Analyze the project for integration points
2. Follow conventions for new files
3. You have freedom for internal implementation

**Prompt:**
```
Add a new {module-name} module to this project.

FIRST, understand:
- Where new modules go (src/, lib/, etc.)
- How modules are imported
- Testing conventions

THEN, create the module:
- You have freedom for internal implementation
- But follow project conventions for:
  - File location
  - Import style
  - Testing approach
  - Documentation
```

### Brownfield in Greenfield

**Scenario:** Starting a new project but following a framework's conventions.

**Strategy:**
1. Treat the framework as "existing code"
2. Follow framework best practices
3. You have freedom within framework constraints

**Prompt:**
```
Create a new FastAPI project called "api".

Follow FastAPI best practices:
- Use dependency injection
- Use Pydantic models
- Organize with routers
- Include proper type hints

Include tests following FastAPI testing patterns.
```

---

## Common Mistakes

### Mistake 1: Treating Brownfield as Greenfield

**Wrong:**
```
Add search to this project. Make it fast and use whatever patterns you think work.
```

**Result:** Agent creates code that doesn't fit the project, gets rejected in review.

**Right:**
```
Add search to this project. First understand the codebase, then follow existing patterns.
```

### Mistake 2: Not Providing Enough Context for Brownfield

**Wrong:**
```
Fix the bug in the parser.
```

**Result:** Agent guesses at the codebase, makes wrong assumptions.

**Right:**
```
Fix the bug in the parser where {specific issue}.

Read src/logparser/parser.py first to understand the LogEntry structure and parsing logic.
The bug occurs when {specific condition}.
```

### Mistake 3: Over-Onboarding for Greenfield

**Wrong:**
```
Create a new CLI tool. First, analyze these 20 files from a different project to understand patterns.
```

**Result:** Wasted context, slower iteration.

**Right:**
```
Create a new CLI tool. Follow PEP 8 and include tests.
```

---

## Decision Tree

```
Are you working on existing code?
├── Yes → Brownfield
│   ├── Run codebase analysis
│   ├── Detect patterns
│   ├── Confirm conventions
│   └── Implement following patterns
│
└── No → Greenfield
    ├── Define requirements clearly
    ├── Specify constraints
    ├── Let agent make architectural decisions
    └── Iterate quickly
```

---

## Exercise Summary

### Greenfield Exercise (30 min)
1. Scaffold a new CLI tool
2. Run it and verify it works
3. Add a feature
4. Commit the work

### Brownfield Exercise (2-3 hours)
1. Pick an open-source project
2. Run onboarding analysis
3. Find and fix a good-first-issue
4. Submit a PR

---

## On Your Hardware

### Small Models (1.5B-3B)

- Greenfield only — small models struggle with brownfield onboarding
- Scaffold small projects (3-5 files max)
- Provide explicit conventions in prompts
- Expect more iteration on brownfield tasks

### Medium Models (7B-14B)

- Handle both greenfield and moderate brownfield
- Can onboard to small codebases (<50 files)
- Good at detecting conventions from existing code

### Large Models (32B+)

- Handle large brownfield codebases
- Better at understanding legacy patterns
- Can propose architectural improvements during onboarding

---

## Next Steps

- [Chapter 13: Refactoring](./04-refactor-migrate.md) - Learn to safely refactor and migrate code
- [Chapter 12 Reference](../reference/prompt-templates.md) - Copy-paste prompts for greenfield/brownfield
