# Chapter 15: Working Long-Term

> Session persistence, branch-per-task workflows, human-in-the-loop, and pair programming mode.

---

## TL;DR

- **Session persistence:** Resume work across days with saved context
- **Branch-per-task:** Clean git history, easy rollbacks
- **Human-in-the-loop:** Approval gates for risky operations
- **Pair programming vs autonomous:** Choose mode based on task importance

---

## Why Long-Term Workflows Matter

Short-term: You're learning how to prompt.
Long-term: You're building a **workflow that scales**.

The difference between:
- **Amateur:** Random prompts, messy git history, lost context
- **Professional:** Structured workflows, clean commits, persistent context

**Key insight:** The best agents aren't just about better prompts—they're about better workflows.

---

## Session Persistence

### The Problem

You have a great conversation with your agent:
- It understands your codebase
- It knows your conventions
- It's halfway through a feature

Then you close the terminal.

Next day: You start fresh. The agent has **no memory** of yesterday's work.

### The Solution: Session Persistence

Tools like Aider and Continue support **session files** that save conversation state.

### Aider Sessions

**Start a session:**
```bash
aider --session my-project.json src/
```

**What gets saved:**
- Conversation history
- File context (what files are in the chat)
- Project understanding
- Git state

**Resume a session:**
```bash
aider --session my-project.json
```

**The agent remembers:**
```
You: "Continue working on the search feature we started yesterday."
Agent: "Right, we were adding search to the log parser. I had just implemented the basic search filter. Should I add the regex support now?"
```

### Continue Sessions

**Save session:**
```json
// .continue/config.json
{
  "sessions": {
    "auto-save": true,
    "session-dir": "~/.continue/sessions"
  }
}
```

**List sessions:**
```bash
continue list-sessions
```

**Resume session:**
```bash
continue resume --session my-project
```

### Session Best Practices

**Do:**
- ✅ Save sessions for long-running features
- ✅ Name sessions descriptively (`feature/search`, `bugfix/memory-leak`)
- ✅ Clean up old sessions periodically
- ✅ Commit session files to git (if not sensitive)

**Don't:**
- ❌ Keep sessions open forever (context bloat)
- ❌ Share session files with secrets
- ❌ Rely on sessions for critical state (use git)
- ❌ Forget to save before closing

### When to Use Sessions

**Use sessions when:**
- Working on a feature over multiple days
- Agent needs to remember context across sessions
- You're doing pair programming mode
- Complex refactors that span hours

**Don't use sessions when:**
- Quick one-off prompts
- Agent is doing something wrong (start fresh)
- Context is getting too large
- You want a clean slate

---

## Branch-Per-Task Workflow

### The Problem

You let the agent work on your main branch:
```bash
git checkout main
aider "Add search feature"
# Agent makes 20 changes
git commit -m "add search"
```

**Problems:**
- Main branch is dirty during work
- Hard to revert if something breaks
- Mixed changes (search + bugfix + refactor)
- Can't easily review what changed

### The Solution: Branch-Per-Task

**Every task gets its own branch:**

```bash
# Start new task
git checkout -b feature/search

# Let agent work
aider "Add search feature"

# Review changes
git diff
git status

# Commit
git add .
git commit -m "feat: Add search feature with regex support"

# Merge or discard
git checkout main
git merge feature/search  # Accept
# or
git branch -D feature/search  # Reject
```

### Branch Naming Conventions

**Feature branches:**
```
feature/search
feature/user-auth
feature/api-endpoint
```

**Bugfix branches:**
```
bugfix/memory-leak
bugfix/null-pointer
bugfix/test-failure
```

**Refactor branches:**
```
refactor/parser
refactor/database
refactor/error-handling
```

**Hotfix branches:**
```
hotfix/security-vulnerability
hotfix/critical-bug
```

