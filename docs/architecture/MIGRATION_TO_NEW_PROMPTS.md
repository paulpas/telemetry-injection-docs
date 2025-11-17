# Migration Guide: Using New Language Instruction System

## Overview

This guide shows how to integrate the new modular language instruction system into `telemetry_generator.py`.

---

## Quick Start

### Add Import

At the top of `telemetry_generator.py`:

```python
# Add this import
from src.prompts import (
    get_function_prompt,
    get_loop_prompt,
    get_variable_prompt,
    get_system_message
)
```

### Replace Old Prompts (Option 1: Gradual Migration)

Keep existing prompts as fallback, use new system:

```python
class TelemetryGenerator:
    # Keep existing FUNCTION_TELEMETRY_PROMPT, etc. as fallback

    def _generate_function_telemetry(self, func: Dict, language: str) -> TelemetrySnippet:
        """Generate telemetry code for a function."""

        # NEW: Use prompt composer
        try:
            prompt = get_function_prompt(
                language=language,
                name=func.get("name"),
                parameters=", ".join(func.get("parameters", [])),
                return_type=func.get("return_type", "unknown"),
                start_line=func.get("start_line"),
                end_line=func.get("end_line")
            )
            system_msg = get_system_message(language)
        except Exception as e:
            # Fallback to old prompt if new system fails
            print(f"Warning: Failed to load new prompts, using fallback: {e}")
            prompt = self.FUNCTION_TELEMETRY_PROMPT.format(...)
            system_msg = f"You are an expert in {language}..."

        code = self._call_llm(system_msg, prompt)

        return TelemetrySnippet(...)
```

### Replace Old Prompts (Option 2: Full Migration)

Remove old prompts entirely, use only new system:

```python
class TelemetryGenerator:
    # REMOVE:
    # FUNCTION_TELEMETRY_PROMPT = """..."""
    # LOOP_TELEMETRY_PROMPT = """..."""
    # VARIABLE_TELEMETRY_PROMPT = """..."""

    def _generate_function_telemetry(self, func: Dict, language: str) -> TelemetrySnippet:
        """Generate telemetry code for a function."""

        # Use new prompt composer
        prompt = get_function_prompt(
            language=language,
            name=func.get("name"),
            parameters=", ".join(func.get("parameters", [])),
            return_type=func.get("return_type", "unknown"),
            start_line=func.get("start_line"),
            end_line=func.get("end_line")
        )
        system_msg = get_system_message(language)

        code = self._call_llm(system_msg, prompt)

        return TelemetrySnippet(
            type="function",
            code=code,
            language=language,
            location={
                "start_line": func.get("start_line"),
                "end_line": func.get("end_line"),
                "name": func.get("name")
            },
            description=f"Telemetry for function '{func.get('name')}'"
        )
```

---

## Complete Migration Example

### Before (telemetry_generator.py):

```python
class TelemetryGenerator:
    FUNCTION_TELEMETRY_PROMPT = """Generate {language} telemetry code...

**EXAMPLE OUTPUT - Python**:
...

**EXAMPLE OUTPUT - Go**:
...
"""

    def _generate_function_telemetry(self, func: Dict, language: str):
        prompt = self.FUNCTION_TELEMETRY_PROMPT.format(
            language=language,
            name=func.get("name"),
            # ...
        )

        if language.lower() in ["go", "golang"]:
            system_msg = f"You are an expert... CRITICAL: For Go, NEVER..."
        else:
            system_msg = f"You are an expert in {language}..."

        code = self._call_llm(system_msg, prompt)
        return TelemetrySnippet(...)
```

### After (telemetry_generator.py):

```python
from src.prompts import get_function_prompt, get_system_message

class TelemetryGenerator:
    # No more hardcoded prompts!

    def _generate_function_telemetry(self, func: Dict, language: str):
        # Prompt composer handles everything
        prompt = get_function_prompt(
            language=language,
            name=func.get("name"),
            parameters=", ".join(func.get("parameters", [])),
            return_type=func.get("return_type", "unknown"),
            start_line=func.get("start_line"),
            end_line=func.get("end_line")
        )

        # Language-specific system message
        system_msg = get_system_message(language)

        code = self._call_llm(system_msg, prompt)
        return TelemetrySnippet(...)
```

