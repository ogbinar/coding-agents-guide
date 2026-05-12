# Evaluation: Measuring and Improving Agent Quality

> How to know if your local agent is actually good, and how to make it better.

## 1. What to Measure

### Core Metrics

| Metric | What It Tells You | Target |
|---|---|---|
| **Task success rate** | % of tasks completed correctly | > 80% |
| **Tool call accuracy** | % of tool calls with correct args | > 85% |
| **Hallucination rate** | % of outputs containing fabricated info | < 5% |
| **Latency (p50, p95)** | Speed of task completion | Depends on use case |
| **Token efficiency** | Tokens consumed per successful task | Minimize |
| **Error recovery rate** | % of failures that self-recover | > 50% |
| **Context utilization** | % of context window used effectively | 60–80% |

### Coding-Specific Metrics

| Metric | What It Tells You | Target |
|---|---|---|
| **Syntax validity** | % of generated code that parses without errors | > 95% |
| **Test pass rate** | % of generated tests that pass on first run | > 80% |
| **Diff accuracy** | % of diffs that apply cleanly without conflicts | > 90% |
| **API hallucination** | % of code referencing non-existent APIs/methods | < 3% |
| **Import correctness** | % of imports that resolve to real modules | > 95% |
| **Build success** | % of modifications that compile/build successfully | > 85% |
| **Lint cleanliness** | % of generated code passing linter on first pass | > 70% |
| **Fix convergence** | Avg iterations to go from error → green build | < 4 |
| **Regression rate** | % of changes that break previously passing tests | < 5% |

### Quality Dimensions

```
Correctness:  Is the answer right?
Completeness: Does it address all parts of the question?
Conciseness:  Is it appropriately brief?
Format:       Does it match the expected structure?
Safety:       Does it avoid harmful/dangerous outputs?
```

## 2. Building Evaluation Datasets

### Golden Dataset Structure

```json
[
  {
    "id": "test_001",
    "category": "file_operations",
    "input": "List all Python files in /src and count lines in each",
    "expected_tools": ["list_files", "count_lines"],
    "expected_output_contains": [".py", "lines"],
    "difficulty": "medium",
    "notes": "Agent should handle nested directories"
  },
  {
    "id": "test_002",
    "category": "data_analysis",
    "input": "What is the average temperature in the weather data?",
    "expected_tools": ["read_csv", "calculate"],
    "expected_output_contains": ["average", "temperature"],
    "difficulty": "medium",
    "notes": "Requires multi-step tool use"
  }
]
```

### Dataset Guidelines

1. **Cover all tool combinations** — Test each tool individually and in chains
2. **Include edge cases** — Missing files, empty results, ambiguous queries
3. **Vary difficulty** — Easy (single tool), Medium (2–3 tools), Hard (5+ tools)
4. **Include negative cases** — Tasks the agent should refuse or flag
5. **Keep it small but diverse** — 20–50 well-chosen tests > 200 random ones

## 3. Automated Evaluation

### Basic Evaluator

```python
import json
from typing import Dict, List

class AgentEvaluator:
    def __init__(self, test_cases: List[Dict]):
        self.test_cases = test_cases
        self.results = []
    
    def evaluate(self, agent) -> Dict:
        for case in self.test_cases:
            result = self.run_test(agent, case)
            self.results.append(result)
        
        return self.summary()
    
    def run_test(self, agent, case: Dict) -> Dict:
        output = agent.run(case["input"])
        
        # Check tool usage
        tools_used = [call["tool"] for call in output.tool_calls]
        tools_correct = all(
            expected in tools_used 
            for expected in case.get("expected_tools", [])
        )
        
        # Check output content
        output_text = str(output.answer)
        content_correct = all(
            expected.lower() in output_text.lower()
            for expected in case.get("expected_output_contains", [])
        )
        
        # Check format
        format_correct = self.validate_format(output, case)
        
        return {
            "id": case["id"],
            "category": case["category"],
            "difficulty": case.get("difficulty", "unknown"),
            "tools_correct": tools_correct,
            "content_correct": content_correct,
            "format_correct": format_correct,
            "success": tools_correct and content_correct and format_correct,
            "latency_ms": output.latency_ms,
            "tokens_used": output.tokens_used,
            "iterations": output.iterations,
            "error": output.error if hasattr(output, "error") else None,
        }
    
    def summary(self) -> Dict:
        total = len(self.results)
        passed = sum(1 for r in self.results if r["success"])
        
        by_category = {}
        for r in self.results:
            cat = r["category"]
            if cat not in by_category:
                by_category[cat] = {"total": 0, "passed": 0}
            by_category[cat]["total"] += 1
            if r["success"]:
                by_category[cat]["passed"] += 1
        
        return {
            "total_tests": total,
            "passed": passed,
            "success_rate": passed / total if total else 0,
            "avg_latency_ms": sum(r["latency_ms"] for r in self.results) / total,
            "avg_tokens": sum(r["tokens_used"] for r in self.results) / total,
            "avg_iterations": sum(r["iterations"] for r in self.results) / total,
            "by_category": {
                cat: {
                    **stats,
                    "rate": stats["passed"] / stats["total"]
                }
                for cat, stats in by_category.items()
            },
            "failures": [r for r in self.results if not r["success"]],
        }
```

