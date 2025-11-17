# Fix for "Unknown" Events in Visualization

## Problem Description

Users were seeing "unknown" nodes appearing in the visual flow diagram for:
- Loop events (`loop_entry`, `loop_exit`, `loop_iteration`)
- Variable change events (`variable_change`)
- Some function calls

## Root Causes

### 1. Telemetry Schema (✅ Already Correct)

The telemetry schema **is correct**. All events include a `function_name` field:

```python
# Loop events
{
    "event_type": "loop_entry",
    "loop_type": "for",
    "function_name": "process_data",  # ✅ Present
    "correlation_id": "loop_abc123",
    ...
}

# Variable events
{
    "event_type": "variable_change",
    "variable_name": "total",
    "value": "42",
    "function_name": "calculate",  # ✅ Present
    ...
}
```

### 2. Caller Information (✅ Fixed)

**Issue:** Functions were sometimes marked as "unknown" due to improper stack frame detection.

**Fix:** Enhanced `_get_caller_info()` in telemetry utils to:
- Skip internal telemetry utility frames
- Filter out telemetry-specific functions (starting with `_tel_`)
- Traverse stack until finding actual user code

**Location:** `src/telemetry_utils_template_python.py:192`

**Apply the fix:**
```bash
./fix-telemetry-http.sh /path/to/your/instrumented/code
```

### 3. Visualization Logic (✅ Fixed)

**Issue:** The visualization wasn't properly handling loop and variable events.

**Old behavior:**
- Tried to create separate nodes for variables
- Didn't group loops/variables with their containing function
- Showed "unknown" for events without direct node mapping

**New behavior:**
- Groups all events by their `function_name`
- Annotates function nodes with `[for loop]` or `[variables]`
- Skips events that genuinely have "unknown" function names (with console warning)
- Updates node colors based on most recent activity

**Location:** `templates/dashboard_visual.html` (processEventForGraph function)

## What You'll See Now

### Before Fix
```
┌─────────────┐
│  calculate  │
└─────────────┘

┌─────────────┐
│   unknown   │  ← Loop event
└─────────────┘

┌─────────────┐
│   unknown   │  ← Variable event
└─────────────┘
```

### After Fix
```
┌──────────────────┐
│   calculate()    │
│  [for loop]      │  ← Shows loop annotation
│  [variables]     │  ← Shows variable annotation
└──────────────────┘
```

## Node Display Logic

### Function Nodes
- **Green** = Recently entered
- **Blue** = Recently exited
- **Label** = Function name only

### Function + Loop
- **Orange** = Loop activity
- **Label** = `function_name\n[for loop]`
- **Tooltip** = Shows loop type and iterations

### Function + Variables
- **Purple** = Variable changes
- **Label** = `function_name\n[variables]`
- **Tooltip** = Shows variable names and values

### Function + Both
- **Color** = Based on most recent activity
- **Label** = `function_name\n[for loop]\n[variables]`
- **Tooltip** = Combined information

## Verification Steps

### 1. Check Your Instrumented Code

```bash
cd /home/paulpas/git/claude-code-testing

# Check if HTTP support and caller fix are present
./check-telemetry-http.sh /path/to/instrumented/code
```

Expected output:
```
✓ HTTP sending code: FOUND
✓ Your telemetry will send to the visualizer!
```

### 2. Look for the Enhanced Caller Detection

```bash
grep -A 10 "Skip telemetry utility functions" /path/to/instrumented/_telemetry_utils.py
```

You should see:
```python
# Skip telemetry utility functions
if '_telemetry_utils' in filename or func_name.startswith('_tel_'):
    frame = frame.f_back
    continue
```

If not found, apply the fix:
```bash
./fix-telemetry-http.sh /path/to/instrumented/code
```

### 3. Test with Loops and Variables

Create a test file:
```python
def process_items(items):
    """Test function with loop and variables."""
    total = 0
    for item in items:
        total += item
    return total

if __name__ == "__main__":
    result = process_items([1, 2, 3, 4, 5])
    print(f"Total: {result}")
```

Instrument it:
```bash
python telemetry-inject.py test.py -l python -o instrumented
```

