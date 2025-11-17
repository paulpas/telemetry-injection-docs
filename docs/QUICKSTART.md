# Quick Start Guide

Get started with the Code Telemetry Injector in 5 minutes!

## Prerequisites

- Python 3.8 or higher
- OpenAI API key ([Get one here](https://platform.openai.com/api-keys))

## Installation

1. **Install dependencies:**
   ```bash
   python3 -m venv venv
   source venv/bin/activate  # Windows: venv\Scripts\activate
   pip install -r requirements.txt
   ```

2. **Set up API key:**
   ```bash
   cp .env.example .env
   # Edit .env and add: OPENAI_API_KEY=your-key-here
   ```

## Quick Test

Run the test suite to verify installation:

```bash
pytest tests/ -v
```

Expected: **44 tests passing**

## First Run

Try it on the example project:

```bash
# Basic usage
python -m src.cli examples/sample_project

# With verbose output
python -m src.cli examples/sample_project -v

# Dry run (no files written)
python -m src.cli examples/sample_project --dry-run -v
```

## View Results

After running, check the instrumented code:

```bash
# View instrumented Python
cat examples/sample_project/instrumented/calculator.py

# Run it to see telemetry output
python examples/sample_project/instrumented/calculator.py
```

You should see output like:
```
[TELEMETRY] Function 'add' started with inputs: {'a': 5, 'b': 3}
[TELEMETRY] Function 'add' took 0.000156 seconds
Add: 8
```

## Use On Your Code

```bash
python -m src.cli /path/to/your/project -v
```

The instrumented code will be in `/path/to/your/project/instrumented/`

## Common Options

```bash
# Process only Python files
python -m src.cli myproject -e .py

# Custom output directory
python -m src.cli myproject -o output/

# Process multiple file types
python -m src.cli myproject -e .py,.js,.go
```

## Troubleshooting

**API Key Error?**
- Check `.env` file has `OPENAI_API_KEY=your-key`
- Or use: `python -m src.cli myproject --api-key YOUR_KEY`

**No files found?**
- Check the directory path is correct
- Use `-e` flag to specify extensions: `-e .py,.js`
- Ensure files aren't in ignored directories (node_modules, venv, etc.)

**Tests failing?**
- Ensure dependencies installed: `pip install -r requirements.txt`
- Activate virtual environment: `source venv/bin/activate`

## Next Steps

- Read the full [README.md](README.md) for detailed documentation
- Check [examples/README.md](examples/README.md) for more examples
- Review generated code to understand telemetry patterns
- Customize for your needs by modifying prompts in `src/`

## Getting Help

- Check existing tests for usage examples
- Review the modular architecture in `src/`
- All components are independently testable and extensible

---

Happy instrumenting! 🔧