### Running Evaluations

```python
# Evaluate different models/prompts
evaluator = AgentEvaluator(test_cases)

results_q4 = evaluator.evaluate(agent_with_q4_model)
results_q5 = evaluator.evaluate(agent_with_q5_model)

print(f"Q4 success rate: {results_q4['success_rate']:.1%}")
print(f"Q5 success rate: {results_q5['success_rate']:.1%}")
```

## 4. LLM-as-Judge (With Caveats)

For subjective evaluation, you can use the LLM itself as a judge.

### Self-Evaluation Prompt

```
Evaluate the following agent response against the criteria:

Task: {task}
Expected tools: {expected_tools}
Agent response: {response}
Tools used: {tools_used}

Rate each dimension 1-5:
- Correctness: Does the answer address the task?
- Tool usage: Were the right tools used correctly?
- Completeness: Are all aspects covered?
- Conciseness: Is the response appropriately brief?

Provide a brief justification for each score.
Return as JSON.
```

### Caveats

- **Same model bias:** The judge model may favor its own output style
- **Use a larger model:** If possible, use a bigger model as judge
- **Calibrate:** Compare LLM-judge scores against human evaluation on a small set
- **Don't over-trust:** Use as a signal, not ground truth

## 5. Regression Testing

### Baseline Tracking

```python
import json
from datetime import datetime

def save_baseline(results, model_name, prompt_version):
    baseline = {
        "timestamp": datetime.now().isoformat(),
        "model": model_name,
        "prompt_version": prompt_version,
        "results": results,
    }
    
    with open("evaluations/baselines.json", "r+") as f:
        baselines = json.load(f)
        baselines.append(baseline)
        f.seek(0)
        json.dump(baselines, f, indent=2)

def compare_to_baseline(new_results, baseline_results):
    delta = new_results["success_rate"] - baseline_results["success_rate"]
    if delta < -0.05:  # More than 5% regression
        print(f"⚠️ Regression detected! Success rate dropped by {delta:.1%}")
        return False
    print(f"✓ No regression. Delta: {delta:+.1%}")
    return True
```

### CI/CD Integration

```yaml
# .github/workflows/evaluate.yml
name: Agent Evaluation
on: [push]
jobs:
  evaluate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run evaluation suite
        run: python evaluate.py --model /models/qwen2.5-7b-q4.gguf
      - name: Check regression
        run: python check_baseline.py --threshold 0.05
```

## 6. Debugging Failed Cases

### Failure Analysis Template

```markdown
## Failure: {test_id}

**Input:** {user_input}
**Expected:** {expected_behavior}
**Got:** {actual_output}
**Failure type:** [tool_error | wrong_tool | bad_format | hallucination | timeout]

### Root Cause
{analysis}

### Fix
{proposed_fix}

### Verification
{how_to_verify_fix}
```

### Common Failure Patterns and Fixes

| Pattern | Cause | Fix |
|---|---|---|
| Wrong tool selected | Tool descriptions unclear | Rewrite descriptions with examples |
| Invalid JSON output | Model too small / temperature too high | Use XML tags, lower temperature |
| Missing arguments | Schema too complex | Simplify tool signatures |
| Infinite loops | No clear stopping condition | Add max iterations, explicit stop signals |
| Context overflow | Too many tool outputs | Truncate/summarize tool outputs |
| Hallucination | No grounding in tool results | Require citing tool outputs in answers |

## 7. Benchmarking Against Baselines

### Simple Benchmark Suite

```python
BENCHMARKS = {
    "tool_calling": {
        "description": "Can the model call tools correctly?",
        "tests": 10,
        "passing_threshold": 0.8,
    },
    "reasoning": {
        "description": "Can the model solve multi-step problems?",
        "tests": 10,
        "passing_threshold": 0.7,
    },
    "following_constraints": {
        "description": "Does the model respect output format constraints?",
        "tests": 10,
        "passing_threshold": 0.9,
    },
    "error_recovery": {
        "description": "Can the model recover from tool errors?",
        "tests": 5,
        "passing_threshold": 0.6,
    },
}
```

