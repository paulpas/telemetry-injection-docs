# Master Meta-Prompt: Cloud-Enriched Telemetry Injector (v3 Enhanced)

> **Goal:**
> Implement an automatic telemetry injector that augments arbitrary source code with runtime instrumentation and cloud metadata enrichment.
> The injector must produce JSONL-formatted telemetry events compatible with OpenTelemetry, Elastic Common Schema (ECS), Datadog, and Splunk.
> It must gather contextual data from the runtime environment (EC2, ECS, EKS, GCP, Azure, Kubernetes, or local).

---

## 🎯 What Makes This v3 Enhanced

Based on analysis of production telemetry systems, this v3 schema adds:

1. **`duration_ms`** - Human-readable milliseconds (UIs prefer this)
2. **`trace_flags`** - OpenTelemetry sampling flags ("01" = sampled)
3. **`sample_rate`** - For high-volume systems (1.0 = 100% sampled)
4. **`event.category`** and `event.type`** - Elastic Common Schema event metadata
5. **Enhanced stack traces** - Full exception context
6. **Nested HTTP metadata** - Structured request/response with headers, body size, content type

**Compatibility Target**: 95%+ with OpenTelemetry, Datadog, Elastic, Splunk, Honeycomb, Jaeger, Tempo

---

## 🧩 Complete v3 Schema Definition

Each JSON line is a single event. Use this exact structure:

```json
{
  "schema": {
    "format": "telemetry.v3",
    "version": "3.0.0"
  },
  "timestamp": "<RFC3339 timestamp in UTC with millisecond precision>",
  "timestamp_ns": "<integer nanoseconds since Unix epoch>",
  "sequence": "<incremental event number per process>",
  "event_type": "<string describing event type>",
  "severity": "<INFO|WARN|ERROR|DEBUG|TRACE>",
  "function_name": "<if applicable>",
  "trace_id": "<32-hex string (128-bit)>",
  "span_id": "<16-hex string (64-bit)>",
  "parent_span_id": "<16-hex string or null>",
  "trace_flags": "<2-char string: '01' if sampled, '00' if not>",
  "sample_rate": "<float 0.0-1.0, sampling percentage>",
  "duration_ns": "<integer or null>",
  "duration_ms": "<float or null>",
  "message": "<short human-readable description>",
  "event": {
    "name": "<event name, often same as function_name>",
    "category": "<code|network|error|auth|database>",
    "type": "<start|end|info|error|change>"
  },
  "attributes": {
    "<arbitrary structured key-value pairs>"
  },
  "service": {
    "name": "<logical service or module name>",
    "version": "<semver or git sha>"
  },
  "environment": "<dev|staging|prod>",
  "resource": {
    "telemetry_sdk_name": "custom-injector",
    "telemetry_sdk_version": "3.0.0",
    "runtime_name": "<python|nodejs|go|rust>",
    "runtime_version": "<detected runtime version>",
    "os_type": "<linux|windows|darwin>",
    "os_description": "<os release string>"
  },
  "cloud": {
    "provider": "<aws|gcp|azure|local>",
    "region": "<region name or null>",
    "availability_zone": "<availability zone or null>",
    "account_id": "<cloud account id or null>",
    "instance_id": "<EC2/GCE/Azure VM instance id or null>",
    "machine_type": "<instance type or VM size>",
    "image_id": "<AMI or image id>",
    "platform": "<ec2|ecs|eks|lambda|gce|gke|aks>",
    "metadata_source": "<how values were obtained: IMDSv2|metadata_server|env|file>"
  },
  "host": {
    "name": "<system hostname>",
    "ip": "<primary private ip>",
    "public_ip": "<public ip if exists>"
  },
  "k8s": {
    "cluster_name": "<cluster name>",
    "namespace_name": "<namespace>",
    "pod_name": "<pod name>",
    "pod_uid": "<unique pod identifier>",
    "pod_labels": {
      "<label_key>": "<label_value>"
    },
    "pod_annotations": {
      "<annotation_key>": "<annotation_value>"
    },
    "container_name": "<container name>",
    "container_id": "<container runtime id>",
    "node_name": "<node host name>",
    "deployment_name": "<deployment name if exists>",
    "service_account_name": "<service account for RBAC>",
    "cpu_request": "<cpu request, e.g., '500m'>",
    "cpu_limit": "<cpu limit, e.g., '1000m'>",
    "memory_request": "<memory request, e.g., '256Mi'>",
    "memory_limit": "<memory limit, e.g., '512Mi'>"
  },
  "process": {
    "pid": "<process id>",
    "executable": "<binary or interpreter>",
    "command_line": "<command line invocation>"
  },
  "http": {
    "request": {
      "method": "<GET|POST|PUT|DELETE|etc>",
      "url": "<full URL>",
      "headers": {
        "<header_name>": "<header_value>"
      },
      "body_bytes": "<size in bytes>",
      "user_agent": "<User-Agent header value>"
    },
    "response": {
      "status_code": "<HTTP status code>",
      "headers": {
        "<header_name>": "<header_value>"
      },
      "body_bytes": "<size in bytes>",
      "content_type": "<Content-Type header value>"
    }
  },
  "error": {
    "type": "<exception type>",
    "message": "<error message>",
    "stacktrace": "<full stacktrace, limit to 2KB>"
  },
  "correlation_id": "<legacy compatibility field>"
}
```

