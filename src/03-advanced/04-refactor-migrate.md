# Chapter 13: Refactoring and Migration

> Safe refactoring with diffs, framework migrations, and incremental strategies.

---

## TL;DR

- **Refactor safely:** Tests first, then diff-based changes
- **Migrate incrementally:** One piece at a time, verify at each step
- **Preserve behavior:** The "what" stays the same, the "how" changes
- **Use AI for scale:** Large refactors that would be tedious by hand

---

## Why Refactor with AI?

Manual refactoring is:
- **Tedious:** Changing 100 functions by hand
- **Error-prone:** Missing edge cases
- **Time-consuming:** Days of mechanical work

AI refactoring is:
- **Fast:** Minutes instead of days
- **Consistent:** Same pattern applied everywhere
- **Safe:** With tests, you catch regressions immediately

**But:** AI can also introduce subtle bugs. The key is **safe refactoring**.

---

## Safe Refactoring Principles

### 1. Tests Are Your Safety Net

**Golden rule:** Never refactor without tests.

**If you have tests:**
```
1. Run tests → All pass ✓
2. Ask AI to refactor
3. Run tests → All pass ✓
4. Commit
```

**If you don't have tests:**
```
1. Write tests for current behavior
2. Run tests → All pass ✓
3. Ask AI to refactor
4. Run tests → All pass ✓
5. Commit
```

**Prompt for test-first refactoring:**
```
Before refactoring this code, write tests that capture current behavior:

{paste code}

Requirements:
- Test all public functions
- Cover edge cases
- Use pytest
- Tests should pass with current code

Return only the test code.
```

### 2. Use Diffs, Not Full Files

**Why diffs?**
- Easier to review
- Clearer what changed
- Less context noise
- Git-friendly

**Prompt:**
```
Refactor this code to use dataclasses:

{paste code}

Requirements:
- Preserve all existing behavior
- Return as a unified diff
- Show only changes, not entire files
- Tests must still pass

Format as:
--- a/src/module.py
+++ b/src/module.py
@@ context @@
-old code
+new code
```

### 3. Small Steps, Frequent Verification

**Wrong approach:**
```
Refactor the entire codebase from functions to classes.
```

**Right approach:**
```
Refactor ModuleA from functions to classes.
Then I'll ask you to do ModuleB, ModuleC, etc.
```

**Why:** If the first refactor breaks something, you know exactly what went wrong.

---

## Refactoring Patterns

### Pattern 1: Functions → Classes

**When:** Code has related functions that share state.

**Prompt:**
```
Refactor these related functions into a class:

{paste code}

Requirements:
- Create a class named {ClassName}
- Move shared state to instance variables
- Keep the same public API
- Return as a diff

Example transformation:
- Before: `parse_data(data), validate_data(data), save_data(data)`
- After: `DataProcessor(data).parse().validate().save()`
```

### Pattern 2: Callbacks → Async/Await

**When:** Migrating from callback-based to async code.

**Prompt:**
```
Convert this callback-based code to async/await:

{paste code}

Requirements:
- Add async/await keywords
- Update function signatures
- Keep the same logic
- Return as a diff

Example:
- Before: `fetch_data(url, callback=handle_result)`
- After: `result = await fetch_data(url)`
```

### Pattern 3: String Formatting → f-strings

**When:** Modernizing Python 2.x or old Python 3.x code.

**Prompt:**
```
Convert all string formatting to f-strings:

{paste code}

Requirements:
- `.format()` → f-strings
- `%` formatting → f-strings
- Keep the same behavior
- Return as a diff

Example:
- Before: `"Hello, {}".format(name)`
- After: `f"Hello, {name}"`
```

### Pattern 4: Dictionary Access → Dataclasses

**When:** Code uses dicts for structured data.

**Prompt:**
```
Replace dict-based data with dataclasses:

{paste code}

Requirements:
- Create dataclasses for each dict structure
- Add type hints
- Update all access points
- Return as a diff

Example:
- Before: `user = {"name": "Alice", "age": 30}`
- After: `@dataclass class User: name: str, age: int`
```

---

## Framework Migration

### When to Migrate

**Reasons to migrate:**
- Framework is deprecated
- New framework has better performance
- Security updates stopped
- Team needs modern features

**Reasons to stay:**
- Migration cost > benefit
- Current framework works fine
- No urgent security issues

**Rule:** Don't migrate just because. Migrate when there's a clear benefit.

