# Telemetry Visualizer - Real-time Monitoring Dashboard

A self-hosted web dashboard for real-time visualization of telemetry data from instrumented applications.

## Related Documentation

- **[Visualizer Quick Start](VISUALIZER_QUICK_START.md)** - Get started quickly
- **[Telemetry Viewing Guide](../TELEMETRY_VIEWING_GUIDE.md)** - View and analyze raw telemetry
- **[Program Name Field](../PROGRAM_NAME_FIELD.md)** - Identify programs in telemetry
- **[Enhanced Telemetry](ENHANCED_TELEMETRY.md)** - Correlation IDs and relationships
- **[Mermaid Visualizer](MERMAID_VISUALIZER.md)** - Sequence diagram view
- **[Visual Flow Diagram](VISUAL_FLOW_DIAGRAM.md)** - Flow diagram visualization
- **[Main README](../README.md)** - Project overview

---

## Features

- **Real-time Monitoring**: WebSocket-based live updates as telemetry events occur
- **Multi-Application Support**: Monitor multiple applications simultaneously
- **Interactive Visualization**:
  - Timeline view of all events
  - Function call traces with timing information
  - Variable changes tracking
  - Loop iteration monitoring
- **Application Management**:
  - Left sidebar with all active/inactive applications
  - Click to filter events by application
  - Status indicators showing active functions
- **Event Filtering**: Filter by event type (functions, loops, variables)
- **Performance Stats**: Real-time event count and event rate tracking

## Quick Start

### 1. Install Dependencies

```bash
# Install FastAPI and uvicorn
pip install fastapi uvicorn websockets
```

### 2. Start the Visualizer

```bash
# Auto-detect available port (starts from 8000)
./start-visualizer.sh

# Or specify a port
./start-visualizer.sh 8888
./start-visualizer.sh --port 9000

# Or run Python directly
python -m src.telemetry_visualizer              # Auto-detect
python -m src.telemetry_visualizer --port 8888  # Specific port
```

The server will auto-detect an available port above 7999, or use the port you specify.
The actual port will be displayed when the server starts.

### 3. Configure Your Instrumented Application

Set these environment variables before running your instrumented code:

```bash
export DEBUG=true
export RECEIVER_URL=http://localhost:8888/telemetry
```

### 4. Run Your Instrumented Application

```bash
# Run your instrumented application
python /path/to/instrumented/app.py
```

### 5. Open the Dashboard

Open your browser to: **http://localhost:8888**

You'll see events appear in real-time as your application runs!

## Usage Guide

### Dashboard Layout

```
┌────────────────────────────────────────────────────────┐
│  Telemetry Monitor                                     │
│  ● Connected                                           │
├────────────────────┬───────────────────────────────────┤
│                    │  All Applications                 │
│  Application List  │  [All] [Functions] [Loops] [Vars]│
│                    ├───────────────────────────────────┤
│  ● app1 (active)   │                                   │
│    15 events       │  Timeline View:                   │
│    • calculate     │  ├─ function_entry: calculate()   │
│    • process       │  ├─ variable_change: x = 10       │
│                    │  ├─ loop_entry: for               │
│  ○ app2 (inactive) │  ├─ loop_iteration                │
│    8 events        │  └─ function_exit: 2.5ms          │
│                    │                                   │
│                    │                                   │
├────────────────────┴───────────────────────────────────┤
│  0 Apps  │  23 Events  │  5 Events/sec                │
└────────────────────────────────────────────────────────┘
```

### Filtering Events

**By Application:**
- Click an application in the left sidebar to show only its events
- Click again to deselect and show all applications

**By Event Type:**
- **All Events**: Show everything (default)
- **Functions**: Show only function entry/exit events
- **Loops**: Show only loop events
- **Variables**: Show only variable change events

### Event Information

Each event card shows:
- **Event Type Badge**: Color-coded badge (green=entry, blue=exit, orange=loop, purple=variable)
- **Application Name**: Which app generated the event
- **Timestamp**: When the event occurred
- **Event Details**:
  - Functions: Name, parameters, duration, return value, location
  - Loops: Type, iteration count
  - Variables: Name, value, location

