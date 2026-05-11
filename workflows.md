# Coding Agent Workflows: End-to-End Topic List

> Core workflows and logical topic groupings for building, running, and
> improving local coding agents on constrained systems.

This document maps **what coding agents actually do** (workflows) against
**what you need to know** (topics). Use it as a checklist for expanding this guide.

---

## Project Contexts: Where the Agent Operates

Before the workflows, understand the **context** — the nature of the codebase
and project dramatically changes what the agent needs to do.

| Context | Description | Agent Challenges |
|---|---|---|
| **Greenfield** | New project, no existing code | Agent defines architecture, conventions, structure from scratch |
| **Brownfield** | Existing codebase, legacy code | Agent must understand, respect, and navigate existing patterns |
| **Spaček / Script** | Single-file scripts, one-off tools | Fast, focused, no architecture needed |
| **Library / Package** | Shared code with public API | Agent must preserve API contracts, versioning, backward compat |
| **Monorepo** | Multiple projects in one repo | Agent navigates cross-project dependencies, shared configs |
| **Service / Microservice** | Networked services with contracts | Agent understands APIs, schemas, inter-service calls |
| **Data / ML Pipeline** | Notebooks, data processing, training | Agent handles data schemas, experiment tracking, reproducibility |
| **Infrastructure / DevOps** | Docker, Terraform, CI/CD, configs | Agent works with declarative configs, not traditional code |
| **Embedded / Systems** | C, Rust, firmware, constrained targets | Agent deals with memory, hardware, build toolchains |
| **Research / Prototyping** | Exploratory code, rapid iteration | Agent tolerates messiness, focuses on speed over polish |

**Why this matters:** A greenfield agent can make architectural decisions.
A brownfield agent must reverse-engineer them. The same "code generation"
workflow looks completely different in each context.

---

## Part A: End-to-End Core Workflows

These are the complete workflows a coding agent performs, from start to finish.
Organized by **phase of the development lifecycle**, not just by task type.

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

### Workflow 9: Architecture and Scaffolding

> The agent designs and bootstraps a new project or major component.

**Steps:**
1. Understand the requirements and constraints
2. Choose architecture (framework, patterns, structure)
3. Generate project scaffold (directories, configs, boilerplate)
4. Set up build system, linter, formatter, CI
5. Create initial module/service structure
6. Document the architectural decisions

**Key techniques needed:**
- Architecture pattern knowledge (MVC, clean arch, hexagonal, etc.)
- Framework conventions and best practices
- Config file generation (package.json, tsconfig, docker-compose, etc.)
- Decision documentation (ADR-style)
- Multi-file coordinated generation

**Contexts:** Primarily greenfield, but also brownfield (new module in existing project)

**Failure modes:**
- Over-engineering simple projects
- Choosing wrong framework/architecture for the domain
- Inconsistent conventions across scaffolded files
- Missing essential configs (secrets, env, CI)

---

### Workflow 10: Code Migration and Modernization

> The agent migrates code between languages, frameworks, or major versions.

**Steps:**
1. Analyze the source code and understand its behavior
2. Understand the target (new language, framework version, platform)
3. Map source constructs to target equivalents
4. Generate migrated code
5. Port and adapt tests
6. Validate behavior is preserved
7. Handle incremental migration (strangler pattern)

**Key techniques needed:**
- Cross-language semantic understanding
- API mapping (old → new)
- Breaking change detection
- Incremental migration strategies
- Behavior preservation verification

**Contexts:** Brownfield almost exclusively

**Common migrations:**
- Language: Python 2→3, JS→TS, Java→Kotlin
- Framework: jQuery→React, Django 1→4, AngularJS→Angular
- Platform: Monolith→microservices, server→edge
- Build: Webpack→Vite, Maven→Gradle

**Failure modes:**
- Semantic drift (migrated code behaves differently)
- Missing edge cases in the mapping
- Not migrating related files (configs, tests, docs)
- All-or-nothing approach instead of incremental

---

### Workflow 11: Dependency and Package Management

> The agent manages dependencies, resolves conflicts, and audits packages.

**Steps:**
1. Audit current dependencies (versions, vulnerabilities, duplicates)
2. Identify updates needed (security, features, compatibility)
3. Plan the upgrade path (breaking changes, order of operations)
4. Update dependency files and code that uses them
5. Run tests to verify compatibility
6. Lock versions and document changes