---

## ⚙️ Calculation & Data Collection Logic

### 1. **Timestamp Fields**

```python
# timestamp: RFC3339 with millisecond precision in UTC
from datetime import datetime, timezone
timestamp = datetime.now(timezone.utc).isoformat(timespec='milliseconds')

# timestamp_ns: integer nanoseconds since Unix epoch
import time
timestamp_ns = time.time_ns()
```

**Requirements**:
- Always use UTC (trailing 'Z')
- Millisecond precision for timestamp (human-readable)
- Nanosecond precision for timestamp_ns (high-resolution timing)

---

### 2. **Sequence**

```python
# Global counter per process, thread-safe
import threading

_sequence_lock = threading.Lock()
_sequence_counter = 0

def get_next_sequence():
    global _sequence_counter
    with _sequence_lock:
        _sequence_counter += 1
        return _sequence_counter
```

**Requirements**:
- Start at 1 for each process
- Increment for every event
- Thread-safe if multi-threaded

---

### 3. **Trace & Span Identifiers (OpenTelemetry Compatible)**

```python
import secrets

def generate_trace_id():
    """Generate 128-bit trace ID (32 hex chars)."""
    return secrets.token_hex(16)

def generate_span_id():
    """Generate 64-bit span ID (16 hex chars)."""
    return secrets.token_hex(8)
```

**Requirements**:
- `trace_id`: 32-character lowercase hex (128-bit)
- `span_id`: 16-character lowercase hex (64-bit)
- `parent_span_id`: Parent's span_id or null for root spans
- Reuse trace_id across entire request trace
- Create new span_id for each function/operation
- Maintain span stack for parent relationships

**Hierarchy Rules**:
1. Root span: `parent_span_id = null`, new `trace_id`
2. Child span: Same `trace_id`, new `span_id`, `parent_span_id = parent.span_id`
3. Sibling spans: Same `parent_span_id`, different `span_id`

---

### 4. **Trace Flags & Sampling (NEW in v3)**

```python
def get_trace_flags(sampled: bool = True):
    """OpenTelemetry trace flags."""
    return "01" if sampled else "00"

def get_sample_rate(config):
    """Sampling rate from config."""
    return config.get("SAMPLE_RATE", 1.0)  # Default: 100%
```

**Requirements**:
- `trace_flags`: "01" if event is sampled, "00" if not
- `sample_rate`: Float 0.0-1.0 (1.0 = 100% sampled)
- Sampling decision made at trace root, inherited by children
- Use head-based sampling (decide at entry) or tail-based (decide at exit)

---

### 5. **Duration**