### Reporting

```
═══════════════════════════════════════════════════════
  Agent Evaluation Report
  Model: qwen2.5-7b-q4_k_m
  Date: 2025-01-15
═══════════════════════════════════════════════════════

  Overall Success Rate: 78.5% (51/65)
  Avg Latency: 2.3s (p50), 5.1s (p95)
  Avg Tokens: 1,247 per task
  Avg Iterations: 3.2 per task

  By Category:
    file_operations:  85.0% ✓ (17/20)
    data_analysis:    70.0% ⚠ (14/20)
    web_search:       80.0% ✓ (8/10)
    code_generation:  65.0% ✗ (12/20)

  Regressions: None
  New failures: 3 (see details below)
═══════════════════════════════════════════════════════
```

## 8. Coding-Specific Evaluation Methods

### Executable Verification (The Gold Standard)

Unlike general agents where you need an LLM-as-judge, coding agents can be
evaluated **objectively** because code either works or it doesn't.

```python
def evaluate_code_output(generated_code, language, test_suite=None):
    results = {}

    # 1. Syntax check
    results["syntax_valid"] = check_syntax(generated_code, language)

    # 2. Import resolution
    if results["syntax_valid"]:
        results["imports_valid"] = check_imports(generated_code, language)

    # 3. Lint check
    if results["syntax_valid"]:
        results["lint_clean"] = run_linter(generated_code, language)

    # 4. Type check (if applicable)
    if language in ["python", "typescript", "rust"]:
        results["type_clean"] = run_type_check(generated_code, language)

    # 5. Test execution
    if test_suite:
        results["tests_pass"] = run_tests(test_suite)
        results["coverage"] = get_coverage(test_suite)

    # 6. Build check
    results["builds"] = run_build()

    return results
```

### Diff Quality Evaluation

For code modification tasks, evaluate the quality of generated diffs:

```python
def evaluate_diff(original_code, modified_code, expected_diff):
    generated_diff = create_diff(original_code, modified_code)

    # Does the diff apply cleanly?
    applies_cleanly = apply_diff(original_code, generated_diff) is not None

    # Does the result match expected behavior?
    result = apply_diff(original_code, generated_diff)
    behavior_preserved = run_tests(result) == run_tests(expected_diff)

    # Is the diff minimal (only changed what was needed)?
    change_ratio = count_changed_lines(generated_diff) / count_total_lines(original_code)
    is_minimal = change_ratio < 0.3  # Less than 30% of file changed

    return {
        "applies_cleanly": applies_cleanly,
        "behavior_preserved": behavior_preserved,
        "is_minimal": is_minimal,
        "change_ratio": change_ratio,
    }
```

### Workflow-Specific Evaluation Scenarios

Build test scenarios that match the 16 workflows from workflows.md:

```python
WORKFLOW_TESTS = {
    "code_generation": {
        "prompt": "Write a Python function that parses CSV and returns a list of dicts",
        "verify": [check_syntax, check_imports, run_unit_tests],
    },
    "debugging": {
        "setup": "Plant a known bug in a working function",
        "prompt": "Fix the bug: {error_message}",
        "verify": [check_bug_fixed, check_no_regressions],
    },
    "refactoring": {
        "setup": "Provide a function that violates SRP",
        "prompt": "Refactor this function to follow single responsibility",
        "verify": [check_behavior_preserved, check_lint, check_tests_still_pass],
    },
    "test_writing": {
        "setup": "Provide a function without tests",
        "prompt": "Write comprehensive tests for this function",
        "verify": [tests_run, coverage_above_threshold, edge_cases_covered],
    },
    "migration": {
        "setup": "Provide callback-based JavaScript code",
        "prompt": "Convert this to async/await",
        "verify": [behavior_preserved, modern_syntax, tests_pass],
    },
}
```

## 9. Continuous Improvement Loop

```
┌─────────────────────────────────────────────┐
│              Improvement Loop               │
│                                             │
│  1. Evaluate → Identify weak categories     │
│       ↓                                     │
│  2. Analyze → Understand failure modes      │
│       ↓                                     │
│  3. Hypothesize → Propose prompt/tool fixes │
│       ↓                                     │
│  4. Implement → Apply changes               │
│       ↓                                     │
│  5. Re-evaluate → Measure improvement       │
│       ↓                                     │
│  6. Compare → Accept or revert              │
│       ↓                                     │
│  (back to 1)                                │
└─────────────────────────────────────────────┘
```

---

*See also: [agents.md](./agents.md) for core principles, [constrained-systems.md](./constrained-systems.md) for hardware benchmarks.*
