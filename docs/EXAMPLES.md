# Usage Examples

This document provides comprehensive examples for all usage scenarios of the Telemetry Injector.

## Table of Contents

1. [Quick Start](#quick-start)
2. [Basic Usage](#basic-usage)
3. [Multi-Provider Examples](#multi-provider-examples)
4. [Parallel Processing](#parallel-processing)
5. [Token Auto-Detection](#token-auto-detection)
6. [GPU VRAM Scheduling](#gpu-vram-scheduling)
7. [Debug Trace Logging](#debug-trace-logging)
8. [Cost Tracking](#cost-tracking)
9. [Benchmark Mode](#benchmark-mode)
10. [Advanced Scenarios](#advanced-scenarios)

---

## Quick Start

### Installation

```bash
# Clone repository
git clone <repository-url>
cd telemetry-injector

# Create virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

### Minimal Example

```bash
# Set LLM provider
export LLM_PROVIDER=ollama
export LLM_MODEL=codellama

# Instrument a file
python telemetry-inject.py --input examples/sample.py --output output/sample.py
```

---

## Basic Usage

### Example 1: Single File Instrumentation

```bash
# Instrument a Python file
python telemetry-inject.py --input examples/sample.py --output output/sample.py --verbose
```

**Output:**
```
🔍 Scanning: examples/sample.py
✓ Found 3 functions to instrument
🤖 Analyzing with LLM...
✓ Generated telemetry for calculate_total()
✓ Generated telemetry for process_order()
✓ Generated telemetry for validate_input()
📝 Injecting telemetry...
✅ File instrumented successfully!
```

### Example 2: Directory Processing

```bash
# Instrument all Python files in a directory
python telemetry-inject.py --input examples/ --output output/ --verbose
```

**Output:**
```
🔍 Scanning directory: examples/
Found 5 Python files
Processing: examples/api.py (4 functions)
Processing: examples/utils.py (7 functions)
Processing: examples/models.py (3 functions)
✅ Instrumented 3 files, 14 functions total
```

### Example 3: Dry Run Mode

```bash
# Preview what would be instrumented without making changes
python telemetry-inject.py --input examples/sample.py --dry-run
```

**Output:**
```
🔍 DRY RUN MODE - No files will be modified
Found 3 functions:
  1. calculate_total() - 15 lines
  2. process_order() - 28 lines
  3. validate_input() - 10 lines
Would generate telemetry for 3 functions
```

---

## Multi-Provider Examples

### Example 4: OpenAI GPT-4

```bash
# Configure OpenAI
export LLM_PROVIDER=openai
export OPENAI_API_KEY=sk-...
export LLM_MODEL=gpt-4o

# Instrument with GPT-4
python telemetry-inject.py --input examples/sample.py --output output/sample.py
```

**Features:**
- Auto-detects token limits (128K input, 16K output)
- Tracks costs (~$0.025/request)
- Fast response times (~2s)

### Example 5: Anthropic Claude

```bash
# Configure Anthropic
export LLM_PROVIDER=anthropic
export ANTHROPIC_API_KEY=sk-ant-...
export LLM_MODEL=claude-3-5-sonnet-20241022

# Instrument with Claude
python telemetry-inject.py --input examples/sample.py --output output/sample.py
```

**Features:**
- Large context window (200K tokens)
- High-quality output
- Competitive pricing (~$0.03/request)

### Example 6: Local Ollama

```bash
# Configure Ollama
export LLM_PROVIDER=ollama
export LLM_BASE_URL=http://localhost:11434/v1
export LLM_MODEL=codellama

# Instrument with local model
python telemetry-inject.py --input examples/sample.py --output output/sample.py
```

**Features:**
- Free (runs locally)
- No API key required
- Privacy (data stays local)
- Slower (~10-30s per request)

---

## Parallel Processing

### Example 7: Simple Parallel Execution

```bash
# Use multiple models in parallel (first valid response wins)
export LLM_PROVIDER=ollama
export LLM_MODEL=codellama
export MAX_WORKERS=4

python telemetry-inject.py --input examples/sample.py --max-parallel 4
```

**Output:**
```
🚀 Querying 4 models in parallel...
✓ codellama: Success in 8.2s
✓ codellama: Success in 8.5s
✓ codellama: Success in 9.1s
⏱️  Average response time: 8.6s
```

### Example 8: Ollama Model Pool Rotation

```bash
# Use different models for parallel requests (multi-GPU optimization)
export LLM_PROVIDER=ollama
export LLM_MODEL="cogito:8b,gpt-oss:20b,llama3"
export MAX_WORKERS=3

python telemetry-inject.py --input examples/ --max-parallel 3 --verbose
```

**Output:**
```
🔄 Ollama Model Pool: Using 3 models for rotation
   1. cogito:8b
   2. gpt-oss:20b
   3. llama3

🔍 Checking Ollama models...
   ✓ cogito:8b - Already available
   ✓ gpt-oss:20b - Already available
   ✓ llama3 - Already available

🚀 Processing with model rotation...
   Function 1 → cogito:8b (GPU 0)
   Function 2 → gpt-oss:20b (GPU 1)
   Function 3 → llama3 (GPU 2)
   Function 4 → cogito:8b (GPU 0)
```

**Why Model Rotation?**

Ollama cannot run the same model on multiple GPUs simultaneously. Model rotation enables true parallel processing by assigning different models to different GPUs.

---

## Token Auto-Detection

### Example 9: Automatic Token Detection

```bash
# Token limits are auto-detected and cached
export LLM_PROVIDER=openai
export LLM_MODEL=gpt-4o

python telemetry-inject.py --input examples/sample.py --verbose
```

**Output:**
```
🔍 Detecting token limits for gpt-4o...
✓ Detected: 128000 input tokens, 16384 output tokens
💾 Cached to: ~/.cache/telemetry_injector/token_limits.json
```

**Cache Location:**
```bash
cat ~/.cache/telemetry_injector/token_limits.json
```

```json
{
  "openai:gpt-4o": [128000, 16384],
  "anthropic:claude-3-5-sonnet-20241022": [200000, 8192],
  "ollama:codellama": [16384, 2048]
}
```

### Example 10: Custom Token Limits (Ollama)

```bash
# Override Ollama context window
export LLM_PROVIDER=ollama
export LLM_MODEL=codellama
export OLLAMA_NUM_CTX=32768  # Set custom context size

python telemetry-inject.py --input examples/sample.py --verbose
```

**Output:**
```
🔍 Detecting token limits for codellama...
✓ Using OLLAMA_NUM_CTX: 32768 input tokens, 2048 output tokens
```

---

## GPU VRAM Scheduling

### Example 11: Automatic GPU Detection (AMD ROCm)

```bash
# VRAM scheduler auto-detects AMD GPUs
export LLM_PROVIDER=ollama
export LLM_MODEL="cogito:8b,gpt-oss:20b"
export MAX_WORKERS=2

python telemetry-inject.py --input examples/ --verbose
```

**Output:**
```
🎮 GPU Scheduler: Detected rocm GPUs

📊 GPU Status:
   GPU 0: AMD Radeon RX 7900 XTX (24GB total, 22GB free)
   GPU 1: AMD Radeon RX 7900 XTX (24GB total, 23GB free)

📊 Model VRAM Estimates:
   cogito:8b → ~10GB (8B params)
   gpt-oss:20b → ~30GB (20B params)

📋 GPU Assignments:
   cogito:8b → GPU 0 (22GB free, 12GB after)
   gpt-oss:20b → GPU 1 (23GB free, -7GB after - may swap)
```

### Example 12: GPU Detection (NVIDIA CUDA)

```bash
# VRAM scheduler auto-detects NVIDIA GPUs
export LLM_PROVIDER=ollama
export LLM_MODEL="llama3,mistral"

python telemetry-inject.py --input examples/ --verbose
```

**Output:**
```
🎮 GPU Scheduler: Detected cuda GPUs

📊 GPU Status:
   GPU 0: NVIDIA GeForce RTX 4090 (24GB total, 20GB free, 15% util)
   GPU 1: NVIDIA GeForce RTX 4090 (24GB total, 22GB free, 5% util)

📋 GPU Assignments:
   llama3 → GPU 1 (22GB free, 12GB after)
   mistral → GPU 0 (20GB free, 10GB after)
```

### Example 13: Manual GPU Selection

```bash
# Manually specify GPU for Ollama
export CUDA_VISIBLE_DEVICES=1  # Use GPU 1 only
export LLM_PROVIDER=ollama
export LLM_MODEL=codellama

python telemetry-inject.py --input examples/sample.py
```

---

## Debug Trace Logging

### Example 14: Enable Debug Trace

```bash
# Enable comprehensive debug logging
export DEBUG_TRACE=true
export DEBUG_TRACE_LEVEL=TRACE  # Most detailed
export DEBUG_TRACE_CONSOLE=true

python telemetry-inject.py --input examples/sample.py --verbose
```

**Console Output:**
```
🔍 [10:30:00.123] [function_call] Calling analyze_code
     code: def calculate_total(items):...
     language: python

🔍 [10:30:00.234] [llm_request] Sending request to gpt-4o
     model: gpt-4o
     provider: openai
     prompt: Generate telemetry code for...
     prompt_length: 1523
     temperature: 0.7
     max_tokens: 16384

🔍 [10:30:02.456] [llm_response] Received response from gpt-4o
     model: gpt-4o
     input_tokens: 1500
     output_tokens: 800
     total_tokens: 2300
     duration_ms: 2222.00
     cost_usd: $0.045000

🔍 [10:30:02.567] [function_return] Returned from analyze_code
     function: analyze_code
     duration_ms: 2333.00
```

**Log File:**
```bash
cat logs/debug_trace_20251029_103000.jsonl | jq .
```

```json
{
  "type": "event",
  "timestamp": 1730192400.123,
  "datetime": "2025-10-29T10:30:00.123",
  "level": "TRACE",
  "category": "function_call",
  "message": "Calling analyze_code",
  "data": {
    "function": "analyze_code",
    "parameters": {
      "code": "def calculate_total(items):...",
      "language": "python"
    }
  }
}
```

### Example 15: Debug Levels

```bash
# INFO level - Major milestones only
export DEBUG_TRACE=true
export DEBUG_TRACE_LEVEL=INFO

python telemetry-inject.py --input examples/
```

**Output:**
```
ℹ️  [10:30:00.000] [parallel_execution] Starting parallel execution with 4 workers
     models: ["gpt-4o", "claude-3-5-sonnet", "codellama", "llama3"]
     worker_count: 4

ℹ️  [10:30:15.234] [parallel_execution] Completed parallel execution
     duration_ms: 15234.00
     success_count: 3
     failure_count: 1
```

### Example 16: Analyze Debug Logs

```bash
# Count events by category
cat logs/debug_trace_*.jsonl | jq -r '.category' | sort | uniq -c | sort -rn
```

**Output:**
```
     156 llm_response
     156 llm_request
      45 function_call
      45 function_return
      12 model_rotation
       8 gpu_assignment
       5 token_detection
       3 retry
       2 blacklist
```

---

## Cost Tracking

### Example 17: Budget Limit

```bash
# Set a budget limit ($5)
export LLM_PROVIDER=openai
export LLM_MODEL=gpt-4o

python telemetry-inject.py --input examples/ --budget 5.00 --verbose
```

**Output:**
```
💰 Budget: $5.00
Processing file 1... Cost: $0.12 (Remaining: $4.88)
Processing file 2... Cost: $0.15 (Remaining: $4.73)
Processing file 3... Cost: $0.18 (Remaining: $4.55)
...
Processing file 25... Cost: $0.14 (Remaining: $0.03)
❌ Budget exceeded! Stopping after 25 files.

Final cost: $5.03 / $5.00
```

### Example 18: Cost Tracking Output

```bash
# Track costs without limit
export LLM_PROVIDER=openai
export LLM_MODEL=gpt-4o

python telemetry-inject.py --input examples/ --verbose
```

**Output:**
```
💰 Cost Summary:
   Total API calls: 45
   Total tokens: 125,000 (input) + 38,000 (output)
   Total cost: $3.75
   Average cost per call: $0.083
```

---

## Benchmark Mode

### Example 19: Quality Benchmarking

```bash
# Use judge ensemble for highest quality
export LLM_PROVIDER=openai
export LLM_MODEL=gpt-4o

python telemetry-inject.py --input examples/sample.py --perform-benchmark --verbose
```

**Output:**
```
🏆 Benchmark Mode - Judge Ensemble Enabled

Function: calculate_total()
🚀 Querying all models...
   ✓ Model 1: Success in 2.3s
   ✓ Model 2: Success in 2.5s
   ✓ Model 3: Success in 2.8s
   ✓ Model 4: Success in 3.1s

📊 Quality Voting:
   Response 1: Score 95/100
   Response 2: Score 92/100
   Response 3: Score 88/100
   Response 4: Score 90/100

🏆 Selected: Response 1 (highest quality)
```

### Example 20: Model Comparison

```bash
# Compare multiple models
export LLM_PROVIDER=openai
export LLM_MODEL="gpt-4o,gpt-4o-mini,gpt-3.5-turbo"

python telemetry-inject.py --input examples/sample.py --perform-benchmark --verbose
```

**Output:**
```
📊 Model Comparison:
   gpt-4o:          Quality: 95/100, Speed: 2.3s, Cost: $0.045
   gpt-4o-mini:     Quality: 88/100, Speed: 1.8s, Cost: $0.008
   gpt-3.5-turbo:   Quality: 82/100, Speed: 1.5s, Cost: $0.002

🏆 Winner: gpt-4o (highest quality)
```

---

## Advanced Scenarios

### Example 21: Multi-Language Project

```bash
# Instrument mixed-language project
python telemetry-inject.py --input project/ --output output/ --verbose
```

**Output:**
```
🔍 Scanning directory: project/
Found files:
   Python: 15 files
   Go: 8 files
   JavaScript: 12 files

Processing Python files... ✓ 45 functions
Processing Go files... ✓ 32 functions
Processing JavaScript files... ✓ 67 functions

✅ Instrumented 35 files, 144 functions total
```

### Example 22: Streaming Large Files

```bash
# Process large files with streaming
export MAX_WORKERS=8

python telemetry-inject.py --input large_file.py --output output.py --max-parallel 8
```

**Output:**
```
🔍 Large file detected (5000 lines, 120 functions)
🚀 Processing in batches of 10 functions...

Batch 1/12: Functions 1-10 ✓ (12.3s)
Batch 2/12: Functions 11-20 ✓ (11.8s)
Batch 3/12: Functions 21-30 ✓ (13.1s)
...
✅ Completed in 2m 34s
```

### Example 23: CI/CD Integration

```bash
#!/bin/bash
# ci-instrument.sh - Instrument code in CI/CD pipeline

set -e

# Configure for CI
export LLM_PROVIDER=openai
export LLM_MODEL=gpt-4o-mini  # Fast and cheap for CI
export OPENAI_API_KEY=$CI_OPENAI_KEY
export MAX_WORKERS=4

# Instrument with budget limit
python telemetry-inject.py \
    --input src/ \
    --output instrumented/ \
    --budget 1.00 \
    --max-parallel 4 \
    --no-validate  # Skip validation for speed

# Run tests on instrumented code
pytest instrumented/

echo "✅ CI instrumentation complete"
```

### Example 24: Configuration File

```bash
# .telemetry-config.json
{
  "llm_provider": "ollama",
  "llm_model": "cogito:8b,gpt-oss:20b,llama3",
  "max_workers": 3,
  "budget_limit": null,
  "debug_trace": true,
  "debug_level": "INFO",
  "receiver_url": "http://localhost:8000/telemetry"
}
```

```bash
# Use config file
python telemetry-inject.py --config .telemetry-config.json --input examples/
```

### Example 25: Custom Receiver URL

```bash
# Send telemetry to custom endpoint
export RECEIVER_URL=http://telemetry.example.com/api/v1/ingest

python telemetry-inject.py --input examples/sample.py --output output/sample.py

# Run instrumented code
python output/sample.py
```

**Telemetry sent to receiver:**
```json
{
  "function_name": "calculate_total",
  "file": "sample.py",
  "start_time": "2025-10-29T10:30:00.000Z",
  "end_time": "2025-10-29T10:30:00.123Z",
  "duration_ms": 123,
  "status": "success",
  "inputs": {"items": "[...]"},
  "outputs": 250.50
}
```

---

## Environment Variables Reference

| Variable | Description | Example |
|----------|-------------|---------|
| `LLM_PROVIDER` | LLM provider (openai, anthropic, ollama) | `openai` |
| `LLM_MODEL` | Model name (comma-separated for rotation) | `gpt-4o` or `cogito:8b,llama3` |
| `LLM_BASE_URL` | Base URL for Ollama | `http://localhost:11434/v1` |
| `OPENAI_API_KEY` | OpenAI API key | `sk-...` |
| `ANTHROPIC_API_KEY` | Anthropic API key | `sk-ant-...` |
| `MAX_WORKERS` | Parallel worker count | `4` |
| `OLLAMA_NUM_CTX` | Ollama context size | `32768` |
| `DEBUG_TRACE` | Enable debug trace | `true` |
| `DEBUG_TRACE_LEVEL` | Log level | `TRACE`, `DEBUG`, `INFO` |
| `DEBUG_TRACE_CONSOLE` | Console output | `true` |
| `RECEIVER_URL` | Telemetry receiver endpoint | `http://localhost:8000/telemetry` |

---

## Summary

The Telemetry Injector supports:

✅ **Multiple LLM providers** (OpenAI, Anthropic, Ollama)
✅ **Parallel processing** with model rotation
✅ **Automatic token detection** with caching
✅ **VRAM-based GPU scheduling** (AMD/NVIDIA)
✅ **Comprehensive debug logging** (TRACE to ERROR)
✅ **Cost tracking** with budget limits
✅ **Benchmark mode** for quality comparison
✅ **Multi-language support** (Python, Go, C/C++, JavaScript)

For more details, see:
- [Architecture](ARCHITECTURE.md) - System architecture and diagrams
- [README](../README.md) - Quick start and configuration
