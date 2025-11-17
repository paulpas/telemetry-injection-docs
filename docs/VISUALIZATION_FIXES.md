# Visualization Fixes - Complete Guide

## Problem Summary

You reported seeing:
1. **Only apps in left sidebar** - calculator, receiver_server, and unknown
2. **Nothing showing in main graph area**
3. **Variable changes had `function_name: "unknown"`**

## Root Cause

The instrumented code was passing **hardcoded "unknown"** for variable tracking:

```python
tel.var_change("sequence", sequence, "unknown", 66)  # ❌ Wrong!
tel.var_change("a", a, "unknown", 67)                 # ❌ Wrong!
```

Should be:
```python
tel.var_change("sequence", sequence, "fibonacci", 66)  # ✅ Correct!
tel.var_change("a", a, "fibonacci", 67)                # ✅ Correct!
```

The visualization was **correctly skipping** events with `function_name: "unknown"` because they're invalid, which is why you saw nothing.

## Solution: Re-Instrument Your Code

### Step 1: Re-Instrument

```bash
cd /home/paulpas/git/claude-code-testing

# Run the re-instrumentation script
./RE-INSTRUMENT.sh examples/python/telemetry_demo examples/python/instrumented/telemetry_demo
```

Or manually:
```bash
python3 telemetry-inject.py examples/python/telemetry_demo -l python -o examples/python/instrumented/telemetry_demo
```

### Step 2: Restart Visualizer

```bash
./start-visualizer.sh
```

### Step 3: Run Your App

```bash
export DEBUG=true
export RECEIVER_URL=http://localhost:8000/telemetry
python3 examples/python/instrumented/telemetry_demo/calculator.py
```

### Step 4: Open Browser

Navigate to: `http://localhost:8000`

## What's Fixed

### 1. Auto-Select First App

The dashboard now **automatically selects the first valid app** when events load, so you immediately see functions in the graph without clicking.

### 2. Show All Events (Even Unknown)

Events with `function_name: "unknown"` are now shown with:
- ⚠️ **Warning icon**
- **Red color**
- **Helpful tooltip**: "This usually means the code needs re-instrumentation"

This helps you identify which files need re-instrumentation.

### 3. Sequence Diagram Timeline

Click the **"Timeline"** button to see a beautiful sequence diagram showing:

- **📊 Time-ordered execution flow**
- **Timestamps** (absolute + relative)
- **Color-coded events**:
  - 🟢 Green = Function Entry
  - 🔵 Blue = Function Exit
  - 🟠 Orange = Loops
  - 🟣 Purple = Variables
  - 🔴 Red = Unknown (needs re-instrumentation)
- **Duration metrics** (for function exits)
- **Loop iteration counts**
- **Variable values**

The sequence diagram updates in real-time as your app runs!

## Expected Results After Re-Instrumentation

### In Flow Diagram View:

```
┌─────────────────┐
│   fibonacci()   │ ← Auto-selected, shows immediately
│   [for loop]    │
│   [variables]   │
└────────┬────────┘
         │ calls
         ▼
┌─────────────────┐
│      main()     │
└─────────────────┘
```

No "unknown" nodes, all properly labeled!

### In Timeline View:

```
10:30:15.123  → fibonacci()
+0ms          (n, post_result)

10:30:15.124  🔁 for loop in fibonacci
+1ms          5 iterations

10:30:15.125  sequence = [0, 1, 1, 2, 3]
+2ms          in fibonacci

10:30:15.126  ← fibonacci()
+3ms          2.45ms
```

All showing correct function names!

## Verification Checklist

After re-instrumentation and running your app:

- [ ] Dashboard shows functions immediately (no clicking needed)
- [ ] All nodes have proper function names (no ⚠️ unknown)
- [ ] Loops show `[for loop]` or `[while loop]` annotations
- [ ] Variables show proper function context
- [ ] Timeline view shows sequence diagram with all events
- [ ] No console warnings about "unknown function"
- [ ] Arrows connect functions showing call relationships

## Troubleshooting

### Still Seeing "Unknown"?

**Check instrumented code:**
```bash
grep -n "tel.var_change" examples/python/instrumented/telemetry_demo/calculator.py
```

Should show:
```python
tel.var_change("data", data, "post_to_receiver", 17)  # ✅ Function name present
```

NOT:
```python
tel.var_change("data", data, "unknown", 17)  # ❌ Hardcoded unknown
```

**Fix:** Re-run instrumentation.

### No Functions Showing?

**Check browser console (F12):**
- Look for: "Auto-selecting first valid app: calculator"
- If missing, no valid events are arriving

**Fix:**
1. Verify `export RECEIVER_URL=http://localhost:8000/telemetry`
2. Verify `export DEBUG=true`
3. Check visualizer is running on port 8000

### Graph is Empty?

**Click on an app** in the left sidebar, or wait 2-3 seconds for auto-selection.

If still empty:
```bash
# Check if events are being received
curl http://localhost:8000/api/events | jq '.[] | {function_name, event_type}'
```

Should show events with valid function names.

## New Features Summary

| Feature | What It Does |
|---------|-------------|
| **Auto-Select** | Automatically shows first valid app's functions |
| **Unknown Warnings** | Shows ⚠️ for events needing re-instrumentation |
| **Sequence Diagram** | Timeline view with execution flow visualization |
| **Real-time Updates** | Both views update as events arrive |
| **Color Coding** | Visual indicators for event types |
| **Duration Metrics** | Shows function execution times |

## Architecture

### Flow Diagram View (Default)
- **Network graph** with vis-network.js
- **Nodes** = Functions
- **Edges** = Function calls
- **Physics simulation** for automatic layout
- **Interactive zoom/pan**

### Timeline View (New!)
- **Chronological event list**
- **Sequence diagram style**
- **Time + duration** display
- **Filterable by app**
- **Real-time updates**

Both views:
- Filter by selected app
- Show all apps if none selected
- Update automatically via WebSocket

## Next Steps

1. **Re-instrument** all your code using `./RE-INSTRUMENT.sh`
2. **Test** with the visualizer to verify all function names appear
3. **Explore** both Flow Diagram and Timeline views
4. **Monitor** your application's execution in real-time!

## Example Session

```bash
# Terminal 1: Start visualizer
cd /home/paulpas/git/claude-code-testing
./start-visualizer.sh

# Terminal 2: Re-instrument and run
./RE-INSTRUMENT.sh examples/python/telemetry_demo examples/python/instrumented/telemetry_demo
export DEBUG=true
export RECEIVER_URL=http://localhost:8000/telemetry
python3 examples/python/instrumented/telemetry_demo/calculator.py

# Browser: http://localhost:8000
# ✅ Should see functions immediately
# ✅ Switch to Timeline view for sequence diagram
# ✅ All functions properly named
```

Done! 🎉
