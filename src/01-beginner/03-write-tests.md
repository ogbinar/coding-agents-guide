# Chapter 3: Write Tests

> Generate comprehensive test suites with AI, covering happy paths, edge cases, and error handling.

---

## TL;DR

- AI can write tests covering happy path, edge cases, and error cases
- Specify test framework and conventions explicitly
- Run tests and iterate on failures
- Target 80%+ coverage for critical code
- Use AI to fix failing tests, not just write them

---

## Prerequisites

- Completed [Chapter 1](./01-write-code.md) and [Chapter 2](./02-debug-fix.md)
- Have code to test
- Basic familiarity with pytest or your preferred test framework

---

## Why Test Generation Matters

Manual test writing is tedious. AI excels at:
- Generating boilerplate test code
- Finding edge cases you might miss
- Creating comprehensive coverage
- Maintaining test consistency

**But:** AI can miss subtle bugs. Always review tests critically.

---

## The Test Generation Loop

```
┌─────────────────────────────────────────────────────┐
│            Test Generation Loop                      │
│                                                      │
│  1. ANALYZE: AI reads the code to understand         │
│     what needs testing                               │
│     ↓                                                │
│  2. PLAN: AI outlines test cases:                    │
│     - Happy path (normal usage)                      │
│     - Edge cases (boundaries, empty, extreme)        │
│     - Error cases (invalid inputs, exceptions)       │
│     ↓                                                │
│  3. GENERATE: AI writes test code                    │
│     ↓                                                │
│  4. RUN: Execute tests                               │
│     ↓                                                │
│  5. FIX: If tests fail, diagnose and fix:            │
│     - Bug in implementation? → Fix code              │
│     - Wrong test expectation? → Fix test             │
│     ↓                                                │
│  6. VERIFY: All tests pass → Done                    │
└─────────────────────────────────────────────────────┘
```

---

## The Prompt Pattern

### Basic Test Generation

```
Write tests for this code using {test framework}:

Code:
{paste code}

Requirements:
- Cover happy path (normal usage)
- Cover edge cases: {specific edge cases}
- Cover error cases: invalid inputs, exceptions
- Use descriptive names: test_{function}_{scenario}
- Each test should be independent (no shared state)
- Mock external dependencies

Return ONLY the test code.
```

### Example: Testing a Function

**Prompt:**
```
Write pytest tests for this function:

def calculate_discount(price: float, discount_percent: float) -> float:
    """
    Calculate discounted price.
    
    Args:
        price: Original price (must be >= 0)
        discount_percent: Discount percentage (0-100)
        
    Returns:
        Discounted price
    """
    if price < 0:
        raise ValueError("Price cannot be negative")
    if discount_percent < 0 or discount_percent > 100:
        raise ValueError("Discount must be between 0 and 100")
    
    return price * (1 - discount_percent / 100)

Requirements:
- Cover happy path: normal prices and discounts
- Cover edge cases: zero price, 0% discount, 100% discount
- Cover error cases: negative price, discount > 100, negative discount
- Use descriptive test names
- Each test should be independent

Return ONLY the test code.
```

**Expected Output:**
```python
import pytest
from your_module import calculate_discount


def test_calculate_discount_normal():
    """Test normal discount calculation."""
    assert calculate_discount(100, 10) == 90.0
    assert calculate_discount(50, 25) == 37.5
    assert calculate_discount(200, 50) == 100.0


def test_calculate_discount_zero_price():
    """Test with zero price."""
    assert calculate_discount(0, 10) == 0.0


def test_calculate_discount_zero_percent():
    """Test with 0% discount."""
    assert calculate_discount(100, 0) == 100.0


def test_calculate_discount_hundred_percent():
    """Test with 100% discount (free)."""
    assert calculate_discount(100, 100) == 0.0


def test_calculate_discount_negative_price():
    """Test that negative price raises error."""
    with pytest.raises(ValueError, match="Price cannot be negative"):
        calculate_discount(-10, 10)


def test_calculate_discount_negative_discount():
    """Test that negative discount raises error."""
    with pytest.raises(ValueError, match="Discount must be between"):
        calculate_discount(100, -5)


def test_calculate_discount_discount_over_100():
    """Test that discount > 100 raises error."""
    with pytest.raises(ValueError, match="Discount must be between"):
        calculate_discount(100, 150)
```