**Prefixes help with:**
- Organization (group by type)
- Automation (CI/CD rules)
- Clarity (what's this branch for?)

### Branch-Per-Task Benefits

| Benefit | Description |
|---|---|
| **Clean history** | Main branch always has working code |
| **Easy rollback** | Just delete the branch if it breaks |
| **Clear boundaries** | Each branch = one task |
| **Easy review** | `git diff` shows only task changes |
| **Parallel work** | Multiple branches for multiple tasks |
| **PR workflow** | Natural fit for pull request reviews |

### Branch-Per-Task Workflow

**Step 1: Create Branch**
```bash
git checkout main
git pull  # Get latest
git checkout -b feature/search
```

**Step 2: Let Agent Work**
```bash
aider "Add search feature to log parser"
# Agent makes changes
```

**Step 3: Review Changes**
```bash
git status  # What changed?
git diff    # Review changes
git diff --stat  # Summary
```

**Step 4: Test**
```bash
pytest  # Run tests
./run.sh  # Manual testing
```

**Step 5: Commit**
```bash
git add .
git commit -m "feat: Add search feature with regex support"

# Use conventional commits:
# feat: New feature
# fix: Bug fix
# refactor: Code restructuring
# docs: Documentation
# test: Adding tests
# chore: Maintenance
```

**Step 6: Merge or Discard**

**If it works:**
```bash
git checkout main
git merge feature/search
git branch -d feature/search  # Clean up
git push origin main
```

**If it breaks:**
```bash
git checkout main
git branch -D feature/search  # Delete branch
# Start over or try different approach
```

### Branch-Per-Task with Pull Requests

**For open-source or team work:**

```bash
# Create and push branch
git checkout -b feature/search
git push origin feature/search

# Create PR on GitHub/GitLab
# URL: github.com/username/repo/compare/feature/search

# Team reviews PR
# Address feedback
# Merge when approved
```

### Exercise: Branch-Per-Task

**Task:** Use branch-per-task for your next feature.

**Steps:**
1. Create new branch for feature
2. Let agent implement
3. Review changes with `git diff`
4. Run tests
5. Commit with conventional commit message
6. Merge to main or discard

**Timebox:** 30 minutes

---

## Human-in-the-Loop

### The Problem

You let the agent run autonomously:
```bash
aider --auto "Fix all bugs and optimize performance"
```

Agent:
- Deletes important files
- Commits broken code
- Pushes to production
- Runs destructive commands

**Result:** Disaster.

### The Solution: Approval Gates

Configure your agent to **ask before risky operations**:

### Aider Approval Settings

```bash
# Require approval for commands
aider --yes-always  # Disable (dangerous)
aider --no-yes-always  # Enable (safe)

# Or configure in config file
aider --config .aider.conf.yml
```

**`.aider.conf.yml`:**
```yaml
# Require approval for commands
require-approval: true

# Auto-commit changes
auto-commits: false

# Auto-save session
auto-save-session: true
```

### Continue Approval Settings

```json
// .continue/config.json
{
  "tools": {
    "requireApproval": ["run_command", "delete_file", "commit"]
  }
}
```

### What to Gate

**Always require approval:**
- ❌ Deleting files
- ❌ Running shell commands
- ❌ Committing code
- ❌ Pushing to remote
- ❌ Modifying production
- ❌ Running migrations

**Optional approval:**
- ⚠️ Creating new files
- ⚠️ Modifying existing files
- ⚠️ Running tests

**No approval needed:**
- ✅ Reading files
- ✅ Analyzing code
- ✅ Suggesting changes

### Human-in-the-Loop Workflow

**Pair Programming Mode (high involvement):**
```
1. Agent suggests change
2. You review the suggestion
3. You apply the change (or reject)
4. Agent continues

Best for: Critical code, new agent, learning
```

**Autonomous Mode (low involvement):**
```
1. You describe task
2. Agent works independently
3. You review at milestones
4. You approve final result

Best for: Routine tasks, trusted agent
```

**Hybrid Mode (balanced):**
```
1. Agent works on safe things automatically
2. Agent asks before risky operations
3. You review and approve
4. Agent continues

Best for: Most situations
```

### When to Stay in the Loop

**Always stay in the loop for:**
- Production code
- Security-sensitive changes
- Architecture decisions
- Database migrations
- First time with a new agent
- Critical bug fixes

**You can go autonomous for:**
- Prototypes
- Experiments
- Routine refactors
- Test generation
- Documentation
- Code you've reviewed before

---

## Pair Programming vs Autonomous

### Pair Programming Mode

**How it works:**
```
You: "I need a function to parse CSV files."
Agent: "Here's a draft. What do you think?"
You: "Good, but handle missing files."
Agent: "Updated. Better?"
You: "Perfect, let's add tests."
```

**Characteristics:**
- Back-and-forth conversation
- You guide every step
- Agent suggests, you decide
- High involvement

**Pros:**
- ✅ You learn the reasoning
- ✅ Full control over decisions
- ✅ Better for complex tasks
- ✅ Easier to correct course

**Cons:**
- ❌ Slower
- ❌ More effort from you
- ❌ Not scalable

**Best for:**
- Learning new patterns
- Critical production code
- Complex architecture
- First time with a feature

### Autonomous Mode

**How it works:**
```
You: "Add search feature to the log parser. Include regex support and tests."
Agent: [works for 10 minutes]
Agent: "Done. Here's what I changed: [diff]"
You: [reviews, approves, merges]
```

**Characteristics:**
- You describe the goal
- Agent figures out how
- You review at the end
- Low involvement

**Pros:**
- ✅ Faster
- ✅ Less effort from you
- ✅ Scalable
- ✅ Good for routine tasks

**Cons:**
- ❌ Less control
- ❌ Harder to correct mid-stream
- ❌ May not match your style
- ❌ Risk of surprises

**Best for:**
- Routine tasks
- Well-understood features
- Trusted agent setup
- When you're in a hurry

### Choosing the Right Mode

| Situation | Recommended Mode |
|---|---|
| **Learning something new** | Pair programming |
| **Production-critical code** | Pair programming |
| **Complex architecture** | Pair programming |
| **Routine refactoring** | Autonomous |
| **Test generation** | Autonomous |
| **Documentation** | Autonomous |
| **Bug fixes (non-critical)** | Autonomous |
| **First time with agent** | Pair programming |
| **Trusted workflow** | Autonomous |

### Switching Modes Mid-Task

You can switch modes during a task:

```
# Start in pair programming mode
You: "Let's design the search feature together."
Agent: [suggests architecture]
You: "Good, I like approach A."

# Once direction is set, go autonomous
You: "Okay, implement approach A with tests. I'll review when done."
Agent: [implements]
Agent: "Done. Here's the diff."
```

---

## Long-Term Workflow Checklist

### Daily Workflow

```
[ ] Start with clean git state
[ ] Create branch for task
[ ] Choose mode (pair programming vs autonomous)
[ ] Let agent work
[ ] Review changes
[ ] Run tests
[ ] Commit with conventional message
[ ] Merge or discard branch
[ ] Update session notes if needed
```

### Weekly Workflow

```
[ ] Review git history (what did agent do this week?)
[ ] Clean up old branches
[ ] Update prompt library
[ ] Note what worked / what didn't
[ ] Adjust workflow based on learnings
```

### Monthly Workflow

```
[ ] Review quality metrics (bug rate, test coverage)
[ ] Evaluate if agent is improving or degrading
[ ] Update model if needed
[ ] Refine prompts based on failures
[ ] Document new patterns that worked
```

---

## Exercise: Long-Term Workflow

### Exercise 1: Session Persistence (20 min)

**Task:** Use session persistence across a break.

**Steps:**
1. Start a session: `aider --session my-project`
2. Work on a feature for 10 minutes
3. Close the terminal
4. Resume session: `aider --session my-project`
5. Continue where you left off

**Success criteria:** Agent remembers context from before the break.

### Exercise 2: Branch-Per-Task (30 min)

**Task:** Use branch-per-task for a feature.

**Steps:**
1. Create branch: `git checkout -b feature/test-feature`
2. Let agent implement a feature
3. Review with `git diff`
4. Commit with conventional message
5. Merge to main

**Success criteria:** Clean git history, feature merged properly.

### Exercise 3: Human-in-the-Loop (15 min)

**Task:** Configure approval gates.

**Steps:**
1. Enable approval requirements in your tool
2. Ask agent to do something risky (delete a file)
3. Verify it asks for approval
4. Approve or reject

**Success criteria:** Agent asks before risky operations.

---

## Common Mistakes

### Mistake 1: No Branch Isolation

**Wrong:**
```bash
git checkout main
aider "Add feature"
# Agent modifies main directly
```

**Right:**
```bash
git checkout -b feature/new-feature
aider "Add feature"
# Agent works in isolated branch
```

### Mistake 2: Never Reviewing

**Wrong:**
```bash
aider --auto "Fix everything"
# Walk away, come back later
```

**Right:**
```bash
aider "Fix everything"
# Stay nearby, review changes as they happen
```

### Mistake 3: Session Bloat

**Wrong:**
```bash
aider --session my-project.json
# Keep same session open for weeks
# Context grows to 100k+ tokens
```

**Right:**
```bash
# Start fresh session for each major feature
aider --session feature/search.json
# Complete feature, start new session
aider --session feature/auth.json
```

### Mistake 4: No Commit Messages

**Wrong:**
```bash
git commit -m "update"
git commit -m "fix stuff"
```

**Right:**
```bash
git commit -m "feat: Add search feature with regex support"
git commit -m "fix: Handle empty search results gracefully"
```

---

## On Your Hardware

### Small Models (1.5B-3B)

- Short sessions (5-10 turns max) before context fills
- Summarize frequently
- Fresh session per task is essential
- Pair programming mode > autonomous mode

### Medium Models (7B-14B)

- Sessions of 15-20 turns manageable
- Can maintain project context across features
- Good balance of autonomy and reliability

### Large Models (32B+)

- Long sessions (30+ turns) feasible
- Better at maintaining project context
- Suitable for autonomous mode with approval gates

---

## Next Steps

- [Chapter 16: Your Setup](../04-daily-driver/01-your-setup.md) - Build your daily driver environment
- [Chapter 15 Reference](../reference/prompt-templates.md) - Copy-paste workflow templates
