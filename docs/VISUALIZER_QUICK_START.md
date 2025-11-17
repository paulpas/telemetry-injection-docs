# Telemetry Visualizer - Quick Start Guide

## Related Documentation

- **[Telemetry Viewing Guide](../TELEMETRY_VIEWING_GUIDE.md)** - Complete guide to viewing and understanding telemetry
- **[Program Name Field](../PROGRAM_NAME_FIELD.md)** - Identifying which program generated telemetry
- **[Telemetry Visualizer](TELEMETRY_VISUALIZER.md)** - Full visualizer documentation
- **[Enhanced Telemetry](ENHANCED_TELEMETRY.md)** - Correlation IDs and sequence numbers
- **[Mermaid Visualizer](MERMAID_VISUALIZER.md)** - Sequence diagram visualization
- **[Main README](../README.md)** - Project overview

---

## Installation

```bash
# Install dependencies
pip install fastapi uvicorn websockets jinja2
```

## Starting the Visualizer

### Option 1: Auto-detect Port (Recommended)
```bash
./start-visualizer.sh
```
Output:
```
🔍 Searching for available port (starting from 8000)...
✓ Found available port: 8000

==========================================================
  🔬 Telemetry Visualization Server
==========================================================

Starting server on http://localhost:8000
```

### Option 2: Specify a Port
```bash
# Using bash script
./start-visualizer.sh 8888
./start-visualizer.sh --port 9000

# Using Python directly
python -m src.telemetry_visualizer --port 8888
python -m src.telemetry_visualizer -p 9000
```

### Option 3: Custom Port Range
```bash
python -m src.telemetry_visualizer --min-port 9000
# Searches for available port starting from 9000
```

## Running Instrumented Applications

Once the visualizer is running, note the port it's using (e.g., 8000).

Then run your instrumented application:

```bash
# Set environment variables (use the actual port from above)
export DEBUG=true
export RECEIVER_URL=http://localhost:8000/telemetry

# Run your instrumented application
python /path/to/instrumented/app.py
```

## Opening the Dashboard

Open your browser to the URL shown by the visualizer:
```
http://localhost:8000
```

## Command-Line Options

```bash
python -m src.telemetry_visualizer --help

Options:
  -p, --port PORT        Port to run server on (default: auto-detect)
  --host HOST           Host to bind to (default: 0.0.0.0)
  --min-port MIN_PORT   Minimum port for auto-detection (default: 8000)
  -h, --help            Show help message
```

## Examples

### Run on specific port
```bash
./start-visualizer.sh 8888
```

### Run multiple visualizers (different ports)
```bash
# Terminal 1 - Production apps
./start-visualizer.sh 8000

# Terminal 2 - Development apps
./start-visualizer.sh 9000

# Terminal 3 - Testing apps
./start-visualizer.sh 10000
```

### Auto-detect with custom minimum
```bash
python -m src.telemetry_visualizer --min-port 9000
# Finds available port >= 9000
```

### Bind to specific network interface
```bash
python -m src.telemetry_visualizer --port 8888 --host 192.168.1.100
# Accessible at http://192.168.1.100:8888
```

## Troubleshooting

### Port Already in Use
```bash
./start-visualizer.sh 8888
# Output: ❌ Error: Port 8888 is already in use!
#         Try a different port or let it auto-detect.

# Solution: Let it auto-detect
./start-visualizer.sh
```

### Can't Find Available Port
```bash
python -m src.telemetry_visualizer --min-port 8000
# Output: ❌ Error: Could not find available port above 8000
#         Try specifying a port manually with --port

# Solution: Try higher port range
python -m src.telemetry_visualizer --min-port 10000
```

### Connection Refused from Instrumented App
```bash
# Make sure:
1. Visualizer is running (check terminal output)
2. Port matches in RECEIVER_URL
3. No firewall blocking the port

# Test connection:
curl http://localhost:8000/api/applications
# Should return: []
```

## Integration Examples

### With Docker
```dockerfile
# Start visualizer in container
docker run -p 8000:8000 myapp python -m src.telemetry_visualizer --port 8000

# Connect from host
export RECEIVER_URL=http://localhost:8000/telemetry
```

### With Docker Compose
```yaml
services:
  visualizer:
    command: python -m src.telemetry_visualizer --port 8000
    ports:
      - "8000:8000"

  app:
    environment:
      - DEBUG=true
      - RECEIVER_URL=http://visualizer:8000/telemetry
    depends_on:
      - visualizer
```

### With systemd
```ini
[Unit]
Description=Telemetry Visualizer
After=network.target

[Service]
Type=simple
User=youruser
WorkingDirectory=/path/to/project
ExecStart=/path/to/venv/bin/python -m src.telemetry_visualizer --port 8000
Restart=always

[Install]
WantedBy=multi-user.target
```

## Tips

1. **Always use auto-detect** for development - avoids conflicts
2. **Specify port** for production - predictable URLs
3. **Use different ports** for different environments (dev/staging/prod)
4. **Check the output** - visualizer shows the actual port and URLs
5. **Save the port** - note which port is being used for your RECEIVER_URL

## Next Steps

- Read full documentation: `docs/TELEMETRY_VISUALIZER.md`
- Try the demo: `python examples/visualizer_demo.py`
- Instrument your code: `python telemetry-inject.py /path/to/code`