---

## Test Frameworks

### pytest (Recommended)

**Why:** Simple syntax, powerful fixtures, great plugin ecosystem

```bash
pip install pytest
pytest tests/ -v
```

**Test Pattern:**
```python
def test_something():
    assert result == expected
```

### unittest (Built-in)

**Why:** No dependencies, part of Python standard library

```bash
python -m unittest tests/test_module.py
```

**Test Pattern:**
```python
import unittest

class TestSomething(unittest.TestCase):
    def test_something(self):
        self.assertEqual(result, expected)
```

### Jest (JavaScript/TypeScript)

**Why:** Standard for JS/TS, fast, great DX

```bash
npm test
```

**Test Pattern:**
```javascript
test('something', () => {
  expect(result).toBe(expected);
});
```

---

## Coverage Targets

### What's "Good" Coverage?

| Coverage | Verdict | Action |
|---|---|---|
| **< 50%** | Poor | Critical paths untested |
| **50-70%** | Acceptable | Test critical code only |
| **70-80%** | Good | Sweet spot for most projects |
| **80-90%** | Excellent | Comprehensive coverage |
| **> 90%** | Diminishing returns | Focus on test quality |

### Coverage Doesn't Equal Quality

**High coverage, low quality:**
```python
def test_add():
    add(1, 2)  # Calls function but doesn't check result
```

**Lower coverage, high quality:**
```python
def test_add_positive_numbers():
    assert add(1, 2) == 3

def test_add_negative_numbers():
    assert add(-1, -2) == -3

def test_add_zeros():
    assert add(0, 0) == 0
```

**3 tests, 100% meaningful coverage > 10 tests, 50% meaningful coverage**

---

## Edge Case Patterns

### Numeric Functions

Test:
- Zero
- Negative numbers
- Very large numbers
- Floating point precision
- Division by zero
- Overflow/underflow

```python
def test_edge_cases():
    # Zero
    assert divide(10, 1) == 10
    
    # Negative
    assert divide(-10, 2) == -5
    
    # Large numbers
    assert divide(10**10, 2) == 5 * 10**9
    
    # Floating point
    assert abs(divide(1, 3) - 0.333333) < 0.0001
```

### String Functions

Test:
- Empty string
- Single character
- Very long string
- Special characters
- Unicode
- Whitespace

```python
def test_string_edge_cases():
    # Empty
    assert reverse("") == ""
    
    # Single char
    assert reverse("a") == "a"
    
    # Unicode
    assert reverse("hello 世界") == "界世 olleh"
    
    # Whitespace
    assert reverse("  ") == "  "
```

### List/Collection Functions

Test:
- Empty list
- Single element
- Duplicate elements
- Very large list
- Nested structures

```python
def test_list_edge_cases():
    # Empty
    assert sum_list([]) == 0
    
    # Single element
    assert sum_list([1]) == 1
    
    # Duplicates
    assert sum_list([1, 1, 1]) == 3
```

---

## Error Case Patterns

### Expected Exceptions

```python
def test_invalid_input_raises():
    """Test that invalid input raises appropriate error."""
    with pytest.raises(ValueError, match="must be positive"):
        process(-1)
```

### Multiple Error Types

```python
def test_various_errors():
    with pytest.raises(ValueError):
        parse("")
    
    with pytest.raises(FileNotFoundError):
        parse("nonexistent.txt")
    
    with pytest.raises(TypeError):
        parse(123)  # Wrong type
```

---

