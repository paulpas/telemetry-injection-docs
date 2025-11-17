# Mermaid.js Telemetry Visualizer

## Related Documentation

- **[Visualizer Quick Start](VISUALIZER_QUICK_START.md)** - Get started quickly
- **[Telemetry Visualizer](TELEMETRY_VISUALIZER.md)** - Full visualizer documentation
- **[Telemetry Viewing Guide](../TELEMETRY_VIEWING_GUIDE.md)** - View and analyze raw telemetry
- **[Program Name Field](../PROGRAM_NAME_FIELD.md)** - Identify programs in telemetry
- **[Enhanced Telemetry](ENHANCED_TELEMETRY.md)** - Correlation IDs and relationships
- **[Visual Flow Diagram](VISUAL_FLOW_DIAGRAM.md)** - Alternative visualization
- **[Main README](../README.md)** - Project overview

---

## Overview

The telemetry visualizer now uses **Mermaid.js** to render beautiful, live-updating sequence diagrams showing:
- **Network → Application → Function** flow
- Real-time event processing via WebSocket
- Windowing/pagination for large datasets
- Zoom and filter controls
- Interactive sequence diagrams

## Features

### ✨ Live Sequence Diagrams

**Shows three layers:**
1. **Network** - External invocations
2. **Application** - Your app (calculator, receiver_server, etc.)
3. **Functions** - Individual functions being called

**Example:**
```
Network ──invoke──> fibonacci()
        ↓ activate
    fibonacci() ──call──> post_to_receiver()
                  ↓ activate
              [for loop start]
              [5 iterations]
              [variables: sequence = [0,1,1,2,3]]
                  ↓ deactivate
              ← 2.45ms
        ↓ deactivate
    ← completed
```

### 📊 Windowing for Large Datasets

When you have hundreds or thousands of events:
- **Window Size**: Control how many events to show (default: 50)
- **Navigation**: Previous/Next buttons to scroll through events
- **Dynamic Updates**: New events appear automatically in real-time

**Controls:**
- `◀ Previous` - Show previous 50 events
- `Next ▶` - Show next 50 events
- `Reset` - Jump back to start
- `Window Size` input - Adjust how many events per page (10-500)

**Info Display:** Shows "Events 1-50 of 237" etc.

### 🔍 Filtering

Filter events by type:
- **All Events** - Everything
- **Functions Only** - Just function_entry/function_exit
- **With Loops** - Only loop-related events
- **With Variables** - Only variable changes

### 🔎 Zoom Controls

Three floating buttons (bottom-right):
- **+** - Zoom in (up to 3x)
- **−** - Zoom out (down to 0.3x)
- **⟲** - Reset to 100%

Smooth transitions, centered zooming.

### 📱 Application Selection

**Left Sidebar:**
- Shows all connected applications
- Click to filter diagram to that app only
- Auto-selects first valid app on startup
- Shows event count per app

### 🔄 Real-Time Updates

**WebSocket Connection:**
- Connects automatically on page load
- Receives events as they happen
- Updates diagram in real-time
- Shows connection status (Connected/Disconnected)
- Auto-reconnects if connection drops

### 📈 Statistics

Bottom bar shows:
- **Applications** - Number of connected apps
- **Total Events** - All events received
- **Visible Events** - Events in current window
- **Functions** - Unique functions seen

## Usage

### Start the Visualizer

```bash
cd /home/paulpas/git/claude-code-testing
./start-visualizer.sh
```

Output:
```
✓ Found available port: 8000
Starting server on http://localhost:8000
```

### Run Your Instrumented App

```bash
# First, re-instrument if needed
./RE-INSTRUMENT.sh examples/python/telemetry_demo examples/python/instrumented/telemetry_demo

# Then run with telemetry enabled
export DEBUG=true
export RECEIVER_URL=http://localhost:8000/telemetry
python3 examples/python/instrumented/telemetry_demo/calculator.py
```

### Open in Browser

Navigate to: **http://localhost:8000**