```python
# At function entry
entry_time_ns = time.time_ns()

# At function exit
exit_time_ns = time.time_ns()
duration_ns = exit_time_ns - entry_time_ns
duration_ms = duration_ns / 1_000_000  # Convert to milliseconds
```

**Requirements**:
- `duration_ns`: Integer nanoseconds (high precision)
- `duration_ms`: Float milliseconds (human-readable, UIs prefer this)
- Only present for exit events (entry, loop_entry, etc.)
- Both fields required for maximum compatibility

---

### 6. **Severity Mapping**

```python
def get_severity(event_type, has_error=False):
    """Determine severity level."""
    if has_error or event_type in ["exception_exit", "error"]:
        return "ERROR"
    if event_type in ["function_entry", "function_exit"]:
        return "INFO"
    if event_type in ["loop_iteration", "variable_change"]:
        return "DEBUG"
    if event_type.startswith("http_"):
        return "INFO"
    return "INFO"  # Default
```

**Requirements**:
- Use ERROR for exceptions and errors
- Use WARN for anomalies (slow response, retries)
- Use INFO for normal operations
- Use DEBUG for verbose details (loop iterations, variable changes)
- Use TRACE for ultra-verbose (every line execution)

---

### 7. **Event Metadata (NEW in v3 - ECS Compatibility)**

```python
# Map event_type to ECS categories
ECS_CATEGORY_MAP = {
    "function_entry": ("code", "start"),
    "function_exit": ("code", "end"),
    "http_request": ("network", "start"),
    "http_response": ("network", "end"),
    "exception_entry": ("error", "start"),
    "exception_exit": ("error", "end"),
    "variable_change": ("code", "change"),
    "loop_entry": ("code", "start"),
    "loop_exit": ("code", "end"),
    "conditional_entry": ("code", "start"),
    "conditional_exit": ("code", "end"),
    "database_query": ("database", "start"),
    "database_result": ("database", "end"),
}

def get_event_metadata(event_type, name):
    """Get ECS event metadata."""
    category, ecs_type = ECS_CATEGORY_MAP.get(event_type, ("code", "info"))
    return {
        "name": name,
        "category": category,
        "type": ecs_type
    }
```

**Requirements**:
- `event.name`: Event name (often function name or operation)
- `event.category`: High-level grouping (code, network, error, auth, database)
- `event.type`: Action type (start, end, info, error, change)
- Critical for Elastic Stack filtering and dashboards

---

### 8. **Service & Environment**

```python
import os
import subprocess

def get_service_metadata():
    """Detect service name and version."""
    # Service name from env or directory
    service_name = os.getenv("SERVICE_NAME")
    if not service_name:
        service_name = os.path.basename(os.getcwd())

    # Version from git or env
    version = os.getenv("SERVICE_VERSION")
    if not version:
        try:
            version = subprocess.check_output(
                ["git", "rev-parse", "--short", "HEAD"],
                stderr=subprocess.DEVNULL
            ).decode().strip()
        except:
            version = "unknown"

    environment = os.getenv("ENVIRONMENT", "dev")

    return {
        "service": {"name": service_name, "version": version},
        "environment": environment
    }
```

**Requirements**:
- `service.name`: Logical service name (not hostname)
- `service.version`: Semver, git SHA, or build number
- `environment`: dev, staging, prod (for filtering)

---

### 9. **Resource Metadata**

```python
import sys
import platform

def get_resource_metadata():
    """Collect runtime and OS metadata."""
    runtime_name = "python"  # or "nodejs", "go", "java", "rust"
    runtime_version = f"{sys.version_info.major}.{sys.version_info.minor}.{sys.version_info.micro}"

    return {
        "telemetry_sdk_name": "custom-injector",
        "telemetry_sdk_version": "3.0.0",
        "runtime_name": runtime_name,
        "runtime_version": runtime_version,
        "os_type": platform.system().lower(),
        "os_description": f"{platform.system()} {platform.release()}"
    }
```

