# Static Telemetry Comments - Implementation Plan

## Overview

Lightweight, non-invasive telemetry system using structured comments instead of runtime instrumentation.

## Goals

1. **Zero Runtime Overhead** - No code execution changes
2. **Static Analysis** - Extract function metadata, data flow, call graphs
3. **Architecture Discovery** - Map system structure without execution
4. **Dead Code Detection** - Identify unused functions
5. **V3 Schema Compatible** - 1:1 mapping with full telemetry system
6. **Human + Machine Readable** - English descriptions + JSONL structure

## Benefits

- ✅ Safe for production (no behavior changes)
- ✅ Works on any codebase (no execution needed)
- ✅ Fast (static analysis only)
- ✅ Version control friendly (just comments)
- ✅ Gradual adoption (can upgrade to full telemetry later)

## Comment Format

### Structure
```
# TELEMETRY:START
# {JSONL data here}
# TELEMETRY:END
```

### Language-Specific Markers
- **Python**: `# TELEMETRY:START`
- **JavaScript/TypeScript**: `// TELEMETRY:START`
- **Go**: `// TELEMETRY:START`

### Placement
Insert immediately **before** function definition:

```python
# TELEMETRY:START
# {"function": "calculate_total", "type": "business_logic", ...}
# TELEMETRY:END
def calculate_total(items: List[Item], tax_rate: float) -> float:
    """Calculate total with tax."""
    subtotal = sum(item.price for item in items)
    return subtotal + (subtotal * tax_rate)
```

## V3 Schema Distillation

### Core Fields (Always Present)

```json
{
  "event_type": "static_function_metadata",
  "timestamp": "2025-11-04T12:34:56.789Z",
  "function_name": "calculate_total",
  "function_type": "business_logic",
  "file_path": "src/billing/calculator.py",
  "line_number": 42,
  "language": "python"
}
```

### Function Metadata (Tree-Sitter Extraction)

```json
{
  "parameters": [
    {"name": "items", "type": "List[Item]", "required": true, "default": null},
    {"name": "tax_rate", "type": "float", "required": true, "default": null}
  ],
  "return_type": "float",
  "decorators": ["@cache", "@validate_input"],
  "is_async": false,
  "is_generator": false,
  "complexity": {
    "cyclomatic": 3,
    "cognitive": 5,
    "lines_of_code": 15
  }
}
```

### Data Flow Analysis (LLM Interpretation)

```json
{
  "data_flow": {
    "inputs": [
      {"name": "items", "source": "user_request", "validation": "non_empty_list"},
      {"name": "tax_rate", "source": "config", "validation": "range_0_to_1"}
    ],
    "outputs": [
      {"name": "total", "type": "float", "destination": "response"}
    ],
    "side_effects": [
      {"type": "log", "description": "Logs calculation for audit"}
    ]
  }
}
```

### Call Graph (Tree-Sitter + LLM)

```json
{
  "calls": {
    "outgoing": [
      {"function": "sum", "module": "builtins", "line": 44},
      {"function": "validate_items", "module": "billing.validators", "line": 43}
    ],
    "incoming": [
      {"function": "process_order", "module": "billing.orders", "estimated": true}
    ]
  }
}
```

### Error Handling (LLM Interpretation)

```json
{
  "error_handling": {
    "raises": [
      {"type": "ValueError", "condition": "items is empty", "line": 43},
      {"type": "TypeError", "condition": "tax_rate not numeric", "line": 44}
    ],
    "catches": [],
    "validates": [
      {"param": "items", "check": "non_empty"},
      {"param": "tax_rate", "check": "0 <= value <= 1"}
    ]
  }
}
```

### Classifiers (LLM Interpretation)

```json
{
  "classifiers": {
    "purpose": "Calculate total cost including tax for billing",
    "category": "business_logic",
    "subcategory": "financial_calculation",
    "tags": ["billing", "tax", "calculation", "critical"],
    "criticality": "high",
    "data_sensitivity": "pii",
    "performance_profile": "fast"
  }
}
```

### Usage Indicators (Static Analysis)

