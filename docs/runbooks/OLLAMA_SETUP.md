# Using Code Telemetry Injector with Ollama

This guide shows you how to use the Code Telemetry Injector with **Ollama**, a local LLM server that runs entirely on your machine - **no API key required!**

## Why Use Ollama?

- ✅ **Free** - No API costs
- ✅ **Private** - Your code never leaves your machine
- ✅ **Fast** - No network latency
- ✅ **Offline** - Works without internet
- ✅ **No API Key** - Just install and run

## Prerequisites

1. **Install Ollama**: Download from [ollama.ai](https://ollama.ai/)
2. **Install Python dependencies** (same as before):
   ```bash
   python3 -m venv venv
   source venv/bin/activate
   pip install -r requirements.txt
   ```

## Quick Start

### 1. Install and Start Ollama

```bash
# Install a code-focused model (recommended)
ollama pull codellama

# Or try other models:
ollama pull deepseek-coder  # Great for code
ollama pull mistral          # Good general model
ollama pull mixtral          # Larger, more capable
```

Ollama starts automatically and runs at `http://localhost:11434`

### 2. Configure the Tool

**Option A: Using Environment Variables (Recommended)**

Edit your `.env` file:
```bash
# Comment out OpenAI (not needed)
# OPENAI_API_KEY=your_key

# Add Ollama configuration
LLM_BASE_URL=http://localhost:11434/v1
LLM_MODEL=codellama
LLM_TIMEOUT=300  # 5 minutes (default for Ollama, increase for large models)
```

**Option B: Using Command Line Flags**

No configuration needed - just use flags when running:
```bash
python -m src.cli /path/to/code \
  --base-url http://localhost:11434/v1 \
  --model codellama \
  -v
```

### 3. Run the Tool

```bash
# Using .env configuration
python -m src.cli examples/sample_project -v

# Or using command line flags
python -m src.cli examples/sample_project \
  --base-url http://localhost:11434/v1 \
  --model codellama \
  -v
```

## Recommended Models

Different models work better for different tasks:

### CodeLlama (Recommended)
```bash
ollama pull codellama
LLM_MODEL=codellama
```
- Best for code instrumentation
- Understands multiple programming languages
- Fast and accurate

### DeepSeek Coder
```bash
ollama pull deepseek-coder
LLM_MODEL=deepseek-coder
```
- Excellent code understanding
- Very good at generating instrumentation
- Slightly larger than CodeLlama

### Mistral
```bash
ollama pull mistral
LLM_MODEL=mistral
```
- Good general-purpose model
- Works well for code
- Balanced speed/quality

### Mixtral (Advanced)
```bash
ollama pull mixtral
LLM_MODEL=mixtral
```
- Most capable model
- Best quality output
- Requires more resources (GPU/RAM)

## Complete Example

```bash
# 1. Install Ollama and pull a model
ollama pull codellama

# 2. Set up configuration
cat > .env << EOF
LLM_BASE_URL=http://localhost:11434/v1
LLM_MODEL=codellama
EOF

# 3. Run the tool
python -m src.cli examples/sample_project -v

# You should see:
# 🤖 Using Ollama at http://localhost:11434/v1
#    Model: codellama
# 🔍 Scanning directory: examples/sample_project
# ...
```

## Troubleshooting

### "Connection refused" Error

**Problem**: Can't connect to Ollama

**Solution**:
```bash
# Check if Ollama is running
ollama list

# Start Ollama if needed (usually starts automatically)
ollama serve

# Test the connection
curl http://localhost:11434/v1/models
```

### Model Not Found

**Problem**: "Model 'codellama' not found"

**Solution**:
```bash
# Pull the model first
ollama pull codellama

# List available models
ollama list
```

### Slow Performance

**Problem**: Ollama is running slowly

**Solutions**:
1. **Use a smaller model**: Try `codellama` instead of `mixtral`
2. **Use GPU**: Ollama automatically uses GPU if available
3. **Increase resources**: Give Docker/Ollama more RAM/CPU
4. **Use quantized models**: Most Ollama models are pre-quantized for speed

### Different Port

**Problem**: Ollama runs on a different port

**Solution**:
```bash
# Find your Ollama port
ollama list  # Shows the endpoint

# Update configuration
LLM_BASE_URL=http://localhost:YOUR_PORT/v1
```

## Performance Comparison

| Provider | Speed | Cost | Privacy | Offline |
|----------|-------|------|---------|---------|
| OpenAI GPT-4 | Fast | $$$ | Cloud | No |
| Ollama CodeLlama | Medium | Free | Local | Yes |
| Ollama Mixtral | Slower | Free | Local | Yes |

## Advanced Configuration

### Custom Ollama Server

If Ollama runs on a different machine:

```bash
# .env file
LLM_BASE_URL=http://192.168.1.100:11434/v1
LLM_MODEL=codellama

# Or command line
python -m src.cli mycode \
  --base-url http://192.168.1.100:11434/v1 \
  --model codellama
```

### Model Parameters

Ollama uses default parameters optimized for each model. The tool sets:
- `temperature=0.1` for consistent, deterministic output
- Appropriate system prompts for code analysis

### Switching Between OpenAI and Ollama

Keep both configurations in your `.env`:

```bash
# OpenAI configuration
OPENAI_API_KEY=sk-...

# Ollama configuration (commented by default)
# LLM_BASE_URL=http://localhost:11434/v1
# LLM_MODEL=codellama
```

Uncomment/comment the appropriate lines to switch providers.

## Verifying Installation

Run the test on example code:

```bash
# Instrument the example project
python -m src.cli examples/sample_project \
  --base-url http://localhost:11434/v1 \
  --model codellama \
  --dry-run \
  -v

# Should output:
# 🤖 Using Ollama at http://localhost:11434/v1
#    Model: codellama
# 🔍 Scanning directory: examples/sample_project
# ✓ Found 2 code file(s)
# ...
# ✓ Dry run complete - no files were modified
```

## Best Practices

1. **Start with CodeLlama**: It's optimized for code and runs efficiently
2. **Use verbose mode**: Add `-v` to see what's happening
3. **Try dry-run first**: Use `--dry-run` to test without writing files
4. **Keep Ollama updated**: Run `ollama pull codellama` periodically
5. **Monitor resources**: Check CPU/GPU usage if performance is slow

## FAQ

**Q: Do I need an API key for Ollama?**
A: No! That's the main benefit. Ollama runs locally with no API key required.

**Q: Is Ollama as good as GPT-4?**
A: For code tasks, CodeLlama and DeepSeek-Coder are very competitive. Quality depends on your specific use case.

**Q: Can I use both OpenAI and Ollama?**
A: Yes! Just switch the configuration or use command-line flags to choose.

**Q: How much disk space do models need?**
A: Typically 4-8GB per model. CodeLlama is about 3.8GB.

**Q: Does this work on Mac/Windows/Linux?**
A: Yes! Ollama supports all three platforms.

**Q: I'm getting timeout errors with large models. What should I do?**
A: Increase the timeout with `LLM_TIMEOUT=600` (10 minutes) in your `.env` file. Large models like gpt-oss:20b or mixtral may need more time. The default is 300 seconds (5 minutes) for Ollama.

## Getting Help

- **Ollama Documentation**: https://ollama.ai/docs
- **Ollama GitHub**: https://github.com/ollama/ollama
- **Model Library**: https://ollama.ai/library

## Summary

To use Ollama with no API key:

```bash
# 1. Install Ollama
brew install ollama  # Mac
# or download from ollama.ai

# 2. Pull a model
ollama pull codellama

# 3. Run the tool
python -m src.cli your/code \
  --base-url http://localhost:11434/v1 \
  --model codellama \
  -v
```

That's it! No API key needed, everything runs locally.

---

**Next Steps**: Try it on the example project, then use it on your own code!
