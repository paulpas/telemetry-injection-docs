# Telemetry v2 Usage Guide

## Quick Start

### 1. Install and Configure

```bash
# Clone repository
git clone <repository-url>
cd telemetry-injector

# Create virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Configure environment
cp .env.example .env
```

### 2. Configure LLM Provider

**Option 1: Local (Free) - Ollama**

```bash
# Install Ollama from ollama.ai
ollama pull codellama

export LLM_PROVIDER=ollama
export LLM_MODEL=codellama
export LLM_BASE_URL=http://localhost:11434/v1
```

**Option 2: OpenAI**

```bash
export LLM_PROVIDER=openai
export OPENAI_API_KEY=sk-...
export LLM_MODEL=gpt-4o
```

**Option 3: Anthropic**

```bash
export LLM_PROVIDER=anthropic
export ANTHROPIC_API_KEY=sk-ant-...
export LLM_MODEL=claude-3-5-sonnet-20241022
```

### 3. Instrument Your Code

```bash
# Instrument a single file
python telemetry-inject.py --input myapp.py --output instrumented/myapp.py

# Instrument entire directory
python telemetry-inject.py --input src/ --output instrumented/
```

### 4. Enable Cloud Metadata

```bash
export ENABLE_CLOUD_METADATA=true
export SERVICE_NAME=my-service
export SERVICE_VERSION=1.0.0
export ENVIRONMENT=prod
```

### 5. Run Instrumented Code

```bash
# Run with telemetry enabled
DEBUG=true python instrumented/myapp.py

# Send telemetry to receiver
RECEIVER_URL=http://localhost:8000/telemetry python instrumented/myapp.py
```

## Understanding Telemetry Output

### Basic Function Call

**Instrumented Code:**
```python
def calculate_sum(a, b):
    result = a + b
    return result
```

**Telemetry Output:**
```json
{
  "schema": {"format": "telemetry.v2", "version": "2.0.0"},
  "timestamp": "2025-10-30T15:04:00.123Z",
  "timestamp_ns": 1730291040123456789,
  "sequence": 1,
  "event_type": "function_entry",
  "severity": "INFO",
  "function_name": "calculate_sum",
  "trace_id": "3f5c21a9af8349b0b4c6d27f56d123ab",
  "span_id": "b7ad6b7169203331",
  "parent_span_id": null,
  "message": "Function 'calculate_sum' entry",
  "attributes": {
    "function.parameters": "a, b",
    "caller.function": "__main__",
    "caller.file": "/app/main.py",
    "caller.line": 15
  },
  "service": {"name": "my-service", "version": "1.0.0"},
  "environment": "prod"
}
```

```json
{
  "event_type": "function_exit",
  "sequence": 2,
  "function_name": "calculate_sum",
  "trace_id": "3f5c21a9af8349b0b4c6d27f56d123ab",
  "span_id": "b7ad6b7169203331",
  "duration_ns": 1234567,
  "message": "Function 'calculate_sum' exit (1.23ms)",
  "timestamp": "2025-10-30T15:04:00.125Z"
}
```

### Nested Function Calls

**Code:**
```python
def process_order(order_id):
    user = fetch_user(order_id)  # Nested call
    return validate_order(user)   # Nested call
```

**Telemetry Output:**
```json
// process_order entry (root span)
{
  "event_type": "function_entry",
  "function_name": "process_order",
  "trace_id": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6",
  "span_id": "1234567890abcdef",
  "parent_span_id": null
}

// fetch_user entry (child span)
{
  "event_type": "function_entry",
  "function_name": "fetch_user",
  "trace_id": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6",  // Same trace_id!
  "span_id": "fedcba0987654321",
  "parent_span_id": "1234567890abcdef"  // Links to process_order
}

// fetch_user exit
{
  "event_type": "function_exit",
  "function_name": "fetch_user",
  "span_id": "fedcba0987654321",
  "duration_ns": 5000000
}

// validate_order entry (sibling span)
{
  "event_type": "function_entry",
  "function_name": "validate_order",
  "trace_id": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6",  // Same trace_id!
  "span_id": "abcdef1234567890",
  "parent_span_id": "1234567890abcdef"  // Links to process_order
}

// validate_order exit
{
  "event_type": "function_exit",
  "function_name": "validate_order",
  "span_id": "abcdef1234567890",
  "duration_ns": 3000000
}

// process_order exit
{
  "event_type": "function_exit",
  "function_name": "process_order",
  "span_id": "1234567890abcdef",
  "duration_ns": 10000000
}
```

**Visualization:**
```
trace_id: a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6

├─ process_order (span: 1234567890abcdef) [10ms]
   ├─ fetch_user (span: fedcba0987654321) [5ms]
   └─ validate_order (span: abcdef1234567890) [3ms]
```

