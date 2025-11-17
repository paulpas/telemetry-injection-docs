# Edge Case Handler Design

## Overview

This document describes the architecture for a self-adaptive telemetry injection system that uses LLMs as a **contingency plan** rather than the primary mechanism.

## Core Principle

**"Scripts first, LLM when needed"**

- Template-based injection is fast, deterministic, and free
- LLM is used only when edge cases are detected
- System learns and adapts without human intervention
- No overfitting - adapts to new patterns dynamically

## Architecture

### 1. EdgeCaseDetector

**Purpose**: Identify when template-based injection needs LLM help

**Detection Strategies**:

```python
class EdgeCaseDetector:
    """Detects when LLM assistance is needed (no overfitting)."""

    def detect(self,
               original_code: str,
               instrumented_code: str,
               validation_result: ValidationResult,
               analysis_result: AnalysisResult) -> EdgeCaseReport:
        """
        Returns EdgeCaseReport with:
        - is_edge_case: bool
        - reasons: List[str]  # Why LLM is needed
        - confidence: float   # 0.0 to 1.0
        - suggested_fix: Optional[str]
        """
```

**Detection Signals** (Ranked by Confidence):

| Signal | Confidence | Description |
|--------|-----------|-------------|
| **Syntax Error** | 1.0 | Definite edge case - code doesn't compile |
| **Missing Telemetry** | 0.9 | Expected 10 calls, found 5 |
| **Incomplete Instrumentation** | 0.9 | Entry without exit |
| **Validation Failure** | 0.8 | Security or test failures |
| **Tree-Sitter Complexity** | 0.6 | Multi-line expressions, nested structures |
| **Historical Pattern** | 0.5 | Similar code failed before |

**Key Design Choices**:

1. **Multiple Signals**: Combine multiple indicators for robustness
2. **Confidence Scores**: Weight different signals appropriately
3. **Threshold-Based**: Only flag if confidence > 0.7 (configurable)
4. **No Auto-Fix**: Detect only, LLM generates fix
5. **Learning-Based**: Update rules based on historical data

### 2. LLM Contingency Handler

**Purpose**: Apply LLM-based fixes when edge cases are detected

**Key Features**:

```python
class LLMContingencyHandler:
    """Applies LLM fixes for edge cases."""

    def __init__(self,
                 max_attempts: int = 3,
                 require_llm: bool = False):
        """
        Args:
            max_attempts: Number of LLM fix attempts (default: 3)
            require_llm: If True, fail if LLM unavailable
                        If False, warn and continue with original code
        """

    def handle_edge_case(self,
                         edge_case: EdgeCaseReport,
                         original_code: str,
                         failed_code: str) -> ContingencyResult:
        """
        Returns ContingencyResult with:
        - success: bool
        - fixed_code: str
        - attempts_used: int
        - llm_used: bool
        - fallback_reason: Optional[str]
        """
```

**Contingency Flow**:

```
EdgeCaseDetector.detect() → is_edge_case=True
    ↓
Check LLM availability
    ├─> Available: Call LLM (up to max_attempts)
    ├─> Unavailable + require_llm=False: Warn, use original code
    └─> Unavailable + require_llm=True: Error
    ↓
LLM generates fix
    ↓
Validate fix (ScriptValidator)
    ├─> Success: Return fixed code ✅
    ├─> Failure + attempts < max_attempts: Retry with error context
    └─> Failure + attempts >= max_attempts: Fallback to original code
    ↓
Log result for self-learning
```

### 3. Self-Learning System

**Purpose**: Adapt and improve without human intervention

**Components**:

```python
class FailurePatternTracker:
    """Tracks and learns from failures."""

    def record_failure(self,
                      code_pattern: str,
                      failure_reason: str,
                      context: dict):
        """Save failure to .telemetry_cache/failures/"""

    def record_success(self,
                      code_pattern: str,
                      fix_applied: str):
        """Save successful fix to .telemetry_cache/successes/"""

    def analyze_patterns(self) -> List[FailurePattern]:
        """Use reflection engine to identify common patterns."""

    def update_detector_rules(self,
                             patterns: List[FailurePattern]):
        """Update EdgeCaseDetector with learned rules."""
```