### Migration Strategy: Incremental

**Never:** Migrate everything at once.

**Always:** Migrate piece by piece.

### Example: Flask → FastAPI

#### Step 1: Side-by-Side Setup

```
# Keep Flask running
# Add FastAPI alongside it

flask_app/
├── app.py          # Existing Flask app
└── ...

fastapi_app/        # New FastAPI app (empty at first)
├── main.py
└── ...
```

#### Step 2: Migrate One Endpoint

**Prompt:**
```
Migrate this Flask endpoint to FastAPI:

{paste Flask route}

Requirements:
- Use FastAPI patterns
- Add Pydantic models for request/response
- Add type hints
- Keep the same behavior
- Return the FastAPI version

Flask code:
@app.route('/users/<int:user_id>')
def get_user(user_id):
    user = db.get_user(user_id)
    return jsonify(user)
```

**Expected FastAPI output:**
```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI()

class User(BaseModel):
    id: int
    name: str
    email: str

@app.get("/users/{user_id}", response_model=User)
async def get_user(user_id: int):
    user = db.get_user(user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user
```

#### Step 3: Test the Migrated Endpoint

```bash
# Test new FastAPI endpoint
curl http://localhost:8000/users/1

# Compare with old Flask endpoint
curl http://localhost:5000/users/1

# Verify they return the same result
```

#### Step 4: Repeat for Each Endpoint

```
Migrate /users → Test → Migrate /posts → Test → Migrate /comments → Test
```

#### Step 5: Switch Traffic

```
1. Deploy FastAPI alongside Flask
2. Route 10% of traffic to FastAPI
3. Monitor for errors
4. Increase to 50%, then 100%
5. Decommission Flask when confident
```

### Example: JavaScript → TypeScript

#### Step 1: Add TypeScript Config

```bash
npm install --save-dev typescript @types/node
npx tsc --init
```

#### Step 2: Rename One File

```
src/utils.js → src/utils.ts
```

#### Step 3: Add Types

**Prompt:**
```
Add TypeScript types to this JavaScript file:

{paste JS code}

Requirements:
- Add type annotations to all parameters
- Add return types
- Create interfaces for objects
- Use strict mode
- Return the TypeScript version

JS code:
function calculateTotal(items, taxRate) {
  const subtotal = items.reduce((sum, item) => {
    return sum + item.price * item.quantity
  }, 0)
  return subtotal * (1 + taxRate)
}
```

**Expected TypeScript output:**
```typescript
interface Item {
  price: number
  quantity: number
}

function calculateTotal(items: Item[], taxRate: number): number {
  const subtotal = items.reduce((sum: number, item: Item) => {
    return sum + item.price * item.quantity
  }, 0)
  return subtotal * (1 + taxRate)
}
```

#### Step 4: Fix Type Errors

```bash
npx tsc --noEmit  # Check for errors
# Fix errors iteratively
```

#### Step 5: Repeat for Each File

```
utils.ts → auth.ts → database.ts → api.ts → ...
```

---

## Refactoring Without Tests

Sometimes you **must** refactor code without tests. Here's how to stay safe:

### Strategy 1: Manual Verification Checklist

**Before refactoring:**
```
[ ] Document current behavior
[ ] Note edge cases
[ ] Record sample inputs/outputs
[ ] Take screenshots if UI
```

**After refactoring:**
```
[ ] Run all sample inputs
[ ] Verify outputs match
[ ] Test edge cases
[ ] Check UI screenshots
```

**Prompt:**
```
Before refactoring, document the current behavior:

{paste code}

For each public function:
1. What does it do?
2. What are the edge cases?
3. What are some example inputs/outputs?

Return a markdown table.
```

### Strategy 2: Small Batches

**Wrong:**
```
Refactor the entire authentication module.
```

**Right:**
```
Refactor just the password hashing function.
I'll verify it works, then ask you to do the next function.
```

### Strategy 3: Manual Test Harness

**Prompt:**
```
Create a manual test script for this code:

{paste code}

Requirements:
- Test each public function
- Include edge cases
- Print inputs and outputs
- Make it easy to run

Return the test script.
```

**Then:**
```bash
# Before refactoring
python test_harness.py > before.txt

# After refactoring
python test_harness.py > after.txt

# Compare
diff before.txt after.txt
```

---

## Exercise: Refactor Your Code

### Exercise 1: With Tests (30 min)

**Task:** Refactor a module that has tests.