**Key techniques needed:**
- Understanding dependency graphs
- Semantic versioning and breaking change detection
- Package manager tool use (npm, pip, cargo, go mod)
- Vulnerability scanning interpretation
- Conflict resolution strategies

**Failure modes:**
- Blindly updating everything → cascade of breakage
- Missing transitive dependency issues
- Not updating code that depends on changed APIs
- Ignoring lock file implications

---

### Workflow 12: Configuration and Environment Setup

> The agent creates, debugs, and manages project configuration and environments.

**Steps:**
1. Understand the target environment (dev, staging, production, Docker, cloud)
2. Generate or fix configuration files
3. Set up environment variables and secrets management
4. Configure CI/CD pipelines
5. Debug environment-specific issues

**Key techniques needed:**
- Config file formats (YAML, JSON, TOML, HCL, INI)
- Environment variable patterns and secrets management
- Docker/container configuration
- CI/CD pipeline syntax (GitHub Actions, GitLab CI, Jenkins)
- Cloud platform configs (AWS, GCP, Azure, K8s manifests)

**Failure modes:**
- Hardcoded secrets in config files
- Environment-specific bugs (works on my machine)
- Misconfigured CI/CD that silently passes
- Inconsistent configs across environments

---

### Workflow 13: Data Schema and Query Workflows

> The agent works with databases, data schemas, migrations, and queries.

**Steps:**
1. Understand the data model and schema
2. Write or optimize queries (SQL, GraphQL, NoSQL)
3. Generate schema migrations
4. Design data access layers (ORM, repositories)
5. Handle data transformations and ETL logic

**Key techniques needed:**
- SQL query generation and optimization
- Schema design and normalization
- Migration file generation (up/down pairs)
- ORM mapping (Prisma, SQLAlchemy, TypeORM, etc.)
- Data validation and seeding

**Contexts:** Full-stack apps, data pipelines, analytics

**Failure modes:**
- N+1 query problems
- Destructive migrations without backups
- Schema drift (code and DB out of sync)
- SQL injection in generated queries

---

### Workflow 14: API Design and Contract Generation

> The agent designs APIs, generates clients, and maintains contracts.

**Steps:**
1. Define API requirements and constraints
2. Design endpoints, schemas, and data models
3. Generate server-side implementation scaffolding
4. Generate client SDKs or types
5. Create OpenAPI/JSON Schema specs
6. Generate mock servers for development

**Key techniques needed:**
- REST/GraphQL/gRPC design patterns
- OpenAPI/Swagger spec generation
- Type generation from specs (openapi-typescript, etc.)
- Versioning strategies
- Contract testing

**Failure modes:**
- Inconsistent API design patterns
- Generated client out of sync with server
- Missing error response definitions
- Poor versioning leading to breaking changes

---

### Workflow 15: Performance Profiling and Optimization

> The agent identifies and fixes performance bottlenecks.

**Steps:**
1. Run profiling tools and capture metrics
2. Analyze profiles to identify bottlenecks
3. Categorize the issue (CPU, memory, I/O, network, algorithm)
4. Propose and implement optimizations
5. Re-profile and verify improvement
6. Document the optimization and its trade-offs

**Key techniques needed:**
- Profiling tool interpretation (flame graphs, traces, logs)
- Algorithm complexity analysis
- Caching strategies
- Database query optimization
- Memory leak detection
- Language-specific optimization patterns

**Failure modes:**
- Premature optimization
- Optimizing the wrong bottleneck
- Trading readability for marginal gains
- Introducing bugs during optimization

---

### Workflow 16: Incident Response and Hotfix

> The agent helps diagnose and fix production issues under time pressure.

**Steps:**
1. Parse incident report (error logs, monitoring alerts, user reports)
2. Triage severity and impact
3. Locate the root cause in the codebase
4. Write a minimal, safe fix
5. Write a regression test
6. Generate a rollback plan
7. Document the incident and fix

**Key techniques needed:**
- Log analysis and pattern matching
- Monitoring/alert interpretation
- Minimal-change fix strategy (surgical, not refactoring)
- Rollback planning
- Post-mortem documentation

**Contexts:** Brownfield, production environments

**Failure modes:**
- Over-reaching fixes that introduce new issues
- Not writing regression tests
- No rollback plan
- Blaming instead of diagnosing

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