### Real-time Updates

- Events appear immediately via WebSocket
- Active functions are highlighted in the application list
- Applications automatically marked as inactive after 5 seconds of no events
- Connection status shown in top-left (green dot = connected)

## Architecture

### Components

1. **FastAPI Server** (`src/telemetry_visualizer.py`):
   - Receives telemetry via POST to `/telemetry`
   - Manages WebSocket connections
   - Stores recent events in memory
   - Broadcasts events to all connected clients

2. **Web Dashboard** (`templates/dashboard.html`):
   - Single-page application with real-time updates
   - WebSocket client for live event streaming
   - Interactive UI with filtering and selection

3. **Telemetry Utils** (`_telemetry_utils.py` in instrumented code):
   - Sends events to stderr (for local debugging)
   - Posts events to RECEIVER_URL (for visualization)
   - Non-blocking with short timeout (no performance impact)

### Data Flow

```
┌─────────────────────┐
│  Instrumented App   │
│  ┌───────────────┐  │
│  │ _telemetry_   │  │
│  │    utils      │  │
│  └───────┬───────┘  │
└──────────┼──────────┘
           │
           │ HTTP POST /telemetry
           │ (non-blocking, 0.5s timeout)
           ▼
    ┌──────────────────┐
    │  Visualizer      │
    │  Server :8888    │
    │                  │
    │  ┌────────────┐  │
    │  │ Collector  │  │
    │  │  Storage   │  │
    │  └──────┬─────┘  │
    └─────────┼────────┘
              │
              │ WebSocket
              │ (real-time)
              ▼
    ┌──────────────────┐
    │  Web Dashboard   │
    │  (Browser)       │
    │                  │
    │  ┌────────────┐  │
    │  │   UI       │  │
    │  │ Timeline   │  │
    │  └────────────┘  │
    └──────────────────┘
```

## API Reference

### POST /telemetry

Receive telemetry events from instrumented applications.

**Request Body** (JSON):
```json
{
  "event_type": "function_entry",
  "function_name": "calculate",
  "parameters": "a, b",
  "correlation_id": "func_abc123",
  "caller_function": "main",
  "caller_file": "/path/to/app.py",
  "caller_line": 42,
  "timestamp_ns": 1234567890000000
}
```

**Response**:
```json
{
  "status": "ok"
}
```

### GET /api/applications

Get list of all monitored applications.

**Response**:
```json
[
  {
    "app_id": "calculator",
    "name": "calculator",
    "first_seen": 1234567890.0,
    "last_seen": 1234567895.0,
    "event_count": 15,
    "active": true,
    "active_functions": ["calculate", "process"]
  }
]
```

### GET /api/applications/{app_id}/events

Get recent events for a specific application.

**Query Parameters**:
- `limit`: Max events to return (default: 100)

**Response**: Array of event objects

### GET /api/events

Get recent events across all applications.

**Query Parameters**:
- `limit`: Max events to return (default: 100)

**Response**: Array of event objects

### WebSocket /ws

Real-time event streaming.

**Messages from Server**:

1. **Initial State**:
```json
{
  "type": "init",
  "applications": [...],
  "recent_events": [...]
}
```

2. **New Event**:
```json
{
  "type": "telemetry_event",
  "data": { ... event object ... }
}
```

## Configuration

### Port Configuration

The visualizer supports flexible port configuration:

**Auto-detection (default):**
```bash
./start-visualizer.sh
# Searches for available port starting from 8000
# Output: "✓ Found available port: 8000"
```

**Specify port:**
```bash
./start-visualizer.sh 8888                    # Using positional argument
./start-visualizer.sh --port 9000             # Using flag
python -m src.telemetry_visualizer -p 8888    # Direct Python invocation
```

**Custom minimum port:**
```bash
python -m src.telemetry_visualizer --min-port 9000
# Auto-detects starting from port 9000
```

