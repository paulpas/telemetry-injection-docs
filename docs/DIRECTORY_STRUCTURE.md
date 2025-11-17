# Source Directory Structure

**Last Updated**: 2025-11-03
**Purpose**: Document the logical organization of the src/ directory
**Status**: Production-ready ✅

---

## 🎯 Overview

The src/ directory is organized into **logical functional groups** to make navigation intuitive. If you want to adjust something, you'll know exactly where to go.

---

## 📁 Directory Structure

```
src/
├── analysis/           # Code analysis (LLM, tree-sitter, AST)
├── generation/         # Telemetry code generation
├── injection/          # Code injection and modification
├── scripting/          # Script-based injection system (caching)
├── llm/                # LLM clients and management
├── processing/         # File and batch processing
├── validation/         # Validation and quality assurance
├── learning/           # Learning and optimization
├── build/              # Build system integration
├── config/             # Configuration and setup
├── utils/              # General utilities
├── visualization/      # Telemetry visualization
└── cli.py              # Main CLI interface
```

---

## 📂 Detailed Breakdown

### 1. analysis/ - Code Analysis

**Purpose**: Analyze code structure, extract functions, track scope

**Files**:
- `llm_analyzer.py` - LLM-based code analysis (for unsupported languages)
- `tree_sitter_analyzer.py` - Tree-sitter AST parsing (Python/JS/Go)
- `hybrid_analyzer.py` - Smart orchestrator (tree-sitter + LLM fallback)
- `function_extractor.py` - Extract functions from code
- `scope_tracker.py` - Scope-aware variable tracking (95%+ coverage)
- `llm_build_guide.py` - LLM build guidance

**When to modify**:
- ✅ Adding new language support to tree-sitter
- ✅ Improving AST parsing logic
- ✅ Enhancing function detection
- ✅ Fixing scope tracking bugs
- ✅ Adding new analysis features

**Key Classes**:
- `TreeSitterAnalyzer` - Fast AST-based analysis
- `LLMAnalyzer` - LLM-based analysis fallback
- `HybridAnalyzer` - Smart orchestrator
- `FunctionExtractor` - Extract function metadata
- `ScopeTracker` - Track variable scope

---

### 2. generation/ - Telemetry Code Generation

**Purpose**: Generate telemetry snippets for instrumentation

**Files**:
- `telemetry_generator.py` - Generate telemetry code snippets
- `templates/` - Language-specific telemetry utility templates
  - `telemetry_utils_template_python.py` - Python telemetry utilities
  - `telemetry_utils_template_javascript.js` - JavaScript telemetry utilities
  - `telemetry_utils_template_golang.go` - Go telemetry utilities

**When to modify**:
- ✅ Adding new telemetry types (func_entry, var_change, etc.)
- ✅ Enhancing telemetry generation logic
- ✅ Adding new language templates
- ✅ Improving telemetry snippet quality
- ✅ Customizing telemetry output format

**Key Classes**:
- `TelemetryGenerator` - Generate telemetry snippets
- `TelemetrySnippet` - Individual telemetry call

---

### 3. injection/ - Code Injection & Modification

**Purpose**: Inject telemetry into source code

**Files**:
- `code_injector.py` - Traditional LLM-based injection
- `file_reconstructor.py` - Rebuild files with instrumentation
- `retry_injector.py` - Retry logic for failed injections
- `function_injector.py` - Function-level injection

**When to modify**:
- ✅ Adjusting LLM injection prompts
- ✅ Improving file reconstruction logic
- ✅ Enhancing retry strategies
- ✅ Fixing code insertion bugs
- ✅ Adding new injection methods

**Key Classes**:
- `CodeInjector` - LLM-based code injection
- `FileReconstructor` - Rebuild instrumented files
- `RetryInjector` - Retry failed injections
- `InjectionResult` - Injection result metadata

---

### 4. scripting/ - Script-Based Injection System

**Purpose**: Generate, cache, and execute insertion scripts (98.7% faster!)

**Files**:
- `script_generator.py` - Generate standalone insertion scripts
- `script_cache.py` - Hash-based script caching
- `script_sandbox.py` - Isolated script execution
- `script_validator.py` - Script validation (syntax, security, tests)
- `script_refactorer.py` - Self-healing refactoring
- `test_generator.py` - Generate pytest tests (TDD)
- `sandbox_executor.py` - Sandbox execution utilities
- `parallel_script_processor.py` - Parallel script processing (12 workers)

**When to modify**:
- ✅ Improving script generation templates
- ✅ Enhancing cache hit rate
- ✅ Adding new validation checks
- ✅ Improving self-healing logic
- ✅ Optimizing parallel processing

**Key Classes**:
- `ScriptGenerator` - Generate insertion scripts
- `ScriptCache` - Cache scripts by hash
- `ScriptSandbox` - Execute scripts safely
- `ScriptValidator` - Validate scripts
- `ScriptRefactorer` - Self-heal failed scripts
- `ParallelScriptProcessor` - Process scripts in parallel

