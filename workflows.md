# Coding Agent Workflows: End-to-End Topic List

> Core workflows and logical topic groupings for building, running, and
> improving local coding agents on constrained systems.

This document maps **what coding agents actually do** (workflows) against
**what you need to know** (topics). Use it as a checklist for expanding this guide.

---

## Part A: End-to-End Core Workflows

These are the complete workflows a coding agent performs, from start to finish.
Each workflow is a self-contained capability that builds on the previous ones.

### Workflow 1: Project Understanding & Context Loading

> The agent understands an existing codebase before making changes.

**Steps:**
1. Discover project structure (file tree, language, framework)
2. Load relevant files into context (selectively, not everything)
3. Identify entry points, dependencies, and architecture patterns
4. Build a mental model of the codebase

**Key techniques needed:**
- File tree analysis and summarization
- Selective context loading (what files matter for what task)
- Architecture pattern recognition
- Dependency mapping
- Token budget management for large codebases

**Failure modes:**
- Loading too much context → context overflow
- Loading wrong files → irrelevant suggestions
- Missing key files → broken changes

---

### Workflow 2: Code Generation from Specification

> The agent writes new code from a description or requirement.

**Steps:**
1. Parse the specification/requirement
2. Plan the implementation (file structure, functions, classes)
3. Generate code with proper structure, types, and documentation
4. Self-review the generated code
5. Output the code in an editable format

**Key techniques needed:**
- Specification parsing and clarification
- Implementation planning (pseudo-code → real code)
- Language-specific code generation patterns
- Self-review / self-correction loops
- Output formatting (file diffs, full files, patches)

**Failure modes:**
- Hallucinated APIs or libraries
- Incomplete implementations
- Wrong language/framework conventions
- Missing error handling

---

### Workflow 3: Code Modification & Refactoring

> The agent modifies existing code: bug fixes, feature additions, refactoring.

**Steps:**
1. Locate the relevant code (search, grep, file navigation)
2. Understand the existing implementation
3. Plan the modification (what to change, what to preserve)
4. Generate the modified code (preferably as a diff/patch)
5. Verify the change doesn't break related code

**Key techniques needed:**
- Code search and navigation
- Diff/patch generation and application
- Impact analysis (what else might break)
- Refactoring patterns (extract, rename, restructure)
- Preserving code style and conventions

**Failure modes:**
- Modifying wrong location
- Breaking unrelated functionality
- Inconsistent style
- Missing related changes (e.g., updating imports)

---

### Workflow 4: Debugging & Error Resolution

> The agent diagnoses and fixes bugs using error messages, logs, and reasoning.

**Steps:**
1. Receive error message, stack trace, or bug description
2. Locate the source of the error (file, line, function)
3. Analyze the root cause (not just the symptom)
4. Propose and implement a fix
5. Verify the fix (run tests, check edge cases)

**Key techniques needed:**
- Error message parsing and interpretation
- Stack trace analysis
- Root cause reasoning (distinguish symptom from cause)
- Hypothesis generation and testing
- Edge case identification

**Failure modes:**
- Fixing the symptom, not the cause
- Introducing new bugs in the fix
- Missing edge cases
- Infinite debugging loops

---

### Workflow 5: Test Writing & Execution

> The agent writes, runs, and fixes tests.

**Steps:**
1. Understand the code to be tested
2. Identify test cases (happy path, edge cases, error cases)
3. Write test code using the appropriate framework
4. Run tests and capture results
5. Fix failures (in tests or in source code)
6. Achieve target coverage

**Key techniques needed:**
- Test case generation from code analysis
- Framework-specific test patterns (pytest, jest, etc.)
- Mock/stub creation
- Test execution and result parsing
- Coverage analysis
- Fix loop (test failure → fix → re-run)

**Failure modes:**
- Tests that don't actually test anything
- Over-mocking (tests pass but code is broken)
- Missing edge cases
- Flaky tests

---

### Workflow 6: Code Review & Quality Improvement

> The agent reviews code for quality, security, and best practices.

**Steps:**
1. Load the code under review
2. Analyze against quality criteria (correctness, style, security, performance)
3. Identify issues and categorize by severity
4. Suggest specific improvements with code examples
5. Prioritize recommendations

**Key techniques needed:**
- Static analysis patterns
- Security vulnerability detection (basic)
- Code style enforcement
- Performance anti-pattern detection
- Constructive feedback generation

**Failure modes:**
- Nitpicking style over substance
- Missing real bugs
- False positives on security
- Overwhelming the developer with suggestions

---

### Workflow 7: Documentation Generation

> The agent writes or updates documentation from code.

**Steps:**
1. Analyze the code (functions, classes, modules, APIs)
2. Extract intent, parameters, return values, side effects
3. Generate documentation in the target format
4. Include examples and usage patterns
5. Update existing docs to stay in sync

**Key techniques needed:**
- Code-to-docs extraction
- Docstring generation (Google, NumPy, JSDoc styles)
- README and architecture doc generation
- API reference generation
- Example code generation

**Failure modes:**
- Generic, unhelpful descriptions
- Outdated documentation
- Missing important details (side effects, exceptions)
- Wrong format/style

---

### Workflow 8: Build, Run, and Iterate Loop

> The agent manages the full development cycle: write → build → test → fix → repeat.

**Steps:**
1. Write or modify code
2. Run the build/compilation
3. Run the test suite
4. Parse build/test output for failures
5. Diagnose and fix failures
6. Repeat until green
7. Commit with meaningful message

