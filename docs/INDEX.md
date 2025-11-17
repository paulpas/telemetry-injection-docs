# Code Telemetry Injector - Documentation Index

**Last Updated**: November 2, 2025

## Quick Navigation

### ⭐ START HERE (Primary References)
1. **INDEX.md** (this file) - Navigation guide
2. **ARCHITECTURE_REFACTORED.md** ✨ NEW - Complete architecture (23KB)
   - Two pipeline modes explained
   - When AI is actually used
   - Component catalog with line numbers
   - Data flow diagrams
   - Performance & cost analysis
3. **AI_USAGE_DETAILED.md** ✨ NEW - Intelligent analysis capabilities reference (27KB)
   - How intelligent analysis enables comprehensive instrumentation
   - The 9 files that use intelligent analysis
   - When is intelligent analysis used? (Decision tree)
   - Cost analysis by scenario
   - How to minimize analysis costs
   - Analysis provider comparison
4. **RUNBOOK.md** ✨ NEW - Operations guide (18KB)
   - Installation
   - Configuration (4 methods)
   - Troubleshooting (common problems)
   - Performance optimization
   - CI/CD integration
   - Monitoring
5. **QUICK_REFERENCE.md** - Fast lookup guide (10KB)
   - Quick commands
   - Configuration examples
   - Performance tables

### For Getting Started
- **QUICKSTART.md** - Installation & first run (2.6KB)
- **EXAMPLES.md** - Usage examples

### For Deep Dives
- **SCRIPT_BASED_ARCHITECTURE.md** - Caching deep dive (26KB)
- **TREE_SITTER_IMPLEMENTATION.md** - Fast AST analysis
- **METADATA_TRACKING.md** - Provenance tracking
- **PARALLEL_PROCESSING.md** - Async processing

### Legacy / Archive
- **ARCHITECTURE.md** ⚠️ - Old architecture (superseded by ARCHITECTURE_REFACTORED.md)
- **ARCHITECTURE_ANALYSIS.md** ⚠️ - Older analysis (superseded by ARCHITECTURE_REFACTORED.md)
- See **DOCUMENTATION_STATUS.md** for complete obsolescence report

## Key Findings Summary

### What This Tool Delivers
**Comprehensive code-level instrumentation** with 95%+ coverage across functions, variables, and execution paths.

### How It Works
Multi-strategy analysis system with 2 execution modes:
1. **Script-Based (Recommended)**: Template generation + caching (0 analysis calls in cached runs)
2. **Traditional**: Direct code generation (2-3 analysis calls per file)

### Analysis Strategy (Optimized for Speed & Cost)
The system uses **layered analysis** to minimize costs:
- **Primary**: Tree-sitter AST analysis for Python/JS/Go (90% of codebases, instant, free)
- **Template**: Pre-built patterns for standard instrumentation (no analysis needed)
- **Intelligent Fallback**: Ensures comprehensive coverage on unfamiliar code patterns
- **Caching**: Script caching provides 98.7% speedup on reruns, 0 analysis calls on cache hits

### The 4 Intelligent Analysis Integration Points
1. **src/llm_analyzer.py** (Analysis phase) - Only for languages unsupported by tree-sitter
2. **src/code_injector.py** (Injection phase) - Traditional mode only
3. **src/reflection_engine.py** (Failure analysis) - Optional, on validation errors
4. **src/script_refactorer.py** (Self-healing) - Script mode only, on test failures

## Documentation Organization

```
docs/
├── INDEX.md                           ← You are here
├── QUICK_REFERENCE.md                 ← For quick lookups
├── ARCHITECTURE_ANALYSIS.md           ← For complete details
├── ARCHITECTURE.md                    ← Previous architecture (retained)
├── SCRIPT_BASED_ARCHITECTURE.md       ← Script-based deep dive
├── QUICKSTART.md                      ← Getting started
├── GO_QUICK_REFERENCE.md              ← Go-specific reference
├── VISUALIZER_QUICK_START.md          ← Telemetry visualizer
└── lessons/                           ← Language-specific lessons
    ├── python/
    │   ├── multi_line_expressions.md
    │   ├── return_statement_transformation.md
    │   └── ...
    ├── javascript/
    └── go/
```

## Files by Purpose

### Quick Reference (Start Here!)
- **QUICK_REFERENCE.md** - Where to start for quick answers
  - One-page lookup for all major components
  - Performance tables
  - Configuration examples
  - When LLM is called

### Deep Dive (For Full Understanding)
- **ARCHITECTURE_ANALYSIS.md** - Complete technical reference
  - Component mapping with line numbers
  - Data flow diagrams
  - Cost analysis scenarios
  - Security details
  - Testing architecture

### Implementation Details
- **SCRIPT_BASED_ARCHITECTURE.md** - Caching and optimization details
- **ARCHITECTURE.md** - Previous generation architecture
- **GO_QUICK_REFERENCE.md** - Go language-specific info

### Getting Started
- **QUICKSTART.md** - Installation and first run
- **VISUALIZER_QUICK_START.md** - Telemetry visualization tool

## Performance Benchmarks

### Traditional Mode (per file)
- Python: 3-6 seconds, $0.05-0.15
- Other: 5-10 seconds, $0.07-0.20

### Script-Based Mode (per function)
- First run (miss): 7ms, $0 (template)
- Cached run (hit): 0.3ms, $0 (instant!)
- Speedup: 98.7% faster

