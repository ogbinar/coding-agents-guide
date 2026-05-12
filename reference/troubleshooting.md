# Reference: Troubleshooting

> Symptom → cause → fix lookup table.

---

## Installation Issues

### "Ollama command not found"

**Cause:** Ollama not in PATH or not installed

**Fix:**
```bash
# Check if installed
which ollama

# If not found, reinstall
curl -fsSL https://ollama.com/install.sh | sh

# On macOS, try full path
~/ollama/bin/ollama --version

# Restart terminal after install
```

### "Permission denied" on install script

**Cause:** Script needs execute permissions

**Fix:**
```bash
curl -fsSL https://ollama.com/install.sh -o install.sh
chmod +x install.sh
./install.sh
```

### "CUDA not available" on NVIDIA GPU

**Cause:** NVIDIA drivers not installed or outdated

**Fix:**
```bash
# Check NVIDIA drivers
nvidia-smi

# If not working, install drivers
# Ubuntu/Debian:
sudo apt update
sudo apt install nvidia-driver-535  # or latest version

# Reboot
sudo reboot
```

---

## Model Loading Issues

### "Out of memory" when loading model

**Cause:** Model too large for your RAM/VRAM

**Fix:**
1. Use a smaller model:
   ```bash
   ollama pull qwen2.5-coder:1.5b  # Instead of 7B
   ```

2. Reduce context window:
   ```bash
   OLLAMA_NUM_CTX=2048 ollama run qwen2.5-coder:7b
   ```

3. For llama.cpp, enable layer offloading:
   ```bash
   ./main -m model.gguf -ngl 35  # Partial GPU offload
   ```

### "Model loads but crashes"

**Cause:** Corrupted download or incompatible quantization

**Fix:**
```bash
# Remove and re-download
ollama rm qwen2.5-coder:7b
ollama pull qwen2.5-coder:7b

# Try a different quantization
ollama pull qwen2.5-coder:7b:q4_k_m  # Explicit Q4
```

---

## Performance Issues

### "Too slow" (< 2 tokens/second)

**Cause:** CPU-only, wrong thread count, or model too large

**Fix:**
1. Enable GPU if available
2. Reduce model size
3. Tune thread count:
   ```bash
   # Match physical cores (not hyperthreads)
   OLLAMA_NUM_THREAD=8 ollama run qwen2.5-coder:7b
   ```

4. Use a faster quantization:
   ```bash
   ollama pull qwen2.5-coder:7b:q4_0  # Faster than q4_k_m
   ```

### "Inconsistent speeds"

**Cause:** Background processes, thermal throttling, or GPU memory pressure

**Fix:**
1. Close other GPU-intensive applications
2. Check thermal throttling:
   ```bash
   # Linux
   watch -n 1 cat /proc/acpi/thermal_zone/*/temperature
   ```

3. Limit concurrent model instances

---

## Output Quality Issues

### "Hallucinates APIs"

**Cause:** Model doesn't know your specific libraries

**Fix:**
1. Add constraints to prompt:
   ```
   Use ONLY Python standard library. Do NOT use external packages.
   ```

2. Provide few-shot examples:
   ```
   Example 1:
   Input: "Parse CSV"
   Output: import csv ... (correct standard library usage)
   ```

3. Use a coding-specific model (Qwen-Coder, DeepSeek-Coder)

### "Ignores output format constraints"

**Cause:** Model too small or temperature too high

**Fix:**
1. Lower temperature:
   ```bash
   ollama run qwen2.5-coder:7b --temperature 0.1
   ```

2. Use XML tags instead of JSON:
   ```
   Wrap your answer in <answer> tags.
   ```

3. Add stronger constraints:
   ```
   IMPORTANT: Return ONLY the JSON. No markdown, no explanation.
   If you include any text outside the JSON, I will reject it.
   ```

### "Produces invalid code"

**Cause:** Model size too small or insufficient context

**Fix:**
1. Use a larger model or better quantization
2. Provide more context:
   ```
   Here's the existing code:
   {paste relevant code}
   
   Write a function that integrates with this:
   ```

3. Ask for self-review:
   ```
   After writing the code, review it for syntax errors and fix them.
   ```

---

## Tool Use Issues

### "Model doesn't use tools correctly"

**Cause:** Tool descriptions unclear or too many tools

**Fix:**
1. Simplify tool descriptions:
   ```
   # BAD:
   search(query: string, options?: {limit: number, offset: number})
   
   # GOOD:
   search(query) - Search for information
     query: The search query string
   ```

2. Limit tools presented at once (max 5–8)