```json
{
  "usage": {
    "last_modified": "2025-10-15T10:30:00Z",
    "caller_count": 5,
    "test_coverage": 95,
    "is_public_api": true,
    "deprecation_status": null
  }
}
```

## Complete V3-Compatible Schema

### Fields Used in Static Telemetry

**From V3 Event Structure:**
- ✅ `event_type` (static_function_metadata)
- ✅ `timestamp` (analysis time)
- ✅ `event_id` (unique per analysis)
- ✅ `correlation_id` (per file/module)
- ✅ `trace_id` (per analysis run)
- ✅ `span_id` (function identifier)

**Function Metadata:**
- ✅ `function_name`
- ✅ `function_type` (NEW: business_logic, data_processing, etc.)
- ✅ `file_path`
- ✅ `line_number`
- ✅ `language`
- ✅ `parameters` (with types)
- ✅ `return_type`

**New Static Analysis Fields:**
- ✅ `data_flow` (inputs, outputs, side effects)
- ✅ `calls` (outgoing, incoming)
- ✅ `error_handling` (raises, catches, validates)
- ✅ `classifiers` (purpose, category, tags)
- ✅ `complexity` (cyclomatic, cognitive, LOC)
- ✅ `usage` (callers, tests, deprecation)

**Unused V3 Fields (Reserved):**
- ⚠️ `duration_ns` (N/A for static)
- ⚠️ `caller_function` (use calls.incoming instead)
- ⚠️ `exception_*` (use error_handling instead)
- ⚠️ `http_*` (N/A for static)
- ⚠️ `db_*` (N/A for static)

## Implementation Strategy

### Phase 1: Tree-Sitter Extraction (Fast, Deterministic)

**What Tree-Sitter Should Extract:**
- Function name
- Parameters (names + types if annotated)
- Return type (if annotated)
- Decorators
- Line numbers
- File path
- Is async/generator
- Basic complexity (count conditionals, loops)

**What to Skip:**
- Purpose/intent (needs LLM)
- Data flow semantics (needs LLM)
- Error conditions (needs LLM)
- Caller relationships (needs cross-file analysis)

### Phase 2: LLM Enhancement (Complex Interpretation)

**When to Use LLM:**
- Determine function purpose/category
- Infer data flow from code
- Identify error conditions
- Classify function type
- Detect validation logic

**Prompt Strategy:**
```
Analyze this function and provide:
1. Purpose (1 sentence)
2. Category (e.g., business_logic, data_processing, api_handler)
3. Data flow (inputs → processing → outputs)
4. Error conditions (what can go wrong)
5. Tags (for searchability)

Function:
{code}

Output as JSON matching this schema:
{schema}
```

### Phase 3: Cross-File Analysis (Call Graph)

**Static Call Detection:**
- Tree-sitter: Find all function calls in body
- LLM: Resolve imports and determine targets
- Build caller → callee graph

**Dead Code Detection:**
- Functions with no incoming calls (except entry points)
- Mark as potentially unused
- Consider test usage

## Example Output

### Python Function
```python
# TELEMETRY:START
# {"event_type":"static_function_metadata","timestamp":"2025-11-04T12:34:56.789Z","function_name":"calculate_total","function_type":"business_logic","file_path":"src/billing/calculator.py","line_number":42,"language":"python","parameters":[{"name":"items","type":"List[Item]","required":true},{"name":"tax_rate","type":"float","required":true}],"return_type":"float","data_flow":{"inputs":[{"name":"items","source":"user_request","validation":"non_empty_list"}],"outputs":[{"name":"total","type":"float"}]},"error_handling":{"raises":[{"type":"ValueError","condition":"items is empty"}]},"classifiers":{"purpose":"Calculate total cost including tax","category":"business_logic","tags":["billing","tax","critical"]},"complexity":{"cyclomatic":3,"lines_of_code":15},"calls":{"outgoing":[{"function":"sum","module":"builtins"}]}}
# TELEMETRY:END
def calculate_total(items: List[Item], tax_rate: float) -> float:
    """Calculate total with tax."""
    if not items:
        raise ValueError("Items cannot be empty")
    subtotal = sum(item.price for item in items)
    return subtotal + (subtotal * tax_rate)
```

