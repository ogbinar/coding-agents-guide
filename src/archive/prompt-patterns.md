# Prompt Patterns for Quality and Control

> Techniques to get reliable, high-quality output from local LLM agents.

## 1. The Foundation: System Prompts

Your system prompt is the most important control mechanism for local agents.

### Anatomy of a Good System Prompt

```
ROLE: Who the model is
TASK: What it should do
CONSTRAINTS: What it should NOT do
FORMAT: How to structure output
EXAMPLES: Few-shot demonstrations (critical for small models)
```

### Example: Agent System Prompt

```
You are a research assistant agent. Your job is to answer questions
by searching available tools and synthesizing findings.

CONSTRAINTS:
- Always use tools before answering factual questions
- Never fabricate information
- If a tool fails, try an alternative approach
- Keep responses under 200 words unless asked for detail

OUTPUT FORMAT:
Use this structure:
1. Thought: What you plan to do next
2. Action: Tool name and arguments (JSON only)
3. Observation: (filled in by the system)
4. Answer: Final response when you have enough information

EXAMPLE:
User: What is the weather in Tokyo?
Thought: I need to check the weather for Tokyo.
Action: {"tool": "weather_search", "args": {"city": "Tokyo"}}
Observation: {"temp": 22, "condition": "sunny"}
Answer: The weather in Tokyo is currently sunny with a temperature of 22°C.
```

### Small Model Specific Tips

- **Be explicit, not implicit:** Small models don't infer intent well
- **Repeat key constraints:** Mention critical rules in both system and user prompts
- **Use concrete examples:** Few-shot examples are worth more than abstract instructions
- **Avoid negation-heavy prompts:** "Don't do X" is less effective than "Do Y instead"

## 2. Structured Output

Getting parseable output from small models requires discipline.

### JSON Output Pattern

```
Return your response as valid JSON matching this schema:
{
  "reasoning": "string - your step-by-step reasoning",
  "answer": "string - the final answer",
  "confidence": "number between 0 and 1",
  "sources": ["string"] - list of sources used
}

IMPORTANT: Return ONLY the JSON object. No markdown, no explanation, no
text before or after the JSON.
```

### Why Small Models Struggle with JSON

- They add markdown code fences (```json ... ```)
- They include explanatory text before/after
- They produce invalid JSON (trailing commas, missing quotes)

### Mitigation Strategies

1. **Post-processing:** Strip markdown fences, extract JSON with regex
2. **JSON mode:** Some engines support `--json` flag for constrained decoding
3. **Output validators:** Catch and retry on parse failures
4. **XML tags as alternative:** `<answer>...</answer>` is often easier for small models

### XML Tag Pattern (Often More Reliable)

```
Wrap your final answer in <answer> tags.
Wrap your reasoning in <thinking> tags.
Do not include any text outside these tags.
```

Parsing: Simple regex, more forgiving than JSON.

## 3. Chain-of-Thought (CoT) Patterns

CoT dramatically improves reasoning quality, even for small models.

### Standard CoT

```
Think through this step by step before answering.
Show your reasoning process.
```

### Structured CoT (Better for Agents)

```
Break down your reasoning:
1. UNDERSTAND: What is the user asking?
2. PLAN: What steps will you take?
3. EXECUTE: Carry out each step
4. VERIFY: Does your answer address the question?
5. ANSWER: State your final answer clearly
```

### Self-Consistency

For critical decisions, generate multiple reasoning paths:

```python
# Generate 3 independent reasoning chains
answers = [model.chat(prompt + f"\nApproach {i}:", temp=0.7) for i in range(3)]
# Take the majority answer or the one with highest confidence
final = consensus(answers)
```

Cost: 3x tokens. Benefit: Significantly more reliable answers.

### Tree of Thoughts (ToT) — Simplified for Small Models

Full ToT is expensive. Here's a constrained version:

```
Consider 2 possible approaches to this problem:

Approach A: [brief description]
Pros: ...
Cons: ...

Approach B: [brief description]  
Pros: ...
Cons: ...

Select the better approach and execute it.
```

## 4. Few-Shot Prompting

The single most effective technique for small models.

### Good Few-Shot Examples