### Topic 13: Project Context Adaptation

- Greenfield vs brownfield agent behavior differences
- Codebase onboarding strategies (how the agent learns a new project)
- Convention detection and adherence
- Legacy code handling (comments, patterns, technical debt)
- Monorepo navigation and cross-project awareness
- Framework-specific agent knowledge loading

### Topic 14: Human-Agent Collaboration Patterns

- Agent as pair programmer (suggest, human decides)
- Agent as autonomous worker (full task execution)
- Agent as reviewer (human writes, agent critiques)
- Approval gates and human-in-the-loop checkpoints
- Communication patterns (what the agent explains vs does silently)
- Trust calibration (when to trust the agent vs verify)

### Topic 15: State and Session Management

- Multi-session continuity (resuming work across days)
- Change tracking (what the agent has already done)
- Undo/rollback of agent-made changes
- Branch-per-task workflows
- Work-in-progress saving and recovery
- Context handoff between sessions

### Topic 16: Domain-Specific Agent Knowledge

- Web development (HTML/CSS/JS, frameworks, deployment)
- Backend services (APIs, databases, caching, queues)
- Mobile development (iOS, Android, cross-platform)
- Data science / ML (notebooks, pipelines, experiments)
- DevOps / SRE (infrastructure, monitoring, incident response)
- Security (vulnerability scanning, secure coding patterns)
- Embedded / systems programming (memory, hardware, toolchains)

### Topic 17: Output Delivery and Integration

- How the agent delivers its work (file writes, diffs, patches, PRs)
- IDE plugin integration
- Terminal/CLI agent interfaces
- Git integration (commits, branches, PR descriptions)
- CI/CD pipeline integration
- Chat-based interfaces (Slack, Discord, Teams)
- Headless/API mode for automation

---

## Part C: Suggested Expansion Priority

Based on impact vs effort for constrained systems:

### Foundational (Build These First)

| Priority | Item | Why |
|---|---|---|
| **P0** | Workflow 2: Code Generation | Core capability, highest demand |
| **P0** | Topic 1: Model Selection | Foundation for everything else |
| **P0** | Topic 3: Structured Code Output | Without this, nothing is usable |
| **P0** | Topic 13: Project Context Adaptation | Greenfield vs brownfield changes everything |

### High-Leverage (Build Next)

| Priority | Item | Why |
|---|---|---|
| **P1** | Workflow 4: Debugging | High value, fundamentally different from generation |
| **P1** | Topic 4: Tool Ecosystem | Tools enable all workflows |
| **P1** | Topic 12: Prompt Engineering for Code | Biggest leverage for quality |
| **P1** | Workflow 1: Project Understanding | Brownfield work starts here |

### Quality Loop (Build After Core Works)

| Priority | Item | Why |
|---|---|---|
| **P2** | Workflow 5: Test Writing | Closes the quality loop |
| **P2** | Workflow 8: Build-Run-Iterate | Full autonomy requires this |
| **P2** | Topic 5: Self-Correction Loops | Quality multiplier |
| **P2** | Workflow 9: Architecture/Scaffolding | Greenfield projects need this |
| **P2** | Topic 15: State and Session Management | Multi-session work requires it |

### Production-Ready (Build When Core is Solid)

| Priority | Item | Why |
|---|---|---|
| **P3** | Workflow 6: Code Review | Adds value but needs solid generation first |
| **P3** | Workflow 10: Migration | High-value but complex |
| **P3** | Workflow 11: Dependency Management | Often overlooked, high pain |
| **P3** | Topic 7: Evaluation | Need to measure before optimizing |
| **P3** | Topic 6: Security | Critical for production |
| **P3** | Workflow 16: Incident Response | High-stakes, specific patterns |

### Advanced (Nice-to-Have)

| Priority | Item | Why |
|---|---|---|
| **P4** | Topic 8: Multi-Agent | Premature for constrained systems |
| **P4** | Topic 10: Memory across sessions | Complex, diminishing returns |
| **P4** | Workflow 15: Performance Optimization | Specialized, needs profiling expertise |
| **P4** | Topic 17: Output Delivery / Integration | Depends on deployment context |

---

*This is a living topic list. Mark items as [WIP], [DONE], or [DEFERRED] as we expand the guide.*
