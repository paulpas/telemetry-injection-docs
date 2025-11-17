# Code Telemetry Injector - Operations Runbook

**Last Updated**: 2025-11-02
**Purpose**: Complete operational guide for running, configuring, and troubleshooting the system
**Audience**: DevOps, SREs, Developers

---

## Quick Reference

| Task | Command |
|------|---------|
| **Basic Usage** | `python telemetry-inject.py ./src --use-scripts -v` |
| **Setup** | `python telemetry-inject.py --configure` |
| **Dry Run** | `python telemetry-inject.py ./src --use-scripts --dry-run` |
| **With Budget** | `python telemetry-inject.py ./src --budget 10.00` |
| **Check API** | `python telemetry-inject.py --check-api` |
| **List Models** | `python telemetry-inject.py --list-models` |

---

## Table of Contents

1. [Installation](#installation)
2. [Configuration](#configuration)
3. [Running the Tool](#running-the-tool)
4. [Troubleshooting](#troubleshooting)
5. [Performance Optimization](#performance-optimization)
6. [CI/CD Integration](#cicd-integration)
7. [Monitoring](#monitoring)
8. [Common Scenarios](#common-scenarios)

---

## Installation

### Prerequisites

```bash
# Python 3.8+
python3 --version

# pip
pip --version
```

### Install Dependencies

```bash
# Clone repository (if not already done)
cd /path/to/claude-code-testing

# Create virtual environment
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### Verify Installation

```bash
# Test the CLI
python telemetry-inject.py --help

# Should output usage information
```

---

## Configuration

### Method 1: Interactive Configuration (Recommended)

```bash
python telemetry-inject.py --configure
```

**Walkthrough**:
1. Select provider (OpenAI, Anthropic, or Ollama)
2. Enter API key (if cloud provider)
3. Select model
4. Test connection
5. Save to .env file

### Method 2: Manual .env File

Create/edit `.env` in project root:

```bash
# Provider Configuration
LLM_PROVIDER=openai              # openai, anthropic, or ollama
OPENAI_API_KEY=sk-...            # Your OpenAI API key
# ANTHROPIC_API_KEY=sk-ant-...   # Or Anthropic API key

# Model Configuration
LLM_MODEL=gpt-4                  # Model name
LLM_BASE_URL=                    # Custom endpoint (for Ollama: http://localhost:11434/v1)
LLM_TIMEOUT=120                  # Timeout in seconds (600 for Ollama recommended)

# Feature Flags
DEBUG=false                      # Enable debug output
DEBUG_TRACE=false                # Enable detailed tracing
DEBUG_TRACE_LEVEL=INFO           # TRACE, DEBUG, INFO, WARNING, ERROR
ENABLE_LINTING=false             # Enable code linting

# Parallel Processing
MAX_PARALLEL=12                  # Max concurrent workers
```

### Method 3: Environment Variables

```bash
# Set for current session
export LLM_PROVIDER=openai
export OPENAI_API_KEY=sk-...
export LLM_MODEL=gpt-4

# Run
python telemetry-inject.py ./src --use-scripts -v
```

### Method 4: Command-Line Flags

```bash
# Override configuration
python telemetry-inject.py ./src \
  --model gpt-4 \
  --base-url http://localhost:11434/v1 \
  --use-scripts \
  -v
```

---

## Running the Tool

### Basic Usage

```bash
# Simplest command (uses .env configuration)
python telemetry-inject.py ./src

# With verbose output (recommended)
python telemetry-inject.py ./src -v

# Script-based mode (fastest, cacheable)
python telemetry-inject.py ./src --use-scripts -v
```

### Output Directory

```bash
# Default: ./src/instrumented
python telemetry-inject.py ./src

# Custom output directory
python telemetry-inject.py ./src -o ./output/instrumented

# Specify absolute path
python telemetry-inject.py ./src -o /tmp/instrumented
```

### Dry Run Mode

```bash
# Analyze without writing files
python telemetry-inject.py ./src --use-scripts --dry-run -v

# Shows what WOULD be done without modifying anything
```

---

## Troubleshooting

### Problem: "API key not configured"

**Symptom**:
```
❌ Error: No API key configured
```

**Solutions**:

1. **Check .env file**:
```bash
cat .env | grep API_KEY

# Should show:
# OPENAI_API_KEY=sk-...
```

2. **Run configuration wizard**:
```bash
python telemetry-inject.py --configure
```

3. **Set environment variable**:
```bash
export OPENAI_API_KEY=sk-...
python telemetry-inject.py ./src
```

---

### Problem: "Connection refused" (Ollama)

**Symptom**:
```
❌ Error: Connection refused to http://localhost:11434
```

**Solutions**:

1. **Check if Ollama is running**:
```bash
ollama list

# If error, start Ollama:
ollama serve
```

2. **Verify port**:
```bash
# Check Ollama port
curl http://localhost:11434/api/tags

# Should return JSON with models
```

3. **Update configuration**:
```bash
# If Ollama uses different port
export LLM_BASE_URL=http://localhost:CUSTOM_PORT/v1
```

---

### Problem: "Model not found"

**Symptom**:
```
❌ Error: Model 'gpt-5' not found
```

**Solutions**:

1. **List available models**:
```bash
python telemetry-inject.py --list-models

# Shows all models with pricing
```

2. **For Ollama, pull model**:
```bash
# Pull the model
ollama pull codellama

# Verify
ollama list

# Update configuration
export LLM_MODEL=codellama
```

3. **Check model name**:
```bash
# OpenAI models: gpt-4, gpt-3.5-turbo, gpt-4o
# Anthropic models: claude-3-5-sonnet-20241022, claude-3-opus-20240229
# Ollama models: codellama, mistral, llama3
```

---

### Problem: "Syntax error in instrumented code"

**Symptom**:
```
❌ Syntax error at line 42
```

**Solutions**:

1. **Check validation is enabled** (default):
```bash
python telemetry-inject.py ./src --validate -v
```

2. **Use script mode** (more reliable):
```bash
python telemetry-inject.py ./src --use-scripts -v
```

3. **Increase retry attempts**:
```bash
python telemetry-inject.py ./src --max-retries 5
```

4. **Enable debug trace**:
```bash
export DEBUG_TRACE=true
export DEBUG_TRACE_LEVEL=DEBUG
python telemetry-inject.py ./src -v

# Check logs in logs/debug_trace_*.jsonl
```

---

### Problem: "Budget exceeded"

**Symptom**:
```
❌ Budget limit exceeded: $10.50 / $10.00
```

**Solutions**:

1. **Increase budget**:
```bash
python telemetry-inject.py ./src --budget 20.00
```

2. **Use script mode** (cheaper):
```bash
# First run: minimal LLM usage
# Cached runs: $0
python telemetry-inject.py ./src --use-scripts --budget 10.00
```

3. **Use Ollama** (free):
```bash
export LLM_PROVIDER=ollama
export LLM_BASE_URL=http://localhost:11434/v1
export LLM_MODEL=codellama

# No budget needed!
python telemetry-inject.py ./src --use-scripts
```

---

### Problem: "Timeout waiting for LLM response"

**Symptom**:
```
❌ Timeout after 120 seconds
```

**Solutions**:

1. **Increase timeout**:
```bash
export LLM_TIMEOUT=600  # 10 minutes

python telemetry-inject.py ./src -v
```

2. **For Ollama, use smaller model**:
```bash
# Instead of mixtral (slow, 47B params):
ollama pull codellama  # Faster, 7B params

export LLM_MODEL=codellama
```

3. **Check network connection** (for cloud APIs):
```bash
# Test API connectivity
python telemetry-inject.py --check-api
```

---

### Problem: "No functions found"

**Symptom**:
```
⚠️ No functions found in file.py
```

**Solutions**:

1. **Verify file has functions**:
```bash
# Check file content
head -20 file.py

# Should have function definitions like:
# def my_function():
```

2. **Check language support**:
```bash
# Supported languages: Python, JavaScript, TypeScript, Go
# File must have correct extension: .py, .js, .ts, .go
```

3. **Enable debug trace**:
```bash
export DEBUG_TRACE=true
python telemetry-inject.py ./src -v

# Check logs for parser errors
```

---

## Performance Optimization

### Optimization 1: Use Script Mode

**Command**:
```bash
python telemetry-inject.py ./src --use-scripts -v
```

**Benefits**:
- First run: 7ms per function (template-based)
- Cached run: 0.3ms per function (**98.7% faster**)
- Cost: $0 on cache hits

---

### Optimization 2: Use Tree-Sitter Languages

**Supported**:
- Python (✅)
- JavaScript (✅)
- TypeScript (✅)
- Go (✅)

**Benefit**: 0 LLM calls for analysis (10-100x faster, $0 cost)

---

### Optimization 3: Use Local Ollama

**Setup**:
```bash
# Install Ollama
brew install ollama  # Mac
# or download from ollama.ai

# Pull model
ollama pull codellama

# Configure
export LLM_PROVIDER=ollama
export LLM_BASE_URL=http://localhost:11434/v1
export LLM_MODEL=codellama

# Run
python telemetry-inject.py ./src --use-scripts -v
```

**Benefits**:
- **$0 cost** (runs locally)
- No API rate limits
- Private (code never leaves machine)

---

### Optimization 4: Parallel Processing

**Command**:
```bash
# Default: 12 workers
python telemetry-inject.py ./src --use-scripts -v

# Custom workers
python telemetry-inject.py ./src --use-scripts --max-parallel 24 -v
```

**Benefit**: ~12x speedup for large codebases

---

### Performance Comparison

| Configuration | 100 Functions | Cost |
|---------------|---------------|------|
| Traditional + OpenAI | 500-700s | $10-25 |
| Script + OpenAI (cached) | 0.03s | $0 |
| Script + Ollama (cached) | 0.03s | $0 |

---

## CI/CD Integration

### GitHub Actions

```yaml
name: Instrument Code

on: [push]

jobs:
  instrument:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.10'

    - name: Install dependencies
      run: |
        pip install -r requirements.txt

    - name: Configure API
      env:
        OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
      run: |
        echo "LLM_PROVIDER=openai" > .env
        echo "OPENAI_API_KEY=$OPENAI_API_KEY" >> .env
        echo "LLM_MODEL=gpt-4" >> .env

    - name: Instrument code
      run: |
        python telemetry-inject.py ./src \
          --use-scripts \
          --validate \
          --budget 5.00 \
          -v

    - name: Upload instrumented code
      uses: actions/upload-artifact@v3
      with:
        name: instrumented
        path: ./src/instrumented
```

---

### GitLab CI

```yaml
instrument:
  stage: build
  image: python:3.10
  script:
    - pip install -r requirements.txt
    - |
      echo "LLM_PROVIDER=openai" > .env
      echo "OPENAI_API_KEY=$OPENAI_API_KEY" >> .env
      echo "LLM_MODEL=gpt-4" >> .env
    - python telemetry-inject.py ./src --use-scripts --budget 5.00 -v
  artifacts:
    paths:
      - ./src/instrumented
  only:
    - main
```

---

### Jenkins

```groovy
pipeline {
    agent any

    stages {
        stage('Setup') {
            steps {
                sh 'pip install -r requirements.txt'
                sh '''
                    echo "LLM_PROVIDER=openai" > .env
                    echo "OPENAI_API_KEY=${OPENAI_API_KEY}" >> .env
                    echo "LLM_MODEL=gpt-4" >> .env
                '''
            }
        }

        stage('Instrument') {
            steps {
                sh '''
                    python telemetry-inject.py ./src \
                        --use-scripts \
                        --validate \
                        --budget 5.00 \
                        -v
                '''
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'src/instrumented/**', fingerprint: true
            }
        }
    }
}
```

---

## Monitoring

### Cost Tracking

```bash
# Set budget limit
python telemetry-inject.py ./src --budget 10.00 -v

# Output shows real-time cost:
# 💰 Cost so far: $2.45 / $10.00
# ✓ File 1/10 processed
# 💰 Cost so far: $4.82 / $10.00
# ✓ File 2/10 processed
# ...
# ✅ Processing complete. Total cost: $8.73
```

---

### Debug Trace Logging

```bash
# Enable detailed logging
export DEBUG_TRACE=true
export DEBUG_TRACE_LEVEL=INFO

python telemetry-inject.py ./src --use-scripts -v

# Logs written to: logs/debug_trace_YYYYMMDD_HHMMSS.jsonl
```

**Analyze logs**:
```bash
# View all LLM requests
cat logs/debug_trace_*.jsonl | jq 'select(.category == "llm_request")'

# Total cost
cat logs/debug_trace_*.jsonl | jq 'select(.category == "llm_response") | .data.cost_usd' | sed 's/\$//g' | paste -sd+ | bc

# Average response time
cat logs/debug_trace_*.jsonl | jq 'select(.category == "llm_response") | .data.duration_s' | jq -s 'add / length'
```

---

### Cache Statistics

```bash
# View cache size
ls -lh .telemetry_cache/scripts/python/ | wc -l

# Cache index
cat .telemetry_cache/metadata/cache_index.json | jq 'length'
# Output: 142 (142 cached scripts)

# Statistics by language
cat .telemetry_cache/metadata/cache_index.json | \
    jq 'group_by(.language) | map({language: .[0].language, count: length})'
```

---

## Common Scenarios

### Scenario 1: First-Time Setup

```bash
# 1. Install dependencies
pip install -r requirements.txt

# 2. Run configuration wizard
python telemetry-inject.py --configure

# 3. Test on example
python telemetry-inject.py examples/sample_project --use-scripts --dry-run -v

# 4. Run on your code
python telemetry-inject.py ./src --use-scripts -v
```

---

### Scenario 2: Local Development (Free)

```bash
# 1. Install Ollama
ollama pull codellama

# 2. Configure
cat > .env << EOF
LLM_PROVIDER=ollama
LLM_BASE_URL=http://localhost:11434/v1
LLM_MODEL=codellama
LLM_TIMEOUT=600
EOF

# 3. Run
python telemetry-inject.py ./src --use-scripts -v
```

---

### Scenario 3: Production Deployment

```bash
# 1. Set budget limit
export OPENAI_API_KEY=sk-...
export LLM_MODEL=gpt-4

# 2. Run with safeguards
python telemetry-inject.py ./src \
  --use-scripts \
  --validate \
  --budget 10.00 \
  -v

# 3. Verify output
ls -lh ./src/instrumented/
```

---

### Scenario 4: Large Codebase (1000+ files)

```bash
# 1. Use script mode + parallel processing
python telemetry-inject.py ./large_project \
  --use-scripts \
  --max-parallel 24 \
  --budget 50.00 \
  -v

# 2. First run: ~1-5 minutes, $0-10
# 3. Cached runs: ~10 seconds, $0

# 4. Monitor progress
tail -f logs/debug_trace_*.jsonl
```

---

## Quick Diagnostic Checklist

When something goes wrong:

1. ✅ Check .env file exists and has API key
2. ✅ Verify API connectivity: `python telemetry-inject.py --check-api`
3. ✅ Test on example: `python telemetry-inject.py examples/sample_project --dry-run -v`
4. ✅ Enable debug trace: `export DEBUG_TRACE=true`
5. ✅ Check logs: `cat logs/debug_trace_*.jsonl | jq`
6. ✅ Verify model exists: `python telemetry-inject.py --list-models`
7. ✅ Try Ollama (free): `ollama pull codellama && export LLM_PROVIDER=ollama`

---

## Support Resources

- **Architecture**: See `docs/ARCHITECTURE_REFACTORED.md`
- **AI Usage**: See `docs/AI_USAGE_DETAILED.md`
- **Quick Reference**: See `docs/QUICK_REFERENCE.md`
- **Examples**: See `examples/` directory

---

**Last Updated**: 2025-11-02
**Maintained By**: DevOps Team
**Next Review**: Quarterly
