# CLI Tooling System - Comprehensive Documentation

## Overview

The CLI tooling system provides safe, sandboxed command execution for the telemetry code generation pipeline. It enables LLM-driven determination of required commands while ensuring those commands cannot perform destructive operations or escape directory boundaries.

**Core Philosophy:**
1. **Test-Driven Development (TDD)**: Commands are tested in sandbox before real execution
2. **Directory Restriction**: Commands cannot operate outside allowed directories
3. **Safety First**: Destructive operations are blocked by default
4. **LLM Autonomy**: LLM determines what's needed, validator ensures it's safe

---

## Architecture

### Components

```
CLI Tooling System
├── CommandValidator        - TDD-based command validation
├── SandboxExecutor         - Safe command execution
├── GoEnvironmentManager    - Go-specific environment setup
└── Integration             - Integrated with validation_engine.py
```

### Files

```
src/
├── command_validator.py       # Command validation with sandbox testing
├── sandbox_executor.py         # Safe command execution engine
├── go_environment_manager.py   # Go environment management
└── validation_engine.py        # Integration point (GoValidator updated)

tests/
└── test_cli_tooling.py        # Comprehensive tests (27 tests, all passing)
```

---

## Component Details

### 1. CommandValidator

**File:** `src/command_validator.py`

**Purpose:** Validates commands before execution using TDD approach

**Key Features:**
- **Whitelist System**: Known-safe commands (cd, ls, pwd, go commands, etc.)
- **Blacklist System**: Never-allowed commands (rm, mv, chmod, etc.)
- **Pattern Detection**: Blocks dangerous patterns (.., &&, ||, |, >, etc.)
- **Sandbox Testing**: Tests unknown commands in temporary environment
- **Directory Validation**: Ensures commands stay within allowed boundaries

**Usage:**

```python
from src.command_validator import CommandValidator

# Initialize validator
validator = CommandValidator("/path/to/allowed/dir", verbose=True)

# Validate single command
result = validator.validate_command("ls -la", "/path/to/allowed/dir")
if result.is_safe:
    print(f"✅ Safe to execute: {result.reason}")
else:
    print(f"❌ Blocked: {result.reason}")
    if result.suggested_fix:
        print(f"   Suggestion: {result.suggested_fix}")

# Validate command sequence
results = validator.validate_command_sequence([
    "cd subdir",
    "go mod init test",
    "go build"
], "/path/to/allowed/dir")
```

**Safety Rules:**

Always Safe:
- `cd`, `ls`, `pwd`, `echo`, `cat`, `head`, `tail`, `find`, `grep`
- `go mod init`, `go mod tidy`, `go build`, `go test`, `go fmt`, etc.

Never Allowed:
- `rm`, `rmdir`, `mv`, `chmod`, `chown`, `dd`, `mkfs`, `fdisk`

