# Telemetry v2 Schema - Complete Reference

## Overview

Telemetry v2 is a modern observability schema that provides:

- ✅ **OpenTelemetry Compatibility**: 128-bit trace IDs, 64-bit span IDs, parent span linking
- ✅ **Cloud Native**: Auto-detects EC2, ECS, EKS, GCP, Azure, and Kubernetes metadata
- ✅ **ECS Compatible**: Follows Elastic Common Schema conventions
- ✅ **RFC3339 Timestamps**: Human-readable with nanosecond precision
- ✅ **Structured Attributes**: Flexible key-value metadata
- ✅ **Severity Levels**: INFO, WARN, ERROR, DEBUG, TRACE

## Schema Version

```json
{
  "schema": {
    "format": "telemetry.v2",
    "version": "2.0.0"
  }
}
```

All v2 events include this schema identifier for compatibility.

## Core Fields

### Timestamps

```json
{
  "timestamp": "2025-10-30T15:04:00.123Z",
  "timestamp_ns": 1730291040123456789
}
```

- **timestamp**: RFC3339 format in UTC with millisecond precision
- **timestamp_ns**: Integer nanoseconds since Unix epoch (high-resolution timing)

### Event Identification

```json
{
  "sequence": 1,
  "event_type": "function_entry",
  "severity": "INFO",
  "message": "Function 'fetch_data' entry"
}
```

- **sequence**: Monotonically increasing event counter (per process)
- **event_type**: Event classification (function_entry, function_exit, http_request, etc.)
- **severity**: Log level (INFO, WARN, ERROR, DEBUG, TRACE)
- **message**: Human-readable event description

### OpenTelemetry Trace Context

```json
{
  "trace_id": "3f5c21a9af8349b0b4c6d27f56d123ab",
  "span_id": "b7ad6b7169203331",
  "parent_span_id": "a1b2c3d4e5f6a7b8"
}
```

- **trace_id**: 32-char hex string (128-bit) - same across entire request trace
- **span_id**: 16-char hex string (64-bit) - unique per function/operation
- **parent_span_id**: Links child spans to parent (null for root spans)

### Function Context

```json
{
  "function_name": "fetch_user_data",
  "duration_ns": 2333333223,
  "attributes": {
    "function.parameters": "user_id, include_profile",
    "caller.function": "process_request",
    "caller.file": "/app/handlers.py",
    "caller.line": 42
  }
}
```

- **function_name**: Name of instrumented function
- **duration_ns**: Execution time in nanoseconds (function_exit only)
- **attributes**: Flexible key-value metadata

### Error Information

```json
{
  "error": {
    "type": "DatabaseConnectionError",
    "message": "Connection timeout after 30s",
    "stacktrace": "Traceback (most recent call last):\n..."
  }
}
```

- **error.type**: Exception class name
- **error.message**: Error message
- **error.stacktrace**: Full stack trace (optional)

### Legacy Compatibility

```json
{
  "correlation_id": "func_a8b3c9d2"
}
```

- **correlation_id**: Legacy identifier for backward compatibility

## Service Metadata

### Service Identification

```json
{
  "service": {
    "name": "user-api",
    "version": "1.2.3"
  },
  "environment": "prod"
}
```

- **service.name**: Logical service name (from SERVICE_NAME env or inferred)
- **service.version**: Service version (from SERVICE_VERSION env or git)
- **environment**: Deployment environment (dev, staging, prod)

### Resource Metadata

```json
{
  "resource": {
    "telemetry.sdk.name": "custom-injector",
    "telemetry.sdk.version": "2.0.0",
    "runtime.name": "python",
    "runtime.version": "3.11.5",
    "os.type": "linux",
    "os.description": "Linux 5.10.0"
  }
}
```

- **telemetry.sdk.name**: SDK identifier
- **telemetry.sdk.version**: SDK version
- **runtime.name**: Programming language
- **runtime.version**: Language version
- **os.type**: Operating system (linux, windows, darwin)
- **os.description**: OS version string

## Cloud Metadata

### AWS EC2

```json
{
  "cloud": {
    "provider": "aws",
    "region": "us-east-1",
    "availability_zone": "us-east-1a",
    "account.id": "123456789012",
    "instance.id": "i-0b123456789abcdef",
    "machine.type": "t3.large",
    "image.id": "ami-0ff8a91507f77f867",
    "platform": "ec2",
    "metadata_source": "IMDSv2"
  }
}
```

- **provider**: Cloud provider (aws, gcp, azure, local)
- **region**: Cloud region
- **availability_zone**: Availability zone
- **account.id**: AWS account ID
- **instance.id**: EC2 instance ID
- **machine.type**: Instance type
- **image.id**: AMI ID
- **platform**: Platform type (ec2, ecs, eks, lambda)
- **metadata_source**: How metadata was obtained (IMDSv2, env, GCE metadata, Azure IMDS)

### AWS EKS / Kubernetes