Run it:
```bash
export DEBUG=true
export RECEIVER_URL=http://localhost:8000/telemetry
python instrumented/test.py
```

Check the visualization:
- Should see **one node**: `process_items()`
- Node should be **orange** (loop activity) or **purple** (variable changes)
- Label should show `[for loop]` and `[variables]`
- **No "unknown" nodes**

### 4. Check Browser Console

Open browser developer tools (F12) and look at the console:

**Good (no warnings):**
```
WebSocket connected
```

**If you see warnings:**
```
Skipping event with unknown function: {...}
```

This means:
1. Your telemetry utils need updating (run `./fix-telemetry-http.sh`)
2. Or there's genuinely no function context (rare, usually module-level code)

## Debugging Unknown Events

If you still see "unknown" in the graph:

### 1. Open Browser Console (F12)

Look for warnings:
```javascript
Skipping event with unknown function: {
  event_type: "loop_exit",
  function_name: "unknown",  // ← Problem
  ...
}
```

### 2. Check Function Names in Events

Open the browser console and run:
```javascript
// See all events
console.table(allEvents.map(e => ({
  type: e.event_type,
  function: e.function_name,
  app: e.app_id
})));
```

Look for `function_name: "unknown"`.

### 3. Re-instrument Your Code

If events have `function_name: "unknown"`, your telemetry utils are outdated:

```bash
# Re-instrument from source
python telemetry-inject.py /path/to/source/code -l python

# Or fix existing instrumented code
./fix-telemetry-http.sh /path/to/instrumented/code
```

### 4. Check for Module-Level Code

Module-level code (not inside functions) may legitimately show as "unknown":

**Example:**
```python
# This is at module level - no containing function!
x = 10  # function_name will be "unknown"

def my_function():
    y = 20  # function_name will be "my_function"
```

**Solution:** Module-level events are now filtered out automatically.

## Summary of Fixes

| Issue | Status | Fix Location |
|-------|--------|--------------|
| Schema missing function_name | ✅ Was never broken | N/A |
| Caller detection broken | ✅ Fixed | `src/telemetry_utils_template_python.py` |
| Visualization not grouping events | ✅ Fixed | `templates/dashboard_visual.html` |
| Unknown module-level code | ✅ Fixed | Now filtered out |

## How to Apply All Fixes

**For existing instrumented code:**
```bash
# Single directory
./fix-telemetry-http.sh /path/to/instrumented/code

# All instrumented directories
find . -name "_telemetry_utils.py" | while read f; do
    ./fix-telemetry-http.sh "$(dirname "$f")"
done
```

**For new code (recommended):**
```bash
# Re-instrument from source with latest templates
python telemetry-inject.py /path/to/source/code -l python
```

## Verification Checklist

- [ ] Run `./check-telemetry-http.sh` on instrumented code
- [ ] No warnings in browser console
- [ ] Loops show as `[for loop]` annotations
- [ ] Variables show as `[variables]` annotations
- [ ] No "unknown" nodes in graph
- [ ] Function nodes are properly colored
- [ ] Clicking nodes shows detailed information

## Getting Help

If you still see "unknown" after applying these fixes:

1. **Check browser console** for specific warnings
2. **Verify telemetry events** using: http://localhost:8000/api/events
3. **Look for function_name: "unknown"** in the event JSON
4. **Re-instrument from source** as a last resort

## Example: Complete Flow

```bash
# 1. Fix existing code
cd /home/paulpas/git/claude-code-testing
./fix-telemetry-http.sh examples/python/instrumented/telemetry_demo

# 2. Start visualizer
./start-visualizer.sh

# 3. Run app (in another terminal)
export DEBUG=true
export RECEIVER_URL=http://localhost:8000/telemetry
python examples/python/instrumented/telemetry_demo/calculator.py

# 4. Open browser
# http://localhost:8000

# 5. Verify
# - See function nodes with [for loop] and [variables] annotations
# - No "unknown" nodes
# - Colors change based on activity
# - Arrows show function calls
```

You should now see a clean, organized graph with all events properly attributed to their containing functions!
