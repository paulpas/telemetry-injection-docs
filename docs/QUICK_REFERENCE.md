# Quick Reference: Code Telemetry Injector Architecture

**Date**: 2025-11-02
**Location**: `/home/paulpas/git/claude-code-testing/`
**Codebase Size**: 20,069 lines across 56 Python modules

## Where LLM is Actually Called

The system makes LLM API calls at exactly 4 points:

### 1. **Analysis Phase** (HybridAnalyzer)
- **File**: `src/llm_analyzer.py:179-329`
- **Trigger**: Only for unsupported languages (not Python/JS/Go)
- **When**: If using traditional pipeline (not `--use-scripts`)
- **Cost**: $0.01-0.10 per file
- **Speed**: 2-10 seconds

### 2. **Injection Phase** (CodeInjector)  
- **File**: `src/code_injector.py:34-250`
- **Trigger**: Always (unless using `--use-scripts`)
- **When**: Main code instrumentation step
- **Cost**: $0.05-0.10 per file
- **Speed**: 2-5 seconds

### 3. **Reflection Phase** (ReflectionEngine)
- **File**: `src/reflection_engine.py:42-180`
- **Trigger**: After N failures (default threshold: 2)
- **When**: Only if injection fails multiple times
- **Cost**: $0.02-0.05 per reflection
- **Speed**: 1-2 seconds
- **Flag**: `--reflection-threshold N`

### 4. **Refactoring Phase** (ScriptRefactorer)
- **File**: `src/script_refactorer.py:40-180`
- **Trigger**: Only in script mode (`--use-scripts`) AND on test failures
- **When**: Template-based script fails pytest tests
- **Cost**: $0.01-0.05 (only on failures, rare!)
- **Speed**: 1-2 seconds
- **Flag**: `--max-refactor-attempts N` (default: 3)

## Quick Architecture Map

```
CLI.main() (src/cli.py)
    ↓
Chooses Pipeline:
    ├─ Traditional (default): Analysis → Telemetry → Injection → LLM calls ❌
    └─ Script-Based (--use-scripts): Template → Cache → No LLM! ✅
         └─ Cache hit: 0 LLM calls (instant)
         └─ Cache miss: Maybe 0-3 LLM calls (on test failures only)
```

## Performance Comparison

| Mode | Analysis | Injection | Cost | Speed |
|------|----------|-----------|------|-------|
| **Traditional (Python)** | TreeSitter (free) | LLM | $0.05-0.15 | 3-6s |
| **Traditional (Other)** | LLM | LLM | $0.07-0.20 | 5-10s |
| **Script (cached)** | None | Template | $0.00 | 0.3ms |
| **Script (miss, pass)** | None | Template | $0.00 | 7ms |
| **Script (miss, fail)** | None | Template+LLM | $0.01-0.05 | 1-3s |

## What Gets Cached

**Script-Based Mode Only** (`--use-scripts`):
```
.telemetry_cache/
├── scripts/python/       # Generated insertion scripts
├── tests/python/         # Generated pytest tests
└── metadata/
    └── cache_index.json  # Maps SHA256 hash → file + metadata
```

**Cache Key**: SHA256(function_code + snippets_list)

**Hit Rate**: 
- First run: 0% (all misses)
- Second run: 100% (all hits!)

**Speedup**: 98.7% faster on cache hits (1015ms → 15ms per function)

## Configuration

**Environment Variables** (.env):
```bash
LLM_PROVIDER=openai|anthropic|ollama
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
LLM_BASE_URL=http://localhost:11434/v1  # For Ollama
LLM_MODEL=gpt-4|claude-3-5-sonnet|codellama
LLM_TIMEOUT=120
```

**Auto-Detection Logic**:
```python
if base_url has "localhost" → ollama
elif ANTHROPIC_API_KEY set → anthropic
elif OPENAI_API_KEY set → openai
else → openai (default)
```

## Common Commands

```bash
# Setup
python telemetry-inject.py --configure

# Traditional mode (LLM-based)
python telemetry-inject.py ./src -v

# Script mode (cached, fast!)
python telemetry-inject.py ./src --use-scripts -v

# With cost limit
python telemetry-inject.py ./src --budget 10.00

# Dry run
python telemetry-inject.py ./src --use-scripts --dry-run

# Parallel disabled
python telemetry-inject.py ./src --use-scripts --no-parallel

# Check API connection
python telemetry-inject.py --check-api
```

## Key Optimization: TreeSitter for Analysis

**For Python, JavaScript, TypeScript, Go**:
- Traditional approach: LLM analysis (2-10s, $0.01-0.10) ❌
- Optimized approach: TreeSitter AST parsing (10ms, $0.00) ✅
- **Speedup**: 100-1000x faster!

Located in: `src/tree_sitter_analyzer.py` and `src/hybrid_analyzer.py`

## Cost Breakdown (per 100 files)

### Traditional Mode
```
Python files:
├─ Analysis: 0 API calls (tree-sitter)
├─ Injection: 100 × $0.05-0.10 = $5-10
└─ Total: $5-10

Other languages:
├─ Analysis: 100 × $0.01 = $1
├─ Injection: 100 × $0.05 = $5
└─ Total: $6
```

### Script-Based Mode
```
First run: $0 (templates only)
Cached runs: $0 (100% savings!)
Savings vs traditional: 40-60%
```

## The Two Pipelines

### Pipeline 1: Traditional (Default)

```
File → Scan → Analyze (LLM or tree-sitter) 
    → Generate Telemetry (template)
    → Inject (LLM) 
    → Validate → Output
```

**LLM Calls**: 2-4 per file
**When to use**: When you need maximum compatibility