### With Error Handling

**Code:**
```python
def risky_operation():
    try:
        result = dangerous_call()
        return result
    except Exception as e:
        log.error(f"Failed: {e}")
        raise
```

**Telemetry Output:**
```json
{
  "event_type": "function_entry",
  "function_name": "risky_operation",
  "trace_id": "f1e2d3c4b5a69788",
  "span_id": "a1b2c3d4e5f6a7b8"
}

{
  "event_type": "function_exit",
  "function_name": "risky_operation",
  "trace_id": "f1e2d3c4b5a69788",
  "span_id": "a1b2c3d4e5f6a7b8",
  "duration_ns": 2000000,
  "severity": "ERROR",
  "error": {
    "type": "ValueError",
    "message": "Invalid input value",
    "stacktrace": "Traceback (most recent call last):\n  File \"app.py\", line 42..."
  }
}
```

## Cloud Environment Examples

### AWS EC2

When running on EC2, telemetry automatically includes:

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
  },
  "host": {
    "name": "ip-172-31-23-45.ec2.internal",
    "ip": "172.31.23.45",
    "public_ip": "54.91.123.45"
  }
}
```

**Setup:**
```bash
# Just enable cloud metadata - detection is automatic!
export ENABLE_CLOUD_METADATA=true
```

### Kubernetes (EKS/GKE/AKS)

When running in Kubernetes, telemetry includes:

```json
{
  "cloud": {
    "provider": "aws",
    "platform": "eks"
  },
  "k8s": {
    "cluster.name": "prod-cluster",
    "namespace.name": "default",
    "pod.name": "myapp-deployment-7b8c9d-xyz",
    "pod_labels": {
      "app": "myapp",
      "version": "v1.2.3"
    },
    "deployment_name": "myapp",
    "container.name": "myapp",
    "services": ["myapp", "myapp-internal"]
  }
}
```

**Setup:**
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      containers:
      - name: myapp
        image: myapp:latest
        env:
        - name: ENABLE_CLOUD_METADATA
          value: "true"
        - name: K8S_CLUSTER_NAME
          value: "prod-cluster"
        - name: SERVICE_NAME
          value: "myapp"
        - name: SERVICE_VERSION
          value: "1.2.3"
        - name: ENVIRONMENT
          value: "prod"
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

## Practical Use Cases

### 1. Performance Debugging

**Find slow functions:**
```bash
# View telemetry output
cat telemetry.jsonl | jq 'select(.event_type=="function_exit" and .duration_ns > 1000000000)'
```

**Output:**
```json
{
  "function_name": "slow_database_query",
  "duration_ns": 5432198765,
  "message": "Function 'slow_database_query' exit (5432.20ms)"
}
```

### 2. Distributed Tracing

**Track a request across services:**

**Service A:**
```python
# Generate trace_id
trace_id = tel._generate_trace_id()
span_id = tel._generate_span_id()

# Make request to Service B
headers = {'traceparent': f'00-{trace_id}-{span_id}-01'}
response = requests.post('http://service-b/api', headers=headers)
```

**Service B:**
```python
# Extract trace context
traceparent = request.headers.get('traceparent')
# Parse: 00-{trace_id}-{parent_span_id}-01

# Use same trace_id for telemetry
tel.func_entry("handle_request", "", trace_id=trace_id, parent_span_id=parent_span_id)
```

**Result:**
Both services emit events with the same `trace_id`, enabling full request tracing!

### 3. Error Analysis

**Find all errors in production:**
```bash
cat telemetry.jsonl | jq 'select(.severity=="ERROR") | {
  function: .function_name,
  error: .error.type,
  message: .error.message,
  environment: .environment
}'
```

**Output:**
```json
{
  "function": "fetch_user_data",
  "error": "DatabaseConnectionError",
  "message": "Connection timeout after 30s",
  "environment": "prod"
}
```

### 4. Cloud Resource Analysis

**Track which instances are slowest:**
```bash
cat telemetry.jsonl | jq 'select(.event_type=="function_exit") | {
  instance: .cloud.instance_id,
  function: .function_name,
  duration_ms: (.duration_ns / 1000000)
} | select(.duration_ms > 1000)'
```

**Output:**
```json
{
  "instance": "i-0b123456789abcdef",
  "function": "process_large_file",
  "duration_ms": 5432.19
}
```

### 5. Kubernetes Pod Troubleshooting

**Find errors by pod:**
```bash
cat telemetry.jsonl | jq 'select(.severity=="ERROR") | {
  pod: .k8s."pod.name",
  namespace: .k8s."namespace.name",
  error: .error.type,
  message: .error.message
}'
```

## Integration Examples

### With Prometheus

Export metrics from telemetry:

```python
from prometheus_client import Counter, Histogram

