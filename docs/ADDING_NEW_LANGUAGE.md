# Adding a New Language to the Telemetry Injection System

This guide demonstrates how to add support for a new programming language to the telemetry injection system, using **Go** as a case study.

## Table of Contents

1. [Overview](#overview)
2. [Components That Need Updates](#components-that-need-updates)
3. [Step-by-Step Guide](#step-by-step-guide)
4. [Testing Your Implementation](#testing-your-implementation)
5. [Common Pitfalls](#common-pitfalls)
6. [Language-Specific Considerations](#language-specific-considerations)

---

## Overview

Adding a new language requires updating **three core components**:

```
┌─────────────────────┐
│  1. Scanner         │  ← Detect files by extension
│  (Already generic)  │
└─────────────────────┘
          ↓
┌─────────────────────┐
│  2. FunctionExtract │  ← Parse functions from source
│  ⚠️ NEEDS UPDATE   │
└─────────────────────┘
          ↓
┌─────────────────────┐
│  3. ScriptGenerator │  ← Generate telemetry code
│  (Mostly generic)   │
└─────────────────────┘
          ↓
┌─────────────────────┐
│  4. FileReconstr... │  ← Rebuild source files
│  ⚠️ NEEDS UPDATE   │
└─────────────────────┘
```

**Components requiring updates**:
1. ✅ **Scanner** - Usually works out-of-box (just add file extensions)
2. ⚠️ **FunctionExtractor** - Add tree-sitter parsing for the language
3. ⚠️ **FileReconstructor** - Add language-specific reconstruction logic
4. ℹ️ **TelemetryGenerator** - May need language-specific prompt tweaks (optional)

---

## Components That Need Updates

### 1. FunctionExtractor (`src/function_extractor.py`)

**Purpose**: Parse source code and extract function metadata (name, parameters, line numbers)

**What to add**:
- Tree-sitter language import
- Language parser initialization
- Language-specific extraction method
- Update `SUPPORTED_LANGUAGES` list

### 2. FileReconstructor (`src/file_reconstructor.py`)

**Purpose**: Rebuild source files with instrumented functions

**What to add**:
- Update `SUPPORTED_LANGUAGES` list
- Add telemetry import statement to `TELEMETRY_IMPORTS`
- Add language routing in `reconstruct()` method
- Implement language-specific reconstruction method

### 3. TelemetryGenerator (`src/telemetry_generator.py`) *(Optional)*

**Purpose**: Generate telemetry code snippets via LLM

**What to add** (if needed):
- Language-specific prompt examples
- Language-specific anti-patterns
- Language-specific syntax guidance

---

## Step-by-Step Guide

### Step 1: Install Tree-Sitter Language Bindings

First, add the tree-sitter package for your language:

```bash
# Example for Go
pip install tree-sitter-go

# Other examples:
# pip install tree-sitter-ruby
# pip install tree-sitter-rust
# pip install tree-sitter-java
```

Update `requirements.txt`:
```
tree-sitter-go>=0.25.0  # Add your language
```

---

### Step 2: Update FunctionExtractor

**File**: `src/function_extractor.py`

#### 2.1 Add Imports

```python
try:
    from tree_sitter import Language, Parser, Query, QueryCursor
    import tree_sitter_python as tspy
    import tree_sitter_javascript as tsjs
    import tree_sitter_go as tsgo  # ← ADD THIS
    TREE_SITTER_AVAILABLE = True
except ImportError:
    TREE_SITTER_AVAILABLE = False
```

#### 2.2 Update SUPPORTED_LANGUAGES

```python
class FunctionExtractor:
    SUPPORTED_LANGUAGES = [
        "python",
        "javascript",
        "typescript",
        "go",      # ← ADD THIS
        "golang",  # ← ADD ALIAS (if applicable)
    ]
```

#### 2.3 Initialize Language Parser

```python
def __init__(self):
    """Initialize the function extractor with tree-sitter parsers."""
    self._python_parser = None
    self._javascript_parser = None
    self._go_parser = None  # ← ADD THIS

    if TREE_SITTER_AVAILABLE:
        try:
            # Existing parsers...

            # ← ADD THIS
            GO_LANGUAGE = Language(tsgo.language())
            self._go_parser = Parser(GO_LANGUAGE)
        except Exception as e:
            print(f"Warning: Could not initialize Go parser: {e}")
```

#### 2.4 Add Language Routing

```python
def extract_functions(self, code: str, language: str) -> List[ExtractedFunction]:
    """Extract functions from code based on language."""

    if language.lower() == "python":
        return self._extract_python_functions(code)
    elif language.lower() in ["javascript", "typescript"]:
        return self._extract_javascript_functions(code)
    elif language.lower() in ["go", "golang"]:  # ← ADD THIS
        return self._extract_go_functions(code)
    else:
        raise UnsupportedLanguageError(f"Language '{language}' not supported")
```

#### 2.5 Implement Extraction Method

**Key Steps**:
1. Parse code with tree-sitter
2. Write tree-sitter query for function declarations
3. Execute query and extract node information
4. Build `ExtractedFunction` objects

**Example (Go)**:

```python
def _extract_go_functions(self, code: str) -> List[ExtractedFunction]:
    """Extract functions from Go code using tree-sitter."""

    if not self._go_parser:
        raise RuntimeError("Go parser not initialized")

    # 1. Parse the code
    code_bytes = code.encode('utf-8')
    tree = self._go_parser.parse(code_bytes)
    root_node = tree.root_node

    # 2. Write tree-sitter query (LANGUAGE-SPECIFIC!)
    query_text = """
        (function_declaration
            name: (identifier) @func.name
            parameters: (parameter_list) @func.params
        ) @func.def
    """

    # 3. Execute query
    query = Query(self._go_parser.language, query_text)
    cursor = QueryCursor(query)
    matches = cursor.matches(root_node)

    functions = []

    # 4. Extract function metadata
    for _, captures_dict in matches:
        func_nodes = captures_dict.get("func.def", [])
        name_nodes = captures_dict.get("func.name", [])
        param_nodes = captures_dict.get("func.params", [])

        if func_nodes and name_nodes:
            func_node = func_nodes[0]
            name_node = name_nodes[0]

            # Extract metadata
            name = name_node.text.decode('utf-8')
            start_line = func_node.start_point[0] + 1  # 1-indexed
            end_line = func_node.end_point[0] + 1

            # Extract parameters
            parameters = []
            if param_nodes:
                param_list = param_nodes[0].text.decode('utf-8')
                # Parse parameter list (language-specific logic)
                parameters = self._parse_go_parameters(param_list)

            # Get function code
            func_code = func_node.text.decode('utf-8')

            functions.append(ExtractedFunction(
                name=name,
                code=func_code,
                language="go",
                start_line=start_line,
                end_line=end_line,
                parameters=parameters
            ))

    return functions
```

**Tree-Sitter Query Tips**:
- Use tree-sitter playground: https://tree-sitter.github.io/tree-sitter/playground
- Inspect language grammar: https://github.com/tree-sitter/tree-sitter-go/blob/master/grammar.js
- Test queries with small code samples first

**Example Queries for Common Languages**:

```python
# Python
"""
(function_definition
    name: (identifier) @func.name
    parameters: (parameters) @func.params
) @func.def
"""

# JavaScript/TypeScript
"""
(function_declaration
    name: (identifier) @func.name
    parameters: (formal_parameters) @func.params
) @func.def
"""

# Go
"""
(function_declaration
    name: (identifier) @func.name
    parameters: (parameter_list) @func.params
) @func.def
"""

# Ruby
"""
(method
    name: (identifier) @func.name
    parameters: (method_parameters)? @func.params
) @func.def
"""

# Rust
"""
(function_item
    name: (identifier) @func.name
    parameters: (parameters) @func.params
) @func.def
"""
```

---

### Step 3: Update FileReconstructor

**File**: `src/file_reconstructor.py`

#### 3.1 Update SUPPORTED_LANGUAGES

```python
class FileReconstructor:
    SUPPORTED_LANGUAGES = [
        "python",
        "javascript",
        "typescript",
        "go",      # ← ADD THIS
        "golang",  # ← ADD ALIAS
    ]
```

#### 3.2 Add Telemetry Import Statement

**CRITICAL**: Understand how your language handles imports!

```python
TELEMETRY_IMPORTS = {
    'python': 'from _telemetry_utils import tel',
    'javascript': "const tel = require('./_telemetry_utils');",
    'typescript': "import { tel } from './_telemetry_utils';",

    # Go: No import needed (same package)
    'go': '',
    'golang': '',

    # Other examples:
    'ruby': "require_relative '_telemetry_utils'",
    'rust': 'use telemetry::tel;',
    'java': 'import com.yourorg.telemetry.Tel;',
}
```

**Import Statement Guidelines**:
- Python: `from module import name`
- JavaScript (CommonJS): `const name = require('module')`
- JavaScript (ES6): `import { name } from 'module'`
- Go: Usually no import if in same package, otherwise `import "package/path"`
- Ruby: `require_relative 'file'` or `require 'gem'`
- Rust: `use module::item;`
- Java: `import package.Class;`

#### 3.3 Add Language Routing

```python
def reconstruct(self, original_code, instrumented_functions, language):
    """Reconstruct a source file with instrumented functions."""

    # ... validation ...

    if language.lower() == "python":
        return self._reconstruct_python(original_code, instrumented_functions)
    elif language.lower() in ["javascript", "typescript"]:
        return self._reconstruct_javascript(original_code, instrumented_functions)
    elif language.lower() in ["go", "golang"]:  # ← ADD THIS
        return self._reconstruct_go(original_code, instrumented_functions)
    else:
        return ReconstructionResult(
            success=False,
            reconstructed_code="",
            language=language,
            functions_replaced=0,
            error_message=f"Reconstruction not implemented for {language}"
        )
```

#### 3.4 Implement Reconstruction Method

**Strategy**: Line-based replacement (works for most languages)

```python
def _reconstruct_go(
    self,
    original_code: str,
    instrumented_functions: List[ExtractedFunction]
) -> ReconstructionResult:
    """
    Reconstruct a Go file with instrumented functions.

    Args:
        original_code: Original Go source code
        instrumented_functions: List of instrumented functions

    Returns:
        ReconstructionResult with reconstructed code
    """
    try:
        lines = original_code.split('\n')

        # Create a mapping of function names to their instrumented versions
        func_map = {func.name: func for func in instrumented_functions}

        # Track which lines belong to which functions (to be replaced)
        function_line_ranges = {}
        for func in instrumented_functions:
            function_line_ranges[func.name] = (func.start_line, func.end_line)

        # Build the reconstructed code
        reconstructed_lines = []
        functions_replaced = 0
        skip_until_line = 0

        for i, line in enumerate(lines, start=1):
            # Check if we're in a line range that should be skipped
            if i < skip_until_line:
                continue

            # Check if this line starts a function we need to replace
            replaced = False
            for func_name, (start, end) in function_line_ranges.items():
                if i == start:
                    # Insert the instrumented function
                    instrumented_code = func_map[func_name].code
                    reconstructed_lines.append(instrumented_code)
                    functions_replaced += 1
                    skip_until_line = end + 1
                    replaced = True
                    break

            # If not replaced, keep the original line
            if not replaced:
                reconstructed_lines.append(line)

        reconstructed_code = '\n'.join(reconstructed_lines)

        return ReconstructionResult(
            success=True,
            reconstructed_code=reconstructed_code,
            language="go",
            functions_replaced=functions_replaced,
            error_message=None
        )

    except Exception as e:
        return ReconstructionResult(
            success=False,
            reconstructed_code="",
            language="go",
            functions_replaced=0,
            error_message=f"Reconstruction error: {str(e)}"
        )
```

**Key Points**:
1. Split code into lines
2. Map function names to instrumented versions
3. Track line ranges for each function
4. Iterate through lines, replacing function ranges
5. Skip lines between start_line and end_line (inclusive)
6. Keep all other lines unchanged

---

### Step 4: Update TelemetryGenerator (Optional)

**File**: `src/telemetry_generator.py`

Most languages work with the existing prompts, but you may want to add language-specific examples:

```python
FUNCTION_TELEMETRY_PROMPT = """Generate {language} telemetry code...

**{language}-Specific Examples**:

✅ GOOD (Go):
```go
func CalculateTotal(items []Item, taxRate float64) float64 {{
    Tel("CalculateTotal", "entry", map[string]interface{{}}{{
        "items": items,
        "taxRate": taxRate,
    }})

    // ... function logic ...

    Tel("CalculateTotal", "exit", map[string]interface{{}}{{
        "result": total,
    }})
    return total
}}
```

❌ BAD (Go):
```go
func CalculateTotal(items []Item, taxRate float64) float64 {{
    import "telemetry"  // ❌ Don't add imports inside functions!
    ...
}}
```
"""
```

---

## Testing Your Implementation

### Test 1: Function Extraction

```python
from src.function_extractor import FunctionExtractor

code = """
func Add(a int, b int) int {
    return a + b
}
"""

extractor = FunctionExtractor()
functions = extractor.extract_functions(code, "go")

assert len(functions) == 1
assert functions[0].name == "Add"
assert functions[0].start_line == 1
```

### Test 2: File Reconstruction

```python
from src.file_reconstructor import FileReconstructor
from src.function_extractor import ExtractedFunction

original = """package main

func Add(a, b int) int {
    return a + b
}
"""

instrumented_func = ExtractedFunction(
    name="Add",
    code="func Add(a, b int) int {\n    // instrumented\n    return a + b\n}",
    language="go",
    start_line=3,
    end_line=5,
    parameters=["a int", "b int"]
)

reconstructor = FileReconstructor()
result = reconstructor.reconstruct(original, [instrumented_func], "go")

assert result.success == True
assert result.functions_replaced == 1
```

### Test 3: End-to-End Pipeline

```bash
# Create test file
mkdir -p /tmp/test_go
cat > /tmp/test_go/calculator.go << 'EOF'
package main

func Add(a, b int) int {
    return a + b
}
EOF

# Run telemetry injection
python telemetry-inject.py /tmp/test_go --use-scripts

# Check output
cat /tmp/test_go/instrumented/calculator.go
```

---

## Common Pitfalls

### 1. **Tree-Sitter Query Syntax Errors**

❌ **Problem**: Query fails to match anything

```python
# Wrong capture name
"""
(function_declaration
    name: (identifier) @name  # ← Inconsistent naming
) @function
"""
```

✅ **Solution**: Use consistent, descriptive capture names

```python
"""
(function_declaration
    name: (identifier) @func.name
    parameters: (parameter_list) @func.params
) @func.def
"""
```

**Debugging Tips**:
- Test queries at https://tree-sitter.github.io/tree-sitter/playground
- Print captured nodes: `print(captures_dict)`
- Check language grammar file for node types

---

### 2. **Incorrect Line Number Handling**

❌ **Problem**: Off-by-one errors in line numbers

```python
start_line = func_node.start_point[0]  # ← 0-indexed!
```

✅ **Solution**: Tree-sitter uses 0-indexed lines, convert to 1-indexed

```python
start_line = func_node.start_point[0] + 1  # Convert to 1-indexed
end_line = func_node.end_point[0] + 1
```

---

### 3. **Import Statement Placement**

❌ **Problem**: Import added in wrong location

```python
# Inserted AFTER code starts
package main

func Add() {}  // ← Code already here

import "fmt"  // ← Import too late!
```

✅ **Solution**: Check language import conventions

```python
# Go: imports come after package declaration
package main

import "fmt"  // ← Correct location

func Add() {}
```

---

### 4. **Parameter Parsing**

❌ **Problem**: Parameters not extracted correctly

```python
# Go: "a int, b int" → Need to parse types
# JavaScript: "(a, b = 10)" → Need to handle defaults
```

✅ **Solution**: Implement language-specific parameter parsing

```python
def _parse_go_parameters(self, param_list: str) -> List[str]:
    """Parse Go parameter list: (a int, b string) → ['a int', 'b string']"""
    param_list = param_list.strip('()')
    if not param_list:
        return []

    # Split by comma, handle grouped types: (a, b int) → ['a int', 'b int']
    # ... language-specific logic ...
    return parameters
```

---

### 5. **Tree-Sitter API Version**

❌ **Problem**: Using wrong API methods

```python
# Old API (deprecated)
captures = query.captures(node)
```

✅ **Solution**: Use QueryCursor API

```python
# Correct API (current)
cursor = QueryCursor(query)
matches = cursor.matches(root_node)
for _, captures_dict in matches:
    nodes = captures_dict.get("capture_name", [])
```

---

## Language-Specific Considerations

### Python
- **Indentation**: Critical! Use AST-based manipulation
- **Imports**: Place after module docstring
- **Decorators**: Preserve decorator syntax

### JavaScript/TypeScript
- **Arrow functions**: `const add = (a, b) => a + b`
- **Class methods**: Extract from class context
- **Async/await**: Handle async function declarations

### Go
- **Package declaration**: Keep package line intact
- **No imports needed**: If `_telemetry_utils.go` in same package
- **Receiver functions**: `func (r *Receiver) Method() {}`

### Ruby
- **Dynamic typing**: Parameters have no type annotations
- **Blocks**: Handle block syntax `do...end` and `{...}`
- **Modules/Classes**: Extract methods from class context

### Rust
- **Ownership**: Understand borrowing in parameters
- **Macros**: Handle macro invocations like `println!`
- **Lifetimes**: Parse lifetime annotations `'a`

### Java
- **Visibility modifiers**: `public`, `private`, `protected`
- **Static methods**: Handle static vs instance methods
- **Generics**: Parse generic type parameters `<T>`

---

## Checklist for Adding a New Language

- [ ] Install tree-sitter language package
- [ ] Update `requirements.txt`
- [ ] Add language import to `FunctionExtractor`
- [ ] Add language to `SUPPORTED_LANGUAGES` (FunctionExtractor)
- [ ] Initialize language parser in `__init__`
- [ ] Write tree-sitter query for function declarations
- [ ] Implement `_extract_<language>_functions()` method
- [ ] Add language routing in `extract_functions()`
- [ ] Test function extraction with sample code
- [ ] Add language to `SUPPORTED_LANGUAGES` (FileReconstructor)
- [ ] Add telemetry import to `TELEMETRY_IMPORTS`
- [ ] Implement `_reconstruct_<language>()` method
- [ ] Add language routing in `reconstruct()`
- [ ] Test reconstruction with sample code
- [ ] (Optional) Add language-specific examples to prompts
- [ ] Run end-to-end pipeline test
- [ ] Update documentation
- [ ] Create demo/example file
- [ ] Commit and push changes

---

## Example: Complete Go Implementation

See commits:
- `ce569ef` - FunctionExtractor Go support
- `f916198`, `ee22095` - Tree-sitter API fixes
- `be2145d` - FileReconstructor Go support

**Files Modified**:
1. `src/function_extractor.py` (+65 lines)
2. `src/file_reconstructor.py` (+76 lines)

**Total Changes**: ~141 lines of code to add complete Go support

---

## Next Steps After Implementation

1. **Documentation**: Update README.md with supported languages
2. **Examples**: Create demo file in `examples/<language>/`
3. **Tests**: Add language-specific tests
4. **CI/CD**: Add language to test matrix
5. **Benchmarks**: Measure performance on real codebases

---

## Resources

- **Tree-Sitter**: https://tree-sitter.github.io/
- **Tree-Sitter Playground**: https://tree-sitter.github.io/tree-sitter/playground
- **Language Grammars**: https://github.com/tree-sitter/
- **Python Bindings**: https://github.com/tree-sitter/py-tree-sitter

---

## Questions?

See also:
- [`docs/TREE_SITTER_IMPLEMENTATION.md`](TREE_SITTER_IMPLEMENTATION.md) - Tree-sitter architecture
- [`docs/SCRIPT_BASED_INJECTION_DESIGN.md`](SCRIPT_BASED_INJECTION_DESIGN.md) - Overall system design
- [`CLAUDE.md`](../CLAUDE.md) - Development session notes

---

**Last Updated**: 2025-11-02
**Languages Supported**: Python, JavaScript, TypeScript, Go
**Next Languages**: Ruby, Rust, Java
