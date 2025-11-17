# Visual Flow Diagram - Interactive Telemetry Visualization

The Telemetry Visualizer now includes an **interactive visual flow diagram** that shows your application's execution as a graph with:
- **Functions as rectangles** with color coding
- **Arrows showing data flow** between functions
- **Interactive zooming and panning**
- **Click-to-explore** function details

## Features

### 🎨 Visual Representation

**Shapes:**
- **Rectangles** = Functions
- **Color-coded** by event type:
  - 🟢 Green = Function Entry
  - 🔵 Blue = Function Exit
  - 🟣 Purple = Variable Changes
  - 🟠 Orange = Loops

**Lines:**
- **Arrows** show function calls (who calls whom)
- **Labels** on arrows show the relationship
- **Curved lines** for better readability

### 🔍 Interactive Features

**Click on Application:**
- Filters the graph to show only that app's functions
- Zooms in to focus on the selected app
- Shows function relationships specific to that app

**Click on Function Node:**
- Shows detailed information about the function
- Displays all events for that function
- Highlights connected functions

**Zoom & Pan:**
- Mouse wheel to zoom in/out
- Click and drag to pan around
- Fit view button to reset

**Hover:**
- Hover over nodes to see tooltips
- Hover over edges to see relationship details

### 📊 View Modes

**Flow Diagram (Default):**
- Visual graph showing function relationships
- Interactive nodes and edges
- Real-time updates as events arrive

**Timeline:**
- Classic event list view
- Chronological order
- Detailed event information

Switch between views using the toggle buttons at the top.

## How It Works

### Data Flow Visualization

```
┌──────────────┐
│    main()    │
│   (caller)   │
└──────┬───────┘
       │ calls
       ▼
┌──────────────┐
│ calculate()  │
│   (called)   │
└──────────────┘
```

The visualizer automatically:
1. **Extracts caller information** from telemetry events
2. **Creates nodes** for each function
3. **Draws edges** showing function calls
4. **Updates in real-time** as your app runs

### Color Coding Logic

Events are analyzed and functions are colored based on their most recent activity:

| Color | Event Type | Meaning |
|-------|-----------|---------|
| 🟢 Green | `function_entry` | Function was just called |
| 🔵 Blue | `function_exit` | Function just returned |
| 🟣 Purple | `variable_change` | Variable was modified |
| 🟠 Orange | `loop_entry/exit` | Loop activity |

## Usage Guide

### 1. Start the Visualizer

```bash
./start-visualizer.sh
```

Output:
```
✓ Found available port: 8000
Starting server on http://localhost:8000
```

### 2. Run Your Instrumented App

```bash
# Make sure your code has the updated telemetry utils
./fix-telemetry-http.sh /path/to/instrumented/code

# Run with telemetry enabled
export DEBUG=true
export RECEIVER_URL=http://localhost:8000/telemetry
python /path/to/instrumented/app.py
```

### 3. Open the Visual Dashboard

Navigate to: **http://localhost:8000**

You'll see:
- **Left sidebar** with list of applications
- **Main area** with the visual flow diagram
- **Legend** in the top-right corner
- **Stats** at the bottom

### 4. Interact with the Graph

**Explore Functions:**
1. Click on an application in the left sidebar
2. Graph zooms to show that app's functions
3. See arrows showing function call relationships

**Get Details:**
1. Click on any function rectangle
2. See popup with function details
3. View event count and metadata

**Navigate:**
- Scroll to zoom in/out
- Click and drag to pan
- Double-click to reset view

## Troubleshooting

### Functions Show as "unknown"

**Problem:** Nodes labeled "unknown" appear in the graph.

**Solution:** Your telemetry utils are outdated. Fix with:
```bash
./fix-telemetry-http.sh /path/to/instrumented/code
```

This updates the caller extraction logic to properly identify functions.

### No Arrows Between Functions

**Problem:** Nodes appear but no arrows connecting them.

**Possible causes:**
1. **Caller information missing** - Telemetry utils need updating
2. **Single-function app** - Only one function, so no calls to show
3. **Events not processed** - Check if events are arriving (look at event count)