You'll see:
1. **Left sidebar** - Applications list (auto-selects first one)
2. **Main area** - Live Mermaid sequence diagram
3. **Top controls** - Filter and refresh buttons
4. **Window controls** - Navigation and window size
5. **Bottom stats** - Event counts and metrics
6. **Floating buttons** - Zoom controls

## Sequence Diagram Explained

### Participants (Actors)

The diagram shows these participants:
- **Network** - External caller (user, cron, API, etc.)
- **Application names** - Your apps (calculator, receiver_server, etc.)
- **Function names** - Individual functions (fibonacci, add, multiply, etc.)

### Flow Arrows

```
Network -->>fibonacci: invoke
    activate fibonacci
    fibonacci-->>post_to_receiver: call
        activate post_to_receiver
        Note over post_to_receiver: params: result, input_value, receiver...
        Note over post_to_receiver: data = {'result': 120}
        deactivate post_to_receiver
    deactivate fibonacci
    Note over fibonacci: 2.45ms
```

- **Solid arrows** - Function calls
- **Activation boxes** - Function is executing
- **Notes** - Parameters, variables, timing
- **Rectangles** - Loops

### Event Types

**Function Entry:**
```
caller-->>function: call
activate function
Note over function: params: x, y
```

**Function Exit:**
```
deactivate function
Note over function: 1.23ms
```

**Loops:**
```
rect rgb(245, 158, 11, 0.1)
Note over function: 🔁 for loop start
Note over function: 5 iterations
end
```

**Variables:**
```
Note over function: total = 120
Note over function: result = [0, 1, 1, 2, 3]
```

## Advanced Features

### Large Dataset Handling

For apps generating thousands of events:

1. **Adjust window size:**
   - Set to 100-200 for detailed analysis
   - Set to 20-30 for quick overview

2. **Use filters:**
   - "Functions Only" - See just the call flow
   - "With Loops" - Focus on iteration behavior
   - "With Variables" - Track data changes

3. **Navigate windows:**
   - Jump to specific time periods
   - Follow execution chronologically

### Multiple Applications

When monitoring multiple apps:

1. **View all** - See interaction between apps
2. **Select one** - Focus on single app
3. **Switch between** - Compare behavior

### Performance Tips

**For 1000+ events:**
- Use window size of 50-100
- Apply filters to reduce noise
- Focus on one app at a time

**For real-time monitoring:**
- Keep window at end (auto-scrolls with new events)
- Use larger window size (200-500)
- Filter to relevant event types

## Troubleshooting

### Diagram Not Rendering

**Symptom:** Loading spinner doesn't go away

**Causes:**
1. Invalid Mermaid syntax (check browser console)
2. Too many events in window (reduce window size)
3. JavaScript error (check console)

**Fix:**
```javascript
// Open browser console (F12) and look for errors
// Try reducing window size to 20
// Click refresh button
```

### Events Not Appearing

**Symptom:** Stats show events but diagram is empty

**Causes:**
1. All events have `function_name: "unknown"` (need re-instrumentation)
2. Filter is too restrictive
3. Window is past the end of events

**Fix:**
```bash
# Re-instrument your code
./RE-INSTRUMENT.sh source/ instrumented/

# Check filter is set to "All Events"
# Click "Reset" to go to start of events
```

### Connection Issues

**Symptom:** Status shows "Disconnected"

**Causes:**
1. Visualizer server not running
2. Network issue
3. Server crashed

**Fix:**
```bash
# Restart visualizer
./start-visualizer.sh

# Refresh browser page
# Check server logs for errors
```

### Unknown Functions

**Symptom:** All functions show as "Unknown"

**Cause:** Code needs re-instrumentation (variables have hardcoded "unknown")

**Fix:**
```bash
./RE-INSTRUMENT.sh examples/python/telemetry_demo examples/python/instrumented/telemetry_demo
```

See: `docs/VISUALIZATION_FIXES.md` for details.

## Comparison with Other Views

### Mermaid Sequence (Default - /)
- **Best for:** Understanding execution flow
- **Shows:** Network → App → Function hierarchy
- **Features:** Real-time, windowing, filters
- **Use when:** Debugging, understanding control flow