---

### 5. llm/ - LLM Clients & Management

**Purpose**: Manage LLM providers, models, GPU scheduling, and prompt formatting

**Files**:
- `model_manager.py` - Model selection and management
- `ollama_model_pool.py` - Ollama multi-model rotation (3+ models)
- `token_detector.py` - Token detection and estimation
- `gpu_vram_scheduler.py` - GPU VRAM scheduling
- `poml_processor.py` - POML (Prompt Oriented Markup Language) processing

**When to modify**:
- ✅ Adding new LLM providers
- ✅ Improving model selection logic
- ✅ Enhancing GPU scheduling
- ✅ Optimizing token usage
- ✅ Adding new model rotation strategies
- ✅ Converting prompts between API formats (OpenAI, Anthropic, Ollama)

**Key Classes**:
- `ModelManager` - Manage model selection
- `OllamaModelPool` - Multi-model rotation
- `TokenDetector` - Estimate token usage
- `POMLProcessor` - Convert prompts between API formats

---

### 6. processing/ - File & Batch Processing

**Purpose**: Scan files, process in batches, parallelize work

**Files**:
- `scanner.py` - File discovery and scanning
- `file_by_file_processor.py` - Sequential file processing
- `parallel_processor.py` - Parallel batch processing
- `parallel_cli.py` - Parallel CLI interface

**When to modify**:
- ✅ Improving file discovery
- ✅ Enhancing batch processing logic
- ✅ Optimizing parallelization
- ✅ Adding new file filters
- ✅ Improving progress reporting

**Key Classes**:
- `CodeScanner` - Discover files
- `FileByFileProcessor` - Sequential processing
- `ParallelProcessor` - Parallel processing

---

### 7. validation/ - Validation & Quality Assurance

**Purpose**: Validate code, run linting, reflect on errors

**Files**:
- `validation_engine.py` - Code validation (syntax, runtime)
- `linting_engine.py` - Linting integration (pylint, eslint)
- `reflection_engine.py` - Error reflection and analysis
- `command_validator.py` - Command validation (security)

**When to modify**:
- ✅ Adding new validation checks
- ✅ Improving linting integration
- ✅ Enhancing error reflection
- ✅ Adding new linters
- ✅ Improving security validation

**Key Classes**:
- `ValidationEngine` - Validate instrumented code
- `LintingEngine` - Run linters
- `ReflectionEngine` - Analyze failures
- `ValidationResult` - Validation results

---

### 8. learning/ - Learning & Optimization

**Purpose**: Learn from past injections, consolidate knowledge

**Files**:
- `learning_engine.py` - Learn from past injections
- `learning_consolidator.py` - Consolidate learnings
- `consolidate_learnings.py` - Consolidation utilities

**When to modify**:
- ✅ Improving learning algorithms
- ✅ Enhancing knowledge consolidation
- ✅ Adding new learning strategies
- ✅ Optimizing lesson storage
- ✅ Improving pattern recognition

**Key Classes**:
- `LearningEngine` - Learn from history
- `LearningConsolidator` - Consolidate lessons

---

### 9. build/ - Build System Integration

**Purpose**: Integrate with build systems (Go modules, etc.)

**Files**:
- `build_instruction_cache.py` - Cache build instructions
- `go_environment_manager.py` - Go build environment

**When to modify**:
- ✅ Adding new build system support
- ✅ Improving build caching
- ✅ Enhancing Go module handling
- ✅ Adding new environment managers
- ✅ Optimizing build performance

**Key Classes**:
- `BuildInstructionCache` - Cache build instructions
- `GoEnvironmentManager` - Manage Go builds

---

### 10. config/ - Configuration & Setup

**Purpose**: Configure the tool, check API connectivity

**Files**:
- `config_menu.py` - Interactive configuration menu
- `api_checker.py` - API connectivity checks

**When to modify**:
- ✅ Adding new configuration options
- ✅ Improving configuration UI
- ✅ Adding new API checks
- ✅ Enhancing setup experience
- ✅ Adding validation for config values

**Key Classes**:
- `ConfigMenu` - Interactive setup
- `APIChecker` - Check API connectivity

---

### 11. utils/ - General Utilities

**Purpose**: Shared utilities used across the system

**Files**:
- `cost_tracker.py` - Cost tracking and budgets
- `debug_trace_logger.py` - Debug logging (JSONL)
- `verbose_logger.py` - Verbose output formatting
- `json_utils.py` - JSON utilities
- `telemetry_utils_writer.py` - Write telemetry utility files
- `telemetry_lib_manager.py` - Manage telemetry libraries
- `cloud_metadata_collector.py` - Cloud metadata collection
- `retry_generator.py` - Retry utilities

**When to modify**:
- ✅ Adding new utility functions
- ✅ Improving logging
- ✅ Enhancing cost tracking
- ✅ Adding new retry strategies
- ✅ Improving JSON handling

