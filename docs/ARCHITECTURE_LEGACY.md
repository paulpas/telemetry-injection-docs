# ⚠️ LEGACY DOCUMENTATION

**This document is superseded by**: [ARCHITECTURE_REFACTORED.md](ARCHITECTURE_REFACTORED.md)

**Last Updated**: 2025-11-02
**Status**: Historical reference only
**For current documentation, see**: [INDEX.md](INDEX.md)

---

# Architecture Documentation

This document provides comprehensive architecture diagrams and explanations for the Telemetry Injector system.

## Table of Contents

1. [System Overview](#system-overview)
2. [Core Components](#core-components)
3. [Execution Flow](#execution-flow)
4. [LLM Integration](#llm-integration)
5. [Parallel Processing](#parallel-processing)
6. [Token Detection](#token-detection)
7. [GPU Scheduling](#gpu-scheduling)
8. [Debug Trace Logging](#debug-trace-logging)

---

## System Overview

```mermaid
graph TB
    subgraph "User Interface"
        CLI[CLI Entry Point<br/>telemetry-inject.py]
    end

    subgraph "Core Processing"
        Scanner[Code Scanner<br/>Analyzes source files]
        Analyzer[LLM Analyzer<br/>Generates telemetry code]
        Injector[Code Injector<br/>Inserts telemetry]
    end

    subgraph "LLM Layer"
        ParallelExec[Parallel Executor<br/>Multi-model execution]
        TokenDetect[Token Detector<br/>Auto-detect limits]
        CostTrack[Cost Tracker<br/>Budget management]
    end

    subgraph "Infrastructure"
        GPUSched[GPU Scheduler<br/>VRAM management]
        ModelPool[Model Pool<br/>Model rotation]
        DebugLog[Debug Logger<br/>Trace logging]
    end

    subgraph "LLM Providers"
        OpenAI[OpenAI API]
        Anthropic[Anthropic API]
        Ollama[Ollama Local]
    end

    CLI --> Scanner
    Scanner --> Analyzer
    Analyzer --> ParallelExec
    ParallelExec --> TokenDetect
    ParallelExec --> GPUSched
    ParallelExec --> ModelPool
    ParallelExec --> OpenAI
    ParallelExec --> Anthropic
    ParallelExec --> Ollama
    ParallelExec --> CostTrack
    Analyzer --> Injector

    DebugLog -.-> Scanner
    DebugLog -.-> Analyzer
    DebugLog -.-> ParallelExec
    DebugLog -.-> Injector

    style CLI fill:#e1f5ff
    style ParallelExec fill:#fff4e1
    style GPUSched fill:#e8f5e9
    style DebugLog fill:#f3e5f5
```

**Key Features:**
- Multi-provider LLM support (OpenAI, Anthropic, Ollama)
- Parallel execution with intelligent model rotation
- Automatic token limit detection
- VRAM-based GPU scheduling
- Comprehensive debug trace logging
- Cost tracking with budget limits

---

## Core Components

### 1. Code Scanner

```mermaid
graph LR
    Input[Source Code] --> Scanner[Code Scanner]
    Scanner --> Extract[Extract Functions]
    Extract --> Validate[Validate Syntax]
    Validate --> Output[Function List]

    Scanner --> Language{Detect<br/>Language}
    Language -->|Python| PyParser[Python AST]
    Language -->|Go| GoParser[Go Parser]
    Language -->|C/C++| CParser[Tree-sitter]
    Language -->|JavaScript| JSParser[Babel Parser]

    style Scanner fill:#e1f5ff
```

**Purpose:** Scans source code files and extracts functions for telemetry injection.

**Key Classes:**
- `CodeScanner`: Main scanner class
- `FunctionExtractor`: Language-specific function extraction

---

### 2. LLM Analyzer

```mermaid
graph TB
    Function[Function Code] --> Analyzer[LLM Analyzer]
    Analyzer --> Mode{Execution<br/>Mode?}

    Mode -->|Default| Parallel[Parallel Executor<br/>Fast, first-valid]
    Mode -->|Benchmark| Judge[Judge Ensemble<br/>Quality voting]

    Parallel --> Models[Multiple Models]
    Models --> First[First Valid Response]

    Judge --> All[All Models]
    All --> Vote[Quality Voting]
    Vote --> Best[Best Response]

    First --> Telemetry[Telemetry Code]
    Best --> Telemetry

    style Analyzer fill:#fff4e1
    style Parallel fill:#e8f5e9
    style Judge fill:#ffebee
```

**Purpose:** Analyzes functions and generates telemetry code using LLMs.

**Execution Modes:**
1. **Default Mode** (Fast): Uses `ParallelExecutor` for first-valid response
2. **Benchmark Mode** (Quality): Uses `JudgeEnsemble` for voting-based quality

**Key Classes:**
- `LLMAnalyzer`: Main analyzer class
- `ParallelExecutor`: Parallel execution engine
- `JudgeEnsemble`: Quality voting system

---

### 3. Code Injector

```mermaid
graph TB
    Original[Original Code] --> Injector[Code Injector]
    Telemetry[Telemetry Code<br/>from LLM] --> Injector

    Injector --> Parse[Parse AST]
    Parse --> Locate[Locate Injection Points]
    Locate --> Insert[Insert Telemetry]
    Insert --> Validate[Validate Syntax]
    Validate --> Format[Format Code]
    Format --> Output[Instrumented Code]

    Validate -->|Error| Retry[Retry with Different Model]
    Retry --> Parse

    style Injector fill:#e1f5ff
```

**Purpose:** Injects telemetry code into source files while maintaining syntax correctness.

**Key Classes:**
- `CodeInjector`: Main injection engine
- `FunctionInjector`: Function-level injection
- `CommandValidator`: Validates telemetry commands

---

## Execution Flow

### Default Execution Flow (Fast Mode)

```mermaid
sequenceDiagram
    participant User
    participant CLI
    participant Scanner
    participant Analyzer
    participant ParallelExec
    participant LLM1
    participant LLM2
    participant LLM3
    participant Injector

    User->>CLI: telemetry-inject.py file.py
    CLI->>Scanner: Scan file
    Scanner->>Scanner: Extract functions
    Scanner-->>CLI: Function list

    loop For each function
        CLI->>Analyzer: Analyze function
        Analyzer->>ParallelExec: Query multiple models

        par Parallel Execution
            ParallelExec->>LLM1: Generate telemetry
            ParallelExec->>LLM2: Generate telemetry
            ParallelExec->>LLM3: Generate telemetry
        end

        alt LLM1 responds first with valid response
            LLM1-->>ParallelExec: Telemetry code
            ParallelExec-->>Analyzer: First valid response
        else LLM2 responds first
            LLM2-->>ParallelExec: Telemetry code
            ParallelExec-->>Analyzer: First valid response
        end

        Analyzer-->>CLI: Telemetry code
        CLI->>Injector: Inject telemetry
        Injector-->>CLI: Success
    end

    CLI-->>User: File instrumented
```

**Characteristics:**
- **Speed**: First valid response wins
- **Parallelism**: All models queried simultaneously
- **Cost**: Lower (stops after first success)
- **Use Case**: Production deployments

---

### Benchmark Execution Flow (Quality Mode)

```mermaid
sequenceDiagram
    participant User
    participant CLI
    participant Analyzer
    participant Judge
    participant Models
    participant Validators

    User->>CLI: telemetry-inject.py --perform-benchmark
    CLI->>Analyzer: Analyze with benchmark=True

    loop For each function
        Analyzer->>Judge: Judge ensemble mode
        Judge->>Models: Query all models

        par All Models Respond
            Models-->>Judge: Response 1
            Models-->>Judge: Response 2
            Models-->>Judge: Response 3
            Models-->>Judge: Response N
        end

        Judge->>Validators: Validate all responses
        Validators-->>Judge: Validation scores

        Judge->>Judge: Quality voting
        Judge-->>Analyzer: Best response
    end

    Analyzer-->>User: Highest quality telemetry
```

**Characteristics:**
- **Quality**: Best response via voting
- **Completeness**: All models must respond
- **Cost**: Higher (waits for all models)
- **Use Case**: Quality benchmarking and model testing

---

## LLM Integration

### Multi-Provider Support

```mermaid
graph TB
    subgraph "Parallel Executor"
        Executor[Parallel Executor]
    end

    subgraph "Provider Detection"
        Detect{Detect Provider<br/>from Model Name}
        Detect -->|gpt-4*, gpt-3*| OpenAI
        Detect -->|claude-*| Anthropic
        Detect -->|Other| Ollama
    end

    subgraph "Provider APIs"
        OpenAI[OpenAI Client]
        Anthropic[Anthropic Client]
        Ollama[Ollama Client]
    end

    subgraph "Token Management"
        TokenDetect[Token Detector]
        TokenDetect --> Cache[Token Cache]
        TokenDetect --> API[API Detection]
        TokenDetect --> Known[Known Limits]
    end

    subgraph "Cost Management"
        CostTrack[Cost Tracker]
        CostTrack --> Budget[Budget Check]
        CostTrack --> Pricing[Model Pricing]
    end

    Executor --> Detect
    Executor --> TokenDetect
    Executor --> CostTrack

    OpenAI --> Response
    Anthropic --> Response
    Ollama --> Response

    Response[Response] --> Executor

    style Executor fill:#fff4e1
    style TokenDetect fill:#e8f5e9
    style CostTrack fill:#ffe0b2
```

**Provider Detection Logic:**

```python
def _get_provider(model: str) -> str:
    """Auto-detect provider from model name."""
    model_lower = model.lower()

    # OpenAI models (precise matching)
    if (model_lower.startswith('gpt-3') or
        model_lower.startswith('gpt-4') or
        model_lower.startswith('o1-')):
        return 'openai'

    # Anthropic models
    elif 'claude' in model_lower:
        return 'anthropic'

    # Default to Ollama for local models
    else:
        return 'ollama'
```

---

## Parallel Processing

### Model Pool Rotation (Ollama)

```mermaid
graph TB
    subgraph "Model Pool"
        Pool[Ollama Model Pool<br/>cogito:8b,gpt-oss:20b,llama3]
        Index[Current Index: 0]
    end

    subgraph "Request Flow"
        Req1[Request 1] --> Get1[Get Next Model]
        Req2[Request 2] --> Get2[Get Next Model]
        Req3[Request 3] --> Get3[Get Next Model]
        Req4[Request 4] --> Get4[Get Next Model]
    end

    subgraph "GPU Assignment"
        GPU0[GPU 0<br/>cogito:8b]
        GPU1[GPU 1<br/>gpt-oss:20b]
        GPU2[GPU 2<br/>llama3]
        GPU3[GPU 0<br/>cogito:8b]
    end

    Get1 --> GPU0
    Get2 --> GPU1
    Get3 --> GPU2
    Get4 --> GPU3

    Pool -.-> Get1
    Pool -.-> Get2
    Pool -.-> Get3
    Pool -.-> Get4

    style Pool fill:#e8f5e9
```

**Why Model Rotation?**

Ollama cannot run the same model on multiple GPUs simultaneously. Model rotation allows parallel processing by:

1. **Parsing comma-separated models**: `LLM_MODEL=cogito:8b,gpt-oss:20b,llama3`
2. **Round-robin assignment**: Each request gets the next model in the pool
3. **Parallel execution**: Different models run on different GPUs concurrently

**Implementation:**

```python
class OllamaModelPool:
    def __init__(self, model_spec: str):
        # Parse "model1,model2,model3"
        self.models = [m.strip() for m in model_spec.split(',')]
        self.current_index = 0
        self.lock = threading.Lock()

    def get_next_model(self) -> str:
        """Thread-safe round-robin rotation."""
        with self.lock:
            model = self.models[self.current_index]
            self.current_index = (self.current_index + 1) % len(self.models)
            return model
```

---

## Token Detection

### Auto-Detection Flow

```mermaid
graph TB
    Request[Token Limit Request] --> Cache{Check<br/>Cache}

    Cache -->|Hit| Return[Return Cached Limits]
    Cache -->|Miss| Provider{Provider?}

    Provider -->|OpenAI| OpenAIAPI[OpenAI API]
    Provider -->|Anthropic| AnthropicAPI[Anthropic API]
    Provider -->|Ollama| OllamaAPI[Ollama API]

    OpenAIAPI --> Known1[Known Limits<br/>Fallback]
    AnthropicAPI --> Known2[Known Limits<br/>Fallback]
    OllamaAPI --> EnvVar[Environment<br/>OLLAMA_NUM_CTX]

    Known1 --> Save[Save to Cache]
    Known2 --> Save
    EnvVar --> Save

    Save --> Return

    style Cache fill:#e8f5e9
    style Save fill:#fff4e1
```

**Token Detection Process:**

1. **Check Cache**: Look for cached limits in `~/.cache/telemetry_injector/token_limits.json`
2. **API Detection**: Try to detect via provider API
3. **Known Limits**: Use pre-configured limits for common models
4. **Conservative Default**: Fall back to safe defaults (8192 input, 2048 output)

**Example Known Limits:**

```python
KNOWN_LIMITS = {
    # OpenAI
    "gpt-4o": (128000, 16384),
    "o1-mini": (128000, 65536),

    # Anthropic
    "claude-3-5-sonnet-20241022": (200000, 8192),

    # Ollama
    "codellama": (16384, 2048),
    "llama3": (8192, 2048),
}
```

---

## GPU Scheduling

### VRAM-Based Scheduling

```mermaid
graph TB
    subgraph "Model Assignment"
        Models[Models to Schedule<br/>codellama, gpt-oss:120b, cogito:8b]
        Estimate[Estimate VRAM Requirements]
    end

    subgraph "GPU Detection"
        Detect{GPU Type?}
        Detect -->|AMD| ROCm[rocm-smi]
        Detect -->|NVIDIA| CUDA[nvidia-smi]
        Detect -->|None| CPU[CPU Mode]
    end

    subgraph "GPU States"
        GPU0[GPU 0<br/>48GB total<br/>40GB free]
        GPU1[GPU 1<br/>48GB total<br/>45GB free]
        GPU2[GPU 2<br/>24GB total<br/>20GB free]
    end

    subgraph "Scheduling Algorithm"
        Sort[Sort by Size<br/>120b → 8b]
        Pack[Best-Fit Packing]
    end

    Models --> Estimate
    Estimate --> Sort

    Detect --> GPU0
    Detect --> GPU1
    Detect --> GPU2

    Sort --> Pack
    GPU0 --> Pack
    GPU1 --> Pack
    GPU2 --> Pack

    Pack --> Assign[Assignment:<br/>gpt-oss:120b → GPU1<br/>codellama → GPU0<br/>cogito:8b → GPU2]

    style GPU0 fill:#e8f5e9
    style GPU1 fill:#e8f5e9
    style GPU2 fill:#fff4e1
```

**VRAM Estimation:**

Models are estimated based on parameter count extracted from name:

```python
def estimate_vram(model_name: str) -> int:
    """
    Estimate VRAM requirements from model name.

    Rule: ~1.3 GB per billion parameters (FP16)
    Buffer: +20% for context and overhead

    Examples:
    - 7b model: ~10 GB
    - 20b model: ~30 GB
    - 120b model: ~180 GB
    """
    param_count = extract_param_count(model_name)  # e.g., "8b" → 8.0
    base_vram_gb = param_count * 1.3
    vram_with_buffer = base_vram_gb * 1.2
    return int(vram_with_buffer * 1024)  # Convert to MB
```

**Scheduling Algorithm:**

1. **Estimate**: Calculate VRAM requirements for all models
2. **Sort**: Largest models first (best-fit packing)
3. **Query GPUs**: Get current VRAM availability
4. **Assign**: Match models to GPUs with sufficient VRAM
5. **Fallback**: Use GPU with most free VRAM if none have enough

---

## Debug Trace Logging

### Logging Architecture

```mermaid
graph TB
    subgraph "Application Layer"
        Scanner[Code Scanner]
        Analyzer[LLM Analyzer]
        ParallelExec[Parallel Executor]
        Injector[Code Injector]
    end

    subgraph "Debug Trace Logger"
        Logger[Debug Logger]
        Levels{Log Level?}

        Levels --> Trace[TRACE<br/>Function calls, LLM conversations]
        Levels --> Debug[DEBUG<br/>Model selection, GPU assignment]
        Levels --> Info[INFO<br/>Major milestones]
        Levels --> Warning[WARNING<br/>Retries, blacklisting]
        Levels --> Error[ERROR<br/>Failures, exceptions]
    end

    subgraph "Output Targets"
        Console[Console Output<br/>Real-time feedback]
        File[JSONL File<br/>logs/debug_trace_*.jsonl]
    end

    Scanner -.-> Logger
    Analyzer -.-> Logger
    ParallelExec -.-> Logger
    Injector -.-> Logger

    Logger --> Levels
    Levels --> Console
    Levels --> File

    style Logger fill:#f3e5f5
    style Console fill:#e1f5ff
    style File fill:#fff4e1
```

**Event Types Logged:**

```python
# Function calls
logger.function_call("analyze_code", code=code, language="python")

# LLM requests
logger.llm_request(
    model="gpt-4o",
    provider="openai",
    prompt=prompt,
    temperature=0.7
)

# LLM responses
logger.llm_response(
    model="gpt-4o",
    response=response,
    input_tokens=1500,
    output_tokens=800,
    duration=2.5,
    cost=0.045
)

# Model rotation
logger.model_rotation(
    model="cogito:8b",
    pool_size=3,
    index=0
)

# GPU assignment
logger.gpu_assignment(
    model="gpt-oss:120b",
    gpu_id=1,
    vram_free_mb=45000
)

# Token detection
logger.token_detection(
    model="claude-3-5-sonnet-20241022",
    provider="anthropic",
    max_input=200000,
    max_output=8192,
    from_cache=True
)
```

**Output Format (JSONL):**

```json
{"type": "event", "timestamp": 1234567890.123, "datetime": "2025-10-29T10:30:00.123", "level": "TRACE", "category": "llm_request", "message": "Sending request to gpt-4o", "data": {"model": "gpt-4o", "provider": "openai", "prompt": "...", "temperature": 0.7}}
{"type": "event", "timestamp": 1234567892.456, "datetime": "2025-10-29T10:30:02.456", "level": "TRACE", "category": "llm_response", "message": "Received response from gpt-4o", "data": {"model": "gpt-4o", "input_tokens": 1500, "output_tokens": 800, "cost_usd": "$0.045000"}}
```

**Usage:**

```bash
# Enable debug trace
export DEBUG_TRACE=true
export DEBUG_TRACE_LEVEL=TRACE  # TRACE, DEBUG, INFO, WARNING, ERROR
export DEBUG_TRACE_CONSOLE=true

# Run with debug trace
python telemetry-inject.py --input file.py

# View logs
cat logs/debug_trace_20251029_103000.jsonl | jq .
```

---

## Summary

The Telemetry Injector architecture is designed for:

1. **Performance**: Parallel execution with first-valid response
2. **Flexibility**: Multi-provider LLM support
3. **Intelligence**: Auto-detection of tokens and VRAM
4. **Scalability**: Model rotation for Ollama multi-GPU
5. **Observability**: Comprehensive debug trace logging
6. **Cost Control**: Budget management and cost tracking

For more details, see:
- [Examples](EXAMPLES.md) - Usage scenarios and examples
- [README](../README.md) - Quick start and configuration