3. Provide examples:
   ```
   Example:
   User: "What's the weather in Tokyo?"
   Thought: I need to check the weather.
   Action: {"tool": "weather_search", "args": {"city": "Tokyo"}}
   ```

### "Tool calls malformed (JSON parse error)"

**Cause:** Model produces invalid JSON

**Fix:**
1. Use XML tag pattern instead:
   ```
   To use a tool, write:
   <tool name="search">
     <arg name="query">latest AI news</arg>
   </tool>
   ```

2. Enable JSON mode if your engine supports it

3. Add post-processing to recover:
   ```python
   import json
   import re
   
   def extract_json(text):
       # Try to extract JSON from markdown fences
       match = re.search(r'```json\s*(.*?)\s*```', text, re.DOTALL)
       if match:
           return json.loads(match.group(1))
       return json.loads(text)  # Try direct parse
   ```

---

## Context Issues

### "Model forgets earlier conversation"

**Cause:** Context window exceeded

**Fix:**
1. Increase context window:
   ```bash
   OLLAMA_NUM_CTX=8192 ollama run qwen2.5-coder:7b
   ```

2. Summarize conversation history periodically

3. Use session memory patterns:
   ```
   Summary of what we've done so far:
   - Created function X
   - Wrote tests for X
   - Fixed bug Y
   
   Now let's {next task}
   ```

### "Context overflow error"

**Cause:** Input + history exceeds model's context window

**Fix:**
1. Start a new session
2. Provide a summary instead of full history
3. Load only relevant files (not entire codebase)

---

## Coding-Specific Issues

### "Code doesn't compile/build"

**Cause:** Missing imports, wrong syntax, or framework confusion

**Fix:**
1. Specify language and framework explicitly:
   ```
   Write Python 3.10 code using only standard library.
   ```

2. Ask for self-review before output:
   ```
   Before returning the code, verify:
   - All imports are valid
   - Syntax is correct
   - Type hints are complete
   ```

3. Run linter/type checker and feed errors back:
   ```
   The code produced these errors:
   {paste error}
   
   Fix the issues.
   ```

### "Tests fail on generated code"

**Cause:** Edge cases not handled or incorrect logic

**Fix:**
1. Provide test expectations:
   ```
   Write a function that:
   - Returns X for input Y
   - Handles empty input by returning Z
   - Raises ValueError for negative numbers
   ```

2. Ask for test-driven development:
   ```
   First write tests, then implement the function to pass those tests.
   ```

3. Iterate:
   ```
   These tests failed:
   {paste test output}
   
   Fix the code to pass all tests.
   ```

### "Diff doesn't apply cleanly"

**Cause:** Line numbers changed or context mismatch

**Fix:**
1. Use file writes instead of diffs for large changes
2. Provide exact file paths and content
3. For diffs, include more context lines:
   ```
   Return a unified diff with 5 lines of context.
   ```

---

## Hardware-Specific Issues

### "Mac: Metal not available"

**Cause:** llama.cpp not compiled with Metal support

**Fix:**
```bash
# Rebuild with Metal
cd llama.cpp
rm -rf build
mkdir build && cd build
cmake .. -DGGML_METAL=ON
make -j$(sysctl -n hw.ncpu)
```

### "Linux: CUDA out of memory"

**Cause:** GPU memory exhausted by other processes

**Fix:**
```bash
# Check GPU memory usage
nvidia-smi

# Kill processes using GPU
sudo kill -9 <PID>

# Or use less VRAM:
./main -m model.gguf -ngl 35  # Offload fewer layers
```

### "Windows: Slow performance"

**Cause:** Antivirus scanning or power settings

**Fix:**
1. Add Ollama/llama.cpp to antivirus exclusion
2. Set power plan to "High performance"
3. Close background applications

---

## Quick Diagnostic Commands

```bash
# Check system resources
free -h           # RAM usage
nvidia-smi        # GPU usage (NVIDIA)
top               # CPU usage

# Check Ollama status
ollama list       # Installed models
ollama ps         # Running models

# Test model performance
time ollama run qwen2.5-coder:7b "Say hello"

# Check logs
tail -f ~/.ollama/logs/server.log  # Ollama logs
```

---

## When to Seek Help

| Issue | Try First | If That Fails |
|---|---|---|
| Installation | Re-run install script | Check system requirements |
| Slow performance | Smaller model, tune threads | Check hardware compatibility |
| Bad output quality | Better prompts, higher quant | Try different model |
| Crashes | Re-download model | Check system logs |
| Tool use failures | Simplify tool schema | Use XML pattern instead |

---

*See also: [Models](./models.md), [Prompt Templates](./prompt-templates.md)*