**Key techniques needed:**
- Build system understanding (Make, npm, Maven, etc.)
- Test framework integration
- Output parsing (compiler errors, test failures)
- Iterative fix loops with memory
- Git operations (diff, commit, branch)

**Failure modes:**
- Infinite fix loops
- Fixing tests instead of code
- Losing context across iterations
- Not knowing when to stop

---

## Part B: Logical Topic Groupings

These are the cross-cutting concerns and foundational topics that support
all workflows above.

### Topic 1: Model Selection for Coding Tasks

- Which models are best at coding (vs general tasks)?
- Coding-specific benchmarks (HumanEval, MBPP, SWE-bench)
- Quantization impact on code quality
- Model size vs coding capability curve
- Language-specific model strengths

### Topic 2: Code-Aware Context Management

- How to load code into context efficiently
- File selection strategies (relevant files only)
- Code summarization techniques
- Handling large files and large codebases
- Context window budgeting for code tasks
- External knowledge bases (code indexes, vector stores)

### Topic 3: Structured Code Output

- Generating diffs and patches
- File-level vs function-level output
- Ensuring syntactically valid code
- Language-specific formatting
- Output validation (can it actually compile/run?)
- Handling partial/incomplete code generation

### Topic 4: Tool Ecosystem for Coding Agents

- **File operations:** read, write, search, grep, find
- **Code analysis:** linters, type checkers, AST parsers
- **Build/test:** compiler, test runner, coverage tools
- **Version control:** git diff, git log, git blame
- **Runtime:** executing code, capturing output, debugging
- **Package management:** dependency resolution, install
- Designing tools that small models can use reliably

### Topic 5: Self-Correction and Verification Loops

- Self-review patterns for code
- Compile-and-fix loops
- Test-driven generation (write tests first, then code)
- Multi-pass refinement (draft → review → revise)
- When to stop iterating (convergence detection)
- Cost of verification vs benefit

### Topic 6: Security and Safety for Coding Agents

- Preventing the agent from executing dangerous code
- Sandboxing code execution
- Preventing credential leaks in generated code
- Input validation for agent tools
- Preventing prompt injection through code files
- Audit trails for all code changes

### Topic 7: Evaluation of Coding Agents

- Automated evaluation (does the code compile? do tests pass?)
- HumanEval/MBPP-style benchmarks
- SWE-bench for real-world task evaluation
- Measuring code quality (not just correctness)
- Regression testing for agent improvements
- Cost/quality trade-off analysis

### Topic 8: Multi-Agent Coding Patterns

- Specialist agents (coder, reviewer, tester, architect)
- Sequential vs parallel agent pipelines
- Agent handoff patterns
- Shared context vs isolated context
- When multi-agent helps vs hurts for coding
- "Fake multi-agent" with single model role-switching

### Topic 9: Integration with Development Environments

- IDE integration (VS Code, JetBrains)
- Terminal-based agent workflows
- CI/CD integration (agent as a build step)
- Git hook integration (pre-commit agent review)
- Chat-based interfaces (Slack, Discord)
- API-based integration (headless agent as service)

### Topic 10: Memory and Learning Across Sessions

- Persistent memory of codebase knowledge
- Learning from past mistakes
- Storing and reusing successful patterns
- Session continuity (resuming interrupted work)
- Personalization (learning developer preferences)
- Vector-based code memory

### Topic 11: Language and Framework Specific Patterns

- Python coding agents (pytest, type hints, virtualenvs)
- JavaScript/TypeScript agents (npm, jest, tsconfig)
- Java/Kotlin agents (Maven/Gradle, JUnit)
- Go agents (go test, go mod)
- Rust agents (cargo, clippy)
- Framework-specific knowledge (React, Django, Spring, etc.)
- Polyglot agents (handling multiple languages)

### Topic 12: Prompt Engineering for Code

- Code-specific system prompts
- Few-shot examples for code generation
- Specifying coding standards and conventions
- Prompt patterns for each workflow (generate, debug, review, test)
- Temperature and sampling for code (low temp for correctness)
- Constrained decoding for code structure

---

## Part C: Suggested Expansion Priority

Based on impact vs effort for constrained systems:

| Priority | Workflow / Topic | Why |
|---|---|---|
| **P0** | Workflow 2: Code Generation | Core capability, highest demand |
| **P0** | Topic 1: Model Selection | Foundation for everything else |
| **P0** | Topic 3: Structured Code Output | Without this, nothing is usable |
| **P1** | Workflow 4: Debugging | High value, different from generation |
| **P1** | Topic 4: Tool Ecosystem | Tools enable all workflows |
| **P1** | Topic 12: Prompt Engineering for Code | Biggest leverage for quality |
| **P2** | Workflow 5: Test Writing | Closes the quality loop |
| **P2** | Workflow 8: Build-Run-Iterate | Full autonomy requires this |
| **P2** | Topic 5: Self-Correction Loops | Quality multiplier |
| **P3** | Workflow 6: Code Review | Adds value but needs solid generation first |
| **P3** | Topic 7: Evaluation | Need to measure before optimizing |
| **P3** | Topic 6: Security | Critical for production, less for dev |
| **P4** | Topic 8: Multi-Agent | Premature optimization for constrained systems |
| **P4** | Topic 10: Memory | Nice-to-have, complex to implement well |

---

*This is a living topic list. Mark items as [WIP], [DONE], or [DEFERRED] as we expand the guide.*