**Learning Cycle**:

```
1. Record Outcome (Success/Failure)
   └─> Save to cache with full context

2. Periodic Analysis (Every N runs)
   └─> Reflection engine identifies patterns

3. Update Detection Rules
   └─> EdgeCaseDetector learns new edge cases

4. Validate Improvements
   └─> Track metrics before/after rule updates
```

### 4. CLI Integration

**New Default Behavior**:

```bash
# Default: Scripts + LLM fallback (3 attempts)
python telemetry-inject.py <directory>

# Custom retry attempts
python telemetry-inject.py <directory> --llm-fallback-attempts=5

# Disable LLM fallback (fastest, may have issues)
python telemetry-inject.py <directory> --no-llm-fallback

# Require LLM (fail if unavailable)
python telemetry-inject.py <directory> --require-llm

# Verbose (show edge case detection)
python telemetry-inject.py <directory> -v
```

**Configuration Check**:

```python
def check_llm_configuration() -> LLMStatus:
    """
    Returns LLMStatus:
    - available: bool
    - provider: Optional[str]
    - model: Optional[str]
    - warning: Optional[str]
    """

    # Check environment variables
    if not os.getenv("LLM_PROVIDER"):
        return LLMStatus(
            available=False,
            warning="No LLM_PROVIDER configured. "
                   "LLM fallback disabled. "
                   "Set LLM_PROVIDER for added integrity."
        )
```

## Edge Case Examples

### Example 1: Multi-Line Dictionary

**Original Code**:
```python
config = {
    "timeout": 30,
    "retry": 3,
    "endpoints": [
        "https://api1.example.com",
        "https://api2.example.com"
    ]
}
```

**Template Injection** (Broken):
```python
config = {
    tel.var_change("config", config)  # ❌ Inside dict!
    "timeout": 30,
    ...
}
```

**Edge Case Detection**:
- Syntax Error: `SyntaxError: invalid syntax` → confidence=1.0
- Tree-Sitter: Multi-line expression (7 lines) → confidence=0.6
- **Result**: `is_edge_case=True`, trigger LLM

**LLM Fix**:
```python
config = {
    "timeout": 30,
    "retry": 3,
    "endpoints": [
        "https://api1.example.com",
        "https://api2.example.com"
    ]
}
tel.var_change("config", config)  # ✅ After closing brace
```

### Example 2: Missing Telemetry

**Original Code**:
```python
def calculate(a, b):
    result = a + b
    intermediate = result * 2
    final = intermediate - 5
    return final
```

**Template Injection** (Incomplete):
```python
def calculate(a, b):
    _tel = tel.func_entry("calculate", "a, b")
    result = a + b
    tel.var_change("result", result)
    # Missing intermediate and final!
    tel.func_exit(_tel, final)
    return final
```

**Edge Case Detection**:
- Expected telemetry: 1 entry + 3 vars + 1 exit = 5 calls
- Actual telemetry: 3 calls
- Missing: 2 variable tracking calls
- **Result**: `is_edge_case=True`, confidence=0.9

**LLM Fix**:
```python
def calculate(a, b):
    _tel = tel.func_entry("calculate", "a, b")
    result = a + b
    tel.var_change("result", result)
    intermediate = result * 2
    tel.var_change("intermediate", intermediate)  # ✅ Added
    final = intermediate - 5
    tel.var_change("final", final)  # ✅ Added
    tel.func_exit(_tel, final)
    return final
```

### Example 3: Complex Conditional

**Original Code**:
```python
if (user.is_authenticated and
    user.has_permission("admin") and
    not user.is_locked):
    do_admin_action()
```

**Template Injection** (Broken):
```python
if (user.is_authenticated and
    tel.cond_entry("if", ...)  # ❌ Middle of condition!
    user.has_permission("admin") and
    not user.is_locked):
    do_admin_action()
```

**Edge Case Detection**:
- Syntax Error: Detected
- Tree-Sitter: Multi-line conditional (3 lines)
- **Result**: `is_edge_case=True`

