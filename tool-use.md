# Tool Use: Strategies for Local Agent Tool Calling

> How to design, implement, and debug tool use in local LLM agents.

## 1. Tool Calling Patterns

### Pattern 1: Structured JSON (Most Common)

```
Available tools:
- search(query: string) -> Search results
- read_file(path: string) -> File contents
- write_file(path: string, content: string) -> Success message

To use a tool, respond with:
{"tool": "tool_name", "args": {"arg1": "value1"}}
```

**Pros:** Well-understood, easy to parse
**Cons:** Small models produce invalid JSON frequently

### Pattern 2: Natural Language with Delimiters

```
To use a tool, write:
TOOL: tool_name(arg1="value1", arg2="value2")

Example:
TOOL: search(query="latest AI news")
```

**Pros:** More natural for small models, easier to parse with regex
**Cons:** Less structured, harder to validate complex arguments

### Pattern 3: XML Tags

```
To use a tool, respond with:
<tool name="search">
  <arg name="query">latest AI news</arg>
</tool>
```

**Pros:** Very parseable, small models handle XML well
**Cons:** More verbose, uses more tokens

### Recommendation

For constrained systems with small models: **Pattern 2 or 3**. They're more forgiving of model imperfections and easier to recover from when parsing fails.

## 2. Tool Schema Design

### Minimal Schema (Best for Small Models)

```
search(query) - Search the web for information
  query: The search query string

read_file(path) - Read a file's contents
  path: Full path to the file

calculate(expression) - Evaluate a math expression
  expression: Mathematical expression like "2 + 3 * 4"
```

### Schema Guidelines

1. **Few parameters:** Max 3–4 per tool for small models
2. **Simple types:** string, number, boolean. Avoid nested objects
3. **Clear descriptions:** One-line, action-oriented
4. **Required vs optional:** Mark clearly, prefer required
5. **Enum values:** List allowed values explicitly

### What to Avoid

```
# BAD: Too complex for small models
analyze_data(
    path: string,
    options: {
        columns: string[],
        filters: { field: string, operator: string, value: any }[],
        aggregations: { field: string, function: string }[],
        output_format: "csv" | "json" | "table"
    },
    callbacks: { on_progress: function, on_complete: function }
)

# GOOD: Simple, composable
read_csv(path: string) -> rows
filter_rows(rows: string, column: string, value: string) -> filtered rows
aggregate(rows: string, column: string, function: string) -> result
```

## 3. Tool Output Management

Tool outputs can blow up your context window. Manage them aggressively.

### Truncation Strategies

```python
def format_tool_output(output, max_length=500):
    text = str(output)
    if len(text) <= max_length:
        return text
    # Keep beginning and end, truncate middle
    half = max_length // 2 - 50
    return (text[:half] + 
            f"\n... [{len(text) - max_length} chars omitted] ...\n" + 
            text[-half:])
```

### Summarization for Large Outputs

```python
def summarize_tool_output(output, model):
    if token_count(str(output)) > 1000:
        summary = model.chat(
            "Summarize this output in 3-5 bullet points:\n" + output
        )
        return summary
    return output
```

### Structured Output Return

Design tools to return structured, concise output:

```python
# BAD: Returns everything
def list_files(path):
    return os.listdir(path)  # Could be hundreds of entries

# GOOD: Returns structured, limited output
def list_files(path, max_results=20):
    files = os.listdir(path)[:max_results]
    total = len(os.listdir(path))
    return {
        "files": files,
        "total": total,
        "truncated": total > max_results
    }
```

## 4. Error Handling in Tool Use

### Tool Execution Errors

```python
def execute_tool(tool_name, args):
    try:
        result = tools[tool_name](**args)
        return {"status": "success", "output": result}
    except Exception as e:
        return {
            "status": "error",
            "error": str(e),
            "suggestion": f"Check your arguments. Tool '{tool_name}' expects: {tools[tool_name].__doc__}"
        }
```

### Feeding Errors Back to the Model

```
Observation: ERROR - Tool 'read_file' failed: File not found: /data/missing.csv
Suggestion: Check the file path. Use list_files to see available files.

Thought: The file doesn't exist. Let me list the directory to find the correct path.
Action: {"tool": "list_files", "args": {"path": "/data"}}
```

### Retry with Adaptation

```python
def tool_call_with_retry(tool_name, args, max_retries=2):
    for attempt in range(max_retries):
        result = execute_tool(tool_name, args)
        if result["status"] == "success":
            return result
        # Adapt based on error
        args = model.adapt_args(tool_name, args, result["error"])
    return result  # Return last error for model to handle
```

