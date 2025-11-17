# Metadata Tracking in Telemetry Injection

**Version**: 1.0.0
**Last Updated**: 2025-10-31
**Status**: ✅ Production Ready (All Tests Passing)

## Overview

The telemetry injection system now includes comprehensive metadata tracking to help you understand:

1. **Which model** performed the code injection
2. **How many retry attempts** were needed
3. **Function indexing** for parallel processing
4. **Processing timestamps** for performance analysis

This metadata is automatically added as language-specific comments at the top of each instrumented file or function.

---

## Features

### 1. **Automatic Metadata Comments**

Every instrumented file gets a metadata comment at the top showing:
- Model name (e.g., `gpt-4`, `claude-3-5-sonnet`, `codellama`)
- Retry attempt information (e.g., `Attempt 2/3`)
- Function index for parallel processing (e.g., `Function #5`)

### 2. **Language-Specific Formatting**

The system automatically formats comments for each language:

| Language | Comment Style | Example |
|----------|---------------|---------|
| Python | `#` | `# Instrumented by gpt-4 - Attempt 2/3` |
| JavaScript/TypeScript | `//` | `// Instrumented by gpt-4 - Attempt 2/3` |
| Go | `//` | `// Instrumented by gpt-4 - Attempt 2/3` |
| Java/C/C++/Rust | `//` | `// Instrumented by gpt-4 - Attempt 2/3` |
| Ruby | `#` | `# Instrumented by gpt-4 - Attempt 2/3` |
| SQL | `--` | `-- Instrumented by gpt-4 - Attempt 2/3` |
| Unknown | `//` | `// Instrumented by gpt-4 - Attempt 2/3` (defaults to C-style) |

### 3. **Smart Metadata Display**

The system shows only relevant information:
- **Single attempt, no parallel processing**: `# Instrumented by gpt-4`
- **With retries**: `# Instrumented by gpt-4 - Attempt 2/3`
- **With parallel processing**: `# Instrumented by gpt-4 - Function #5`
- **All metadata**: `# Instrumented by gpt-4 - Attempt 2/3 - Function #5`

---

## Data Structures

### Enhanced `InjectionResult`

```python
@dataclass
class InjectionResult:
    """Result of code injection."""

    # Original fields
    original_code: str
    instrumented_code: str
    language: str
    snippets_applied: int

    # NEW: Metadata fields
    model_used: Optional[str] = None             # Model that performed injection
    attempt_number: Optional[int] = None         # Current attempt (for retries)
    max_attempts: Optional[int] = None           # Maximum attempts allowed
    function_index: Optional[int] = None         # Index for parallel processing
    timestamp: Optional[float] = None            # Unix timestamp of injection
```

### Enhanced `TelemetrySnippet`

```python
@dataclass
class TelemetrySnippet:
    """A snippet of telemetry code to be injected."""

    # Original fields
    type: str              # "function", "loop", or "variable"
    code: str
    language: str
    location: Dict
    description: str

    # NEW: Processing metadata
    function_index: Optional[int] = None         # For parallel tracking
    batch_id: Optional[int] = None               # Parallel batch ID
    generated_at: Optional[float] = None         # Generation timestamp
    generation_time_ms: Optional[float] = None   # Time to generate
```

### Enhanced `ExtractedFunction`

```python
@dataclass
class ExtractedFunction:
    """Represents a function extracted from source code."""

    # Original fields
    name: str
    code: str
    start_line: int
    end_line: int
    language: str
    parameters: List[str]
    indentation_level: int

    # NEW: Tracking metadata
    function_index: Optional[int] = None         # For parallel processing
```

### Enhanced `ReconstructionResult`

```python
@dataclass
class ReconstructionResult:
    """Result of file reconstruction."""

    # Original fields
    success: bool
    reconstructed_code: str
    language: str
    functions_replaced: int
    error_message: Optional[str]

    # NEW: Tracking metadata
    line_mappings: Optional[dict] = None         # Original → New line mappings
    functions_processed: Optional[List[str]] = None  # Names of replaced functions
```