# Track function calls
function_calls = Counter('function_calls_total', 'Total function calls',
                        ['function_name', 'service', 'environment'])

# Track function duration
function_duration = Histogram('function_duration_seconds', 'Function duration',
                             ['function_name', 'service'])

# Process telemetry events
for event in telemetry_stream:
    if event['event_type'] == 'function_entry':
        function_calls.labels(
            function_name=event['function_name'],
            service=event['service']['name'],
            environment=event['environment']
        ).inc()

    elif event['event_type'] == 'function_exit':
        function_duration.labels(
            function_name=event['function_name'],
            service=event['service']['name']
        ).observe(event['duration_ns'] / 1e9)
```

### With Grafana

Query telemetry in Grafana:

```promql
# Average function duration by service
avg(rate(function_duration_seconds_sum[5m])) by (service)

# Error rate by environment
sum(rate(function_calls_total{severity="ERROR"}[5m])) by (environment)

# Requests per second by cloud provider
sum(rate(function_calls_total[5m])) by (cloud_provider)
```

### With Jaeger

Export to Jaeger for distributed tracing:

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

# Convert telemetry events to Jaeger spans
for event in telemetry_stream:
    if event['event_type'] == 'function_entry':
        span = tracer.start_span(
            operation_name=event['function_name'],
            tags={
                'service.name': event['service']['name'],
                'cloud.provider': event.get('cloud', {}).get('provider'),
                'environment': event['environment']
            }
        )
```

## Advanced Configuration

### Sampling

For high-throughput applications, implement sampling:

```python
import random

# Sample 10% of traces
SAMPLE_RATE = 0.1

if random.random() < SAMPLE_RATE:
    tel.func_entry("high_frequency_function", parameters)
```

### Custom Attributes

Add custom metadata to events:

```python
tel.func_entry("process_order", f"order_id={order_id}")

# Attributes automatically include:
# - caller.function
# - caller.file
# - caller.line
# - function.parameters
```

### Multiple Services

Configure different services:

```bash
# Service A
export SERVICE_NAME=api-gateway
export SERVICE_VERSION=2.1.0
python service_a.py

# Service B
export SERVICE_NAME=user-service
export SERVICE_VERSION=1.5.3
python service_b.py
```

## Troubleshooting

### Problem: No cloud metadata in output

**Solution:**
```bash
# Check if enabled
echo $ENABLE_CLOUD_METADATA

# Enable it
export ENABLE_CLOUD_METADATA=true

# Clear cache to force refresh
rm ~/.telemetry_metadata.json

# Run again
python myapp.py
```

### Problem: Kubernetes metadata missing

**Solution:**
```bash
# Check K8s environment
echo $KUBERNETES_SERVICE_HOST

# Check service account exists
ls /var/run/secrets/kubernetes.io/serviceaccount/

# Set cluster name if not auto-detected
export K8S_CLUSTER_NAME=my-cluster
```

### Problem: Trace IDs not correlating

**Solution:**
```python
# Ensure functions are instrumented
# Parent function:
parent_tel = tel.func_entry("parent", "")

# Child function automatically inherits trace_id
child_tel = tel.func_entry("child", "")

# Check output - both should have same trace_id
```

### Problem: High overhead

**Solution:**
```python
# Implement sampling
if random.random() < 0.1:  # 10% sampling
    tel.func_entry("my_function", params)

# Or disable telemetry for specific paths
if not request.path.startswith('/api/'):
    return  # Skip telemetry for non-API requests
```

## Best Practices

### 1. Set Meaningful Service Names

```bash
# Good
export SERVICE_NAME=payment-api
export SERVICE_VERSION=2.1.0

# Bad
export SERVICE_NAME=app
export SERVICE_VERSION=1.0.0
```

### 2. Use Consistent Environments

```bash
# Standard environments
export ENVIRONMENT=dev      # Local development
export ENVIRONMENT=staging  # Pre-production
export ENVIRONMENT=prod     # Production
```

### 3. Enable Cloud Metadata in Production

```bash
# Always enable in cloud environments
export ENABLE_CLOUD_METADATA=true
```

### 4. Include Error Context

Functions should automatically capture exceptions with full stack traces.

### 5. Use Span Hierarchies

Instrument nested functions to build complete call trees.

## See Also

- [TELEMETRY_V2_REFERENCE.md](TELEMETRY_V2_REFERENCE.md) - Complete schema reference
- [TELEMETRY_V2_EXAMPLES.md](../TELEMETRY_V2_EXAMPLES.md) - Real-world examples
- [CLOUD_METADATA_GUIDE.md](CLOUD_METADATA_GUIDE.md) - Cloud metadata collection details
- [EXAMPLES.md](../EXAMPLES.md) - General usage examples
- [README.md](../../README.md) - Project overview
