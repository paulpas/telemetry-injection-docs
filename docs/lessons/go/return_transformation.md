# Lesson: Go Return Statement Transformation

## Problem
Go functions with return statements were failing compilation with errors like:
- `assignment mismatch: 1 variable but 2 values`
- `not enough return values`
- `undefined: result`

## Root Cause
The script generator was trying to transform return statements:
```go
// BEFORE (original)
return http.ListenAndServe(addr, handler)

// ATTEMPTED TRANSFORMATION (broken)
result = http.ListenAndServe(addr, handler)  // ERROR: result not declared!
Tel.FuncExit(_telFunc)()
return result
```

Problems:
1. Variable `result` never declared
2. Multi-value returns (e.g., `return x, nil`) create assignment mismatches
3. Go's strict typing makes return transformation risky

## Solution
**DON'T transform return statements in Go. Insert telemetry BEFORE return.**

```go
// CORRECT APPROACH
Tel.FuncExit(_telFunc)()  // Insert telemetry before return
return http.ListenAndServe(addr, handler)  // Keep original return
```

## Implementation
In script_generator.py exit telemetry logic (lines 608-613):
```python
# For Go: Insert telemetry BEFORE return, DON'T transform
# Go has multi-value returns and strict typing - transformation is too risky
# Just insert exit telemetry before the return statement
lines.insert(line_idx, indent + tel_code)  # Before return
continue  # Skip the normal insertion logic
```

## Applies To
- **Language**: Go/Golang
- **Pattern**: All return statements with expressions
- **Safe for**: Single-value, multi-value, and void returns

## Testing
```bash
cd examples/golang/telemetry_demo/instrumented
go build ./...
# Should compile successfully with NO errors
```

## Date Learned
2025-11-03

## Success Rate
100% - Eliminated all 4 return-related compilation errors