**Check available ports:**
```bash
# The visualizer will tell you if a port is already in use
./start-visualizer.sh 8888
# Output: "❌ Error: Port 8888 is already in use!"
```

### Environment Variables

**For the Visualizer Server:**
- None required - auto-detects available port above 7999 (typically 8000)

**For Instrumented Applications:**
- `DEBUG=true` - Enable telemetry output
- `RECEIVER_URL=http://localhost:8888/telemetry` - Send events to visualizer

### Performance Considerations

- **Non-blocking**: HTTP POST uses 0.5s timeout, won't slow down your app
- **Efficient**: Events sent asynchronously, no waiting for response
- **Graceful degradation**: Network errors are silently ignored
- **Memory limits**: Dashboard stores last 1000 events per application

### Network Configuration

If running visualizer on a different machine:

```bash
# On machine A (visualizer):
python src/telemetry_visualizer.py

# On machine B (instrumented app):
export RECEIVER_URL=http://machine-a:8888/telemetry
export DEBUG=true
python instrumented_app.py
```

## Advanced Usage

### Monitoring Multiple Applications

The visualizer automatically detects and groups events by application:

```bash
# Terminal 1: Start visualizer
python src/telemetry_visualizer.py

# Terminal 2: Run app 1
export DEBUG=true RECEIVER_URL=http://localhost:8888/telemetry
python instrumented/app1.py

# Terminal 3: Run app 2
export DEBUG=true RECEIVER_URL=http://localhost:8888/telemetry
python instrumented/app2.py

# Both apps appear in the dashboard sidebar!
```

### Custom Application Names

The visualizer uses the filename as the app ID by default. To customize:

```python
# In your instrumented code, you can modify _telemetry_utils.py
# to add an "app_id" field to events
```

### Integration with CI/CD

Run the visualizer in test environments:

```yaml
# Example GitHub Actions workflow
steps:
  - name: Start Telemetry Visualizer
    run: |
      python src/telemetry_visualizer.py &
      sleep 2

  - name: Run Instrumented Tests
    env:
      DEBUG: true
      RECEIVER_URL: http://localhost:8888/telemetry
    run: |
      pytest instrumented/tests/
```

## Troubleshooting

### Events Not Appearing

**Check:**
1. Is `DEBUG=true` set?
2. Is `RECEIVER_URL` set correctly?
3. Is the visualizer running on port 8888?
4. Can the app reach the visualizer? (try `curl http://localhost:8888/api/applications`)

**Test Connection:**
```bash
curl -X POST http://localhost:8888/telemetry \
  -H "Content-Type: application/json" \
  -d '{"event_type": "test", "timestamp_ns": 123}'
```

### WebSocket Disconnects

- Check browser console for errors
- Ensure no proxy/firewall blocking WebSocket connections
- Refresh the page to reconnect

### High Event Rate

If you're generating thousands of events per second:

1. **Filter Events**: Use the event type filters to reduce visual noise
2. **Adjust Limits**: Modify `max_events_per_app` in `TelemetryCollector.__init__()`
3. **Sample Events**: Modify instrumented code to emit fewer events

### Performance Impact

The visualizer is designed to have minimal impact:
- HTTP POST uses 0.5s timeout (very short)
- Network errors are silently ignored
- No retry logic
- Events sent in background

If you still see performance issues:
- Disable `RECEIVER_URL` and use only stderr output
- Reduce instrumentation density in your code

## Examples

See `/examples` directory for complete working examples:
- `examples/python/instrumented/` - Instrumented Python calculator demo
- Run with visualizer to see live telemetry

## Development

### Running from Source

```bash
# Install development dependencies
pip install fastapi uvicorn websockets jinja2

# Start with auto-reload
uvicorn src.telemetry_visualizer:app --reload --port 8888
```

### Modifying the Dashboard

The dashboard is a single HTML file at `templates/dashboard.html`.

Edit the HTML/CSS/JavaScript directly and refresh your browser.

## License

Part of the Code Telemetry Injector project.