---

## Update All Three Methods

### 1. Function Telemetry

```python
def _generate_function_telemetry(self, func: Dict, language: str) -> TelemetrySnippet:
    prompt = get_function_prompt(
        language=language,
        name=func.get("name"),
        parameters=", ".join(func.get("parameters", [])),
        return_type=func.get("return_type", "unknown"),
        start_line=func.get("start_line"),
        end_line=func.get("end_line")
    )
    system_msg = get_system_message(language)
    code = self._call_llm(system_msg, prompt)

    return TelemetrySnippet(
        type="function",
        code=code,
        language=language,
        location={
            "start_line": func.get("start_line"),
            "end_line": func.get("end_line"),
            "name": func.get("name")
        },
        description=f"Telemetry for function '{func.get('name')}'"
    )
```

### 2. Loop Telemetry

```python
def _generate_loop_telemetry(self, loop: Dict, language: str) -> TelemetrySnippet:
    prompt = get_loop_prompt(
        language=language,
        loop_type=loop.get("type"),
        variable=loop.get("variable", ""),
        start_line=loop.get("start_line"),
        end_line=loop.get("end_line")
    )
    system_msg = get_system_message(language)
    code = self._call_llm(system_msg, prompt)

    return TelemetrySnippet(
        type="loop",
        code=code,
        language=language,
        location={
            "start_line": loop.get("start_line"),
            "end_line": loop.get("end_line")
        },
        description=f"Telemetry for {loop.get('type')} loop"
    )
```

### 3. Variable Telemetry

```python
def _generate_variable_telemetry(self, var: Dict, language: str) -> TelemetrySnippet:
    prompt = get_variable_prompt(
        language=language,
        name=var.get("name"),
        var_type=var.get("type", "unknown"),
        line=var.get("line")
    )
    system_msg = get_system_message(language)
    code = self._call_llm(system_msg, prompt)

    return TelemetrySnippet(
        type="variable",
        code=code,
        language=language,
        location={"line": var.get("line"), "name": var.get("name")},
        description=f"Telemetry for variable '{var.get('name')}'"
    )
```

---

## Update Parallel Generation

For `_generate_parallel()` method:

```python
def _generate_parallel(self, analysis: AnalysisResult) -> List[TelemetrySnippet]:
    requests = []

    # Functions
    for func in analysis.functions:
        prompt = get_function_prompt(
            language=analysis.language,
            name=func.get("name"),
            parameters=", ".join(func.get("parameters", [])),
            return_type=func.get("return_type", "unknown"),
            start_line=func.get("start_line"),
            end_line=func.get("end_line")
        )
        system_msg = get_system_message(analysis.language)

        requests.append(_LLMRequest(
            request_type="function",
            construct=func,
            language=analysis.language,
            system_message=system_msg,
            user_message=prompt
        ))

    # Loops
    for loop in analysis.loops:
        prompt = get_loop_prompt(
            language=analysis.language,
            loop_type=loop.get("type"),
            variable=loop.get("variable", ""),
            start_line=loop.get("start_line"),
            end_line=loop.get("end_line")
        )
        system_msg = get_system_message(analysis.language)

        requests.append(_LLMRequest(
            request_type="loop",
            construct=loop,
            language=analysis.language,
            system_message=system_msg,
            user_message=prompt
        ))

    # Variables
    for var in analysis.variables:
        prompt = get_variable_prompt(
            language=analysis.language,
            name=var.get("name"),
            var_type=var.get("type", "unknown"),
            line=var.get("line")
        )
        system_msg = get_system_message(analysis.language)

        requests.append(_LLMRequest(
            request_type="variable",
            construct=var,
            language=analysis.language,
            system_message=system_msg,
            user_message=prompt
        ))

    # Process in parallel...
    results = self._parallel_manager.process_batch(...)
    return snippets
```