```
Here are examples of correct behavior:

Example 1:
Input: "What files are in the /data directory?"
Thought: I need to list the contents of /data.
Action: {"tool": "list_directory", "args": {"path": "/data"}}
Observation: ["file1.txt", "file2.csv", "subdir/"]
Answer: The /data directory contains: file1.txt, file2.csv, and a subdirectory called subdir.

Example 2:
Input: "Find all CSV files and count their rows"
Thought: I need to find CSV files first, then count rows in each.
Action: {"tool": "find_files", "args": {"pattern": "*.csv", "path": "/data"}}
Observation: ["/data/file2.csv", "/data/subdir/data.csv"]
Thought: Now I'll count rows in each CSV file.
Action: {"tool": "count_csv_rows", "args": {"path": "/data/file2.csv"}}
Observation: {"rows": 150}
Action: {"tool": "count_csv_rows", "args": {"path": "/data/subdir/data.csv"}}
Observation: {"rows": 340}
Answer: Found 2 CSV files: file2.csv has 150 rows, data.csv has 340 rows.
```

### Rules for Effective Few-Shot

1. **Match the distribution:** Examples should resemble actual inputs
2. **Show tool use patterns:** Demonstrate correct tool calling
3. **Include edge cases:** Show how to handle errors and ambiguity
4. **Keep it concise:** 2–4 examples is usually optimal for small models
5. **Update regularly:** As your agent evolves, update examples

## 5. Temperature and Sampling

### Temperature Guide for Agents

| Task | Temperature | Why |
|---|---|---|
| Tool calling | 0.0–0.1 | Need deterministic, parseable output |
| Factual Q&A | 0.1–0.3 | Balance accuracy and fluency |
| Creative tasks | 0.5–0.8 | Allow variation |
| Reasoning/CoT | 0.3–0.5 | Some variation helps explore paths |
| Code generation | 0.1–0.2 | Need correctness |

### Top-P vs Temperature

- **Temperature:** Controls randomness globally
- **Top-P (nucleus sampling):** Controls the tail of the distribution
- **For agents:** Use low temperature (0.1) + top_p=0.9 for most steps
- **For creative exploration:** Use higher temperature in specific steps only

### Reproducibility

```python
# Always set seed for debugging
model.chat(prompt, seed=42, temperature=0.1)

# Once working, you can increase temperature for production
model.chat(prompt, temperature=0.3)
```

## 6. Prompt Compression

Save tokens without losing quality.

### Techniques

1. **Abbreviate instructions:** "Think step by step" → "Think step-by-step" (small savings)
2. **Remove pleasantries:** No "Please", "Could you", "I would like to"
3. **Use symbols and structure:** Bullet points > paragraphs
4. **Compress examples:** Keep the pattern, trim verbose descriptions
5. **Template variables:** Store common prompts as templates, inject only variables

### Context Summarization Prompt

When you need to compress conversation history:

```
Summarize the following conversation in 3-5 bullet points.
Preserve: decisions made, tools used, key findings, open questions.
Omit: intermediate reasoning, repeated information, pleasantries.

Conversation:
{history}

Summary:
```

## 7. Guardrails and Safety

### Output Validation

```python
def validate_agent_output(output, schema):
    checks = [
        is_valid_json(output),
        has_required_fields(output, schema),
        length_within_bounds(output, max_tokens=500),
        no_prohibited_content(output),
    ]
    return all(checks)
```

### Input Sanitization

```python
def sanitize_user_input(user_input):
    # Strip potential prompt injection patterns
    dangerous_patterns = [
        r"ignore previous instructions",
        r"system prompt:",
        r"you are now",
        r"disregard the above",
    ]
    for pattern in dangerous_patterns:
        user_input = re.sub(pattern, "[FILTERED]", user_input, flags=re.IGNORECASE)
    return user_input
```

### Constrained Decoding

Some inference engines support grammar-based constrained decoding:

```bash
# llama.cpp with grammar file
./main -m model.gguf -f prompt.txt --grammar json.grammar

# This forces the model to produce valid JSON
# Significantly reduces parsing errors
```

## 8. Prompt Patterns by Task

### Classification

```
Classify the following text into one of these categories:
{categories}

Return ONLY the category name, nothing else.

Text: {input}
Category:
```

### Extraction

```
Extract the following information from the text:
- Name
- Date
- Amount
- Status

Return as JSON. Use null for missing fields.

Text: {input}
```

### Summarization

```
Summarize the following in exactly 3 bullet points.
Focus on: key findings, actions needed, risks.

Text: {input}
```

### Code Generation

```
Write a Python function that {description}.

Requirements:
- Type hints on all parameters and return value
- Docstring with description, params, returns
- Handle edge cases: {list}
- No external dependencies

Return ONLY the code, wrapped in a function.
```

## 9. Iterative Prompt Refinement

Treat prompts as code — version, test, and iterate.

### The Prompt Testing Loop

