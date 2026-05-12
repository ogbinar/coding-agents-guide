# Quick Start: Your First AI-Coded Function (30 Minutes)

> **Goal:** Install one tool, run one model, get AI to write code. No theory yet.

---

## What You'll Have in 30 Minutes

- A working AI coding agent on your machine
- A function that the AI wrote for you
- The "wow" moment before you've read a chapter of theory

---

## Step 1: Install Ollama (5 minutes)

Ollama is the easiest way to run local LLMs. It's opinionated, well-documented, and just works.

### macOS

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

### Linux

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

### Windows

Download from [ollama.com](https://ollama.com) and run the installer.

### Verify Installation

```bash
ollama --version
# Should print something like: ollama version 0.1.0
```

---

## Step 2: Choose a Model Based on Your Hardware (2 minutes)

Run this command to check your available RAM:

```bash
# macOS
sysctl -n hw.memsize | awk '{print $1/1024/1024/1024 " GB"}'

# Linux
free -h | awk '/^Mem:/ {print $2}'

# Windows (PowerShell)
Get-CimInstance Win32_PhysicalMemory | Measure-Object -Property Capacity -Sum | %{ "{0:N1} GB" -f ($_.Sum/1GB) }
```

### Model Recommendation Matrix

| Your RAM | Recommended Model | VRAM Needed | Speed |
|---|---|---|---|
| **4–8 GB** | `qwen2.5-coder:1.5b` | ~1 GB | Fast |
| **8–16 GB** | `qwen2.5-coder:7b` | ~5 GB | Good |
| **16–32 GB** | `qwen2.5-coder:14b` | ~9 GB | Slower |
| **32+ GB** | `qwen2.5-coder:32b` | ~20 GB | Slow but smart |

**For most people:** Start with `qwen2.5-coder:7b`. It's the sweet spot.

---

## Step 3: Pull the Model (5–10 minutes)

```bash
ollama pull qwen2.5-coder:7b
```

This downloads ~4 GB. Time depends on your internet speed.

### Verify Download

```bash
ollama list
# Should show qwen2.5-coder:7b in the list
```

---

## Step 4: Run Your First Prompt (3 minutes)

```bash
ollama run qwen2.5-coder:7b
```

You should see:
```
>>> 
```

Type this prompt:

```
Write a Python function that takes a list of integers and returns the sum of all even numbers.

Requirements:
- Type hints on all parameters and return value
- Docstring with description and examples
- Handle edge cases: empty list, negative numbers
- No external dependencies

Return ONLY the code. No explanation.
```

Press Enter. The model will generate code.

### Expected Output

```python
def sum_even_numbers(numbers: list[int]) -> int:
    """
    Calculate the sum of all even numbers in a list.
    
    Args:
        numbers: A list of integers
        
    Returns:
        The sum of all even numbers in the list
        
    Examples:
        >>> sum_even_numbers([1, 2, 3, 4, 5, 6])
        12
        >>> sum_even_numbers([])
        0
        >>> sum_even_numbers([-2, -1, 0, 1, 2])
        0
    """
    return sum(num for num in numbers if num % 2 == 0)
```

**Congratulations!** You just had AI write code for you.

---

## Step 5: Save and Run the Code (5 minutes)

1. Copy the code from the terminal
2. Save it to a file: `sum_even.py`
3. Test it:

```bash
python sum_even.py
```

Actually, let's add a quick test:

```python
if __name__ == "__main__":
    print(sum_even_numbers([1, 2, 3, 4, 5, 6]))  # Should print 12
    print(sum_even_numbers([]))                   # Should print 0
    print(sum_even_numbers([-2, -1, 0, 1, 2]))    # Should print 0
```

Run it:

```bash
python sum_even.py
# Output:
# 12
# 0
# 0
```

**It works!**

---

## Step 6: Try a More Complex Task (5 minutes)

Now ask the AI to write a test for this function:

```
Write pytest tests for this function:

```python
def sum_even_numbers(numbers: list[int]) -> int:
    """Calculate the sum of all even numbers in a list."""
    return sum(num for num in numbers if num % 2 == 0)
```

Requirements:
- Cover happy path, edge cases, error cases
- Use descriptive test names: test_{function}_{scenario}
- Each test should be independent

Return ONLY the test code.
```

Save the output as `test_sum_even.py` and run:

```bash
pip install pytest
pytest test_sum_even.py -v
```

You should see all tests pass.

---

## What Just Happened

You've completed the **Quick Start**:
- Installed Ollama (5 min)
- Pulled a model (5–10 min)
- Had AI write code (3 min)
- Saved and tested the code (5 min)
- Had AI write tests (5 min)

**Total time:** ~30 minutes

---

## What's Next

You've experienced the "magic" of AI coding. Now you're ready to understand how it works and how to get better results.

### Continue to Part 1: Beginner — Do Things

- [Chapter 1: Write New Code with AI](../01-beginner/01-write-code.md) — Learn to scaffold projects and generate code systematically
- [Chapter 2: Debug and Fix Bugs](../01-beginner/02-debug-fix.md) — Use AI to diagnose and fix errors
- [Chapter 3: Write Tests](../01-beginner/03-write-tests.md) — Generate comprehensive test suites

Or skip ahead:
- [Chapter 6: How Your Agent Thinks](../02-understand/01-how-agents-think.md) — Understand the Plan-Code-Verify loop
- [Chapter 17: Models and Quantization](../04-daily-driver/02-models-quantization.md) — Deep dive on model selection

---

## On Your Hardware

### If You Have 8GB RAM or Less

- Use `qwen2.5-coder:1.5b` or `phi3:mini`
- Expect 5–15 tokens/second
- Keep context small (don't load entire codebases)
- CPU-only is fine for this model size

### If You Have 16GB RAM

- Use `qwen2.5-coder:7b` (recommended)
- Should get 10–30 tokens/second with GPU
- CPU-only: ~3–8 tokens/second (still usable)

### If You Have 32GB+ RAM

- Use `qwen2.5-coder:14b` or `32b` for better quality
- Consider `qwen2.5-coder:70b` if you have 48GB+ VRAM
- You can run multiple models simultaneously

---

## Troubleshooting

### "Ollama command not found"

- Make sure Ollama is in your PATH
- Restart your terminal
- On macOS, try: `~/ollama/bin/ollama`

### "Model not found"

- Run `ollama pull qwen2.5-coder:7b` again
- Check your internet connection
- Try a different model: `ollama pull llama3.1:8b`

### "Out of memory"

- Use a smaller model: `qwen2.5-coder:1.5b`
- Close other GPU-intensive applications
- Reduce context window: `OLLAMA_NUM_CTX=2048`

### "Too slow"

- Use a smaller model
- Enable GPU acceleration (check Ollama docs for your GPU)
- Reduce thread count: `OLLAMA_NUM_THREAD=4`

---

## Next Steps

1. **Play around** — Ask the AI to write more functions
2. **Try different prompts** — See what works best
3. **Move to Chapter 1** — Learn the systematic approach

You're now ready to start **Part 1: Beginner — Do Things**.