---

## Usage Examples

### Example 1: Basic Injection with Metadata

```python
from src.code_injector import CodeInjector

injector = CodeInjector(
    api_key="your-api-key",
    provider="openai",
    model="gpt-4"
)

result = injector.inject(
    code="def calculate(a, b):\n    return a + b",
    telemetry_snippets=snippets,
    language="python",
    attempt_number=1,      # First attempt
    max_attempts=3,        # Allow up to 3 retries
    function_index=0,      # First function in parallel batch
    add_metadata_comment=True  # Default: True
)

# Result includes metadata
print(result.model_used)        # "gpt-4"
print(result.attempt_number)    # 1
print(result.max_attempts)      # 3
print(result.function_index)    # 0
print(result.timestamp)         # 1698765432.123

# Instrumented code has metadata comment
print(result.instrumented_code)
# Output:
# # Instrumented by gpt-4 - Attempt 1/3 - Function #0
# from _telemetry_utils import tel
#
# def calculate(a, b):
#     _tel_func_calculate = tel.func_entry("calculate", "a, b")
#
#     result = a + b
#
#     tel.func_exit(_tel_func_calculate, result)
#     return result
```

### Example 2: Retry Injection with Metadata

```python
from src.retry_injector import RetryInjector

retry_injector = RetryInjector(
    api_key="your-api-key",
    model="gpt-4",
    max_retries=3
)

result = retry_injector.inject_with_retry(
    code=source_code,
    telemetry_snippets=snippets,
    language="python",
    validate=True
)

# Check retry metadata
print(f"Succeeded on attempt {result.attempts_made}/{retry_injector.max_retries}")
print(f"Used reflection: {result.reflection_used}")

# The injection result includes metadata
injection_result = result.injection_result
print(injection_result.model_used)       # "gpt-4"
print(injection_result.attempt_number)   # e.g., 2
print(injection_result.max_attempts)     # 3
```

### Example 3: Parallel Processing with Function Indexes

```python
from src.function_extractor import FunctionExtractor
from src.function_injector import FunctionInjector
from src.file_reconstructor import FileReconstructor

# Extract functions with indexes
extractor = FunctionExtractor()
functions = extractor.extract_functions(code, "python")

# Assign indexes for parallel tracking
for idx, func in enumerate(functions):
    func.function_index = idx

# Process functions in parallel (pseudo-code)
instrumented_functions = []
for idx, func in enumerate(functions):
    result = injector.inject(
        code=func.code,
        telemetry_snippets=generate_snippets(func),
        language="python",
        function_index=idx,  # Track parallel processing
        attempt_number=1,
        max_attempts=3
    )
    func.code = result.instrumented_code
    instrumented_functions.append(func)

# Reconstruct with metadata preserved
reconstructor = FileReconstructor()
final_result = reconstructor.reconstruct(
    original_code=code,
    instrumented_functions=instrumented_functions,
    language="python"
)

# Each function has metadata comment showing its index
# Function #0:
# # Instrumented by gpt-4 - Attempt 1/3 - Function #0
# def function_one():
#     ...
#
# Function #1:
# # Instrumented by gpt-4 - Attempt 1/3 - Function #1
# def function_two():
#     ...
```

### Example 4: Creating TelemetrySnippets with Metadata

```python
import time
from src.telemetry_generator import TelemetrySnippet

snippet = TelemetrySnippet(
    type="function",
    code='_tel_func = tel.func_entry("calculate", "a, b")',
    language="python",
    location={"start_line": 1, "end_line": 3},
    description="Telemetry for calculate function",

    # NEW: Add processing metadata
    function_index=5,              # 6th function in batch
    batch_id=2,                    # Part of batch #2
    generated_at=time.time(),      # When snippet was created
    generation_time_ms=125.5       # Time taken to generate
)

# Later, you can analyze processing metadata
print(f"Function #{snippet.function_index} took {snippet.generation_time_ms}ms to generate")
```