```json
{
  "cloud": {
    "provider": "aws",
    "region": "us-east-1",
    "platform": "eks",
    "metadata_source": "env"
  },
  "k8s": {
    "cluster.name": "prod-eks-cluster",
    "namespace.name": "user-services",
    "namespace_labels": {
      "environment": "production",
      "team": "platform"
    },
    "pod.name": "user-api-deployment-7b8c9d-abcde",
    "pod.uid": "abc123-def456-ghi789",
    "pod_labels": {
      "app": "user-api",
      "version": "v1.2.3",
      "tier": "backend"
    },
    "pod_annotations": {
      "prometheus.io/scrape": "true",
      "istio.io/rev": "default"
    },
    "deployment_name": "user-api",
    "replicaset_name": "user-api-7b8c9d",
    "container.name": "user-api",
    "container.id": "abc123def456",
    "service_account_name": "user-api-sa",
    "cpu_request": "500m",
    "cpu_limit": "1000m",
    "memory_request": "512Mi",
    "memory_limit": "1Gi",
    "pod_phase": "Running",
    "pod_ip": "10.244.1.5",
    "host_ip": "172.31.45.67",
    "node.name": "ip-172-31-45-67.ec2.internal",
    "services": ["user-api", "user-api-internal"]
  }
}
```

### GCP Compute Engine

```json
{
  "cloud": {
    "provider": "gcp",
    "region": "us-central1",
    "availability_zone": "us-central1-a",
    "account.id": "my-project-12345",
    "instance.id": "1234567890123456789",
    "machine.type": "n1-standard-4",
    "platform": "compute",
    "metadata_source": "GCE metadata"
  }
}
```

### Azure VM

```json
{
  "cloud": {
    "provider": "azure",
    "region": "eastus",
    "instance.id": "12345678-1234-1234-1234-123456789012",
    "machine.type": "Standard_D4s_v3",
    "platform": "vm",
    "metadata_source": "Azure IMDS"
  }
}
```

### Local Development

```json
{
  "cloud": {
    "provider": "local",
    "metadata_source": "none"
  }
}
```

## Host Metadata

```json
{
  "host": {
    "name": "ip-172-31-23-45.ec2.internal",
    "ip": "172.31.23.45",
    "public_ip": "54.91.123.45"
  }
}
```

- **host.name**: System hostname
- **host.ip**: Primary private IP address
- **host.public_ip**: Public IP address (if available)

## Process Metadata

```json
{
  "process": {
    "pid": 3421,
    "executable": "/usr/bin/python3",
    "command_line": "python api_server.py --port 8000"
  }
}
```

- **process.pid**: Process ID
- **process.executable**: Binary or interpreter path
- **process.command_line**: Full command line invocation

## Event Types

### function_entry

Emitted when a function is entered.

```json
{
  "event_type": "function_entry",
  "function_name": "fetch_user_data",
  "trace_id": "3f5c21a9af8349b0b4c6d27f56d123ab",
  "span_id": "b7ad6b7169203331",
  "parent_span_id": null,
  "message": "Function 'fetch_user_data' entry",
  "attributes": {
    "function.parameters": "user_id, include_profile",
    "caller.function": "process_request",
    "caller.file": "/app/handlers.py",
    "caller.line": 42
  }
}
```

### function_exit

Emitted when a function returns or throws an exception.

```json
{
  "event_type": "function_exit",
  "function_name": "fetch_user_data",
  "trace_id": "3f5c21a9af8349b0b4c6d27f56d123ab",
  "span_id": "b7ad6b7169203331",
  "duration_ns": 2333333223,
  "message": "Function 'fetch_user_data' exit (2333.33ms)",
  "error": {
    "type": "DatabaseConnectionError",
    "message": "Connection timeout after 30s"
  }
}
```

### http_request

Emitted when an HTTP request is made.

```json
{
  "event_type": "http_request",
  "trace_id": "3f5c21a9af8349b0b4c6d27f56d123ab",
  "span_id": "c8d9e1f2a3b4c5d6",
  "parent_span_id": "b7ad6b7169203331",
  "attributes": {
    "http.method": "POST",
    "http.url": "https://api.example.com/users",
    "http.user_agent": "MyApp/1.0"
  }
}
```

### http_response

Emitted when an HTTP response is received.

```json
{
  "event_type": "http_response",
  "trace_id": "3f5c21a9af8349b0b4c6d27f56d123ab",
  "span_id": "c8d9e1f2a3b4c5d6",
  "duration_ns": 125000000,
  "attributes": {
    "http.status_code": 200,
    "http.response_content_length": 1024
  }
}
```

## Configuration

### Environment Variables

```bash
# Enable cloud metadata collection
export ENABLE_CLOUD_METADATA=true

# Service identification
export SERVICE_NAME=my-service
export SERVICE_VERSION=1.0.0
export ENVIRONMENT=prod

# Kubernetes (usually auto-detected)
export K8S_CLUSTER_NAME=prod-cluster
export KUBERNETES_SERVICE_HOST=10.96.0.1

# Telemetry endpoint
export RECEIVER_URL=http://localhost:8000/telemetry

# Debug mode
export DEBUG=true
```

### Metadata Caching

Cloud metadata is cached to `~/.telemetry_metadata.json` to avoid repeated API calls:

```json
{
  "cloud": {...},
  "host": {...},
  "k8s": {...},
  "resource": {...},
  "cached_at": "2025-10-30T15:04:00.123Z"
}
```

Cache is loaded once on first telemetry event and persists across runs.

## OpenTelemetry Integration

### Trace Context Propagation

Telemetry v2 uses W3C Trace Context compatible IDs:

```
traceparent: 00-3f5c21a9af8349b0b4c6d27f56d123ab-b7ad6b7169203331-01
              │  └─────────────┬─────────────────┘  └──────┬───────┘  │
              │              trace_id              span_id            flags
             version
```

### Span Hierarchy

```
trace_id: 3f5c21a9af8349b0b4c6d27f56d123ab

├─ span_id: b7ad6b7169203331 (root)
│  └─ function: process_request
│     ├─ span_id: c8d9e1f2a3b4c5d6 (child)
│     │  └─ function: fetch_user_data
│     │     └─ span_id: d1e2f3a4b5c6d7e8 (grandchild)
│     │        └─ function: query_database
```

## Integration with Observability Platforms

### Jaeger

Export to Jaeger using trace_id and span_id:

```python
from jaeger_client import Config

config = Config(
    config={
        'sampler': {'type': 'const', 'param': 1},
        'logging': True,
    },
    service_name='my-service',
)
tracer = config.initialize_tracer()
```

### Elastic APM

Compatible with Elastic Common Schema (ECS):

```json
{
  "service.name": "user-api",
  "trace.id": "3f5c21a9af8349b0b4c6d27f56d123ab",
  "transaction.id": "b7ad6b7169203331",
  "cloud.provider": "aws",
  "cloud.region": "us-east-1"
}
```

### Grafana/Prometheus

Use attributes for metrics labels:

```promql
# Query by service
telemetry_duration_seconds{service_name="user-api"}

# Query by environment
telemetry_errors_total{environment="prod"}

# Query by cloud provider
telemetry_requests_total{cloud_provider="aws"}
```

## Best Practices

### 1. Always Propagate Trace Context

```python
# Pass trace_id to downstream services
headers = {
    'traceparent': f'00-{trace_id}-{span_id}-01'
}
```

### 2. Use Structured Attributes

```python
# Good: Structured
attributes = {
    "http.method": "POST",
    "http.url": url,
    "http.status_code": 200
}

# Bad: Unstructured
message = f"POST {url} returned 200"
```

### 3. Set Appropriate Severity

```python
# INFO: Normal operation
severity = "INFO"

# WARN: Degraded performance
if latency > threshold:
    severity = "WARN"

# ERROR: Exceptions
if exception:
    severity = "ERROR"
```

### 4. Include Error Context

```python
error = {
    "type": type(e).__name__,
    "message": str(e),
    "stacktrace": traceback.format_exc()
}
```

## Performance Considerations

### Metadata Collection

- **Cloud metadata**: ~100-200ms first call, then cached
- **Kubernetes API**: ~50-100ms first call, then cached
- **Per-event overhead**: ~10-50μs (nanosecond timestamps, ID generation)

### Sampling

For high-throughput applications, implement sampling:

```python
import random

# Sample 10% of traces
if random.random() < 0.1:
    tel.func_entry("my_function", parameters)
```

## Troubleshooting

### Cloud metadata not appearing?

```bash
# Enable cloud metadata collection
export ENABLE_CLOUD_METADATA=true

# Check cache
cat ~/.telemetry_metadata.json

# Delete cache to force refresh
rm ~/.telemetry_metadata.json
```

### Kubernetes metadata missing?

```bash
# Check K8s environment
echo $KUBERNETES_SERVICE_HOST

# Check service account
ls /var/run/secrets/kubernetes.io/serviceaccount/

# Check RBAC permissions (may need cluster role)
kubectl auth can-i get pods --as=system:serviceaccount:default:default
```

### Trace IDs not correlating?

```python
# Ensure parent span is passed correctly
parent_tel = tel.func_entry("parent_function", "")
# ...
child_tel = tel.func_entry("child_function", "")  # Auto-inherits trace_id
```

## Migration from v1

v1 telemetry used a different format. Key differences:

| Feature | v1 | v2 |
|---------|----|----|
| **Trace IDs** | correlation_id only | trace_id + span_id |
| **Cloud metadata** | None | Full EC2/ECS/EKS/GCP/Azure |
| **Timestamps** | timestamp_ns only | timestamp + timestamp_ns |
| **Schema version** | None | {"format": "telemetry.v2"} |
| **Severity** | Implicit | Explicit (INFO/WARN/ERROR) |
| **OpenTelemetry** | Not compatible | Fully compatible |

v2 maintains backward compatibility with `correlation_id` field.

## See Also

- [TELEMETRY_PROMPT.md](../TELEMETRY_PROMPT.md) - Complete v2 schema specification
- [TELEMETRY_V2_EXAMPLES.md](../TELEMETRY_V2_EXAMPLES.md) - Real-world examples
- [ARCHITECTURE.md](../ARCHITECTURE.md) - System architecture
- [EXAMPLES.md](../EXAMPLES.md) - Usage examples