### Network Graph (/graph)
- **Best for:** Seeing call relationships
- **Shows:** Function nodes with arrows between them
- **Features:** Physics layout, interactive zoom
- **Use when:** Analyzing architecture, finding bottlenecks

### Timeline (/timeline)
- **Best for:** Chronological event list
- **Shows:** All events in order
- **Features:** Event details, filtering
- **Use when:** Detailed debugging, event inspection

## Examples

### Simple Function Call

```
Network-->>main: invoke
    activate main
    main-->>add: call
        activate add
        Note over add: params: 5, 3
        Note over add: result = 8
        deactivate add
        Note over add: 0.05ms
    deactivate main
```

### Loop Execution

```
Network-->>fibonacci: invoke
    activate fibonacci
    Note over fibonacci: params: n=5
    rect rgb(245, 158, 11, 0.1)
    Note over fibonacci: 🔁 for loop start
    Note over fibonacci: sequence = []
    Note over fibonacci: sequence = [0]
    Note over fibonacci: sequence = [0, 1]
    Note over fibonacci: sequence = [0, 1, 1]
    Note over fibonacci: sequence = [0, 1, 1, 2]
    Note over fibonacci: sequence = [0, 1, 1, 2, 3]
    Note over fibonacci: 5 iterations
    end
    deactivate fibonacci
    Note over fibonacci: 2.45ms
```

### Multiple Function Calls

```
Network-->>main: invoke
    activate main
    main-->>load_data: call
        activate load_data
        load_data-->>fetch_api: call
            activate fetch_api
            deactivate fetch_api
        deactivate load_data
    main-->>process: call
        activate process
        process-->>clean_data: call
            activate clean_data
            deactivate clean_data
        process-->>transform: call
            activate transform
            deactivate transform
        deactivate process
    deactivate main
```

## Keyboard Shortcuts (Future)

Planned shortcuts:
- `Space` - Pause/resume updates
- `←` / `→` - Previous/next window
- `+` / `-` - Zoom in/out
- `F` - Fit to screen
- `R` - Refresh
- `C` - Clear

## API Integration

The visualizer exposes REST endpoints:

```bash
# Get all applications
curl http://localhost:8000/api/applications

# Get recent events
curl http://localhost:8000/api/events?limit=100

# Get events for specific app
curl http://localhost:8000/api/applications/calculator/events
```

## Configuration

### Mermaid Theme

Customize in dashboard HTML:
```javascript
mermaid.initialize({
    theme: 'dark',  // or 'default', 'forest', 'neutral'
    themeVariables: {
        primaryColor: '#667eea',  // Change colors
        // ... more variables
    }
});
```

### Window Size Default

Change default window size:
```html
<input type="number" id="window-size" value="100" min="10" max="500" step="10">
```

### Auto-Refresh Rate

Add auto-refresh (future):
```javascript
setInterval(() => {
    if (autoRefresh) {
        renderDiagram();
    }
}, 1000);  // Every second
```

## Best Practices

1. **Re-instrument regularly** - Ensure function names are correct
2. **Use filters** - Reduce noise for large apps
3. **Adjust window size** - Match your dataset size
4. **Monitor one app** - Focus for detailed analysis
5. **Check console** - Look for warnings/errors
6. **Use zoom** - Examine complex flows
7. **Export diagrams** - Take screenshots for documentation

## Future Enhancements

Planned features:
- Export as PNG/SVG
- Custom color schemes
- Collaborative viewing
- Historical playback
- Performance annotations
- Error highlighting
- Search functionality
- Bookmarks for interesting events

## Summary

The Mermaid.js visualizer provides:
- ✅ **Live sequence diagrams** showing execution flow
- ✅ **WebSocket updates** in real-time
- ✅ **Windowing/pagination** for large datasets
- ✅ **Zoom controls** for detailed inspection
- ✅ **Filtering** to reduce noise
- ✅ **Auto-selection** of first app
- ✅ **Multi-app support** with sidebar
- ✅ **Statistics** showing metrics

Access it at: **http://localhost:8000** (default view)

Alternative views:
- Network Graph: **http://localhost:8000/graph**
- Timeline: **http://localhost:8000/timeline**