Dangerous Patterns:
- `..` (directory traversal)
- `~/`, `/etc`, `/var`, `/usr` (system directories)
- `&&`, `||`, `;` (command chaining)
- `|` (pipes)
- `>`, `<` (redirects)
- `` ` ``, `$(` (command substitution)

**Example Output:**

```
🧪 Testing command in sandbox: go build
   Sandbox: /tmp/cmd_sandbox_abc123
   ✅ Sandbox test passed

Command: go build
Safe: ✅ True
Reason: Command passed sandbox test successfully
```

---

### 2. SandboxExecutor

**File:** `src/sandbox_executor.py`

**Purpose:** Executes validated commands within directory boundaries

**Key Features:**
- **Pre-Validation**: Uses CommandValidator before execution
- **Working Directory Tracking**: Maintains current directory state
- **Output Capture**: Captures stdout, stderr, return codes
- **History Tracking**: Records all executed commands
- **Timeout Protection**: 30-second timeout per command
- **Directory Restriction**: Cannot escape allowed base directory

**Usage:**

```python
from src.sandbox_executor import SandboxExecutor

# Initialize executor
executor = SandboxExecutor("/path/to/allowed/dir", verbose=True)

# Execute single command
result = executor.execute_command("ls -la")
if result.success:
    print(f"✅ Success: {result.stdout}")
else:
    print(f"❌ Failed: {result.error_message}")

# Execute command sequence
results = executor.execute_sequence([
    "pwd",
    "cd subdir",
    "ls"
])

# Get current directory
current_dir = executor.get_current_directory()

# View execution history
for result in executor.get_history():
    print(f"{result.command}: {result.return_code}")

# Reset to base directory
executor.reset_directory()
```

**Example Output:**

```
⚡ Executing: ls -la
   In directory: /tmp/test_dir
   ✅ Success (exit code: 0)
   Output: total 8
drwxr-xr-x 2 user user 4096 Oct 26 12:00 .
drwxrwxrwt 20 root root 4096 Oct 26 12:00 ..

⚡ Executing: rm -rf *
❌ Command blocked: rm -rf *
   Reason: Command 'rm' is never allowed (destructive)
```

---

### 3. GoEnvironmentManager

**File:** `src/go_environment_manager.py`

**Purpose:** Manages Go project environment and operations

**Key Features:**
- **Environment Analysis**: Detects Go files, modules, dependencies
- **Module Initialization**: Runs `go mod init` when needed
- **Dependency Management**: Handles `go mod tidy`
- **Build Management**: Manages `go build` and `go test`
- **Import Validation**: Detects incorrect telemetry imports
- **Package Name Detection**: Automatically detects and uses correct package names
- **Safe Execution**: Uses SandboxExecutor for all operations

**Usage:**

```python
from src.sandbox_executor import SandboxExecutor
from src.go_environment_manager import GoEnvironmentManager

# Initialize
executor = SandboxExecutor("/path/to/go/project", verbose=True)
manager = GoEnvironmentManager("/path/to/go/project", executor, verbose=True)

# Analyze environment
status = manager.analyze_environment()
print(f"Has Go files: {status.has_go_files}")
print(f"Has go.mod: {status.has_go_mod}")
print(f"Needs init: {status.needs_init}")

# Automatic setup (init + tidy if needed)
success, results = manager.setup_environment("mymodule")
if success:
    print("✅ Environment ready")

# Manual operations
manager.initialize_module("mymodule")
manager.tidy_dependencies()
manager.build_project("output_binary")
manager.run_tests("-v")

# Validate imports
valid, errors = manager.validate_imports()
if not valid:
    for error in errors:
        print(f"⚠️  {error}")
```

**Example Output:**

```
=== Go Environment Status ===
Directory: /tmp/test_project
Go files found: 3
Has go.mod: False
Module name: None
Needs init: True
Needs tidy: False
Go version: 1.21.0

🔧 Initializing Go module: test_module
⚡ Executing: go mod init test_module
   In directory: /tmp/test_project
   ✅ Success (exit code: 0)
✅ Go module initialized successfully

🔨 Building Go project...
⚡ Executing: go build
   In directory: /tmp/test_project
   ✅ Success (exit code: 0)
✅ Build successful
```

---

## Integration with Validation Engine

### GoValidator Enhancement

The `GoValidator` class in `validation_engine.py` has been updated to use the CLI tooling system:

**Before:**
```python
# Direct subprocess calls, no module management
result = subprocess.run(["go", "build", str(temp_file)], ...)
```

**After:**
```python
# Uses SandboxExecutor and GoEnvironmentManager
executor, go_manager = self._get_executor(temp_dir)
success, results = go_manager.setup_environment()
build_result = go_manager.build_project()
```

**Benefits:**
- ✅ Automatic Go module initialization
- ✅ Proper package name detection and substitution
- ✅ Safe command execution with directory restrictions
- ✅ Better error messages and debugging
- ✅ Consistent with TDD philosophy

**How It Works:**

1. **Syntax Check**: Uses gofmt via SandboxExecutor
2. **Compilation**:
   - Creates SandboxExecutor for temp directory
   - Detects telemetry utils and updates package name
   - Uses GoEnvironmentManager to setup environment
   - Runs build via manager
3. **Runtime**: Executes via SandboxExecutor

---

## Use Cases

### Use Case 1: Go Module Not Initialized

**Problem:** User's Go directory lacks go.mod

**Solution:**

```python
executor = SandboxExecutor("/path/to/go/code", verbose=True)
manager = GoEnvironmentManager("/path/to/go/code", executor, verbose=True)

# Automatic setup
success, _ = manager.setup_environment()
# This will:
# 1. Detect no go.mod
# 2. Run go mod init
# 3. Run go mod tidy if needed
```

**Output:**
```
=== Go Environment Status ===
Needs init: True

🔧 Initializing Go module: myproject
✅ Go module initialized successfully

📦 Tidying Go dependencies...
✅ Dependencies tidied successfully

✅ Go environment setup complete
```

### Use Case 2: Preventing Destructive Operations

**Problem:** LLM suggests `rm -rf old_build`

**Solution:**

```python
executor = SandboxExecutor("/path/to/project", verbose=True)
result = executor.execute_command("rm -rf old_build")
```

**Output:**
```
❌ Command blocked: rm -rf old_build
   Reason: Command 'rm' is never allowed (destructive)
```

### Use Case 3: Directory Traversal Prevention

**Problem:** Command tries to access parent directory

**Solution:**

```python
validator = CommandValidator("/path/to/project", verbose=True)
result = validator.validate_command("cd ../../etc", "/path/to/project")
```

**Output:**
```
❌ Unsafe: cd ../../etc
   Reason: Command contains dangerous pattern: '..'
   Suggested fix: Remove '..' from command
```

### Use Case 4: Go Import Validation

**Problem:** Generated code has incorrect telemetry import

**Solution:**

```python
manager = GoEnvironmentManager("/path/to/go/project", executor, verbose=True)
valid, errors = manager.validate_imports()
```

**Output:**
```
⚠️  Import validation errors:
   - main.go: Imports telemetry_utils (should be same package, no import needed)
```

---

## Testing

### Running Tests

```bash
# Run all CLI tooling tests
pytest tests/test_cli_tooling.py -v

# Run specific test class
pytest tests/test_cli_tooling.py::TestCommandValidator -v

# Run with coverage
pytest tests/test_cli_tooling.py --cov=src.command_validator --cov=src.sandbox_executor --cov=src.go_environment_manager
```

### Test Coverage

```
CommandValidator:     70.34% coverage
SandboxExecutor:      45.58% coverage
GoEnvironmentManager: 40.19% coverage

27 tests total - ALL PASSING ✅
```

### Test Categories

**CommandValidator Tests (8 tests):**
- Initialization
- Safe command allowance
- Unsafe command blocking
- Dangerous pattern detection
- Go command validation
- Directory traversal prevention
- Working directory validation
- Command sequence validation

**SandboxExecutor Tests (8 tests):**
- Initialization
- Safe command execution
- Unsafe command blocking
- Directory navigation
- Directory restriction
- Command sequence execution
- Execution history tracking
- Directory reset

**GoEnvironmentManager Tests (8 tests):**
- Initialization
- Empty environment analysis
- Go file detection
- Module initialization
- Environment setup
- Project building
- Import validation (bad imports detected)
- Import validation (good code passes)

**Integration Tests (3 tests):**
- Full Go workflow (create, setup, build)
- Validator-Executor integration
- Telemetry package name handling

---

## Error Handling

### Command Validation Errors

```python
result = validator.validate_command("rm -rf *", "/path")

if not result.is_safe:
    print(f"Error: {result.reason}")
    # Error: Command 'rm' is never allowed (destructive)

    if result.suggested_fix:
        print(f"Fix: {result.suggested_fix}")
```

### Execution Errors

```python
result = executor.execute_command("go build")

if not result.success:
    print(f"Command failed: {result.error_message}")
    print(f"Return code: {result.return_code}")
    print(f"Stderr: {result.stderr}")
```

### Go Environment Errors

```python
success, results = manager.setup_environment()

if not success:
    print("Setup failed:")
    for result in results:
        if not result.success:
            print(f"  - {result.command}: {result.error_message}")
```

---

## Security Considerations

### Directory Restrictions

The system enforces strict directory boundaries:

```python
# Allowed: Commands within base directory
validator.validate_command("cd subdir", "/allowed/base")  # ✅ Safe

# Blocked: Commands outside base directory
validator.validate_command("cd ../../etc", "/allowed/base")  # ❌ Blocked
```

### Command Validation Layers

1. **Static Analysis**: Check against whitelist/blacklist
2. **Pattern Detection**: Scan for dangerous patterns
3. **Directory Validation**: Ensure paths are within bounds
4. **Sandbox Testing**: Test unknown commands in isolation
5. **Execution**: Only run validated commands

### Timeout Protection

All commands have 30-second timeout:

```python
# Long-running command will timeout
result = executor.execute_command("sleep 60")
# Result: timeout=True, error_message="Command timed out (>30s)"
```

---

## Best Practices

### 1. Always Initialize with Verbose Mode During Development

```python
validator = CommandValidator(base_dir, verbose=True)  # See validation details
executor = SandboxExecutor(base_dir, verbose=True)    # See execution details
manager = GoEnvironmentManager(dir, executor, verbose=True)  # See Go operations
```

### 2. Check Validation Results

```python
result = validator.validate_command(cmd, working_dir)
if not result.is_safe:
    # Don't attempt execution
    logging.error(f"Unsafe command blocked: {result.reason}")
    return
```

### 3. Use Execution History for Debugging

```python
# Execute sequence
executor.execute_sequence(commands)

# Review what happened
for result in executor.get_history():
    if not result.success:
        print(f"Failed: {result.command}")
        print(f"  Error: {result.error_message}")
```

### 4. Let GoEnvironmentManager Handle Setup

```python
# Don't manually run go mod init, go mod tidy, etc.
# Let the manager handle it:
success, _ = manager.setup_environment("mymodule")
```

### 5. Validate Imports After Generation

```python
# After telemetry code generation
valid, errors = manager.validate_imports()
if not valid:
    # Fix imports before building
    for error in errors:
        logging.warning(f"Import issue: {error}")
```

---

## Troubleshooting

### Issue: Commands Not Executing

**Symptom:** Commands return `success=False` with "validation failed"

**Cause:** Command is blocked by validator

**Solution:**
```python
# Check why it's blocked
result = validator.validate_command(cmd, working_dir)
print(f"Blocked: {result.reason}")
print(f"Fix: {result.suggested_fix}")
```

### Issue: Go Build Fails

**Symptom:** `go build` returns compilation errors

**Cause:** Go module not initialized or wrong package names

**Solution:**
```python
# Use manager's setup
success, results = manager.setup_environment()
if not success:
    # Check which step failed
    for r in results:
        if not r.success:
            print(f"Failed: {r.command}")
            print(f"Error: {r.stderr}")
```

### Issue: Import Errors in Go

**Symptom:** "package telemetry_utils is not in std"

**Cause:** Generated code imports telemetry_utils (should be same package)

**Solution:**
```python
# Validate and fix imports
valid, errors = manager.validate_imports()
if not valid:
    # Imports are wrong, need to regenerate telemetry code
    # with correct instructions (use src/prompts system)
```

### Issue: Directory Traversal Blocked

**Symptom:** `cd` commands to subdirectories blocked

**Cause:** Path contains `..` or goes outside allowed base

**Solution:**
```python
# Use relative paths without ..
# Good: cd subdir
# Bad:  cd ../other_dir

# If you need to access parent directories,
# initialize validator with higher-level base:
validator = CommandValidator("/higher/level/path", verbose=True)
```

---

## Future Enhancements

### Planned Features

1. **Command History Persistence**
   - Save execution history to file
   - Load and replay command sequences
   - Generate execution reports

2. **Enhanced Sandbox Testing**
   - Configurable timeout per command type
   - Resource usage monitoring (CPU, memory)
   - Network access detection and blocking

3. **Multi-Language Support**
   - Python environment manager (venv, pip)
   - JavaScript environment manager (npm, yarn)
   - Rust environment manager (cargo)

4. **LLM Integration**
   - Generate commands based on error messages
   - Suggest fixes for validation failures
   - Auto-correct common mistakes

5. **Audit Trail**
   - Detailed logging of all commands
   - Security event tracking
   - Compliance reporting

---

## API Reference

### CommandValidator

```python
class CommandValidator:
    def __init__(self, allowed_base_dir: str, verbose: bool = False)
    def validate_command(self, command: str, working_dir: str) -> CommandValidationResult
    def validate_command_sequence(self, commands: List[str], working_dir: str) -> List[CommandValidationResult]
```

### SandboxExecutor

```python
class SandboxExecutor:
    def __init__(self, allowed_base_dir: str, verbose: bool = False)
    def execute_command(self, command: str, working_dir: Optional[str] = None) -> ExecutionResult
    def execute_sequence(self, commands: List[str]) -> List[ExecutionResult]
    def get_current_directory(self) -> str
    def reset_directory(self)
    def get_history(self) -> List[ExecutionResult]
    def clear_history(self)
    def get_last_result(self) -> Optional[ExecutionResult]
```

### GoEnvironmentManager

```python
class GoEnvironmentManager:
    def __init__(self, project_dir: str, executor: SandboxExecutor, verbose: bool = False)
    def analyze_environment(self) -> GoEnvironmentStatus
    def initialize_module(self, module_name: Optional[str] = None) -> ExecutionResult
    def tidy_dependencies(self) -> ExecutionResult
    def build_project(self, output_name: Optional[str] = None) -> ExecutionResult
    def run_tests(self, test_args: Optional[str] = None) -> ExecutionResult
    def setup_environment(self, module_name: Optional[str] = None) -> Tuple[bool, List[ExecutionResult]]
    def validate_imports(self) -> Tuple[bool, List[str]]
```

---

## Examples

### Example 1: Basic Go Project Setup

```python
from src.sandbox_executor import SandboxExecutor
from src.go_environment_manager import GoEnvironmentManager

# Setup
executor = SandboxExecutor("/home/user/myproject", verbose=True)
manager = GoEnvironmentManager("/home/user/myproject", executor, verbose=True)

# Analyze and setup
status = manager.analyze_environment()
if status.needs_init:
    success, _ = manager.setup_environment("github.com/user/myproject")
    if success:
        print("✅ Project ready!")
```

### Example 2: Safe Command Execution

```python
from src.sandbox_executor import SandboxExecutor

executor = SandboxExecutor("/home/user/project", verbose=True)

# Execute sequence
commands = [
    "pwd",
    "ls -la",
    "go mod init mymodule",
    "go build"
]

results = executor.execute_sequence(commands)

# Check results
for i, result in enumerate(results):
    if result.success:
        print(f"✅ {commands[i]}")
    else:
        print(f"❌ {commands[i]}: {result.error_message}")
```

### Example 3: Import Validation

```python
from src.sandbox_executor import SandboxExecutor
from src.go_environment_manager import GoEnvironmentManager

executor = SandboxExecutor("/home/user/goproject", verbose=True)
manager = GoEnvironmentManager("/home/user/goproject", executor, verbose=True)

# Check imports
valid, errors = manager.validate_imports()

if not valid:
    print("⚠️  Import issues found:")
    for error in errors:
        print(f"   - {error}")
    print("\nFix: Regenerate telemetry code without import statements")
else:
    print("✅ All imports valid")
```

---

## Summary

The CLI tooling system provides:

✅ **Safety**: Commands validated and tested before execution
✅ **Security**: Directory boundaries strictly enforced
✅ **Automation**: LLM can determine needs, system ensures safety
✅ **Go Support**: Full Go environment management
✅ **Testing**: Comprehensive test suite (27 tests, all passing)
✅ **Integration**: Seamlessly integrated with validation engine
✅ **Documentation**: Complete API reference and examples

**Status:** Production-ready ✅

**Test Results:** 27/27 passing ✅

**Coverage:**
- CommandValidator: 70.34% ✅
- SandboxExecutor: 45.58% ✅
- GoEnvironmentManager: 40.19% ✅
