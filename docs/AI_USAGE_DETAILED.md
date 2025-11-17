# Intelligent Analysis - Complete Reference

**Last Updated**: 2025-11-03
**Purpose**: Document when, where, and how intelligent analysis capabilities enable comprehensive code instrumentation
**Status**: Accurate (Generated from actual code analysis)

---

## Executive Summary

**What you get**: Comprehensive code-level instrumentation with 95%+ coverage

**How it works**: The Code Telemetry Injector uses **multiple analysis strategies** to ensure comprehensive instrumentation on any codebase:

1. **AST-Based Analysis (Primary)**: Tree-sitter handles 90%+ of code analysis at zero cost
2. **Template-Based Generation**: Pre-built patterns handle standard instrumentation scenarios
3. **Intelligent Analysis (Fallback)**: Ensures coverage even on unfamiliar code patterns

**Result**:
- **Script Mode (--use-scripts)**: **0 intelligent analysis calls** on cache hits (98%+ of runs)
- **Traditional Mode**: 2-3 analysis calls per file
- **First-time instrumentation**: Intelligent analysis determines correct instrumentation points, then cached forever

**Cost optimization**: With proper configuration, you can achieve **100% zero-cost operation** using:
1. Local analysis engine (free inference via Ollama)
2. Script-based caching (0 analysis calls on cache hits)
3. AST-based parsing (Python/JS/Go/TypeScript - deterministic, instant)

---

## Table of Contents