**Solution:**
```bash
# Update telemetry utils
./check-telemetry-http.sh /path/to/instrumented/code

# Re-instrument if needed
python telemetry-inject.py /path/to/source/code -l python
```

### Graph is Cluttered

**Problem:** Too many nodes, hard to see relationships.

**Solutions:**
1. **Click on a specific app** to filter the view
2. **Zoom in** on areas of interest
3. **Switch to Timeline view** for chronological events
4. **Reduce instrumentation** - instrument fewer functions

### Nodes Overlap

**Problem:** Function rectangles overlap and are hard to read.

**Solution:** The graph uses physics simulation to spread nodes apart. Give it a few seconds to stabilize. You can also:
- Zoom in for better spacing
- Click and drag nodes to rearrange manually
- Click "Fit View" to re-layout

## Advanced Features

### Hierarchical Layout

For applications with clear call hierarchies, the graph can be switched to hierarchical mode (edit in dashboard_visual.html):

```javascript
layout: {
    hierarchical: {
        enabled: true,
        direction: 'UD',  // Up-Down
        sortMethod: 'directed'
    }
}
```

### Custom Node Colors

You can customize colors in the dashboard HTML. Look for:
```javascript
if (event.event_type === 'function_entry') {
    color = '#10b981';  // Change this
    borderColor = '#059669';
}
```

### Export Graph

To export the current graph view (add this feature):
```javascript
// Take screenshot
network.once('afterDrawing', function() {
    const canvas = network.body.container.getElementsByTagName('canvas')[0];
    canvas.toBlob(function(blob) {
        saveAs(blob, 'telemetry-graph.png');
    });
});
```

## Examples

### Simple Calculator App

```python
def add(a, b):
    return a + b

def multiply(a, b):
    return a * b

def calculate():
    x = add(5, 3)
    y = multiply(x, 2)
    return y
```

**Visual Result:**
```
    calculate()
       ↓        ↓
    add()   multiply()
```

### Complex Application

```python
def main():
    data = load_data()
    result = process(data)
    save(result)

def load_data():
    return fetch_from_api()

def process(data):
    clean = clean_data(data)
    return transform(clean)
```

**Visual Result:**
```
        main()
        ↓  ↓  ↓
    load  process  save
      ↓      ↓  ↓
   fetch  clean transform
```

## Performance

**Scalability:**
- Handles up to **1000 nodes** smoothly
- Physics simulation can be disabled for >500 nodes
- Filtering by app keeps graphs manageable

**Real-time Updates:**
- Nodes and edges added as events arrive
- Graph re-layouts automatically
- No performance impact on instrumented apps

## Keyboard Shortcuts

Add these to your dashboard for power users:

| Key | Action |
|-----|--------|
| `Space` | Toggle physics on/off |
| `F` | Fit view to screen |
| `+` / `-` | Zoom in/out |
| `Arrow keys` | Pan view |

## Integration with CI/CD

The visual flow diagram is perfect for:
- **Debugging** - See execution flow visually
- **Performance analysis** - Identify hot paths
- **Architecture review** - Understand system structure
- **Documentation** - Auto-generated call graphs

Export graphs for reports or documentation.

## Next Steps

1. **Try it out** - Start the visualizer and run an instrumented app
2. **Click around** - Explore the interactive features
3. **Filter by app** - See focused views of individual applications
4. **Compare modes** - Switch between Flow Diagram and Timeline views

For the classic timeline view, go to: **http://localhost:8000/timeline**

## FAQ

**Q: Can I have both views open at once?**
A: Yes! Open multiple browser tabs to the same visualizer URL. Each shows real-time updates.

**Q: Does this work with JavaScript apps?**
A: Yes! The visualizer is language-agnostic. Instrument your JS/TS code and it will appear in the graph.

**Q: Can I export the graph?**
A: Currently you can screenshot. A proper export feature is planned.

**Q: How do I reset the graph?**
A: Refresh the page or click the Fit View button.

**Q: Why do some functions not connect?**
A: They might not have caller information. Make sure you've updated your telemetry utils with `./fix-telemetry-http.sh`.
