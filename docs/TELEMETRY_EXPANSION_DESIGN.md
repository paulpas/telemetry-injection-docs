# Telemetry Expansion Design Document

**Version**: 1.0
**Date**: 2025-11-02
**Status**: Design Phase
**Author**: Claude Code Analysis

---

## Executive Summary

This document outlines the design for expanding telemetry coverage beyond the current functions, loops, and variables to include:
1. **Conditionals** (if/else/elif/switch/case)
2. **Exception Handling** (try/catch/finally)
3. **Arrays/Collections** (creation, modification, access)
4. **Return Value Tracking** (capture return expressions)
5. **Method/Function Calls** (important API calls)

**Goal**: Provide comprehensive code execution visibility without performance degradation.

**Estimated Impact**: 2,500+ lines of new code across 25+ files

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Construct Specifications](#construct-specifications)
3. [Data Schema](#data-schema)
4. [Implementation Plan](#implementation-plan)
5. [Performance Analysis](#performance-analysis)
6. [Testing Strategy](#testing-strategy)
7. [Examples](#examples)

---

## Architecture Overview

### Current State (Phase 1 Complete)

```
User runs: python telemetry-inject.py ./src (script-based is DEFAULT)
    │
    ▼
┌──────────────────────────────────────────────────────────────────┐
│ 1. SCAN: Find code files                                         │
└──────────────────────────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────────────────────────┐
│ 2. EXTRACT: Parse functions (FunctionExtractor)                  │
└──────────────────────────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────────────────────────┐
│ 3. ANALYZE: HybridAnalyzer                                       │
│    ├─> TreeSitterAnalyzer (Python/JS/Go) - Detects:             │
│    │   • Functions ✅                                             │
│    │   • Loops (for/while) ✅                                     │
│    │   • Variables ✅                                             │
│    │   • Conditionals ❌ (TO ADD)                                │
│    │   • Exceptions ❌ (TO ADD)                                  │
│    │   • Arrays ❌ (TO ADD)                                      │
│    │   • Returns ❌ (TO ADD)                                     │
│    │   • Calls ❌ (TO ADD)                                       │
│    └─> LLMAnalyzer (fallback) - Updated prompt, ready for new    │
└──────────────────────────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────────────────────────┐
│ 4. GENERATE: TelemetryGenerator                                  │
│    Creates snippets for all detected constructs                   │
└──────────────────────────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────────────────────────┐
│ 5. INJECT: ScriptGenerator + Cache + Execute                     │
└──────────────────────────────────────────────────────────────────┘
```

### Target State (After Phase 2)

All construct types will be detected and instrumented at every level:
- **Analysis**: Tree-sitter queries for Python/JS/Go, LLM fallback for others
- **Generation**: Template-based telemetry snippet creation
- **Injection**: Script-based insertion with proper placement
- **Utilities**: New methods in `_telemetry_utils` for each construct

---

## Construct Specifications

### 1. Conditionals (if/else/elif/switch/case)

**Purpose**: Track which branches are taken and with what conditions

**Data Schema**:
```python
{
    "type": "if" | "else" | "elif" | "switch" | "case",
    "start_line": int,
    "end_line": int,
    "condition": str,  # e.g., "x > 0", "age >= 18"
    "branch_id": str,  # Unique ID for this branch
}
```

**Telemetry Methods**:
```python
# Python
tel.cond_entry(branch_id: str, condition: str, result: bool) -> str
tel.cond_exit(correlation_id: str)

# JavaScript
tel.condEntry(branchId, condition, result)
tel.condExit(correlationId)

# Go
Tel.CondEntry(branchId string, condition string, result bool) string
Tel.CondExit(correlationId string)
```

**Output Format**:
```json
{
    "event": "conditional",
    "branch_id": "if_line_42",
    "condition": "x > 0",
    "result": true,
    "timestamp_ns": 1234567890123456789,
    "correlation_id": "cond_abc123"
}
```

**Tree-Sitter Queries**:

*Python*:
```scheme
(if_statement
    condition: (_) @cond.expr
    consequence: (block) @cond.then
    alternative: (elif_clause)? @cond.elif
    alternative: (else_clause)? @cond.else
) @cond.if
```

*JavaScript*:
```scheme
(if_statement
    condition: (parenthesized_expression) @cond.expr
    consequence: (_) @cond.then
    alternative: (_)? @cond.else
) @cond.if

(switch_statement
    value: (_) @switch.expr
    body: (switch_body) @switch.body
) @switch
```

*Go*:
```scheme
(if_statement
    condition: (_)? @cond.expr
    consequence: (block) @cond.then
    alternative: (_)? @cond.else
) @cond.if

(expression_switch_statement
    value: (_)? @switch.expr
) @switch
```

**Instrumentation Strategy**:
```python
# BEFORE:
if x > 0:
    result = x * 2
else:
    result = 0

# AFTER:
_cond_1 = tel.cond_entry("if_line_5", "x > 0", x > 0)
if x > 0:
    result = x * 2
    tel.cond_exit(_cond_1)
else:
    _cond_2 = tel.cond_entry("else_line_7", "else", True)
    result = 0
    tel.cond_exit(_cond_2)
```

---

### 2. Exception Handling (try/catch/finally)

**Purpose**: Track exception flow, what gets caught, recovery paths

**Data Schema**:
```python
{
    "type": "try" | "catch" | "finally",
    "start_line": int,
    "end_line": int,
    "exception_types": List[str],  # e.g., ["ValueError", "TypeError"]
}
```

**Telemetry Methods**:
```python
# Python
tel.exception_entry(block_type: str) -> str
tel.exception_caught(correlation_id: str, exc_type: str, exc_msg: str)
tel.exception_exit(correlation_id: str, success: bool)

# JavaScript
tel.exceptionEntry(blockType)
tel.exceptionCaught(correlationId, excType, excMsg)
tel.exceptionExit(correlationId, success)

# Go
Tel.ExceptionEntry(blockType string) string
Tel.ExceptionCaught(correlationId string, excType string, excMsg string)
Tel.ExceptionExit(correlationId string, success bool)
```

**Output Format**:
```json
{
    "event": "exception",
    "block_type": "try",
    "exception_type": "ValueError",
    "exception_message": "invalid literal for int()",
    "caught": true,
    "timestamp_ns": 1234567890123456789,
    "correlation_id": "exc_xyz789"
}
```

**Tree-Sitter Queries**:

*Python*:
```scheme
(try_statement
    body: (block) @try.body
    (except_clause
        type: (_)? @except.type
        body: (block) @except.body
    )* @except
    (finally_clause
        body: (block) @finally.body
    )? @finally
) @try
```

*JavaScript*:
```scheme
(try_statement
    body: (statement_block) @try.body
    handler: (catch_clause
        parameter: (_)? @catch.param
        body: (statement_block) @catch.body
    )? @catch
    finalizer: (finally_clause
        body: (statement_block) @finally.body
    )? @finally
) @try
```

*Go*:
```scheme
; Go uses defer/panic/recover instead of try/catch
; Will track defer statements
(defer_statement) @defer
```

**Instrumentation Strategy**:
```python
# BEFORE:
try:
    result = risky_operation()
except ValueError as e:
    result = default_value
finally:
    cleanup()

# AFTER:
_exc_1 = tel.exception_entry("try")
try:
    result = risky_operation()
    tel.exception_exit(_exc_1, True)
except ValueError as e:
    tel.exception_caught(_exc_1, "ValueError", str(e))
    result = default_value
    tel.exception_exit(_exc_1, False)
finally:
    cleanup()
```

---

### 3. Arrays/Collections (create, modify, access)

**Purpose**: Track collection operations for debugging data flow

**Data Schema**:
```python
{
    "type": "create" | "append" | "access" | "modify" | "delete",
    "name": str,  # Variable name
    "line": int,
    "operation": str,  # e.g., "initialization", "append", "index"
    "size": int,  # Optional: collection size
}
```

**Telemetry Methods**:
```python
# Python
tel.array_create(name: str, initial_size: int)
tel.array_modify(name: str, operation: str, index: Optional[int], size: int)
tel.array_access(name: str, index: int, value: Any)

# JavaScript
tel.arrayCreate(name, initialSize)
tel.arrayModify(name, operation, index, size)
tel.arrayAccess(name, index, value)

# Go
Tel.ArrayCreate(name string, initialSize int)
Tel.ArrayModify(name string, operation string, index int, size int)
Tel.ArrayAccess(name string, index int, value interface{})
```

**Output Format**:
```json
{
    "event": "array_operation",
    "array_name": "items",
    "operation": "append",
    "index": 3,
    "size": 4,
    "value": "new_item",
    "timestamp_ns": 1234567890123456789
}
```

**Tree-Sitter Queries**:

*Python*:
```scheme
; List creation
(assignment
    left: (identifier) @array.name
    right: (list) @array.create
)

; List comprehension
(list_comprehension) @array.comp

; Append/extend
(call
    function: (attribute
        object: (identifier) @array.name
        attribute: (identifier) @array.method
    )
) @array.call

; Index access
(subscript
    value: (identifier) @array.name
    subscript: (_) @array.index
) @array.access
```

*JavaScript*:
```scheme
; Array creation
(variable_declarator
    name: (identifier) @array.name
    value: (array) @array.create
)

; Push/pop
(call_expression
    function: (member_expression
        object: (identifier) @array.name
        property: (property_identifier) @array.method
    )
) @array.call

; Index access
(subscript_expression
    object: (identifier) @array.name
    index: (_) @array.index
) @array.access
```

*Go*:
```scheme
; Slice creation
(short_var_declaration
    left: (expression_list (identifier) @slice.name)
    right: (expression_list (slice_expression) @slice.create)
)

; make() call
(call_expression
    function: (identifier) @func.name
    arguments: (argument_list) @slice.args
) @slice.make
```

**Instrumentation Strategy**:
```python
# BEFORE:
items = []
items.append("first")
value = items[0]

# AFTER:
items = []
tel.array_create("items", 0)
items.append("first")
tel.array_modify("items", "append", None, len(items))
value = items[0]
tel.array_access("items", 0, value)
```

---

### 4. Return Value Tracking

**Purpose**: Capture what functions actually return (not just that they returned)

**Data Schema**:
```python
{
    "line": int,
    "has_value": bool,
    "expression": str,  # e.g., "result * 2"
    "value_type": str,  # Optional: type hint
}
```

**Telemetry Methods**:
```python
# Already exists in func_exit():
tel.func_exit(func_correlation_id, return_value)

# Enhanced to capture:
# - Return value
# - Return type
# - Return expression (for debugging)
```

**Output Format** (part of existing func_exit):
```json
{
    "event": "function_exit",
    "function": "calculate",
    "return_value": 42,
    "return_type": "int",
    "return_expression": "x + y",
    "duration_ns": 123456,
    "correlation_id": "func_abc123"
}
```

**Tree-Sitter Queries**:

*Python*:
```scheme
(return_statement
    (expression_list)? @return.value
) @return
```

*JavaScript*:
```scheme
(return_statement
    (_)? @return.value
) @return
```

*Go*:
```scheme
(return_statement
    (expression_list)? @return.value
) @return
```

**Instrumentation Strategy**:
```python
# BEFORE:
def calculate(x, y):
    return x + y

# AFTER (already mostly done):
def calculate(x, y):
    _tel_func = tel.func_entry("calculate", "x, y")
    result = x + y
    tel.func_exit(_tel_func, result)
    return result
```

---

### 5. Method/Function Calls

**Purpose**: Track important API calls, database queries, external services

**Data Schema**:
```python
{
    "name": str,  # Function/method name
    "line": int,
    "receiver": str,  # Object being called on
    "arguments": List[str],  # Argument expressions
}
```

**Telemetry Methods**:
```python
# Python
tel.call_trace(func_name: str, receiver: str, args: List[Any])

# JavaScript
tel.callTrace(funcName, receiver, args)

# Go
Tel.CallTrace(funcName string, receiver string, args []interface{})
```

**Output Format**:
```json
{
    "event": "function_call",
    "function": "database.query",
    "receiver": "db",
    "arguments": ["SELECT * FROM users", "param1"],
    "timestamp_ns": 1234567890123456789
}
```

**Tree-Sitter Queries**:

*Python*:
```scheme
(call
    function: (attribute
        object: (_) @call.receiver
        attribute: (identifier) @call.name
    )
    arguments: (argument_list) @call.args
) @call
```

*JavaScript*:
```scheme
(call_expression
    function: (member_expression
        object: (_) @call.receiver
        property: (property_identifier) @call.name
    )
    arguments: (arguments) @call.args
) @call
```

*Go*:
```scheme
(call_expression
    function: (selector_expression
        operand: (_) @call.receiver
        field: (field_identifier) @call.name
    )
    arguments: (argument_list) @call.args
) @call
```

**Instrumentation Strategy**:
```python
# BEFORE:
result = db.query("SELECT * FROM users")

# AFTER:
tel.call_trace("query", "db", ["SELECT * FROM users"])
result = db.query("SELECT * FROM users")
```

---

## Data Schema

### Updated AnalysisResult (Already Done in Phase 1)

```python
@dataclass
class AnalysisResult:
    """Result of code analysis."""

    language: str
    code: str
    functions: List[Dict]  # ✅ Existing
    loops: List[Dict]      # ✅ Existing
    variables: List[Dict]  # ✅ Existing
    conditionals: List[Dict] = None  # ✅ New (Phase 1)
    arrays: List[Dict] = None        # ✅ New (Phase 1)
    exceptions: List[Dict] = None    # ✅ New (Phase 1)
    returns: List[Dict] = None       # ✅ New (Phase 1)
    method_calls: List[Dict] = None  # ✅ New (Phase 1)
```

### Updated TelemetrySnippet

```python
@dataclass
class TelemetrySnippet:
    """A snippet of telemetry code to be injected."""

    type: str  # "function", "loop", "variable", "conditional", "exception", "array", "return", "call"
    code: str
    language: str
    location: Dict
    description: str
    # Existing optional fields...
    function_index: Optional[int] = None
    batch_id: Optional[int] = None
    generated_at: Optional[float] = None
    generation_time_ms: Optional[float] = None
```

---

## Implementation Plan

### Phase 2.1: Conditionals ✅ PRIORITY

**Estimated**: 500 lines, 3-4 hours

**Files to Modify**:
1. `src/tree_sitter_analyzer.py` (+150 lines)
   - Add conditional queries for Python/JS/Go
   - Add `_extract_conditionals()` method
   - Update `analyze_code()` to call it

2. `src/telemetry_utils_template_python.py` (+80 lines)
   - Add `cond_entry(branch_id, condition, result)` method
   - Add `cond_exit(correlation_id)` method
   - Add `ConditionalTelemetry` data class

3. `src/telemetry_utils_template_javascript.js` (+80 lines)
   - Same as Python but in JavaScript

4. `src/telemetry_utils_template_golang.go` (+80 lines)
   - Same as Python but in Go

5. `src/telemetry_generator.py` (+100 lines)
   - Add conditional snippet generation
   - Handle if/else/elif branch logic

6. `tests/test_tree_sitter_conditionals.py` (+100 lines)
   - Test Python if/elif/else detection
   - Test JavaScript if/switch detection
   - Test Go if/switch detection

**Acceptance Criteria**:
- ✅ Detect all if/else/elif statements in Python
- ✅ Detect if/else and switch/case in JavaScript
- ✅ Detect if/else and switch in Go
- ✅ Generate telemetry for each branch
- ✅ Proper placement (before condition, after branch)
- ✅ All tests passing

---

### Phase 2.2: Exception Handling

**Estimated**: 400 lines, 3 hours

**Files to Modify**:
1. `src/tree_sitter_analyzer.py` (+120 lines)
2. `src/telemetry_utils_template_python.py` (+60 lines)
3. `src/telemetry_utils_template_javascript.js` (+60 lines)
4. `src/telemetry_utils_template_golang.go` (+60 lines) - defer tracking
5. `src/telemetry_generator.py` (+60 lines)
6. `tests/test_tree_sitter_exceptions.py` (+80 lines)

**Acceptance Criteria**:
- ✅ Detect try/except/finally in Python
- ✅ Detect try/catch/finally in JavaScript
- ✅ Detect defer in Go
- ✅ Track exception types and messages
- ✅ All tests passing

---

### Phase 2.3: Arrays/Collections

**Estimated**: 600 lines, 4-5 hours

**Files to Modify**:
1. `src/tree_sitter_analyzer.py` (+180 lines)
   - Most complex queries (many array operations)
2. `src/telemetry_utils_template_python.py` (+100 lines)
3. `src/telemetry_utils_template_javascript.js` (+100 lines)
4. `src/telemetry_utils_template_golang.go` (+100 lines)
5. `src/telemetry_generator.py` (+80 lines)
6. `tests/test_tree_sitter_arrays.py` (+120 lines)

**Acceptance Criteria**:
- ✅ Detect array/list creation
- ✅ Detect append/push operations
- ✅ Detect index access
- ✅ Track collection size
- ✅ All tests passing

---

### Phase 2.4: Return Value Tracking

**Estimated**: 300 lines, 2 hours

**Files to Modify**:
1. `src/tree_sitter_analyzer.py` (+80 lines)
2. `src/telemetry_utils_template_python.py` (+40 lines) - enhance func_exit
3. `src/telemetry_utils_template_javascript.js` (+40 lines)
4. `src/telemetry_utils_template_golang.go` (+40 lines)
5. `src/telemetry_generator.py` (+60 lines)
6. `tests/test_tree_sitter_returns.py` (+60 lines)

**Acceptance Criteria**:
- ✅ Detect all return statements
- ✅ Capture return expressions
- ✅ Enhanced func_exit output
- ✅ All tests passing

---

### Phase 2.5: Method Call Tracking

**Estimated**: 400 lines, 3 hours

**Files to Modify**:
1. `src/tree_sitter_analyzer.py` (+120 lines)
2. `src/telemetry_utils_template_python.py` (+60 lines)
3. `src/telemetry_utils_template_javascript.js` (+60 lines)
4. `src/telemetry_utils_template_golang.go` (+60 lines)
5. `src/telemetry_generator.py` (+60 lines)
6. `tests/test_tree_sitter_calls.py` (+80 lines)

**Acceptance Criteria**:
- ✅ Detect important method/function calls
- ✅ Capture receiver, function name, arguments
- ✅ Filter noise (only important calls)
- ✅ All tests passing

---

## Performance Analysis

### Current Performance Baseline

| Metric | Current (Functions + Loops + Variables) |
|--------|----------------------------------------|
| **Analysis Time** | <10ms (tree-sitter) |
| **Telemetry Overhead** | ~1-5% CPU, <1MB memory |
| **File Size Increase** | ~10-30% |
| **Cache Hit Speed** | 0.3ms per function |

### Projected Performance with All Constructs

| Metric | Projected (All 8 Construct Types) | Impact |
|--------|-----------------------------------|---------|
| **Analysis Time** | <20ms (tree-sitter) | +10ms (acceptable) |
| **Telemetry Overhead** | ~3-8% CPU, <2MB memory | Still acceptable |
| **File Size Increase** | ~30-60% | More telemetry code |
| **Cache Hit Speed** | 0.3ms per function | No change (cached!) |

### Mitigation Strategies

1. **Selective Instrumentation**: Add flags to disable specific constructs
   ```bash
   --no-conditionals --no-arrays  # Reduce overhead
   ```

2. **Sampling**: Only instrument N% of constructs
   ```bash
   --sample-rate 0.5  # 50% of constructs
   ```

3. **Hot Path Detection**: Avoid instrumenting performance-critical paths

---

## Testing Strategy

### Test Organization

```
tests/
├── test_tree_sitter_conditionals.py
├── test_tree_sitter_exceptions.py
├── test_tree_sitter_arrays.py
├── test_tree_sitter_returns.py
├── test_tree_sitter_calls.py
├── test_telemetry_generator_conditionals.py
├── test_telemetry_generator_exceptions.py
├── test_telemetry_generator_arrays.py
├── test_telemetry_generator_returns.py
├── test_telemetry_generator_calls.py
└── test_integration_all_constructs.py
```

### Test Coverage Goals

- **Unit Tests**: 90%+ coverage for each new method
- **Integration Tests**: End-to-end for each construct type
- **Performance Tests**: Ensure <20ms analysis time
- **Regression Tests**: Ensure existing functionality unchanged

### Example Test Structure

```python
# tests/test_tree_sitter_conditionals.py
class TestConditionalDetection:
    def test_python_if_statement(self):
        code = """
if x > 0:
    result = x * 2
"""
        analyzer = TreeSitterAnalyzer()
        result = analyzer.analyze_code(code, "python")

        assert len(result.conditionals) == 1
        assert result.conditionals[0]["type"] == "if"
        assert result.conditionals[0]["condition"] == "x > 0"

    def test_python_if_elif_else(self):
        # Test elif/else detection
        pass

    def test_javascript_switch(self):
        # Test switch/case detection
        pass
```

---

## Examples

### Example 1: Conditionals in Action

**Input Code**:
```python
def process_age(age):
    if age < 18:
        return "minor"
    elif age < 65:
        return "adult"
    else:
        return "senior"
```

**Instrumented Code**:
```python
def process_age(age):
    _tel_func = tel.func_entry("process_age", "age")

    _cond_1 = tel.cond_entry("if_line_2", "age < 18", age < 18)
    if age < 18:
        result = "minor"
        tel.cond_exit(_cond_1)
        tel.func_exit(_tel_func, result)
        return result

    _cond_2 = tel.cond_entry("elif_line_4", "age < 65", age < 65)
    elif age < 65:
        result = "adult"
        tel.cond_exit(_cond_2)
        tel.func_exit(_tel_func, result)
        return result
    else:
        _cond_3 = tel.cond_entry("else_line_6", "else", True)
        result = "senior"
        tel.cond_exit(_cond_3)
        tel.func_exit(_tel_func, result)
        return result
```

**Telemetry Output**:
```json
[
  {"event": "function_entry", "function": "process_age", "params": {"age": 25}},
  {"event": "conditional", "branch_id": "if_line_2", "condition": "age < 18", "result": false},
  {"event": "conditional", "branch_id": "elif_line_4", "condition": "age < 65", "result": true},
  {"event": "function_exit", "function": "process_age", "return_value": "adult", "duration_ns": 12345}
]
```

---

### Example 2: Exception Handling

**Input Code**:
```python
def safe_divide(a, b):
    try:
        result = a / b
    except ZeroDivisionError as e:
        result = 0
    finally:
        cleanup()
    return result
```

**Instrumented Code**:
```python
def safe_divide(a, b):
    _tel_func = tel.func_entry("safe_divide", "a, b")

    _exc_1 = tel.exception_entry("try")
    try:
        result = a / b
        tel.exception_exit(_exc_1, True)
    except ZeroDivisionError as e:
        tel.exception_caught(_exc_1, "ZeroDivisionError", str(e))
        result = 0
        tel.exception_exit(_exc_1, False)
    finally:
        cleanup()

    tel.func_exit(_tel_func, result)
    return result
```

**Telemetry Output**:
```json
[
  {"event": "function_entry", "function": "safe_divide", "params": {"a": 10, "b": 0}},
  {"event": "exception", "block_type": "try", "correlation_id": "exc_xyz789"},
  {"event": "exception_caught", "exception_type": "ZeroDivisionError", "exception_message": "division by zero"},
  {"event": "exception_exit", "success": false},
  {"event": "function_exit", "function": "safe_divide", "return_value": 0, "duration_ns": 23456}
]
```

---

### Example 3: Array Operations

**Input Code**:
```python
def process_items(items):
    results = []
    for item in items:
        results.append(item * 2)
    return results
```

**Instrumented Code**:
```python
def process_items(items):
    _tel_func = tel.func_entry("process_items", "items")

    results = []
    tel.array_create("results", 0)

    for item in items:
        tel.loop_entry("for", "item", item)
        results.append(item * 2)
        tel.array_modify("results", "append", None, len(results))
        tel.loop_exit()

    tel.func_exit(_tel_func, results)
    return results
```

**Telemetry Output**:
```json
[
  {"event": "function_entry", "function": "process_items", "params": {"items": [1, 2, 3]}},
  {"event": "array_operation", "array_name": "results", "operation": "create", "size": 0},
  {"event": "loop_entry", "loop_type": "for", "variable": "item", "value": 1},
  {"event": "array_operation", "array_name": "results", "operation": "append", "size": 1, "value": 2},
  {"event": "loop_exit"},
  // ... more loop iterations
  {"event": "function_exit", "function": "process_items", "return_value": [2, 4, 6], "duration_ns": 34567}
]
```

---

## Risk Analysis

### High Risk Items

1. **Performance Degradation**: Too much telemetry overhead
   - **Mitigation**: Selective instrumentation flags, sampling

2. **Code Bloat**: Instrumented files become too large
   - **Mitigation**: Keep telemetry utils DRY, use short method names

3. **False Positives**: Tree-sitter detects non-important constructs
   - **Mitigation**: Filtering heuristics, user configuration

### Medium Risk Items

1. **Complexity**: Too many construct types to maintain
   - **Mitigation**: Modular design, separate files per construct

2. **Testing Coverage**: Hard to test all edge cases
   - **Mitigation**: Comprehensive test suite, real-world examples

### Low Risk Items

1. **Backward Compatibility**: Existing code still works (schema already updated)
2. **LLM Fallback**: Always available for unsupported cases

---

## Success Criteria

### Phase 2 Complete When:

1. ✅ All 5 construct types detected by tree-sitter (Python/JS/Go)
2. ✅ All 5 construct types detected by LLM (fallback)
3. ✅ Telemetry utils updated with new methods (3 templates)
4. ✅ TelemetryGenerator creates snippets for all constructs
5. ✅ All tests passing (50+ new tests)
6. ✅ Performance within acceptable range (<20ms analysis)
7. ✅ Documentation updated (examples, API reference)
8. ✅ Real-world testing on bitcoin analyzer (2500+ lines)

---

## Timeline

| Phase | Description | Estimated Time | Status |
|-------|-------------|---------------|---------|
| **Phase 1** | Architecture Inversion | 2 hours | ✅ Complete |
| **Phase 2.1** | Conditionals | 3-4 hours | 🔜 Next |
| **Phase 2.2** | Exception Handling | 3 hours | ⏳ Pending |
| **Phase 2.3** | Arrays/Collections | 4-5 hours | ⏳ Pending |
| **Phase 2.4** | Return Tracking | 2 hours | ⏳ Pending |
| **Phase 2.5** | Method Calls | 3 hours | ⏳ Pending |
| **Documentation** | Update all docs | 2 hours | ⏳ Pending |
| **Testing** | Integration & performance | 2 hours | ⏳ Pending |
| **Total** | - | 21-23 hours | 9% complete |

---

## Appendix A: Tree-Sitter Query Reference

Complete tree-sitter query patterns for all constructs across all languages.

### Python Patterns

```scheme
; Conditionals
(if_statement) @conditional
(elif_clause) @conditional
(else_clause) @conditional

; Exceptions
(try_statement) @exception
(except_clause) @exception
(finally_clause) @exception

; Arrays
(list) @array
(list_comprehension) @array
(subscript) @array_access

; Returns
(return_statement) @return

; Calls
(call) @call
```

### JavaScript Patterns

```scheme
; Conditionals
(if_statement) @conditional
(switch_statement) @conditional
(case_clause) @conditional

; Exceptions
(try_statement) @exception
(catch_clause) @exception
(finally_clause) @exception

; Arrays
(array) @array
(subscript_expression) @array_access
(call_expression) @array_method

; Returns
(return_statement) @return

; Calls
(call_expression) @call
```

### Go Patterns

```scheme
; Conditionals
(if_statement) @conditional
(expression_switch_statement) @conditional

; Defer (exception-like)
(defer_statement) @defer

; Slices
(slice_expression) @slice
(index_expression) @slice_access

; Returns
(return_statement) @return

; Calls
(call_expression) @call
```

---

## Appendix B: Telemetry Utils API

Complete API reference for all new methods.

### Conditionals API

```python
# Python
tel.cond_entry(branch_id: str, condition: str, result: bool) -> str
tel.cond_exit(correlation_id: str) -> None

# JavaScript
tel.condEntry(branchId: string, condition: string, result: boolean): string
tel.condExit(correlationId: string): void

# Go
Tel.CondEntry(branchId string, condition string, result bool) string
Tel.CondExit(correlationId string)
```

### Exception API

```python
# Python
tel.exception_entry(block_type: str) -> str
tel.exception_caught(correlation_id: str, exc_type: str, exc_msg: str) -> None
tel.exception_exit(correlation_id: str, success: bool) -> None

# JavaScript
tel.exceptionEntry(blockType: string): string
tel.exceptionCaught(correlationId: string, excType: string, excMsg: string): void
tel.exceptionExit(correlationId: string, success: boolean): void

# Go
Tel.ExceptionEntry(blockType string) string
Tel.ExceptionCaught(correlationId string, excType string, excMsg string)
Tel.ExceptionExit(correlationId string, success bool)
```

### Array API

```python
# Python
tel.array_create(name: str, initial_size: int) -> None
tel.array_modify(name: str, operation: str, index: Optional[int], size: int) -> None
tel.array_access(name: str, index: int, value: Any) -> None

# JavaScript
tel.arrayCreate(name: string, initialSize: number): void
tel.arrayModify(name: string, operation: string, index: number, size: number): void
tel.arrayAccess(name: string, index: number, value: any): void

# Go
Tel.ArrayCreate(name string, initialSize int)
Tel.ArrayModify(name string, operation string, index int, size int)
Tel.ArrayAccess(name string, index int, value interface{})
```

---

## Conclusion

This design provides a comprehensive blueprint for expanding telemetry coverage from the current 3 construct types (functions, loops, variables) to 8 types (adding conditionals, exceptions, arrays, returns, calls).

**Key Benefits**:
1. **Comprehensive Coverage**: Track all major code constructs
2. **Performance Maintained**: <20ms analysis time target
3. **Modular Implementation**: Each construct is independent
4. **Backward Compatible**: Existing code continues to work
5. **Extensible**: Easy to add more constructs later

**Next Step**: Proceed with Phase 2.1 (Conditionals) implementation.

---

**Document Status**: Ready for Review
**Last Updated**: 2025-11-02
**Approval Required**: Engineering Lead