**Steps:**
1. Pick a module from Project A with tests
2. Run tests → Verify all pass
3. Ask AI to refactor (e.g., functions → classes)
4. Run tests → Verify all pass
5. Review the diff
6. Commit the change

**Prompt:**
```
Refactor src/logparser/parser.py to use a Parser class instead of standalone functions.

Requirements:
- Preserve all existing behavior
- Keep the same public API
- Return as a unified diff
- Tests must still pass
```

### Exercise 2: Without Tests (45 min)

**Task:** Refactor code without tests safely.

**Steps:**
1. Pick a small module without tests
2. Create a manual test harness
3. Document current behavior
4. Ask AI to refactor
5. Run test harness → Compare outputs
6. Commit if everything matches

**Prompt:**
```
Before refactoring, document the current behavior of this code:

{paste code}

Then create a test script that exercises all functions with sample inputs.
```

### Exercise 3: Migration — Project C (Optional, 2+ hours)

**Task:** Migrate a real project using AI.

**Scenarios:**
- Python 2.x → Python 3.x
- Flask → FastAPI
- jQuery → Vanilla JS or a framework
- Class-based components → Functional components + hooks
- Any framework migration your team is considering

**Steps:**
1. Pick a small migration task (one file, one component, one endpoint)
2. Set up the target environment (install new framework, config, etc.)
3. Migrate one piece at a time
4. Test after each migration step
5. Commit each step
6. Gradually increase migration scope as confidence grows

**Prompt:**
```
I'm migrating from {source} to {target}.

Here's my current code:
{paste code}

Requirements:
- Use {target} idioms and patterns
- Preserve all behavior
- Return as a diff
- Note any behavior that can't be preserved
- List any dependencies that need updating

Target code should be:
{paste target framework examples or links}
```

**Success criteria:**
- [ ] Migrated code passes tests
- [ ] No behavior regression detected
- [ ] Migration steps committed incrementally

---

## Common Pitfalls

### Pitfall 1: Over-Refactoring

**Mistake:** Refactoring code that works fine.

**Rule:** Only refactor when:
- Code is hard to understand
- You're adding features and hitting friction
- There's a clear benefit (performance, maintainability)

**Ask:** "Will this refactor make future work easier?" If no, don't do it.

### Pitfall 2: Ignoring Tests

**Mistake:** Refactoring without running tests.

**Rule:** Always run tests:
- Before refactoring (baseline)
- After refactoring (verification)
- During refactoring (if incremental)

### Pitfall 3: Big Bang Migrations

**Mistake:** Migrating the entire codebase at once.

**Rule:** Migrate incrementally:
- One file at a time
- One feature at a time
- Verify at each step

### Pitfall 4: Changing Behavior

**Mistake:** "Refactoring" that changes what the code does.

**Rule:** Refactoring preserves behavior. If you're changing behavior, you're not refactoring—you're rewriting.

**Prompt fix:**
```
Refactor this code to {new pattern}.

CRITICAL: Preserve all existing behavior. The output must be identical
to the current output for all inputs. Only change the implementation,
not the interface or behavior.
```

---

## Migration Decision Matrix

| Migration Type | Risk | Effort | When to Do |
|---|---|---|---|
| **Syntax update** (Python 2→3) | Low | Medium | When security/support requires it |
| **Framework upgrade** (Flask 1→2) | Medium | Medium | When new features needed |
| **Framework migration** (Flask→FastAPI) | High | High | When clear benefit (performance, team needs) |
| **Language migration** (JS→TS) | Medium | High | When team wants type safety |
| **Architecture change** (monolith→microservices) | Very High | Very High | Only when scaling requires it |

---

## On Your Hardware

### Small Models (1.5B-3B)

- Refactoring is risky — small models often change behavior unintentionally
- Stick to simple renames and extractions
- Always run tests after every change
- Avoid framework migrations with small models

### Medium Models (7B-14B)

- Handle moderate refactoring (extract functions, reorganize modules)
- Can perform simple migrations (e.g., Python 2→3 syntax)
- Use incremental approach: one file at a time

### Large Models (32B+)

- Capable of complex refactoring and migrations
- Better at preserving behavior during transformation
- Can handle framework migrations with verification

---

## Next Steps

- [Chapter 14: Patterns](./05-patterns.md) - Learn patterns for better results
- [Chapter 13 Reference](../reference/prompt-templates.md) - Copy-paste refactoring prompts
