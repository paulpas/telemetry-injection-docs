# Anti-Pattern: Markdown Code Fences in Source Code

## Date: 2025-11-05 08:25:00

## Category: Code Generation / LLM Response Parsing

## Severity: CRITICAL

## Problem Statement

LLM responses containing markdown code fences (` ``` `) were being inserted directly into Go source code, causing syntax errors:

```
# command-line-arguments
cmd/receiver_server/main.go:87:1: syntax error: non-declaration statement outside function body
```

The actual file content showed:
```
Line 85: }
Line 86:
Line 87: ```           ← MARKDOWN CODE FENCE!
Line 88: func main() {
```

## Root Cause

**Multiple failure points in the pipeline:**

1. **Incomplete code fence removal** (`src/injection/code_injector.py:684-692`)
   - Only removed fences at the START of code
   - Did NOT remove fences in the MIDDLE of code
   - Pattern: `if instrumented_code.startswith("```")`

2. **File reconstructor not cleaning** (`src/injection/file_reconstructor.py`)
   - Inserted instrumented function code without cleaning
   - Assumed input was already clean
   - No validation of code quality

3. **LLM responding with markdown formatting**
   - LLMs naturally wrap code in ``` fences for readability
   - System did not explicitly forbid this in prompts
   - Multiple injection points in the pipeline

## The Anti-Pattern (NEVER DO THIS)

**❌ WRONG - Only cleaning start of code:**
```python
if instrumented_code.startswith("```"):
    lines = instrumented_code.split("\n")
    lines = lines[1:]  # Remove first line
    if lines and lines[-1].strip() == "```":
        lines = lines[:-1]  # Remove last line
    instrumented_code = "\n".join(lines)
```

**Problem:** This ONLY handles code that starts with fences. If LLM inserts fences between functions, they remain.

**❌ WRONG - Trusting input is clean:**
```python
for func_name, (start, end) in function_line_ranges.items():
    if i == start:
        instrumented_code = func_map[func_name].code  # ASSUME IT'S CLEAN!
        reconstructed_lines.append(instrumented_code)
```

**Problem:** Never assumes inputs are clean, always validate/sanitize.

## The Correct Pattern (DO THIS)

**✅ CORRECT - Comprehensive fence removal:**
```python
def _remove_code_fences(code: str) -> str:
    """Remove ALL markdown code fences, not just at start/end."""
    lines = code.split("\n")
    cleaned_lines = []

    for line in lines:
        stripped = line.strip()
        # Skip lines that are ONLY code fences
        is_fence = (stripped == "```" or
                   stripped.startswith("```go") or
                   stripped.startswith("```python") or
                   # ... other languages ...
                   )

        if not is_fence:
            cleaned_lines.append(line)

    return "\n".join(cleaned_lines)
```

**✅ CORRECT - Always sanitize before using:**
```python
for func_name, (start, end) in function_line_ranges.items():
    if i == start:
        # ALWAYS clean before inserting!
        instrumented_code = self._remove_code_fences(func_map[func_name].code)
        reconstructed_lines.append(instrumented_code)
```

**✅ CORRECT - Defense in depth:**
```python
# Clean at EVERY stage:
# 1. When LLM returns code
instrumented_code = response.choices[0].message.content
instrumented_code = self._remove_code_fences(instrumented_code)

# 2. When reconstructing files
instrumented_code = self._remove_code_fences(func.code)

# 3. Before writing to disk
final_code = self._remove_code_fences(reconstructed_code)
```

## Prevention Rules

### Rule 1: Defense in Depth
**NEVER trust ANY input to be clean. ALWAYS sanitize at EVERY pipeline stage.**

- Clean when receiving from LLM
- Clean when reconstructing files
- Clean before writing to disk
- Clean when reading from cache

### Rule 2: Explicit LLM Instructions
**ALWAYS tell LLM explicitly NOT to use markdown:**

```python
prompt = """
CRITICAL: Return ONLY raw Go code.
DO NOT wrap in markdown code fences.
DO NOT use ``` or ```go.
Output pure Go source code only.
"""
```

### Rule 3: Comprehensive Cleaning
**ALWAYS clean ALL fence types, not just one:**

- Clean `\`\`\``
- Clean `\`\`\`go`
- Clean `\`\`\`python`
- Clean `\`\`\`javascript`
- Add new languages as needed

### Rule 4: Validation Before Use
**ALWAYS validate code before inserting into files:**

```python
def _validate_no_fences(code: str) -> bool:
    """Ensure no markdown artifacts remain."""
    return "```" not in code
```

## Files Fixed

1. **src/injection/code_injector.py** (lines 684-710)
   - Enhanced code fence removal
   - Added comprehensive language support
   - Clean at LLM response stage

2. **src/injection/file_reconstructor.py** (lines 27-60, 233)
   - Added `_remove_code_fences()` static method
   - Applied cleaning at reconstruction stage
   - Applied to all language paths (Python/JS/Go)

## Test Case to Prevent Regression

```python
def test_code_fence_removal():
    """Ensure code fences are NEVER inserted into source files."""

    # Test input with fences in multiple locations
    code_with_fences = """package main

```
func Add(a, b int) int {
    return a + b
}
```go
func main() {
    result := Add(5, 3)
}
```
"""

    cleaned = FileReconstructor._remove_code_fences(code_with_fences)

    # Validate NO fences remain
    assert "```" not in cleaned
    assert "func Add" in cleaned
    assert "func main" in cleaned
```

## Impact

**Before fix:**
- ❌ 14+ compilation failures with "syntax error: non-declaration statement outside function body"
- ❌ Same error repeated despite 20+ retry attempts
- ❌ System NOT learning from failures

**After fix:**
- ✅ Code compiles successfully
- ✅ No markdown artifacts in generated code
- ✅ Telemetry working correctly
- ✅ Anti-pattern documented for future prevention

## Key Lessons

1. **LLMs naturally use markdown** - They're trained on markdown, they'll use it unless explicitly told not to
2. **Single-point cleaning is insufficient** - Need defense in depth at every stage
3. **Trust nothing** - Always sanitize, validate, verify
4. **Document anti-patterns** - Prevent future developers from repeating the mistake

## References

- Go Cheat Sheet: `/home/paulpas/git/claude-code-testing/lessons/golang/go_cheat_sheet.md`
- Validation failures: `docs/lessons/go/validation_failure_*.md`
- Code fence removal: `src/injection/code_injector.py:684-710`
- File reconstruction: `src/injection/file_reconstructor.py:27-60`

## Success Criteria

- ✅ All generated Go code compiles without syntax errors
- ✅ No `\`\`\`` characters appear in any instrumented file
- ✅ Validation passes without "non-declaration statement" errors
- ✅ System learns from this failure and never repeats it

---

**Status**: ✅ RESOLVED
**Verified**: 2025-11-05 08:30:00
**Test Case**: `/tmp/test_go_project` - compiles and runs successfully