## Fixing Failing Tests

### Scenario 1: Bug in Implementation

**Test output:**
```
tests/test_parser.py:15: AssertionError
assert parse("a,b,c") == ['a', 'b', 'c']
E       AssertionError: assert ['a,b,c'] == ['a', 'b', 'c']
```

**Prompt:**
```
This test is failing:

Code:
def parse(csv_string):
    return [csv_string]  # Bug: doesn't split

Test:
def test_parse_csv():
    assert parse("a,b,c") == ['a', 'b', 'c']

Error:
AssertionError: assert ['a,b,c'] == ['a', 'b', 'c']

Fix the implementation to pass the test.
```

### Scenario 2: Wrong Test Expectation

**Test output:**
```
tests/test_math.py:10: AssertionError
assert add(2, 3) == 6
E       assert 5 == 6
```

**Prompt:**
```
This test expectation is wrong:

Code:
def add(a, b):
    return a + b

Test:
def test_add():
    assert add(2, 3) == 6  # Wrong: should be 5

Fix the test expectation.
```

---

## Exercise: Achieve 80% Coverage

**Task:** Generate tests for the `Task` class from Chapter 1 and achieve 80%+ coverage.

**Step 1:** Generate tests

```
Write pytest tests for this class:

class Task:
    def __init__(self, title: str, description: str, priority: int = 3):
        if not title:
            raise ValueError("Title cannot be empty")
        if priority < 1 or priority > 5:
            raise ValueError("Priority must be 1-5")
        
        self.title = title
        self.description = description
        self.priority = priority
        self.completed = False
    
    def toggle(self):
        self.completed = not self.completed
    
    def __str__(self):
        status = "✓" if self.completed else "○"
        return f"[{status}] {self.title}"
    
    def __eq__(self, other):
        if not isinstance(other, Task):
            return False
        return (self.title == other.title and 
                self.description == other.description and
                self.priority == other.priority and
                self.completed == other.completed)

Requirements:
- Cover all methods
- Cover edge cases for validation
- Cover __eq__ with different scenarios
- Use descriptive test names
```

**Step 2:** Run tests
```bash
pytest tests/test_task.py -v
```

**Step 3:** Check coverage
```bash
pytest --cov=src.task tests/ -v
```

**Step 4:** Iterate on any failures

---

## On Your Hardware

### Small Models (1.5B-3B)

- Generate tests one function at a time
- May need more iteration to get right
- Focus on happy path first, edge cases later

### Medium Models (7B-14B)

- Can handle entire modules
- Better at finding edge cases
- Good balance of speed and quality

### Large Models (32B+)

- Can generate comprehensive test suites
- Better at understanding context
- May suggest additional test scenarios

---

## Common Pitfalls

### Pitfall 1: Tests That Always Pass

**Bad:**
```python
def test_something():
    result = do_something()
    assert True  # Always passes
```

**Fix:**
```python
def test_something():
    result = do_something()
    assert result is not None
    assert len(result) > 0
```

### Pitfall 2: Tests That Depend on Each Other

**Bad:**
```python
def test_setup():
    global data
    data = create_data()

def test_process():
    process(data)  # Depends on test_setup
```

**Fix:**
```python
def test_process():
    data = create_data()
    process(data)
```

### Pitfall 3: Testing Implementation Details

**Bad:**
```python
def test_internal_method():
    result = obj._internal_method()  # Testing private method
    assert result == expected
```

**Fix:**
```python
def test_public_api():
    result = obj.public_method()  # Test through public API
    assert result == expected
```

---

## Next Steps

- [Chapter 4: Review and Improve Code](./04-review-improve.md) — Use AI for code review
- [Chapter 5: The Full Loop](./05-full-loop.md) — Put it all together

---

## Further Reading

- [Prompt Templates](../reference/prompt-templates.md) — Test generation templates
- [Troubleshooting](../reference/troubleshooting.md) — Common test issues