**Requirements**:
- Detect runtime automatically (python, node, go, java, rust)
- Get exact version (3.11.5, go1.21.0, node v18.12.0)
- OS type: linux, windows, darwin
- OS description: Full platform string

---

### 10. **Cloud Metadata Collection**

#### AWS EC2 (IMDSv2)

```python
import urllib.request
import json

def get_ec2_metadata():
    """Fetch EC2 metadata using IMDSv2."""
    try:
        # Step 1: Get token (IMDSv2)
        token_req = urllib.request.Request(
            "http://169.254.169.254/latest/api/token",
            method="PUT",
            headers={"X-aws-ec2-metadata-token-ttl-seconds": "60"}
        )
        token = urllib.request.urlopen(token_req, timeout=1).read().decode()

        # Step 2: Get instance identity document
        doc_req = urllib.request.Request(
            "http://169.254.169.254/latest/dynamic/instance-identity/document",
            headers={"X-aws-ec2-metadata-token": token}
        )
        doc = json.loads(urllib.request.urlopen(doc_req, timeout=1).read())

        return {
            "provider": "aws",
            "region": doc.get("region"),
            "availability_zone": doc.get("availabilityZone"),
            "account_id": doc.get("accountId"),
            "instance_id": doc.get("instanceId"),
            "machine_type": doc.get("instanceType"),
            "image_id": doc.get("imageId"),
            "platform": "ec2",
            "metadata_source": "IMDSv2"
        }
    except:
        return None
```

#### GCP Compute Engine

```python
def get_gcp_metadata():
    """Fetch GCP metadata from metadata server."""
    try:
        headers = {"Metadata-Flavor": "Google"}

        def fetch(path):
            req = urllib.request.Request(
                f"http://metadata.google.internal/computeMetadata/v1/{path}",
                headers=headers
            )
            return urllib.request.urlopen(req, timeout=1).read().decode()

        return {
            "provider": "gcp",
            "region": fetch("instance/zone").split("/")[-1].rsplit("-", 1)[0],
            "availability_zone": fetch("instance/zone").split("/")[-1],
            "account_id": fetch("project/project-id"),
            "instance_id": fetch("instance/id"),
            "machine_type": fetch("instance/machine-type").split("/")[-1],
            "platform": "gce",
            "metadata_source": "metadata_server"
        }
    except:
        return None
```

#### Azure VM

```python
def get_azure_metadata():
    """Fetch Azure VM metadata."""
    try:
        req = urllib.request.Request(
            "http://169.254.169.254/metadata/instance?api-version=2021-02-01",
            headers={"Metadata": "true"}
        )
        data = json.loads(urllib.request.urlopen(req, timeout=1).read())

        compute = data.get("compute", {})
        return {
            "provider": "azure",
            "region": compute.get("location"),
            "availability_zone": compute.get("zone"),
            "account_id": compute.get("subscriptionId"),
            "instance_id": compute.get("vmId"),
            "machine_type": compute.get("vmSize"),
            "image_id": compute.get("imageReference", {}).get("id"),
            "platform": "azure_vm",
            "metadata_source": "metadata_server"
        }
    except:
        return None
```

#### Kubernetes Detection

```python
def get_k8s_metadata():
    """Detect Kubernetes environment."""
    if not os.path.exists("/var/run/secrets/kubernetes.io/serviceaccount"):
        return None

    k8s = {}

    # Read namespace from service account
    try:
        with open("/var/run/secrets/kubernetes.io/serviceaccount/namespace") as f:
            k8s["namespace_name"] = f.read().strip()
    except:
        pass

    # Get metadata from environment variables (set by Downward API)
    k8s["cluster_name"] = os.getenv("K8S_CLUSTER_NAME")
    k8s["pod_name"] = os.getenv("HOSTNAME")  # Hostname = pod name in K8s
    k8s["pod_uid"] = os.getenv("POD_UID")
    k8s["container_name"] = os.getenv("CONTAINER_NAME")
    k8s["node_name"] = os.getenv("NODE_NAME")
    k8s["deployment_name"] = os.getenv("DEPLOYMENT_NAME")
    k8s["service_account_name"] = os.getenv("SERVICE_ACCOUNT_NAME")

    # Parse resource limits/requests
    k8s["cpu_request"] = os.getenv("CPU_REQUEST")
    k8s["cpu_limit"] = os.getenv("CPU_LIMIT")
    k8s["memory_request"] = os.getenv("MEMORY_REQUEST")
    k8s["memory_limit"] = os.getenv("MEMORY_LIMIT")

    # Parse labels (JSON from env)
    labels_json = os.getenv("POD_LABELS")
    if labels_json:
        k8s["pod_labels"] = json.loads(labels_json)

    return k8s if any(k8s.values()) else None
```