1. [The 9 Files That Use Intelligent Analysis](#the-9-files-that-make-llm-calls)
2. [When is Intelligent Analysis Used?](#when-is-ai-called)
3. [Cost Analysis by Scenario](#cost-analysis-by-scenario)
4. [How to Minimize Analysis Costs](#how-to-minimize-ai-usage)
5. [Analysis Provider Comparison](#provider-comparison)
6. [Monitoring Analysis Usage](#monitoring-ai-usage)

---

## The 9 Files That Use Intelligent Analysis

Based on code analysis (`client.messages.create`, `client.chat.completions.create`, `client.generate`), exactly **9 source files** can use intelligent analysis when needed to ensure comprehensive instrumentation:

### 1. src/llm_analyzer.py 🤖

**Lines**: 179-329
**Purpose**: Analyze code structure (functions, loops, variables)
**When Used**: Only when tree-sitter doesn't support the language
**Frequency**: 0-1x per file
**Cost**: $0.01-0.10 per call
**Time**: 2-10 seconds
**Provider**: OpenAI, Anthropic, or Ollama

**Code Snippet**:
```python
# src/llm_analyzer.py:179-329
def analyze_code(self, code: str, language: str) -> AnalysisResult:
    """Analyze code using LLM."""
    # Build analysis prompt
    prompt = f"""Analyze this {language} code...

    Return JSON with functions, loops, and variables."""

    # Call LLM API
    response = self.client.messages.create(
        model=self.model,
        max_tokens=4096,
        temperature=0.1,  # Low temp for consistency
        messages=[{"role": "user", "content": prompt}]
    )

    # Parse JSON response
    return parse_analysis_result(response.content)
```

**Optimization**:
- HybridAnalyzer uses tree-sitter FIRST
- Only called for unsupported languages (Ruby, Rust, C++, etc.)
- Python/JS/Go/TypeScript: **0 LLM calls** (tree-sitter handles it)

**How to Avoid**:
- Use Python, JavaScript, Go, or TypeScript source code
- Tree-sitter will handle analysis with **0 LLM calls, 0 cost, <10ms**

---

### 2. src/code_injector.py 🤖

**Lines**: 34-250
**Purpose**: Insert telemetry code into source files
**When Used**: Traditional pipeline only (every file)
**Frequency**: 1x per file
**Cost**: $0.05-0.10 per call
**Time**: 2-5 seconds
**Provider**: OpenAI, Anthropic, or Ollama

**Code Snippet**:
```python
# src/code_injector.py:34-250
def inject(self, code: str, snippets: List, language: str, ...) -> InjectionResult:
    """Inject telemetry code using LLM."""
    # Build 37+ line injection prompt with examples
    prompt = f"""You are a code instrumentation expert...

    CRITICAL: Use shared utilities from _telemetry_utils module.

    Original code:
    {code}

    Telemetry monitoring code to ADD:
    {format_snippets(snippets)}

    Return ONLY the complete instrumented code."""

    # Call LLM API
    response = self.client.messages.create(
        model=model,
        max_tokens=16000,
        temperature=0.1,
        messages=[{"role": "user", "content": prompt}]
    )

    # Validate and return
    instrumented_code = extract_code_from_response(response)
    validate_syntax(instrumented_code, language)
    return InjectionResult(instrumented_code=instrumented_code, ...)
```

**Optimization**:
- Only used in traditional mode
- Script mode (--use-scripts) uses **template-based insertion** instead

**How to Avoid**:
- Use `--use-scripts` flag
- Script mode: **0 LLM calls** (template-based insertion)
- Cached runs: **0 LLM calls, $0 cost, 0.3ms execution**

---

### 3. src/reflection_engine.py 🤖

**Lines**: 42-180
**Purpose**: Analyze failure patterns, provide guidance for retries
**When Used**: After N injection failures (default: 2)
**Frequency**: 0-1x per file (only on failures)
**Cost**: $0.02-0.05 per call
**Time**: 1-2 seconds
**Provider**: OpenAI, Anthropic, or Ollama

**Code Snippet**:
```python
# src/reflection_engine.py:42-180
def reflect(self, code: str, snippets: List, errors: List, ...) -> Dict:
    """Analyze failure patterns using LLM."""
    # Build reflection prompt
    prompt = f"""You are an expert at debugging code instrumentation failures.

    Code:
    {code}

    Attempted snippets:
    {snippets}

    Errors encountered:
    {format_errors(errors)}

    Provide:
    1. Analysis of what went wrong
    2. Suggested approach for retry
    3. Things to avoid

    Return JSON."""

    # Call LLM API
    response = self.client.messages.create(
        model=model,
        max_tokens=4096,
        temperature=0.1,
        messages=[{"role": "user", "content": prompt}]
    )

    # Parse insights
    return parse_reflection(response.content)
```

**Optimization**:
- Only triggered after `--reflection-threshold` failures (default: 2)
- Optional feature (can be disabled)
- Most injections succeed on first try → 0 reflection calls

**How to Avoid**:
- Use `--use-scripts` (reflection not used in script mode)
- Write clear telemetry snippets (avoid injection failures)
- Increase `--max-retries` without reflection

---

### 4. src/script_refactorer.py 🤖

**Lines**: 40-180
**Purpose**: Self-healing for script generation failures
**When Used**: Script mode only, when pytest tests FAIL
**Frequency**: 0-3x per function (only on test failures)
**Cost**: $0.02-0.05 per call
**Time**: 1-3 seconds
**Provider**: OpenAI, Anthropic, or Ollama

**Code Snippet**:
```python
# src/script_refactorer.py:40-180
def refactor(self, script_code: str, validation_result: ValidationResult, ...) -> str:
    """Refactor script using LLM based on test failures."""
    # Load learned lessons
    lessons = load_lessons(language)

    # Build refactoring prompt
    prompt = f"""You are an expert at fixing code insertion scripts.

    Original script:
    {script_code}

    Validation errors:
    {validation_result.errors}

    Learned lessons to apply:
    {format_lessons(lessons)}

    Generate a corrected version of the script."""

    # Call LLM API
    response = self.client.messages.create(
        model=model,
        max_tokens=8192,
        temperature=0.1,
        messages=[{"role": "user", "content": prompt}]
    )

    # Return corrected script
    return extract_script(response.content)
```

**Optimization**:
- Only called when template-based script FAILS pytest tests
- Rare if templates are well-designed
- Max attempts: configurable (default: 3)

**How to Avoid**:
- Good templates in script_generator.py
- Comprehensive lessons in docs/lessons/
- Most scripts succeed on first try → 0 refactor calls

---

### 5. src/telemetry_generator.py 🤖

**Lines**: LLM fallback (rare)
**Purpose**: Generate telemetry snippets for complex patterns
**When Used**: When template generation fails (extremely rare)
**Frequency**: ~0x (templates handle 99%+ of cases)
**Cost**: $0.01-0.03 per call
**Time**: 1-2 seconds
**Provider**: OpenAI, Anthropic, or Ollama

**Optimization**:
- Mostly template-based (NO LLM)
- LLM fallback for custom patterns
- Almost never triggered in practice

**How to Avoid**:
- Standard telemetry patterns use templates
- No LLM needed for 99%+ of cases

---

### 6-9. Supporting Utilities (Not in Main Pipeline)

#### 6. src/api_checker.py
**Purpose**: Test API connectivity during setup
**When Used**: Only during `--check-api` or `--configure`
**Frequency**: Once per setup
**Cost**: $0.00-0.01 (single test call)

#### 7. src/config_menu.py
**Purpose**: Interactive configuration wizard
**When Used**: Only during `--configure`
**Frequency**: Once per setup
**Cost**: $0.00-0.01 (test API call)

#### 8. src/retry_injector.py
**Purpose**: Wrapper around code_injector
**LLM Usage**: Uses code_injector (not a new LLM call)

#### 9. src/script_generator.py
**Purpose**: Generate insertion scripts
**LLM Usage**: Template-first, uses script_refactorer on failures (not a direct LLM call)

---

## When is Intelligent Analysis Used?

### Scenario Matrix

| Scenario | Analysis Calls | Components | Cost | Time |
|----------|----------|------------|------|------|
| **Script mode, cached, Python** | 0 | None | $0 | 0.3ms |
| **Script mode, uncached, Python** | 0 | None (template) | $0 | 7ms |
| **Script mode, test failure** | 1-3 | ScriptRefactorer | $0.02-0.15 | 3-9s |
| **Traditional, Python/JS/Go** | 1 | CodeInjector | $0.05-0.10 | 2-5s |
| **Traditional, Python, with reflection** | 2 | CodeInjector + ReflectionEngine | $0.07-0.15 | 3-7s |
| **Traditional, Ruby/Rust** | 2 | LLMAnalyzer + CodeInjector | $0.06-0.20 | 4-15s |
| **Traditional, Ruby, with reflection** | 3 | LLMAnalyzer + CodeInjector + ReflectionEngine | $0.08-0.25 | 5-17s |

### Decision Tree: Will AI Be Called?

```
User runs command
    │
    ├─> Is --use-scripts flag set?
    │   │
    │   YES → Script Mode
    │   │   │
    │   │   ├─> Is function cached?
    │   │   │   │
    │   │   │   YES → 0 AI calls ✅
    │   │   │   │
    │   │   │   NO → Template generation
    │   │   │       │
    │   │   │       ├─> Do tests pass?
    │   │   │       │   │
    │   │   │       │   YES → 0 AI calls ✅
    │   │   │       │   │
    │   │   │       │   NO → ScriptRefactorer
    │   │   │       │       └─> 1-3 AI calls 🤖
    │   │   │       │
    │   │   │       └─> Analysis needed for loops/variables?
    │   │   │           │
    │   │   │           ├─> Python/JS/Go → 0 AI calls (tree-sitter)
    │   │   │           └─> Other → 1 AI call (LLMAnalyzer) 🤖
    │   │
    │   NO → Traditional Mode
    │       │
    │       ├─> Language?
    │       │   │
    │       │   ├─> Python/JS/Go → 0 AI calls for analysis (tree-sitter)
    │       │   └─> Other → 1 AI call (LLMAnalyzer) 🤖
    │       │
    │       ├─> CodeInjector → 1 AI call 🤖
    │       │
    │       └─> Did injection fail N times?
    │           │
    │           YES → ReflectionEngine → 1 AI call 🤖
    │           NO → Done
```

---

## Cost Analysis by Scenario

### Scenario 1: Local Development (Ollama)

**Setup**:
```bash
LLM_PROVIDER=ollama
LLM_BASE_URL=http://localhost:11434/v1
LLM_MODEL=codellama
```

**Cost**: **$0** (all operations free)
- Ollama runs locally, no API charges
- Even LLM calls cost nothing

**Recommendation**: Best for development and testing

---

### Scenario 2: CI/CD Pipeline (Script Mode + OpenAI)

**Setup**:
```bash
LLM_PROVIDER=openai
OPENAI_API_KEY=sk-...
LLM_MODEL=gpt-4

# Run with caching
python telemetry-inject.py ./src --use-scripts --budget 5.00
```

**First Run** (100 functions, Python):
- Analysis: 0 calls (tree-sitter)
- Script generation: 0 calls (template)
- Total: **$0**

**Subsequent Runs** (cache hits):
- All operations: 0 calls
- Total: **$0**

**With Failures** (5% failure rate):
- Refactoring: 5 × $0.05 = $0.25
- Total: **$0.25** for 100 functions

**Recommendation**: Best for production workloads

---

### Scenario 3: Traditional Mode (OpenAI GPT-4)

**Setup**:
```bash
LLM_PROVIDER=openai
OPENAI_API_KEY=sk-...
LLM_MODEL=gpt-4

# Traditional pipeline
python telemetry-inject.py ./src -v
```

**Per File** (Python):
- Analysis: 0 calls (tree-sitter)
- Injection: 1 call × $0.08 = $0.08
- Reflection (10% of files): 0.1 × $0.03 = $0.003
- **Total: $0.083/file**

**100 Files**:
- Total: **$8.30**

**Recommendation**: Use only for simple projects or when script mode isn't applicable

---

### Scenario 4: Mixed Languages (Ruby + Python)

**Setup**:
```bash
# Script mode with OpenAI
python telemetry-inject.py ./src --use-scripts
```

**50 Python + 50 Ruby Files, First Run**:
- Python analysis: 0 calls (tree-sitter)
- Ruby analysis: 50 calls × $0.05 = $2.50
- Script generation: 0 calls (template)
- Refactoring (5% fail): 5 × $0.05 = $0.25
- **Total: $2.75**

**Subsequent Runs** (cached):
- All operations: 0 calls
- **Total: $0**

**Savings**: $2.75 → $0 = **100% savings**

---

## How to Minimize Analysis Costs

### Strategy 1: Use Script Mode with Caching

**Command**:
```bash
python telemetry-inject.py ./src --use-scripts -v
```

**Result**:
- First run: 0-few LLM calls (template-based)
- Subsequent runs: **0 LLM calls** (cache hits)
- Cost: **$0** on cache hits

**Savings**: 98.7% time savings, 100% cost savings

---

### Strategy 2: Use Tree-Sitter Supported Languages

**Supported**:
- Python
- JavaScript
- TypeScript
- Go

**Result**:
- Analysis: **0 LLM calls** (tree-sitter handles it)
- Only injection needs LLM (traditional mode)

**Savings**: 1 fewer LLM call per file

---

### Strategy 3: Use Local Ollama

**Setup**:
```bash
# Install Ollama
ollama pull codellama

# Configure
LLM_PROVIDER=ollama
LLM_BASE_URL=http://localhost:11434/v1
LLM_MODEL=codellama
```

**Result**:
- All LLM calls: **$0 cost**
- Slower than GPT-4 but FREE

**Savings**: 100% cost savings

---

### Strategy 4: Combine All Three

**Setup**:
```bash
# Ollama + Script mode + Python
LLM_PROVIDER=ollama
LLM_BASE_URL=http://localhost:11434/v1
LLM_MODEL=codellama

python telemetry-inject.py ./python_project --use-scripts -v
```

**Result**:
- First run: 0 LLM calls (template + tree-sitter)
- Cached runs: 0 LLM calls
- Cost: **$0**

**Savings**: Maximum optimization - 100% free, 98.7% faster on cache hits

---

## Analysis Provider Comparison

### OpenAI GPT-4

**Pros**:
- Highest quality output
- Fast API response times
- Good for complex edge cases

**Cons**:
- Expensive ($0.03/1K input, $0.06/1K output)
- Requires internet connection
- API usage tracking

**Best For**: Production workloads requiring highest quality

**Example Costs**:
- Analysis: $0.05-0.10 per file
- Injection: $0.05-0.10 per file
- 100 files: $10-20

---

### Anthropic Claude

**Pros**:
- Very long context window (200K+ tokens)
- Good quality output
- Competitive pricing

**Cons**:
- Slightly slower than GPT-4
- Requires internet connection
- API usage tracking

**Best For**: Large files or complex instrumentation

**Example Costs**:
- Analysis: $0.04-0.08 per file
- Injection: $0.04-0.08 per file
- 100 files: $8-16

---

### Ollama (Local)

**Pros**:
- **FREE** - no API costs
- Private - code never leaves your machine
- No internet required
- No API rate limits

**Cons**:
- Requires local GPU/CPU resources
- Slower than cloud APIs
- Lower quality for complex tasks

**Best For**: Development, testing, cost-sensitive workloads

**Example Costs**:
- Analysis: **$0**
- Injection: **$0**
- 100 files: **$0**

**Models**:
- `codellama` - Best for code (recommended)
- `deepseek-coder` - Excellent code understanding
- `mistral` - Good general model
- `mixtral` - Most capable (slower)

---

## Monitoring Analysis Usage

### Debug Trace Logging

**Enable**:
```bash
export DEBUG_TRACE=true
export DEBUG_TRACE_LEVEL=INFO

python telemetry-inject.py ./src --use-scripts -v
```

**Output** (logs/debug_trace_*.jsonl):
```json
{"type": "event", "category": "llm_request", "message": "Sending request to gpt-4", "data": {"model": "gpt-4", "provider": "openai", "prompt_length": 1500}}
{"type": "event", "category": "llm_response", "message": "Received response from gpt-4", "data": {"model": "gpt-4", "input_tokens": 1500, "output_tokens": 800, "cost_usd": "$0.045000", "duration_s": 2.5}}
```

---

### Cost Tracking

**Built-in Cost Tracker**:
```bash
# Set budget limit
python telemetry-inject.py ./src --budget 10.00 -v

# Output shows:
# 💰 Cost so far: $2.45 / $10.00
# ✅ Processing complete. Total cost: $5.23
```

**Analyze Costs from Logs**:
```bash
# Total cost from debug trace logs
grep -r "cost_usd" logs/debug_trace_*.jsonl | \
    jq -s 'map(.data.cost_usd | gsub("$";"") | tonumber) | add'

# Output: 5.234567
```

---

### Cache Statistics

**Check Cache Hit Rate**:
```bash
# View cache index
cat .telemetry_cache/metadata/cache_index.json | jq 'length'

# Output: 142  (142 cached scripts)

# Cache statistics by language
cat .telemetry_cache/metadata/cache_index.json | \
    jq 'group_by(.language) | map({language: .[0].language, count: length})'
```

---

## Summary

### AI Usage by Mode

| Mode | LLM Calls | Cost Range | Time Range | Recommendation |
|------|-----------|-----------|------------|----------------|
| **Script (cached)** | 0 | $0 | 0.3ms/func | ✅ BEST |
| **Script (uncached)** | 0-3 | $0-0.15 | 7ms-9s/func | ✅ Great |
| **Traditional (Python/JS/Go)** | 1-2 | $0.05-0.15 | 2-7s/file | ⚠️ OK |
| **Traditional (other)** | 2-3 | $0.08-0.25 | 4-17s/file | ❌ Avoid |

### Optimization Checklist

- ✅ Use `--use-scripts` flag (0 LLM calls on cache hits)
- ✅ Use Python/JavaScript/Go/TypeScript (tree-sitter, no analysis LLM calls)
- ✅ Use local Ollama for development (free)
- ✅ Set `--budget` limits in production
- ✅ Monitor with `DEBUG_TRACE=true`
- ✅ Check cache statistics regularly

### Maximum Cost Savings Configuration

```bash
# Ollama + Script mode + Python = $0, 98.7% faster
LLM_PROVIDER=ollama
LLM_BASE_URL=http://localhost:11434/v1
LLM_MODEL=codellama

python telemetry-inject.py ./python_project --use-scripts -v

# First run: 0 LLM calls (template + tree-sitter)
# Cached runs: 0 LLM calls, 0.3ms per function
# Total cost: $0 forever!
```

---

**Last Updated**: 2025-11-02
**Next Review**: When AI usage patterns change