### JavaScript Function
```javascript
// TELEMETRY:START
// {"event_type":"static_function_metadata","timestamp":"2025-11-04T12:34:56.789Z","function_name":"processOrder","function_type":"api_handler","file_path":"src/api/orders.js","line_number":15,"language":"javascript","parameters":[{"name":"orderId","type":"string","required":true},{"name":"options","type":"Object","required":false,"default":"{}"}],"return_type":"Promise<Order>","is_async":true,"data_flow":{"inputs":[{"name":"orderId","validation":"uuid"}],"outputs":[{"name":"order","type":"Order"}]},"error_handling":{"raises":[{"type":"NotFoundError","condition":"order not found"}]},"classifiers":{"purpose":"Process and validate order","category":"api_handler","tags":["orders","api","async"]},"calls":{"outgoing":[{"function":"validateOrder","module":"./validators"},{"function":"saveOrder","module":"./db"}]}}
// TELEMETRY:END
async function processOrder(orderId, options = {}) {
    const order = await fetchOrder(orderId);
    if (!order) throw new NotFoundError('Order not found');
    await validateOrder(order);
    return await saveOrder(order, options);
}
```

## File Structure

```
src/
├── analysis/
│   ├── static_telemetry_analyzer.py      # Main analyzer
│   ├── static_comment_generator.py       # Comment generation
│   └── static_comment_parser.py          # Parse existing comments
├── generation/
│   └── static_telemetry_schema.py        # V3-compatible schema
└── cli.py                                 # Add --static-comments flag
```

## CLI Integration

### New Flag
```bash
# Generate static telemetry comments (default)
telemetry-inject.py <directory> --static-comments

# Hybrid: Comments + full telemetry
telemetry-inject.py <directory> --static-comments --use-scripts

# Full telemetry only (no comments)
telemetry-inject.py <directory> --use-scripts --no-static-comments
```

### Output
```
Generating static telemetry comments...
✓ Analyzed 25 functions
✓ Added comments to 23 functions (2 already present)
✓ Detected 12 call relationships
✓ Identified 3 potentially unused functions
```

## Future: Visualization & Analysis

### What Can Be Built (Later)

1. **Architecture Diagrams**
   - Parse comments → build call graph → render
   - Tools: Graphviz, Mermaid, D3.js

2. **Data Flow Maps**
   - Track data through function calls
   - Identify data sources and sinks

3. **Dead Code Reports**
   - Functions with no incoming calls
   - Unused imports

4. **Complexity Heatmaps**
   - Visualize cyclomatic complexity
   - Identify refactoring candidates

5. **API Documentation**
   - Auto-generate from comments
   - Keep docs in sync with code

## Compatibility Matrix

| Feature | Static Comments | Full Telemetry | Compatible |
|---------|----------------|----------------|------------|
| Function metadata | ✅ | ✅ | ✅ 1:1 |
| Data flow | ✅ (inferred) | ✅ (observed) | ✅ |
| Call graph | ✅ (static) | ✅ (dynamic) | ✅ |
| Error handling | ✅ (declared) | ✅ (caught) | ✅ |
| Performance | ❌ | ✅ | N/A |
| Runtime values | ❌ | ✅ | N/A |
| Execution count | ❌ | ✅ | N/A |

## Success Criteria

1. ✅ Comments are valid JSONL
2. ✅ Schema matches V3 telemetry
3. ✅ Tree-sitter handles 80%+ extraction
4. ✅ LLM only used for complex interpretation
5. ✅ Comments are human-readable
6. ✅ No code execution changes
7. ✅ Can parse comments to build visualizations
8. ✅ Works on any codebase (no execution needed)

## Next Steps

1. Define complete JSON schema (V3-compatible)
2. Implement StaticTelemetryAnalyzer (tree-sitter + LLM)
3. Implement StaticCommentGenerator (JSONL formatting)
4. Add CLI flag (--static-comments, default ON)
5. Test on real codebases
6. Document format for future visualization tools