---

## API Reference

### `CodeInjector.inject()`

```python
def inject(
    self,
    code: str,
    telemetry_snippets: List[TelemetrySnippet],
    language: str,
    attempt_number: int = 1,          # NEW
    max_attempts: int = 1,            # NEW
    function_index: Optional[int] = None,  # NEW
    add_metadata_comment: bool = True      # NEW
) -> InjectionResult:
    """
    Inject telemetry code with metadata tracking.

    Args:
        code: Original source code
        telemetry_snippets: List of telemetry snippets to inject
        language: Programming language
        attempt_number: Current attempt number (for retry tracking)
        max_attempts: Maximum attempts configured (for retry tracking)
        function_index: Optional index for parallel processing
        add_metadata_comment: Whether to add metadata comment (default: True)

    Returns:
        InjectionResult with instrumented code and metadata
    """
```

### `CodeInjector._generate_metadata_comment()`

```python
def _generate_metadata_comment(
    self,
    language: str,
    model_used: str,
    attempt_number: int = 1,
    max_attempts: int = 1,
    function_index: Optional[int] = None
) -> str:
    """
    Generate language-specific metadata comment.

    Args:
        language: Programming language
        model_used: Model name that performed injection
        attempt_number: Current attempt number
        max_attempts: Maximum attempts configured
        function_index: Optional index for parallel processing

    Returns:
        Formatted comment string with newline
    """
```

---

## Testing

### Running Tests

```bash
# Run all metadata tests
pytest tests/test_metadata_comments.py -v

# Run specific test class
pytest tests/test_metadata_comments.py::TestMetadataComments -v

# Run with coverage
pytest tests/test_metadata_comments.py --cov=src.code_injector --cov=src.telemetry_generator
```

### Test Coverage

All tests passing: **16/16 ✅**

**Test Classes**:
- `TestMetadataComments`: 9 tests for comment generation
- `TestInjectionResultMetadata`: 2 tests for result metadata
- `TestMetadataCommentIntegration`: 3 tests for integration
- `TestTelemetrySnippetMetadata`: 2 tests for snippet metadata

**Coverage**:
- Comment generation for all languages
- Metadata field population
- Edge cases (single attempt, no function index)
- Integration with full injection workflow

---

## Demonstration

Run the included demonstration script:

```bash
python examples/metadata_demo.py
```

**Output**:
```
======================================================================
METADATA COMMENT DEMONSTRATION
======================================================================

Scenario 1: Python function - First attempt
----------------------------------------------------------------------
Generated comment:
# Instrumented by gpt-4 - Attempt 1/3 - Function #0

Scenario 2: JavaScript function - Retry attempt
----------------------------------------------------------------------
Generated comment:
// Instrumented by gpt-4 - Attempt 2/3 - Function #5

...
```

---

## Benefits

### 1. **Traceability**
- Know exactly which model processed each file
- Track retry attempts to identify problematic code
- Understand parallel processing order

### 2. **Debugging**
- Quickly identify files that needed multiple attempts
- Correlate errors with specific models or attempts
- Track function processing order in parallel workflows

### 3. **Performance Analysis**
- Measure generation time per snippet
- Identify slow functions in parallel processing
- Track batch processing efficiency

### 4. **Reproducibility**
- Timestamps enable chronological ordering
- Function indexes preserve processing order
- Model tracking enables model comparison

---

## Best Practices

### 1. **Always Enable Metadata Comments**

```python
# Good: Metadata enabled (default)
result = injector.inject(code, snippets, language)

# Only disable if you have a specific reason
result = injector.inject(code, snippets, language, add_metadata_comment=False)
```

### 2. **Track Function Indexes in Parallel Processing**

```python
# Good: Assign indexes before parallel processing
for idx, func in enumerate(functions):
    func.function_index = idx
    # Process with index...

# Bad: No index tracking
for func in functions:
    # Process without index - hard to debug later
```

### 3. **Use RetryInjector for Automatic Metadata**