**Key Classes**:
- `CostTracker` - Track costs and budgets
- `DebugTraceLogger` - JSONL logging
- `VerboseLogger` - Colored output

---

### 12. visualization/ - Telemetry Visualization

**Purpose**: Visualize telemetry data and execution traces

**Files**:
- `telemetry_visualizer.py` - Visualize telemetry data

**When to modify**:
- ✅ Adding new visualization types
- ✅ Improving visualization quality
- ✅ Adding new output formats
- ✅ Improving data rendering
- ✅ Creating trace diagrams and charts

**Key Classes**:
- `TelemetryVisualizer` - Visualize execution traces and telemetry data

---

### cli.py - Main CLI Interface

**Location**: `src/cli.py` (stays at root)

**Purpose**: Main entry point for the CLI

**When to modify**:
- ✅ Adding new CLI commands
- ✅ Improving argument parsing
- ✅ Adding new flags
- ✅ Enhancing user experience
- ✅ Improving error messages

---

## 🗺️ Navigation Guide

### "I want to adjust LLM injection logic"

→ Go to **`src/injection/`**
- Modify `code_injector.py` for prompt changes
- Modify `retry_injector.py` for retry logic
- Modify `file_reconstructor.py` for reconstruction

### "I want to adjust tree-sitter parsing"

→ Go to **`src/analysis/`**
- Modify `tree_sitter_analyzer.py` for AST parsing
- Modify `hybrid_analyzer.py` for orchestration
- Modify `function_extractor.py` for function extraction

### "I want to add a new language"

→ Touch **multiple directories**:
1. **`src/analysis/tree_sitter_analyzer.py`** - Add tree-sitter queries
2. **`src/generation/templates/`** - Add telemetry template
3. **`src/injection/file_reconstructor.py`** - Add reconstruction logic
4. **`src/analysis/function_extractor.py`** - Add function extraction

### "I want to improve script caching"

→ Go to **`src/scripting/`**
- Modify `script_cache.py` for caching logic
- Modify `script_generator.py` for script generation
- Modify `script_validator.py` for validation

### "I want to add new telemetry types"

→ Go to **`src/generation/`**
- Modify `telemetry_generator.py` for generation logic
- Modify `templates/*.py/*.js/*.go` for utility methods

### "I want to improve cost tracking"

→ Go to **`src/utils/cost_tracker.py`**

### "I want to add new validation checks"

→ Go to **`src/validation/validation_engine.py`**

---

## 📚 Import Examples

### Before Reorganization

```python
from src.llm_analyzer import LLMAnalyzer
from src.script_cache import ScriptCache
from src.cost_tracker import CostTracker
```

### After Reorganization

```python
from src.analysis.llm_analyzer import LLMAnalyzer
from src.scripting.script_cache import ScriptCache
from src.utils.cost_tracker import CostTracker
```

---

## 🔄 Migration Notes

**Date**: 2025-11-03
**Changes**: 204 imports updated across 67 files
**Automated**: Yes, using `update_imports.py` script
**Verified**: CLI and all imports working ✅

---

## 🎓 Benefits

### For Developers

- ✅ **Intuitive Navigation**: Know where to go by functionality
- ✅ **Clear Organization**: Logical groupings
- ✅ **Easy Onboarding**: New developers understand structure quickly
- ✅ **Reduced Confusion**: No more "where is X?" questions

### For Maintenance

- ✅ **Easier Refactoring**: Related code grouped together
- ✅ **Better Testing**: Test by functional area
- ✅ **Clear Dependencies**: See what depends on what
- ✅ **Scalability**: Easy to add new features

### For Documentation

- ✅ **Self-Documenting**: Directory names explain purpose
- ✅ **Clear References**: "See src/analysis/ for analysis code"
- ✅ **Better Diagrams**: Can visualize by directory
- ✅ **Easier to Explain**: "We have 12 functional areas..."

---

## 🛠️ Maintenance

### Adding a New File

1. Determine which functional group it belongs to
2. Place it in the appropriate directory
3. Update imports if it's used elsewhere
4. Add to documentation if it's a key file

### Moving a File

1. Use `git mv src/old/path.py src/new/path.py` to preserve history
2. Run `update_imports.py` to update all imports
3. Test that everything still works
4. Update documentation

### Adding a New Directory

1. Create directory: `mkdir src/new_category`
2. Add `__init__.py`: `touch src/new_category/__init__.py`
3. Move files: `git mv src/file.py src/new_category/`
4. Update imports: Run `update_imports.py`
5. Document in this file

---

## 📝 Related Documentation

- **ARCHITECTURE_REFACTORED.md** - Complete architecture
- **REORGANIZATION_PLAN.md** - Original reorganization plan
- **README.md** - High-level overview
- **QUICKSTART.md** - Getting started guide

---

**Last Updated**: 2025-11-03
**Maintained By**: Development team
**Status**: Production-ready ✅