```
1. Write initial prompt
2. Test on 10 diverse examples
3. Identify failure modes
4. Add constraints or examples for failures
5. Re-test
6. Repeat until success rate > 90%
7. Document the final prompt and why it works
```

### A/B Testing Prompts

```python
def compare_prompts(prompt_a, prompt_b, test_cases):
    results_a = [run_agent(prompt_a, case) for case in test_cases]
    results_b = [run_agent(prompt_b, case) for case in test_cases]
    
    score_a = evaluate(results_a, test_cases)
    score_b = evaluate(results_b, test_cases)
    
    return {"prompt_a": score_a, "prompt_b": score_b}
```

## 10. Coding-Specific Prompt Patterns

### Code Generation with Constraints

```
Write a {language} function that {description}.

Context:
- This is part of a {framework/language} project
- The project uses {conventions: e.g., PEP 8, snake_case, etc.}
- Available dependencies: {list}
- Do NOT use: {restricted libraries/patterns}

Requirements:
1. Type hints on all parameters and return values
2. Docstring in {style} format
3. Handle these edge cases: {list}
4. No external dependencies beyond: {list}
5. Include inline comments for non-obvious logic

Return ONLY the code. No explanation, no markdown fences.
```

### Code Modification via Diff

```
Modify the following code to {change description}.

Current code (file: {path}):
{code}

Return a unified diff showing the changes. Use this format:
--- a/{path}
+++ b/{path}
@@ -X,Y +A,B @@
-removed line
+added line

Rules:
- Only change what is necessary
- Preserve all existing comments and docstrings
- Keep the same indentation style
- If the change requires adding imports, show that in the diff too
```

### Debugging Prompt

```
The following code produces this error:

Code:
{code}

Error:
{error_message_or_stack_trace}

Analyze the error:
1. What is the direct cause of this error?
2. What is the root cause in the code?
3. What is the minimal fix?

Return your analysis and the fixed code as a diff.
```

### Code Review Prompt

```
Review the following code for issues. Categorize findings:

🔴 Critical: Bugs, security vulnerabilities, data loss
🟡 Warning: Performance issues, maintainability concerns
🔵 Suggestion: Style improvements, best practices

Code (file: {path}):
{code}

For each finding, provide:
- Line number(s)
- Category (🔴/🟡/🔵)
- Description of the issue
- Suggested fix (code snippet)

If no issues found, say "No issues found." Do not manufacture problems.
```

### Test Generation Prompt

```
Write tests for the following {language} code using {test framework}.

Code:
{code}

Test requirements:
- Cover the happy path (normal usage)
- Cover edge cases: {list specific edge cases}
- Cover error cases: invalid inputs, exceptions, boundary values
- Use descriptive test names: test_{function}_{scenario}
- Each test should be independent (no shared state)
- Include setup/teardown where needed

Return ONLY the test code.
```

### Architecture/Design Prompt

```
Design a {type: API / service / module / CLI tool} for {purpose}.

Constraints:
- Language: {language}
- Framework: {framework}
- Must integrate with: {existing systems}
- Deployment target: {environment}
- Team size: {size} (affects complexity tolerance)

Provide:
1. High-level architecture (text diagram)
2. Module/file structure (directory tree)
3. Key interfaces and data models
4. Technology choices with brief justification
5. Potential risks and mitigations
```

### Brownfield Onboarding Prompt

```
I am working in an existing codebase. Here is the project structure:

{file_tree}

Here are the key configuration files:
{configs}

Analyze this project and tell me:
1. What language, framework, and major libraries does it use?
2. What is the entry point and general architecture?
3. What are the coding conventions (naming, style, patterns)?
4. How are tests organized and run?
5. How is the project built and deployed?
6. What are the most complex or risky parts to modify?

Be specific and reference actual file names from the structure.
```

## 11. Prompt Patterns by Project Context

### Greenfield (New Project)

- Emphasize architecture decisions and scaffolding
- Provide framework-specific conventions upfront
- Ask for complete, runnable code (not snippets)
- Include setup instructions and README generation

### Brownfield (Existing Project)

- Load project context before asking for changes
- Ask for diffs, not full file rewrites
- Reference existing patterns and conventions
- Ask the agent to identify related files that might need changes

### Migration (Code Transformation)

- Provide both source and target examples (few-shot)
- Specify what must be preserved (behavior, API contracts)
- Ask for incremental migration steps, not all-at-once
- Include test preservation strategy

---

*See also: [agents.md](./agents.md) for core principles, [tool-use.md](./tool-use.md) for tool calling patterns.*