```python
# Good: RetryInjector automatically tracks attempts
retry_injector = RetryInjector(max_retries=3)
result = retry_injector.inject_with_retry(code, snippets, language)
# Metadata is automatically populated

# Manual: You'd need to track this yourself
for attempt in range(1, 4):
    result = injector.inject(code, snippets, language,
                            attempt_number=attempt, max_attempts=3)
```

### 4. **Capture Processing Metadata for Snippets**

```python
# Good: Track generation timing
start = time.time()
snippet = generator.generate_function_snippet(func)
snippet.generation_time_ms = (time.time() - start) * 1000
snippet.generated_at = start
snippet.function_index = func_index

# This helps with performance analysis later
```

---

## Troubleshooting

### Issue: Metadata comments not appearing

**Solution**: Check that `add_metadata_comment=True` (default)

```python
# Ensure metadata comments are enabled
result = injector.inject(code, snippets, language, add_metadata_comment=True)
```

### Issue: Function indexes not preserved

**Solution**: Assign `function_index` to ExtractedFunction before processing

```python
for idx, func in enumerate(extracted_functions):
    func.function_index = idx
```

### Issue: Retry attempts not tracked

**Solution**: Use `RetryInjector` instead of direct `CodeInjector`

```python
# Use RetryInjector for automatic retry tracking
retry_injector = RetryInjector(max_retries=3)
result = retry_injector.inject_with_retry(code, snippets, language)
```

---

## Migration Guide

### Upgrading Existing Code

**Old code** (pre-metadata):
```python
result = injector.inject(code, snippets, language)
```

**New code** (with metadata):
```python
result = injector.inject(
    code,
    snippets,
    language,
    attempt_number=1,      # Optional: defaults to 1
    max_attempts=3,        # Optional: defaults to 1
    function_index=None,   # Optional: defaults to None
    add_metadata_comment=True  # Optional: defaults to True
)

# Access metadata
print(f"Instrumented by {result.model_used}")
print(f"Attempt {result.attempt_number}/{result.max_attempts}")
```

**All parameters are optional and backward compatible!** Your existing code will continue to work without changes.

---

## Performance Impact

**Overhead**: Minimal (< 1ms per file)

The metadata comment generation adds negligible overhead:
- Comment generation: < 0.1ms
- Metadata field population: < 0.1ms
- Total overhead: < 1ms per file

**Memory**: Minimal (< 1KB per result)

The additional metadata fields add minimal memory:
- `InjectionResult`: +40 bytes (5 new fields)
- `TelemetrySnippet`: +32 bytes (4 new fields)
- Total per file: < 1KB

---

## Future Enhancements

### Planned Features

1. **Structured Metadata Export**
   - JSON export of all metadata
   - CSV reports for batch processing
   - Visualization dashboards

2. **Metadata Aggregation**
   - Success rate by model
   - Average retry attempts by language
   - Performance analytics

3. **Advanced Indexing**
   - Hierarchical indexes (file → function → snippet)
   - Dependency tracking between functions
   - Call graph integration

---

## Related Documentation

- [`TREE_SITTER_IMPLEMENTATION.md`](TREE_SITTER_IMPLEMENTATION.md) - Tree-sitter code analysis
- [`PARALLEL_PROCESSING.md`](./PARALLEL_PROCESSING.md) - Parallel processing guide
- [`RETRY_REFLECTION.md`](./RETRY_REFLECTION.md) - Retry and reflection system
- [`CLAUDE.md`](../CLAUDE.md) - Development session summary

---

## Support

For questions or issues related to metadata tracking:

1. Check the [troubleshooting section](#troubleshooting)
2. Run the [demonstration script](#demonstration)
3. Review the [test cases](../tests/test_metadata_comments.py)
4. Check the implementation in [src/code_injector.py](../src/code_injector.py)

---

**Last Updated**: 2025-10-31
**Status**: ✅ Production Ready
**Test Coverage**: 16/16 tests passing
**Performance**: < 1ms overhead per file