**LLM Fix**:
```python
_tel_cond = tel.cond_entry("if", "user.is_authenticated and ...", ...)
if (user.is_authenticated and
    user.has_permission("admin") and
    not user.is_locked):
    do_admin_action()
    tel.cond_exit(_tel_cond)
```

## Metrics & Monitoring

### Success Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Template Success Rate** | > 85% | Files processed without LLM |
| **LLM Fallback Rate** | < 15% | Files needing LLM assistance |
| **Edge Case Detection Accuracy** | > 95% | True positives / (TP + FP) |
| **False Positive Rate** | < 5% | Unnecessary LLM calls |
| **Cost Savings** | > 80% | vs Always-LLM approach |
| **Performance** | < 20ms/func | Average processing time |

### Learning Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Pattern Recognition** | > 90% | Known patterns detected |
| **Rule Improvement** | +5% per cycle | Success rate increase |
| **Adaptation Speed** | < 100 samples | Samples to learn new pattern |

## Implementation Phases

### Phase 1: Edge Case Detection (TDD)
1. Write tests for EdgeCaseDetector
2. Implement syntax error detection
3. Implement missing telemetry detection
4. Implement incomplete instrumentation detection
5. Implement tree-sitter complexity analysis
6. Integrate with ScriptValidator

### Phase 2: LLM Contingency Handler (TDD)
1. Write tests for LLMContingencyHandler
2. Implement LLM availability checking
3. Implement retry logic with max_attempts
4. Implement fallback to original code
5. Add warning system for missing LLM
6. Integrate with ParallelScriptProcessor

### Phase 3: Self-Learning System
1. Implement FailurePatternTracker
2. Add cache storage for patterns
3. Integrate reflection engine
4. Implement rule update mechanism
5. Add metrics tracking

### Phase 4: CLI Integration
1. Make script-based injection default
2. Add `--llm-fallback-attempts` flag
3. Add `--no-llm-fallback` flag
4. Add `--require-llm` flag
5. Add LLM configuration check
6. Update help text and documentation

### Phase 5: Testing & Validation
1. End-to-end tests with edge cases
2. Performance benchmarks
3. Cost analysis
4. Learning system validation
5. Documentation updates

## Anti-Overfitting Strategies

1. **Multiple Signals**: Don't rely on single heuristic
2. **Confidence Thresholds**: Require high confidence (> 0.7)
3. **Learning-Based**: Adapt to actual failures, not predicted
4. **Validation Loop**: Always validate LLM fixes
5. **Fallback Safety**: Original code if all attempts fail
6. **Metrics Tracking**: Monitor false positives/negatives

## Benefits

### 1. Cost Savings
- 85%+ files: $0 (template-based)
- 15% files: $0.05-0.15 (LLM fallback)
- **Total savings: 80-90% vs always-LLM**

### 2. Performance
- Template-based: 15-25ms per function
- LLM fallback: 2-5 seconds (only when needed)
- **Average: < 500ms per function** (vs 5s always-LLM)

### 3. Reliability
- Template-based: 100% deterministic
- Edge case detection: 95%+ accuracy
- LLM fixes: 90%+ success rate
- **Overall: 98%+ success rate**

### 4. Self-Improvement
- Learns from failures automatically
- No human intervention needed
- Adapts to new code patterns
- Improves over time

## Success Criteria

✅ Template-based injection works for 85%+ of code
✅ Edge case detection accuracy > 95%
✅ LLM fallback success rate > 90%
✅ False positive rate < 5%
✅ Cost savings > 80% vs always-LLM
✅ Performance < 500ms per function average
✅ System learns and adapts without human intervention
✅ Graceful degradation when LLM unavailable

## References

- [Script-Based Injection Architecture](SCRIPT_BASED_ARCHITECTURE.md)
- [Tree-Sitter Implementation](TREE_SITTER_IMPLEMENTATION.md)
- [Reflection Engine](../src/reflection_engine.py)
- [Self-Healing Documentation](SELF_HEALING.md)
