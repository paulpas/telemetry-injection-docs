# LLM Provider Switching Guide

Quick guide for switching between OpenAI, Anthropic, and Ollama providers.

## Method 1: Environment Variables (Recommended)

### Switch to Ollama (Local, Free)

```bash
# Edit your .env file
cat > .env << 'EOF'
LLM_PROVIDER=ollama
LLM_BASE_URL=http://localhost:11434/v1
LLM_MODEL=codellama
# No API key needed!
EOF

# Make sure Ollama is running
ollama serve
ollama pull codellama

# Run instrumentation
python telemetry-inject.py examples/sample_project -o output/
```

### Switch to OpenAI

```bash
# Edit your .env file
cat > .env << 'EOF'
LLM_PROVIDER=openai
OPENAI_API_KEY=your-openai-key-here
LLM_MODEL=gpt-4o-mini
EOF

# Run instrumentation
python telemetry-inject.py examples/sample_project -o output/
```

### Switch to Anthropic

```bash
# Edit your .env file
cat > .env << 'EOF'
LLM_PROVIDER=anthropic
ANTHROPIC_API_KEY=your-anthropic-key-here
LLM_MODEL=claude-3-5-sonnet-20241022
EOF

# Run instrumentation
python telemetry-inject.py examples/sample_project -o output/
```

## Method 2: Interactive Configuration

```bash
python telemetry-inject.py --configure
```

This will guide you through provider setup interactively.

## Method 3: Command Line Override

Override provider for a single run without changing .env:

```bash
# Use Ollama for this run only
LLM_PROVIDER=ollama LLM_BASE_URL=http://localhost:11434/v1 \
  python telemetry-inject.py examples/sample_project -o output/

# Use OpenAI with specific model
LLM_PROVIDER=openai OPENAI_API_KEY=your-key \
  python telemetry-inject.py examples/sample_project -o output/ --model gpt-4o-mini

# Use Anthropic
LLM_PROVIDER=anthropic ANTHROPIC_API_KEY=your-key \
  python telemetry-inject.py examples/sample_project -o output/
```

## Troubleshooting

### "Anthropic API key required" when using Ollama

**Problem**: You have `ANTHROPIC_API_KEY` in your .env but want to use Ollama.

**Solution**: Set `LLM_PROVIDER=ollama` explicitly:

```bash
# In .env file
LLM_PROVIDER=ollama  # Add this line
LLM_BASE_URL=http://localhost:11434/v1
LLM_MODEL=codellama
# Comment out or remove: ANTHROPIC_API_KEY=...
```

### "OpenAI API key required" when using Ollama

**Problem**: No provider specified and defaulting to OpenAI.

**Solution**: Set provider explicitly:

```bash
export LLM_PROVIDER=ollama
export LLM_BASE_URL=http://localhost:11434/v1
python telemetry-inject.py examples/sample_project -o output/
```

### Ollama connection failed

**Problem**: "Cannot connect to Ollama" error.

**Solution**: Start Ollama service:

```bash
# Start Ollama
ollama serve

# In another terminal, pull model if needed
ollama pull codellama

# Now run instrumentation
python telemetry-inject.py examples/sample_project -o output/
```

## Provider Auto-Detection

If `LLM_PROVIDER` is not set, the system auto-detects based on:

1. **Base URL contains localhost/127.0.0.1** → Ollama
2. **ANTHROPIC_API_KEY set but not OPENAI_API_KEY** → Anthropic
3. **OPENAI_API_KEY set** → OpenAI
4. **Nothing set** → Defaults to OpenAI (shows warning)

**Best Practice**: Always set `LLM_PROVIDER` explicitly to avoid confusion.

## Complete .env File Examples

### Ollama Only

```bash
LLM_PROVIDER=ollama
LLM_BASE_URL=http://localhost:11434/v1
LLM_MODEL=codellama
```

### OpenAI Only

```bash
LLM_PROVIDER=openai
OPENAI_API_KEY=sk-...
LLM_MODEL=gpt-4o-mini
```

### Anthropic Only

```bash
LLM_PROVIDER=anthropic
ANTHROPIC_API_KEY=sk-ant-...
LLM_MODEL=claude-3-5-sonnet-20241022
```

### Multiple Providers (Switch with LLM_PROVIDER)

```bash
# Active provider
LLM_PROVIDER=ollama

# OpenAI credentials (comment out when using others)
#OPENAI_API_KEY=sk-...

# Anthropic credentials (comment out when using others)
#ANTHROPIC_API_KEY=sk-ant-...

# Ollama config (active)
LLM_BASE_URL=http://localhost:11434/v1
LLM_MODEL=codellama
```

## Testing Provider Configuration

```bash
# Verbose mode shows which provider is being used
python telemetry-inject.py examples/sample_project -o output/ --verbose

# Output should show:
# 🤖 Using Ollama at http://localhost:11434/v1
#    Model: codellama
```

## Provider Comparison

| Provider | Cost | Speed | Quality | Setup |
|----------|------|-------|---------|-------|
| **Ollama** | Free | Fast (local) | Good | Medium (install) |
| **OpenAI** | $0.01-0.50/file | Fast | Excellent | Easy (API key) |
| **Anthropic** | $0.01-0.50/file | Fast | Excellent | Easy (API key) |

**Recommendation**:
- **Development**: Use Ollama (free, fast, no API costs)
- **Production**: Use OpenAI or Anthropic (higher quality, better reliability)
- **Testing**: Use Ollama or OpenAI gpt-4o-mini (low cost)

## Advanced: Provider-Specific Settings

### Ollama - Custom Port

```bash
LLM_PROVIDER=ollama
LLM_BASE_URL=http://localhost:8080/v1  # Custom port
LLM_MODEL=codellama
```

### OpenAI - Azure OpenAI

```bash
LLM_PROVIDER=openai
OPENAI_API_KEY=your-azure-key
OPENAI_API_BASE=https://your-resource.openai.azure.com/
LLM_MODEL=gpt-4
```

### Anthropic - Custom Model

```bash
LLM_PROVIDER=anthropic
ANTHROPIC_API_KEY=your-key
LLM_MODEL=claude-3-opus-20240229  # Highest quality model
```

---

For more help, run: `python telemetry-inject.py --help`