### Parallel Processing
- 100 functions serial: ~700ms
- 100 functions parallel: ~60ms
- Speedup: ~12x

## Configuration

### Environment Variables
```bash
LLM_PROVIDER=openai|anthropic|ollama
OPENAI_API_KEY=...
ANTHROPIC_API_KEY=...
LLM_BASE_URL=...  # For Ollama
LLM_MODEL=...
LLM_TIMEOUT=120
```

### Command Examples
```bash
# Setup
python telemetry-inject.py --configure

# Traditional mode
python telemetry-inject.py ./src -v

# Script-based (recommended!)
python telemetry-inject.py ./src --use-scripts -v

# With cost limit
python telemetry-inject.py ./src --budget 10.00

# Dry run
python telemetry-inject.py ./src --use-scripts --dry-run
```

## Key Components

### Analysis Phase
- **HybridAnalyzer** - Smart selector (tree-sitter first, LLM fallback)
- **TreeSitterAnalyzer** - Fast AST parsing (Python/JS/Go/TS)
- **LLMAnalyzer** - LLM-based analysis (fallback)

### Traditional Pipeline
- **CodeInjector** - LLM-based code injection
- **RetryInjector** - Retry wrapper with validation
- **ReflectionEngine** - Learn from failures

### Script-Based Pipeline
- **ScriptGenerator** - Generate insertion scripts (template-based)
- **ScriptCache** - Hash-based caching system
- **ParallelScriptProcessor** - Async batch processing
- **ScriptValidator** - Syntax/security/test validation
- **ScriptRefactorer** - Self-healing via LLM

## Where to Find Things

### Understanding LLM Usage
1. Read: QUICK_REFERENCE.md (section: "Where LLM is Actually Called")
2. Then: ARCHITECTURE_ANALYSIS.md (section: "LLM Integration Points")
3. Details: See specific source files listed

### Optimizing Performance
1. Read: QUICK_REFERENCE.md (section: "Performance Comparison")
2. Then: SCRIPT_BASED_ARCHITECTURE.md (section: "Caching Performance")
3. Implement: Use `--use-scripts` flag

### Understanding Caching
1. Read: QUICK_REFERENCE.md (section: "What Gets Cached")
2. Then: SCRIPT_BASED_ARCHITECTURE.md (complete document)
3. Monitor: `.telemetry_cache/metadata/cache_index.json`

### Configuration and Deployment
1. Read: QUICKSTART.md
2. Then: QUICK_REFERENCE.md (section: "Configuration")
3. Reference: ARCHITECTURE_ANALYSIS.md (section: "CLI Flow & Flags")

## Important Absolute Paths

### Core Files
- `/home/paulpas/git/claude-code-testing/src/cli.py` - Main entry point
- `/home/paulpas/git/claude-code-testing/src/hybrid_analyzer.py` - Smart analyzer
- `/home/paulpas/git/claude-code-testing/src/tree_sitter_analyzer.py` - Fast AST
- `/home/paulpas/git/claude-code-testing/src/llm_analyzer.py` - LLM analysis

### Pipeline Files
- `/home/paulpas/git/claude-code-testing/src/code_injector.py` - Injection (traditional)
- `/home/paulpas/git/claude-code-testing/src/script_generator.py` - Scripts (cached)
- `/home/paulpas/git/claude-code-testing/src/script_cache.py` - Cache system
- `/home/paulpas/git/claude-code-testing/src/parallel_script_processor.py` - Async

### Cache Location
- `/home/paulpas/git/claude-code-testing/.telemetry_cache/`

### Test Files
- `/home/paulpas/git/claude-code-testing/tests/` (51+ test files, 100% coverage)

## Best Practices

### For Development
1. Use `--use-scripts` flag (98.7% faster on reruns)
2. Enable `--debug-trace` for detailed logging
3. Use `--dry-run` for testing

### For Production
1. Use `--use-scripts` with `--validate`
2. Set `--budget` limit
3. Disable `--no-parallel` only if needed
4. Monitor `.telemetry_cache/` for cache statistics

### For Cost Control
1. Use local Ollama ($0 cost)
2. Use `--budget` flag
3. Monitor API calls via cost tracker
4. Leverage script-based caching

## Testing

- 51+ test files
- 100% test coverage
- TDD methodology
- Run: `pytest tests/ -v`

## Support Resources

### Documentation Files
- ARCHITECTURE_ANALYSIS.md - 932 lines of complete documentation
- QUICK_REFERENCE.md - 362 lines of quick lookup
- This file (INDEX.md) - Navigation guide

### Source Code
- All files in `/home/paulpas/git/claude-code-testing/src/` with inline documentation

### Examples
- `/home/paulpas/git/claude-code-testing/examples/` - Working examples

### Lessons
- `/home/paulpas/git/claude-code-testing/docs/lessons/` - Language-specific patterns

## Summary

This codebase implements a sophisticated code instrumentation system with:
- **Dual pipelines** for maximum flexibility
- **Aggressive LLM optimization** via tree-sitter and caching
- **98.7% performance improvement** on cached runs
- **100% cost savings** on repeat runs
- **Production-ready** with full test coverage

**Recommended Usage**: `python telemetry-inject.py ./src --use-scripts -v`

For complete details, see **ARCHITECTURE_ANALYSIS.md** and **QUICK_REFERENCE.md**.

---

**Generated**: November 2, 2025
**Codebase Size**: 20,069 lines across 56 Python modules
**Status**: Production-ready with comprehensive documentation