## 5. Tool Selection and Routing

### When You Have Many Tools

Small models struggle with too many tool options.

### Tool Grouping

```
Category: File Operations
- read_file(path)
- write_file(path, content)
- list_files(path)
- search_files(pattern)

Category: Data Operations
- read_csv(path)
- query_sql(database, query)
- analyze_data(path, metric)

Category: Web Operations
- search(query)
- fetch_url(url)
- extract_links(url)
```

### Two-Stage Tool Selection

```
Stage 1: Select category
Thought: This is a file operation task.
Category: File Operations

Stage 2: Select specific tool (from reduced set)
Thought: I need to find files matching a pattern.
Action: {"tool": "search_files", "args": {"pattern": "*.csv"}}
```

This reduces the decision space at each step, improving accuracy.

### Dynamic Tool Loading

Only present relevant tools based on context:

```python
def get_relevant_tools(user_input, all_tools):
    # Simple keyword matching or LLM classification
    category = classify_task(user_input)
    return {name: tool for name, tool in all_tools.items() 
            if tool.category == category}
```

## 6. Observability for Tool Use

### Tool Call Logging

```python
class ToolCallLogger:
    def __init__(self):
        self.calls = []
    
    def log(self, tool_name, args, result, duration_ms, step):
        self.calls.append({
            "step": step,
            "tool": tool_name,
            "args": args,
            "status": result["status"],
            "output_length": len(str(result.get("output", ""))),
            "duration_ms": duration_ms,
        })
    
    def report(self):
        total_calls = len(self.calls)
        errors = sum(1 for c in self.calls if c["status"] == "error")
        avg_duration = sum(c["duration_ms"] for c in self.calls) / total_calls
        return {
            "total_calls": total_calls,
            "error_rate": errors / total_calls,
            "avg_duration_ms": avg_duration,
            "tools_used": list(set(c["tool"] for c in self.calls)),
        }
```

## 7. Security for Tool Execution

### Principle of Least Privilege

```python
# Define tool permissions explicitly
TOOL_PERMISSIONS = {
    "read_file": {
        "allowed_paths": ["/data", "/tmp"],
        "max_file_size": 1_000_000,  # 1MB
    },
    "write_file": {
        "allowed_paths": ["/tmp/output"],
        "max_file_size": 500_000,
    },
    "execute_command": {
        "allowed_commands": ["ls", "cat", "grep", "wc"],
        "timeout_seconds": 10,
        "no_network": True,
    },
}

def execute_with_permissions(tool_name, args):
    permissions = TOOL_PERMISSIONS.get(tool_name, {})
    
    # Validate paths
    if "allowed_paths" in permissions:
        path = args.get("path", "")
        if not any(path.startswith(allowed) for allowed in permissions["allowed_paths"]):
            raise PermissionError(f"Path '{path}' not allowed for {tool_name}")
    
    # Execute tool
    return tools[tool_name](**args)
```

### Sandboxed Execution

```python
import subprocess

def safe_execute(command, timeout=10):
    allowed = ["ls", "cat", "grep", "wc", "head", "tail", "sort", "uniq"]
    cmd_parts = command.split()
    
    if cmd_parts[0] not in allowed:
        raise PermissionError(f"Command '{cmd_parts[0]}' not allowed")
    
    result = subprocess.run(
        cmd_parts,
        capture_output=True,
        text=True,
        timeout=timeout,
        cwd="/safe/sandbox",  # Restricted working directory
    )
    return result.stdout
```

## 8. Testing Tool Use

### Unit Tests for Tools

```python
import pytest

def test_read_file():
    result = read_file("/data/test.csv")
    assert result["status"] == "success"
    assert len(result["output"]) > 0

def test_read_file_not_found():
    result = read_file("/data/nonexistent.csv")
    assert result["status"] == "error"
    assert "not found" in result["error"].lower()

def test_read_file_permission():
    result = read_file("/etc/shadow")
    assert result["status"] == "error"
    assert "not allowed" in result["error"].lower()
```

### Integration Tests for Agent Tool Use

```python
def test_agent_uses_tool_correctly():
    agent = Agent(model, tools={"search": mock_search})
    result = agent.run("What is the capital of France?")
    
    # Verify tool was called
    assert mock_search.called
    assert "France" in mock_search.call_args[0]["query"]
    
    # Verify answer is correct
    assert "Paris" in result.answer
```

---

*See also: [agents.md](./agents.md) for core principles, [prompt-patterns.md](./prompt-patterns.md) for prompting tool use.*