---

## Benefits of Migration

### Before
- ❌ 3 huge prompts in telemetry_generator.py
- ❌ Mixed language examples
- ❌ Hard to maintain
- ❌ Language-specific logic scattered
- ❌ Few examples

### After
- ✅ Clean, simple imports
- ✅ Language-specific instruction files
- ✅ Easy to maintain and extend
- ✅ Comprehensive examples (60+ per language)
- ✅ Multi-language context for better learning

---

## Testing Migration

### Test Each Language

```python
# Test Go
from src.prompts import get_function_prompt, get_system_message

go_prompt = get_function_prompt("go", "Calculate", "x, y", "int", 10, 20)
go_system = get_system_message("go")

print("GO PROMPT:")
print(go_prompt)
print("\nGO SYSTEM MESSAGE:")
print(go_system)

# Test Python
py_prompt = get_function_prompt("python", "calculate", "x, y", "int", 10, 20)
py_system = get_system_message("python")

print("\nPYTHON PROMPT:")
print(py_prompt)

# Test JavaScript
js_prompt = get_function_prompt("javascript", "calculate", "x, y", "number", 10, 20)
js_system = get_system_message("javascript")

print("\nJAVASCRIPT PROMPT:")
print(js_prompt)
```

### Verify Prompt Structure

Each prompt should have:
1. ✅ Primary language section (full instructions)
2. ✅ Other languages section (brief context)
3. ✅ Clear visual separation
4. ✅ Critical reminder at end

### Test with Real Code

```bash
# Before migration - baseline
python -m src.cli examples/golang/telemetry_demo -e .go -v

# After migration - should be better or same
python -m src.cli examples/golang/telemetry_demo -e .go -v

# Check for improvements:
# - No import errors
# - Better code quality
# - Fewer syntax errors
```

---

## Rollback Plan

If issues occur, easy to rollback:

```python
class TelemetryGenerator:
    # Keep old prompts commented
    # FUNCTION_TELEMETRY_PROMPT = """..."""

    def _generate_function_telemetry(self, func: Dict, language: str):
        try:
            # Try new system
            prompt = get_function_prompt(...)
            system_msg = get_system_message(language)
        except Exception as e:
            # Rollback to old system
            print(f"Using fallback prompts: {e}")
            prompt = self.FUNCTION_TELEMETRY_PROMPT.format(...)
            system_msg = f"You are an expert..."

        code = self._call_llm(system_msg, prompt)
        return TelemetrySnippet(...)
```

---

## Cleanup (After Successful Migration)

Once new system is stable, remove old prompts:

```python
class TelemetryGenerator:
    # DELETE these (no longer needed):
    # FUNCTION_TELEMETRY_PROMPT = """..."""
    # LOOP_TELEMETRY_PROMPT = """..."""
    # VARIABLE_TELEMETRY_PROMPT = """..."""

    # All prompt logic now in src/prompts/
```

---

## Summary

### Migration Steps

1. ✅ Add imports from `src.prompts`
2. ✅ Replace `FUNCTION_TELEMETRY_PROMPT.format(...)` with `get_function_prompt(...)`
3. ✅ Replace system message logic with `get_system_message(language)`
4. ✅ Repeat for loops and variables
5. ✅ Update parallel generation
6. ✅ Test all languages
7. ✅ Remove old prompts (after verification)

### Result

- **Simpler** telemetry_generator.py
- **Better** code generation quality
- **Easier** to maintain and extend
- **Comprehensive** language-specific examples
- **Non-overfitted** multi-language learning

---

## Next Steps

1. **Migrate** telemetry_generator.py
2. **Test** with all languages
3. **Monitor** error rates
4. **Add** new examples as issues are discovered
5. **Extend** to new languages (Rust, Java, etc.)