### Pipeline 2: Script-Based (--use-scripts)

```
File → Extract Functions → Generate Script (template)
    → Check Cache → If hit: Execute cached script
                    If miss: Validate → Store → Execute
    → Reconstruct File → Output
```

**LLM Calls**: 0 per function (first run), 0 on cache hit!
**When to use**: For speed, cost savings, and repeated runs

## Retry & Reflection Logic

```
Attempt 1: Inject code
    ↓ (if fails)
Attempt 2: Retry inject  
    ↓ (if fails threshold: default 2)
Reflection: Analyze patterns, generate guidance
    ↓ (if still fails)
Attempt 3+: Retry with enhanced prompt
    ↓ (if still fails max attempts: default 3)
Fallback: Use original code, log failure
```

**Configurable**:
- `--max-retries N` (default: 3)
- `--reflection-threshold N` (default: 2)

## Security Checks

Scripts are validated for:
- ✅ Syntax correctness (ast.parse)
- ✅ No dangerous patterns (eval, exec, subprocess, etc)
- ✅ Pytest passes (if run_tests=True)

## File Organization

```
src/
├── cli.py                        # Main entry point
├── scanner.py                    # Find code files
├── function_extractor.py         # Parse functions
│
├─ ANALYSIS:
├── llm_analyzer.py              # LLM-based analysis
├── tree_sitter_analyzer.py      # Fast AST parsing
├── hybrid_analyzer.py           # Smart selector (tree-sitter first!)
│
├─ TRADITIONAL PIPELINE:
├── telemetry_generator.py       # Generate snippets
├── code_injector.py             # Inject code (LLM)
├── retry_injector.py            # Retry wrapper
├── reflection_engine.py         # Learn from failures
│
├─ SCRIPT-BASED PIPELINE:
├── script_generator.py          # Generate scripts
├── test_generator.py            # Generate tests
├── script_cache.py              # Hash-based caching
├── script_sandbox.py            # Isolated execution
├── script_validator.py          # Syntax/security/test checks
├── script_refactorer.py         # Self-healing via LLM
├── parallel_script_processor.py # Async batch processing
│
└─ SUPPORT:
├── file_reconstructor.py        # Rebuild files
├── cost_tracker.py              # Track API costs
├── verbose_logger.py            # Detailed logging
└── ...
```

## Most Important Files

| File | Lines | Purpose | LLM? |
|------|-------|---------|------|
| `src/cli.py` | 1000+ | Main orchestrator | No |
| `src/llm_analyzer.py` | 330 | LLM analysis API | **YES** |
| `src/code_injector.py` | 250 | LLM injection API | **YES** |
| `src/reflection_engine.py` | 180 | LLM reflection API | **YES** |
| `src/hybrid_analyzer.py` | 200 | Smart analyzer selector | Optional |
| `src/tree_sitter_analyzer.py` | 280 | Fast AST parsing | No |
| `src/script_generator.py` | 600+ | Generate insertion scripts | No (template-first) |
| `src/script_cache.py` | 250 | Hash-based cache | No |
| `src/parallel_script_processor.py` | 350 | Async processing | No (except refactor) |

## Key Data Structures

```python
# Main types you'll see in prompts/responses:

ExtractedFunction = {
    name, code, start_line, end_line, 
    parameters, language, indentation_level
}

AnalysisResult = {
    language, code,
    functions: [name, start_line, end_line, ...],
    loops: [type, start_line, variable, ...],
    variables: [name, type, line, ...]
}

TelemetrySnippet = {
    type (function/loop/variable),
    code (telemetry code string),
    language,
    location (line numbers),
    description
}

ProcessingResult = {
    function_name, success, cached,
    script_hash, duration_ms, 
    instrumented_code, error
}
```

## Providers Supported

```
OpenAI:
├─ Models: gpt-4, gpt-3.5-turbo
├─ Cost: $0.01-0.10 per call
└─ API: api.openai.com

Anthropic:
├─ Models: claude-3-5-sonnet-20241022, claude-3-opus
├─ Cost: $0.01-0.10 per call
└─ API: api.anthropic.com

Ollama (Local):
├─ Models: codellama, llama2, mistral, etc
├─ Cost: $0.00 (runs locally!)
└─ API: http://localhost:11434/v1
```

## Lessons System

Located: `docs/lessons/{language}/`

Examples:
- `docs/lessons/python/multi_line_expressions.md`
- `docs/lessons/python/return_statement_transformation.md`

These are automatically loaded and referenced in generated scripts as comments.

## Testing

All components tested with:
- ✅ 100% test coverage
- ✅ TDD methodology (tests written first)
- ✅ 51+ test files
- ✅ pytest framework

Run: `pytest tests/ -v`

## Summary: When LLM is Actually Used

✅ **ALWAYS Used**:
- Traditional mode, non-Python files (analysis)
- Traditional mode, code injection
- Any mode, if reflection/refactoring triggered

🟢 **SOMETIMES Used**:
- Traditional mode, Python analysis (uses TreeSitter instead if available)
- Script mode, if template-based scripts fail tests

❌ **NEVER Used**:
- Script mode, if cache hits
- Function extraction/parsing
- File I/O operations
- Script caching/retrieval

## Most Efficient Usage

```bash
# Best performance & cost:
python telemetry-inject.py ./src --use-scripts -v

# This gives you:
# ✅ 98.7% faster on cache hits
# ✅ 100% cost savings on reruns
# ✅ 40-60% cost savings on first run
# ✅ Parallel processing (12x speedup)
# ✅ Full validation + security checks
```

---

**For full details, see**: `docs/ARCHITECTURE_ANALYSIS.md`