**Requirements**:
- Try cloud metadata in order: EC2 → GCP → Azure
- Use IMDSv2 for EC2 (token-based, more secure)
- Always use short timeouts (1 second max)
- Fallback to `"provider": "local"` if all fail
- Cache metadata (only fetch once per process)

---

### 11. **Host Information**

```python
import socket

def get_host_metadata():
    """Collect host information."""
    hostname = socket.gethostname()

    try:
        ip = socket.gethostbyname(hostname)
    except:
        ip = None

    # Get public IP (optional, may require external service)
    public_ip = None
    try:
        # AWS EC2 public IP from metadata
        req = urllib.request.Request(
            "http://169.254.169.254/latest/meta-data/public-ipv4",
            headers={"X-aws-ec2-metadata-token": token}  # Reuse token
        )
        public_ip = urllib.request.urlopen(req, timeout=1).read().decode()
    except:
        pass

    return {
        "name": hostname,
        "ip": ip,
        "public_ip": public_ip
    }
```

**Requirements**:
- Always include hostname
- Include private IP if available
- Include public IP only if cloud metadata provides it
- Don't call external IP services (security/privacy)

---

### 12. **Process Information**

```python
import os
import sys

def get_process_metadata():
    """Collect process information."""
    return {
        "pid": os.getpid(),
        "executable": sys.executable,
        "command_line": " ".join(sys.argv)
    }
```

**Requirements**:
- Process ID for correlation
- Executable path (interpreter or binary)
- Full command line for debugging

---

### 13. **HTTP Request/Response Metadata (ENHANCED in v3)**

```python
def emit_http_request(method, url, headers=None, body=None):
    """Emit HTTP request event with enhanced metadata."""
    headers = headers or {}

    event = {
        # ... standard fields ...
        "event_type": "http_request",
        "http": {
            "request": {
                "method": method,
                "url": url,
                "headers": {k: v for k, v in headers.items()},  # Full headers
                "body_bytes": len(body) if body else 0,
                "user_agent": headers.get("User-Agent", "unknown")
            }
        }
    }

    return event

def emit_http_response(status_code, headers=None, body=None, duration_ns=0):
    """Emit HTTP response event with enhanced metadata."""
    headers = headers or {}

    event = {
        # ... standard fields ...
        "event_type": "http_response",
        "duration_ns": duration_ns,
        "duration_ms": duration_ns / 1_000_000,
        "http": {
            "response": {
                "status_code": status_code,
                "headers": {k: v for k, v in headers.items()},
                "body_bytes": len(body) if body else 0,
                "content_type": headers.get("Content-Type", "unknown")
            }
        }
    }

    return event
```

**Requirements**:
- Nested `http.request` and `http.response` objects
- Capture headers (sanitize sensitive ones like Authorization)
- Track body sizes for performance analysis
- User-Agent for client identification
- Content-Type for API analytics

---

### 14. **Error Handling (ENHANCED in v3)**

```python
import traceback

def emit_error(exception, function_name=None):
    """Emit error event with full stack trace."""
    stack_lines = traceback.format_exception(
        type(exception),
        exception,
        exception.__traceback__
    )

    event = {
        # ... standard fields ...
        "event_type": "exception_exit",
        "severity": "ERROR",
        "function_name": function_name,
        "error": {
            "type": type(exception).__name__,
            "message": str(exception)[:200],  # Limit message
            "stacktrace": "".join(stack_lines)[:2000]  # Limit to 2KB
        }
    }

    return event
```

