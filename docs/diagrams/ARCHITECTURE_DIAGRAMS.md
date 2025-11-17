# System Architecture Diagrams

## Table of Contents
1. [System Overview](#system-overview)
2. [Instrumentation Flow](#instrumentation-flow)
3. [Validation Pipeline](#validation-pipeline)
4. [Retry & Reflection](#retry--reflection)
5. [DRY Implementation](#dry-implementation)

---

## System Overview

### High-Level Component Diagram

```mermaid
graph TB
    CLI[CLI Interface<br/>telemetry-inject.py] --> Scanner[CodeScanner]
    CLI --> Config[Configuration<br/>API Keys, Models]

    Scanner --> Analyzer[LLMAnalyzer]
    Analyzer --> Generator[TelemetryGenerator]
    Generator --> Injector[RetryInjector]

    Injector --> CodeInj[CodeInjector]
    Injector --> Validator[ValidationEngine]
    Injector --> Reflector[ReflectionEngine]

    CodeInj --> LLM1[LLM API<br/>GPT-4/Claude/Ollama]
    Analyzer --> LLM1
    Generator --> LLM1
    Reflector --> LLM1

    Validator --> PyVal[PythonValidator]
    Validator --> JSVal[JavaScriptValidator]

    CLI --> UtilsWriter[TelemetryUtilsWriter]
    UtilsWriter --> Templates[Utility Templates]

    Injector --> Output[(Output<br/>Directory)]
    UtilsWriter --> Output

    CLI --> CostTracker[CostTracker]
    LLM1 --> CostTracker
```

---

## Instrumentation Flow

### Complete End-to-End Sequence

```mermaid
sequenceDiagram
    actor User
    participant CLI as CLI
    participant Scanner as CodeScanner
    participant Analyzer as LLMAnalyzer
    participant Generator as TelemetryGenerator
    participant Injector as RetryInjector
    participant Validator as ValidationEngine
    participant Writer as UtilsWriter
    participant FS as File System

    User->>CLI: telemetry-inject.py project/ -o output/

    Note over CLI: Phase 1: Discovery
    CLI->>Scanner: scan(directory)
    Scanner->>FS: Walk directory tree
    FS-->>Scanner: File list
    Scanner->>Scanner: Filter by extension<br/>Skip ignored dirs
    Scanner-->>CLI: List of code files

    Note over CLI: Phase 2: Write Utilities
    CLI->>Writer: write_utilities(output_dir, file_paths)
    Writer->>Writer: Detect languages
    Writer->>Writer: Group files by directory
    loop For each directory
        Writer->>FS: Copy utility template
    end
    Writer-->>CLI: Utility files created

    Note over CLI: Phase 3: Process Each File
    loop For each file
        CLI->>Analyzer: analyze_code(code, language)
        Analyzer->>Analyzer: Build analysis prompt
        Analyzer->>LLM: Send analysis request
        LLM-->>Analyzer: Function/loop/variable locations
        Analyzer-->>CLI: AnalysisResult

        CLI->>Generator: generate(analysis_result)
        Generator->>Generator: Build generation prompt
        Generator->>LLM: Send generation request
        LLM-->>Generator: Telemetry snippets
        Generator-->>CLI: List of TelemetrySnippet

        CLI->>Injector: inject_with_retry(code, snippets, language)
        Note over Injector: Retry loop begins

        Injector->>CodeInj: inject(code, snippets, language)
        CodeInj->>CodeInj: Build injection prompt
        CodeInj->>LLM: Send injection request
        LLM-->>CodeInj: Instrumented code
        CodeInj-->>Injector: InjectionResult

        Injector->>Validator: validate(instrumented_code, language)

        alt Code is valid
            Validator-->>Injector: ValidationResult(success=True)
            Injector-->>CLI: RetryInjectionResult(succeeded=True)
        else Code is invalid
            Validator-->>Injector: ValidationResult(success=False, errors)
            Injector->>Reflector: reflect(failures, code, snippets)
            Reflector->>LLM: Analyze errors, suggest fixes
            LLM-->>Reflector: Guidance
            Reflector-->>Injector: ReflectionResult(guidance)
            Note over Injector: Retry with guidance
        end

        CLI->>FS: Write instrumented file
    end

    CLI-->>User: Instrumentation complete
```

---

## Validation Pipeline

### Multi-Stage Validation Process

```mermaid
sequenceDiagram
    participant Injector as RetryInjector
    participant Validator as ValidationEngine
    participant Complexity as ComplexityChecker
    participant Syntax as SyntaxValidator
    participant Compile as CompileValidator
    participant Runtime as RuntimeValidator
    participant Utils as UtilityDetector

    Injector->>Validator: validate(code, language, check_runtime=True)

    Note over Validator: Stage 1: Complexity Check
    Validator->>Complexity: is_simple(code, language)
    Complexity->>Complexity: Check for:<br/>- Nested lambdas<br/>- globals() manipulation<br/>- Long assignment lines<br/>- Frame inspection

    alt Code is complex
        Complexity-->>Validator: False (too complex)
        Validator-->>Injector: ValidationResult(success=False,<br/>error="Code too complex")
    else Code is simple
        Complexity-->>Validator: True

        Note over Validator: Stage 2: Syntax Check
        Validator->>Syntax: validate_syntax(code)
        Syntax->>Syntax: ast.parse(code) for Python<br/>or esprima for JavaScript

        alt Syntax invalid
            Syntax-->>Validator: False, error_message
            Validator-->>Injector: ValidationResult(success=False,<br/>error="Syntax error")
        else Syntax valid
            Syntax-->>Validator: True

            Note over Validator: Stage 3: Compile Check
            Validator->>Compile: validate_compile(code)
            Compile->>Compile: compile(code, bytecode)

            alt Compilation fails
                Compile-->>Validator: False, error_message
                Validator-->>Injector: ValidationResult(success=False,<br/>error="Compilation error")
            else Compilation succeeds
                Compile-->>Validator: True

                Note over Validator: Stage 4: Runtime Check (conditional)
                Validator->>Utils: detect_instrumented_code(code)

                alt Code imports _telemetry_utils
                    Utils-->>Validator: True (instrumented code)
                    Note over Validator: Skip runtime validation<br/>(instrumented code)
                    Validator-->>Injector: ValidationResult(success=True,<br/>runtime_skipped=True)
                else Code is not instrumented
                    Utils-->>Validator: False

                    alt check_runtime=True
                        Validator->>Runtime: run_code(code, timeout=15)
                        Runtime->>Runtime: Execute in subprocess<br/>with timeout

                        alt Execution succeeds
                            Runtime-->>Validator: True
                            Validator-->>Injector: ValidationResult(success=True)
                        else Execution fails/timeouts
                            Runtime-->>Validator: False, error_message
                            Validator-->>Injector: ValidationResult(success=False,<br/>error="Runtime error")
                        end
                    else check_runtime=False
                        Validator-->>Injector: ValidationResult(success=True)
                    end
                end
            end
        end
    end
```

---

## Retry & Reflection

### Self-Healing Injection Process

```mermaid
sequenceDiagram
    participant CLI as CLI
    participant Injector as RetryInjector
    participant CodeInj as CodeInjector
    participant Validator as ValidationEngine
    participant Reflector as ReflectionEngine
    participant LLM as LLM API

    CLI->>Injector: inject_with_retry(code, snippets, language, max_retries=3)

    Note over Injector: Attempt 1
    Injector->>CodeInj: inject(code, snippets, language)
    CodeInj->>LLM: Inject telemetry<br/>(no guidance)
    LLM-->>CodeInj: Instrumented code v1
    CodeInj-->>Injector: InjectionResult v1

    Injector->>Validator: validate(instrumented_code)

    alt Validation succeeds
        Validator-->>Injector: ValidationResult(success=True)
        Injector-->>CLI: RetryInjectionResult(succeeded=True, attempts=1)
    else Validation fails
        Validator-->>Injector: ValidationResult(success=False, errors=["Syntax error line 42"])

        Note over Injector: Attempt 2 with Reflection
        Injector->>Reflector: reflect(failures, code, snippets)
        Reflector->>Reflector: Analyze failure patterns
        Reflector->>LLM: What went wrong?<br/>How to fix?<br/>Previous errors: ...
        LLM-->>Reflector: Guidance:<br/>"Indentation mismatch at line 42.<br/>Ensure telemetry matches function body."
        Reflector-->>Injector: ReflectionResult(guidance)

        Injector->>CodeInj: inject(code, snippets, language, guidance)
        CodeInj->>LLM: Inject telemetry<br/>(with guidance)
        LLM-->>CodeInj: Instrumented code v2
        CodeInj-->>Injector: InjectionResult v2

        Injector->>Validator: validate(instrumented_code)

        alt Validation succeeds
            Validator-->>Injector: ValidationResult(success=True)
            Injector-->>CLI: RetryInjectionResult(succeeded=True, attempts=2)
        else Still failing after max retries
            Validator-->>Injector: ValidationResult(success=False)
            Note over Injector: Max retries exhausted
            Injector-->>CLI: RetryInjectionResult(succeeded=False,<br/>attempts=3,<br/>failure_reasons=[...])
        end
    end
```

---

## DRY Implementation

**Important**: The utility files (`_telemetry_utils.py`, `_telemetry_utils.js`) are **automatically excluded** from being scanned and instrumented by the `CodeScanner`. This prevents recursive instrumentation and ensures these utility files can be safely modified without being processed.

The exclusion is configured in `src/scanner.py:IGNORED_FILES` and works for utility files in any directory (root or nested subdirectories).

### Utility File Management

```mermaid
sequenceDiagram
    participant CLI as CLI
    participant Writer as TelemetryUtilsWriter
    participant Scanner as Language Detector
    participant FS as File System

    CLI->>Writer: write_utilities(output_dir, file_paths, input_dir)

    Note over Writer: Phase 1: Detect Languages & Directories
    Writer->>Scanner: Group files by language and directory
    loop For each file_path
        Scanner->>Scanner: Extract extension (.py, .js, .ts)
        Scanner->>Scanner: Map to language (python, javascript, typescript)
        Scanner->>Scanner: Compute output directory:<br/>rel_path = file_path.relative_to(input_dir)<br/>output_dir = output_dir / rel_path.parent
    end
    Scanner-->>Writer: {<br/>  "python": {<br/>    "/output": ["file1.py"],<br/>    "/output/subdir": ["file2.py"]<br/>  },<br/>  "javascript": {<br/>    "/output": ["app.js"]<br/>  }<br/>}

    Note over Writer: Phase 2: Copy Utilities to Each Directory
    loop For each (language, directories)
        Writer->>Writer: Select template:<br/>python → telemetry_utils_template_python.py<br/>javascript → telemetry_utils_template_javascript.js

        loop For each directory in directories
            Writer->>FS: Check if directory exists
            FS-->>Writer: Directory status

            alt Directory doesn't exist
                Writer->>FS: mkdir -p directory
            end

            Writer->>FS: Copy template to directory/_telemetry_utils.{py|js}
            FS-->>Writer: File copied
            Writer->>Writer: Record written file path
        end
    end

    Note over Writer: Phase 3: Update .gitignore
    Writer->>FS: Read .gitignore
    FS-->>Writer: Existing patterns

    Writer->>Writer: Check if _telemetry_utils.* already ignored

    alt Not already ignored
        Writer->>FS: Append to .gitignore:<br/>"_telemetry_utils.py"<br/>"_telemetry_utils.js"
    end

    Writer-->>CLI: List of written utility files

    Note over CLI: Result
    CLI->>CLI: Display:<br/>"✓ Created N telemetry utility file(s)"
```

### Import Resolution

```mermaid
graph TB
    subgraph "Input Structure"
        I1[input/root.py]
        I2[input/subdir/nested.py]
    end

    subgraph "Output Structure"
        O1[output/root.py<br/>from _telemetry_utils import tel]
        O2[output/_telemetry_utils.py]
        O3[output/subdir/nested.py<br/>from _telemetry_utils import tel]
        O4[output/subdir/_telemetry_utils.py]
    end

    I1 --> O1
    I2 --> O3

    O1 -.import.-> O2
    O3 -.import.-> O4

    style O2 fill:#90EE90
    style O4 fill:#90EE90

    note1[Utility copied to<br/>same directory as<br/>instrumented file]
    note1 -.-> O2
    note1 -.-> O4
```

---

## Component Interactions

### Cost Tracking Integration

```mermaid
sequenceDiagram
    participant CLI as CLI
    participant Tracker as CostTracker
    participant Analyzer as LLMAnalyzer
    participant Generator as TelemetryGenerator
    participant Injector as CodeInjector
    participant Reflector as ReflectionEngine
    participant LLM as LLM API

    CLI->>Tracker: Initialize(budget=5.00)

    Note over CLI: Process each file
    loop For each file
        CLI->>Analyzer: analyze_code(code, language)
        Analyzer->>Tracker: check_budget(estimated_tokens)

        alt Budget exceeded
            Tracker-->>Analyzer: BudgetExceededError
            Analyzer-->>CLI: Error: Budget exhausted
        else Budget available
            Tracker-->>Analyzer: OK
            Analyzer->>LLM: Send request
            LLM-->>Analyzer: Response
            Analyzer->>Tracker: add_call(model, input_tokens, output_tokens)
            Tracker->>Tracker: Calculate cost<br/>Update total
        end

        CLI->>Generator: generate(analysis_result)
        Generator->>Tracker: check_budget(estimated_tokens)
        Tracker-->>Generator: OK
        Generator->>LLM: Send request
        LLM-->>Generator: Response
        Generator->>Tracker: add_call(model, input_tokens, output_tokens)

        CLI->>Injector: inject(code, snippets)
        Injector->>Tracker: check_budget(estimated_tokens)
        Tracker-->>Injector: OK
        Injector->>LLM: Send request
        LLM-->>Injector: Response
        Injector->>Tracker: add_call(model, input_tokens, output_tokens)
    end

    CLI->>Tracker: get_summary()
    Tracker-->>CLI: {<br/>  total_calls: 47,<br/>  total_cost: 0.2961,<br/>  remaining: 4.7039<br/>}
    CLI->>CLI: Display cost summary
```

---

## Notes

### Diagram Conventions

- **Solid lines**: Data flow or function calls
- **Dashed lines**: Return values or responses
- **Dotted lines**: Imports or references
- **Green boxes**: Generated files
- **Blue boxes**: External services
- **Yellow boxes**: User interactions

### Updating Diagrams

These diagrams use Mermaid syntax. To update:

1. Edit the Mermaid code directly
2. Preview using GitHub or Mermaid Live Editor
3. Ensure diagrams render correctly before committing

### Additional Diagrams

For more specific component interactions, see:
- `docs/architecture/COMPONENT_DETAILS.md` - Detailed component specifications
- `docs/architecture/DATA_FLOW.md` - Data transformation details
- `docs/runbooks/DEBUGGING.md` - Debugging workflows

---

**Last Updated**: 2025-10-26