**Requirements**:
- Full stack trace capture (limit to 2KB to avoid huge events)
- Exception type (class name)
- Exception message (limit to 200 chars)
- Always set severity to ERROR
- Include in function_exit events when exception occurs

---

## 🎯 Event Type Catalog

| Event Type | When to Emit | Required Fields |
|------------|--------------|-----------------|
| `function_entry` | Function starts | function_name, trace_id, span_id |
| `function_exit` | Function ends | function_name, duration_ns, duration_ms |
| `http_request` | HTTP call starts | http.request.* |
| `http_response` | HTTP call completes | http.response.*, duration_ns |
| `database_query` | DB query starts | db.system, db.statement |
| `database_result` | DB query completes | db.rows_returned, duration_ns |
| `exception_entry` | Try block entered | exception_type |
| `exception_exit` | Exception caught/not caught | exception_raised, error.* |
| `variable_change` | Variable assigned | variable_name, value, value_type |
| `loop_entry` | Loop starts | loop_type |
| `loop_iteration` | Loop iteration | iteration_count |
| `loop_exit` | Loop ends | total_iterations, duration_ns |
| `conditional_entry` | If/switch starts | condition_type, condition |
| `conditional_exit` | Branch taken | branch_taken |

---

## 📊 Example Complete v3 Event

```json
{
  "schema": {
    "format": "telemetry.v3",
    "version": "3.0.0"
  },
  "timestamp": "2025-11-03T19:15:41.499Z",
  "timestamp_ns": 1762197341499000000,
  "sequence": 5,
  "event_type": "function_exit",
  "severity": "INFO",
  "function_name": "fetch_data",
  "trace_id": "0af7651916cd43dd8448eb211c80319",
  "span_id": "b7ad6b7169203331",
  "parent_span_id": null,
  "trace_flags": "01",
  "sample_rate": 1.0,
  "duration_ns": 345678000,
  "duration_ms": 345.678,
  "message": "Function 'fetch_data' exit (345.68ms)",
  "event": {
    "name": "fetch_data",
    "category": "code",
    "type": "end"
  },
  "attributes": {
    "return_value": "{'status': 'ok'}",
    "caller_function": "main",
    "caller_file": "/app/main.py",
    "caller_line": 42
  },
  "correlation_id": "func_abc123",
  "service": {
    "name": "data-fetcher",
    "version": "1.0.0"
  },
  "environment": "prod",
  "resource": {
    "telemetry_sdk_name": "custom-injector",
    "telemetry_sdk_version": "3.0.0",
    "runtime_name": "python",
    "runtime_version": "3.11.5",
    "os_type": "linux",
    "os_description": "Linux 6.14.0-34-generic"
  },
  "cloud": {
    "provider": "aws",
    "region": "us-east-1",
    "availability_zone": "us-east-1a",
    "account_id": "123456789012",
    "instance_id": "i-0b123456789abcdef",
    "machine_type": "t3.large",
    "image_id": "ami-0ff8a91507f77f867",
    "platform": "ec2",
    "metadata_source": "IMDSv2"
  },
  "host": {
    "name": "ip-172-31-23-45.ec2.internal",
    "ip": "172.31.23.45",
    "public_ip": "54.91.123.45"
  },
  "k8s": {
    "cluster_name": "prod-cluster-1",
    "namespace_name": "default",
    "pod_name": "data-fetcher-7d4c8f9b-xk2j9",
    "pod_uid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "pod_labels": {
      "app": "data-fetcher",
      "version": "v1.0.0"
    },
    "container_name": "main",
    "node_name": "node-1",
    "deployment_name": "data-fetcher",
    "service_account_name": "default"
  },
  "process": {
    "pid": 3421,
    "executable": "/usr/bin/python3",
    "command_line": "python app.py"
  }
}
```

---

## ✅ Implementation Requirements

When implementing this schema:

### 1. **Lazy Metadata Loading**
- Don't fetch cloud metadata on import
- Cache metadata after first fetch
- Use 1-second timeouts for all network calls

### 2. **Thread Safety**
- Use locks for sequence counter
- Consider thread-local span stacks

### 3. **Performance**
- Metadata fetch: < 100ms total
- Event emission: < 1ms per event
- Zero overhead when telemetry disabled (DEBUG=false)

### 4. **Configuration**

Support these environment variables:

```bash
# Core config
DEBUG=true                    # Enable/disable telemetry
SERVICE_NAME=my-app           # Service name
SERVICE_VERSION=1.0.0         # Service version
ENVIRONMENT=prod              # Environment

# Output config
TELEMETRY_OUTPUT=stderr       # Where to emit (stderr, file, http)
TELEMETRY_FILE=/logs/tel.jsonl  # File path if OUTPUT=file
RECEIVER_URL=http://...       # Remote collector URL

# Sampling
SAMPLE_RATE=1.0               # 1.0 = 100%, 0.1 = 10%

# Features
ENABLE_CLOUD_METADATA=true    # Fetch cloud metadata
ENABLE_K8S_METADATA=true      # Fetch Kubernetes metadata
ENABLE_STACKTRACES=true       # Capture full stack traces
```

### 5. **Output Formats**

```python
# Default: JSONL to stderr
print(json.dumps(event), file=sys.stderr, flush=True)

# File output
with open(telemetry_file, 'a') as f:
    f.write(json.dumps(event) + '\n')

# HTTP output (non-blocking)
requests.post(receiver_url, json=event, timeout=0.5)
```

---

## 🎁 Deliverable Checklist

LLM must produce:

- [ ] Complete source file with all schema fields
- [ ] Metadata collection functions for all cloud providers
- [ ] Thread-safe sequence and span management
- [ ] Configuration via environment variables
- [ ] Example instrumented function showing entry/exit/http/error events
- [ ] Comments mapping code sections to schema fields
- [ ] Zero overhead when telemetry disabled
- [ ] Tested on Python 3.11+, Node 18+, or Go 1.21+

---

## 🔧 Optional Extensions

- **Local metadata cache** (`~/.telemetry_metadata.json`)
- **OTLP exporter** (gRPC or HTTP to OpenTelemetry Collector)
- **Sensitive field redaction** (passwords, tokens in headers/bodies)
- **Compression** (gzip for file output)
- **Batching** (buffer events, flush every N events or N seconds)
- **Tail-based sampling** (keep all events for traces with errors)

---

## 📚 Validation

Use this JSON Schema to validate events:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "Telemetry v3 Event",
  "type": "object",
  "required": ["schema", "timestamp", "event_type", "trace_id", "span_id"],
  "properties": {
    "schema": {
      "type": "object",
      "properties": {
        "format": {"const": "telemetry.v3"},
        "version": {"const": "3.0.0"}
      }
    },
    "trace_id": {"type": "string", "pattern": "^[0-9a-f]{32}$"},
    "span_id": {"type": "string", "pattern": "^[0-9a-f]{16}$"},
    "trace_flags": {"type": "string", "pattern": "^0[01]$"},
    "duration_ns": {"type": "integer", "minimum": 0},
    "duration_ms": {"type": "number", "minimum": 0}
  }
}
```

---

## 🎯 Success Criteria

Implementation is complete when:

1. ✅ Events validate against JSON Schema
2. ✅ All metadata fields populated (when available)
3. ✅ Trace/span hierarchy correct across nested functions
4. ✅ Duration tracking accurate (ns and ms)
5. ✅ Cloud metadata fetched in < 100ms
6. ✅ Zero overhead when DEBUG=false
7. ✅ Compatible with OpenTelemetry Collector
8. ✅ Compatible with Elastic Stack
9. ✅ Compatible with Datadog agent
10. ✅ Full stack traces captured on errors

---

**End of Meta-Prompt**

*Feed this prompt to any LLM to generate production-ready telemetry injection code for your target language.*
