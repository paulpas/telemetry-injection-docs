# Telemetry v2 Schema - Example Outputs

This document shows real examples of telemetry v2 events with OpenTelemetry compatibility and cloud metadata enrichment.

## Overview

Telemetry v2 includes:
- ✅ OpenTelemetry-compatible trace/span IDs
- ✅ Cloud metadata (EC2, ECS, EKS, GCP, Azure)
- ✅ Kubernetes metadata
- ✅ ECS-compatible resource fields
- ✅ Severity levels (INFO, WARN, ERROR, DEBUG, TRACE)
- ✅ RFC3339 timestamps
- ✅ Attributes for structured data

## Example 1: Function Entry (EC2 with Full Metadata)

```json
{
  "schema": {
    "format": "telemetry.v2",
    "version": "2.0.0"
  },
  "timestamp": "2025-10-30T15:04:00.123Z",
  "timestamp_ns": 1730291040123456789,
  "sequence": 1,
  "event_type": "function_entry",
  "severity": "INFO",
  "function_name": "fetch_user_data",
  "trace_id": "3f5c21a9af8349b0b4c6d27f56d123ab",
  "span_id": "b7ad6b7169203331",
  "parent_span_id": null,
  "message": "Function 'fetch_user_data' entry",
  "attributes": {
    "function.parameters": "user_id, include_profile",
    "caller.function": "process_request",
    "caller.file": "/app/handlers.py",
    "caller.line": 42,
    "program_name": "api_server.py"
  },
  "correlation_id": "func_a8b3c9d2",
  "service": {
    "name": "user-api",
    "version": "1.2.3"
  },
  "environment": "prod",
  "resource": {
    "telemetry.sdk.name": "custom-injector",
    "telemetry.sdk.version": "2.0.0",
    "runtime.name": "python",
    "runtime.version": "3.11.5",
    "os.type": "linux",
    "os.description": "Linux 5.10.0"
  },
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
  },
  "process": {
    "pid": 3421,
    "executable": "/usr/bin/python3",
    "command_line": "python api_server.py --port 8000"
  }
}
```

## Example 2: Function Exit with Error (Kubernetes on EKS)

```json
{
  "schema": {
    "format": "telemetry.v2",
    "version": "2.0.0"
  },
  "timestamp": "2025-10-30T15:04:02.456Z",
  "timestamp_ns": 1730291042456789012,
  "sequence": 2,
  "event_type": "function_exit",
  "severity": "ERROR",
  "function_name": "fetch_user_data",
  "trace_id": "3f5c21a9af8349b0b4c6d27f56d123ab",
  "span_id": "b7ad6b7169203331",
  "parent_span_id": null,
  "duration_ns": 2333333223,
  "message": "Function 'fetch_user_data' exit (2333.33ms)",
  "attributes": {
    "program_name": "api_server.py"
  },
  "correlation_id": "func_a8b3c9d2",
  "error": {
    "type": "DatabaseConnectionError",
    "message": "Connection timeout after 30s",
    "stacktrace": null
  },
  "service": {
    "name": "user-api",
    "version": "1.2.3"
  },
  "environment": "prod",
  "resource": {
    "telemetry.sdk.name": "custom-injector",
    "telemetry.sdk.version": "2.0.0",
    "runtime.name": "python",
    "runtime.version": "3.11.5",
    "os.type": "linux",
    "os.description": "Linux 5.10.0"
  },
  "cloud": {
    "provider": "aws",
    "region": "us-east-1",
    "platform": "eks",
    "metadata_source": "env"
  },
  "host": {
    "name": "ip-172-31-45-67.ec2.internal",
    "ip": "172.31.45.67"
  },
  "k8s": {
    "cluster.name": "prod-eks-cluster",
    "namespace.name": "user-services",
    "pod.name": "user-api-deployment-7b8c9d-abcde",
    "container.name": "user-api",
    "node.name": "ip-172-31-45-67.ec2.internal"
  },
  "process": {
    "pid": 1,
    "executable": "/usr/bin/python3",
    "command_line": "python api_server.py --port 8000"
  }
}
```

## Example 3: Nested Function Calls (Local Development)

### Parent Function Entry

```json
{
  "schema": {
    "format": "telemetry.v2",
    "version": "2.0.0"
  },
  "timestamp": "2025-10-30T15:04:00.100Z",
  "timestamp_ns": 1730291040100000000,
  "sequence": 1,
  "event_type": "function_entry",
  "severity": "INFO",
  "function_name": "process_order",
  "trace_id": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6",
  "span_id": "1234567890abcdef",
  "parent_span_id": null,
  "message": "Function 'process_order' entry",
  "attributes": {
    "function.parameters": "order_id, customer_id",
    "caller.function": "__main__",
    "caller.file": "/home/user/app.py",
    "caller.line": 15,
    "program_name": "app.py"
  },
  "correlation_id": "func_12ab34cd",
  "service": {
    "name": "app",
    "version": "unknown"
  },
  "environment": "dev",
  "resource": {
    "telemetry.sdk.name": "custom-injector",
    "telemetry.sdk.version": "2.0.0",
    "runtime.name": "python",
    "runtime.version": "3.11.5",
    "os.type": "linux",
    "os.description": "Linux 6.2.0"
  },
  "cloud": {
    "provider": "local",
    "metadata_source": "none"
  },
  "host": {
    "name": "dev-laptop",
    "ip": "192.168.1.100"
  },
  "process": {
    "pid": 12345,
    "executable": "/usr/bin/python3",
    "command_line": "python app.py"
  }
}
```

### Child Function Entry (Same Trace ID, Different Span ID)

```json
{
  "schema": {
    "format": "telemetry.v2",
    "version": "2.0.0"
  },
  "timestamp": "2025-10-30T15:04:00.150Z",
  "timestamp_ns": 1730291040150000000,
  "sequence": 2,
  "event_type": "function_entry",
  "severity": "INFO",
  "function_name": "validate_payment",
  "trace_id": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6",
  "span_id": "fedcba0987654321",
  "parent_span_id": "1234567890abcdef",
  "message": "Function 'validate_payment' entry",
  "attributes": {
    "function.parameters": "amount, payment_method",
    "caller.function": "process_order",
    "caller.file": "/home/user/app.py",
    "caller.line": 23,
    "program_name": "app.py",
    "parent_correlation_id": "func_12ab34cd"
  },
  "correlation_id": "func_56ef78gh",
  "service": {
    "name": "app",
    "version": "unknown"
  },
  "environment": "dev"
}
```

## Example 4: GCP Compute Engine

```json
{
  "schema": {
    "format": "telemetry.v2",
    "version": "2.0.0"
  },
  "timestamp": "2025-10-30T15:04:00.200Z",
  "timestamp_ns": 1730291040200000000,
  "sequence": 1,
  "event_type": "function_entry",
  "severity": "INFO",
  "function_name": "analyze_data",
  "trace_id": "9a8b7c6d5e4f3a2b1c0d9e8f7a6b5c4d",
  "span_id": "abcd1234efgh5678",
  "parent_span_id": null,
  "message": "Function 'analyze_data' entry",
  "attributes": {
    "function.parameters": "dataset_id",
    "caller.function": "__main__",
    "caller.file": "/app/analyzer.py",
    "caller.line": 10,
    "program_name": "analyzer.py"
  },
  "correlation_id": "func_aa11bb22",
  "service": {
    "name": "data-analyzer",
    "version": "2.0.0"
  },
  "environment": "prod",
  "resource": {
    "telemetry.sdk.name": "custom-injector",
    "telemetry.sdk.version": "2.0.0",
    "runtime.name": "python",
    "runtime.version": "3.11.5",
    "os.type": "linux",
    "os.description": "Linux 5.15.0"
  },
  "cloud": {
    "provider": "gcp",
    "region": "us-central1",
    "availability_zone": "us-central1-a",
    "account.id": "my-project-12345",
    "instance.id": "1234567890123456789",
    "machine.type": "n1-standard-4",
    "platform": "compute",
    "metadata_source": "GCE metadata"
  },
  "host": {
    "name": "instance-1",
    "ip": "10.128.0.2"
  },
  "process": {
    "pid": 8765,
    "executable": "/usr/bin/python3",
    "command_line": "python analyzer.py"
  }
}
```

## Example 5: Azure Virtual Machine

```json
{
  "schema": {
    "format": "telemetry.v2",
    "version": "2.0.0"
  },
  "timestamp": "2025-10-30T15:04:00.300Z",
  "timestamp_ns": 1730291040300000000,
  "sequence": 1,
  "event_type": "function_entry",
  "severity": "INFO",
  "function_name": "process_batch",
  "trace_id": "1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d",
  "span_id": "9876543210fedcba",
  "parent_span_id": null,
  "message": "Function 'process_batch' entry",
  "attributes": {
    "function.parameters": "batch_id",
    "caller.function": "__main__",
    "caller.file": "/app/processor.py",
    "caller.line": 8,
    "program_name": "processor.py"
  },
  "correlation_id": "func_zz99yy88",
  "service": {
    "name": "batch-processor",
    "version": "1.5.0"
  },
  "environment": "prod",
  "resource": {
    "telemetry.sdk.name": "custom-injector",
    "telemetry.sdk.version": "2.0.0",
    "runtime.name": "python",
    "runtime.version": "3.11.5",
    "os.type": "linux",
    "os.description": "Linux 5.15.0"
  },
  "cloud": {
    "provider": "azure",
    "region": "eastus",
    "instance.id": "12345678-1234-1234-1234-123456789012",
    "machine.type": "Standard_D4s_v3",
    "platform": "vm",
    "metadata_source": "Azure IMDS"
  },
  "host": {
    "name": "vm-processor-01",
    "ip": "10.0.0.4"
  },
  "process": {
    "pid": 4567,
    "executable": "/usr/bin/python3",
    "command_line": "python processor.py"
  }
}
```

## Example 6: Kubernetes with Full API Metadata

This example demonstrates comprehensive Kubernetes API metadata collection, including labels, annotations, owner references, resource limits, and service discovery - all critical for distributed tracing in Kubernetes environments.

```json
{
  "schema": {
    "format": "telemetry.v2",
    "version": "2.0.0"
  },
  "timestamp": "2025-10-30T15:04:00.400Z",
  "timestamp_ns": 1730291040400000000,
  "sequence": 1,
  "event_type": "function_entry",
  "severity": "INFO",
  "function_name": "handle_request",
  "trace_id": "f1e2d3c4b5a69788c9d0e1f2a3b4c5d6",
  "span_id": "abcd1234efgh5678",
  "parent_span_id": null,
  "message": "Function 'handle_request' entry",
  "attributes": {
    "function.parameters": "request_id, user_id",
    "caller.function": "__main__",
    "caller.file": "/app/server.py",
    "caller.line": 45,
    "program_name": "server.py"
  },
  "correlation_id": "func_kk88ll99",
  "service": {
    "name": "payment-api",
    "version": "2.1.0"
  },
  "environment": "prod",
  "resource": {
    "telemetry.sdk.name": "custom-injector",
    "telemetry.sdk.version": "2.0.0",
    "runtime.name": "python",
    "runtime.version": "3.11.5",
    "os.type": "linux",
    "os.description": "Linux 5.15.0"
  },
  "cloud": {
    "provider": "aws",
    "region": "us-west-2",
    "platform": "eks",
    "metadata_source": "env"
  },
  "host": {
    "name": "ip-10-0-45-123.ec2.internal",
    "ip": "10.0.45.123"
  },
  "k8s": {
    "cluster.name": "prod-eks-cluster",
    "namespace.name": "payments",
    "namespace_labels": {
      "environment": "production",
      "team": "payments",
      "cost-center": "engineering"
    },
    "pod.name": "payment-api-7b8c9d-xyz123",
    "pod.uid": "abc123-def456-ghi789-jkl012",
    "pod_labels": {
      "app": "payment-api",
      "version": "v2.1.0",
      "tier": "backend",
      "component": "api-server"
    },
    "pod_annotations": {
      "prometheus.io/scrape": "true",
      "prometheus.io/port": "9090",
      "istio.io/rev": "default",
      "linkerd.io/inject": "enabled",
      "kubernetes.io/config.hash": "a1b2c3d4"
    },
    "deployment_name": "payment-api",
    "replicaset_name": "payment-api-7b8c9d",
    "container.name": "payment-api",
    "container.id": "abc123def456789ghijk",
    "service_account_name": "payment-api-sa",
    "service_account_uid": "sa-uid-123456",
    "cpu_request": "500m",
    "cpu_limit": "2000m",
    "memory_request": "1Gi",
    "memory_limit": "4Gi",
    "pod_phase": "Running",
    "pod_ip": "10.244.5.67",
    "host_ip": "10.0.45.123",
    "node.name": "ip-10-0-45-123.ec2.internal",
    "node_labels": {
      "kubernetes.io/arch": "amd64",
      "kubernetes.io/os": "linux",
      "node.kubernetes.io/instance-type": "m5.xlarge",
      "topology.kubernetes.io/zone": "us-west-2a"
    },
    "services": [
      "payment-api",
      "payment-api-internal",
      "payment-metrics"
    ]
  },
  "process": {
    "pid": 1,
    "executable": "/usr/bin/python3",
    "command_line": "python server.py --port 8080"
  }
}
```

### Key Kubernetes Features for Tracing

**Owner References** - Track deployment hierarchy:
- `deployment_name`: "payment-api" (logical deployment)
- `replicaset_name`: "payment-api-7b8c9d" (specific replica set)
- Useful for: Rolling updates, canary deployments, blue-green tracing

**Labels** - Filter and correlate traces:
- `pod_labels`: Identify app, version, tier, component
- `namespace_labels`: Environment, team, cost center
- `node_labels`: Infrastructure details (instance type, zone)
- Useful for: Multi-version tracing, A/B testing, regional analysis

**Annotations** - Service mesh integration:
- `prometheus.io/*`: Metrics scraping configuration
- `istio.io/rev`, `linkerd.io/inject`: Service mesh trace IDs
- Useful for: Correlating with service mesh distributed traces

**Service Discovery** - Request routing:
- `services`: ["payment-api", "payment-api-internal", "payment-metrics"]
- Useful for: Understanding which services route to this pod

**Resource Limits** - Performance correlation:
- `cpu_request/limit`, `memory_request/limit`
- Useful for: Correlating performance issues with resource constraints

**Container Identity**:
- `container.id`: Unique container identifier
- `pod.uid`: Unique pod identifier
- Useful for: Cross-referencing with container logs, metrics

**Service Account** - Security context:
- `service_account_name`: RBAC identity
- Useful for: Security auditing, permission tracing

## Key Features Demonstrated

### 1. OpenTelemetry Compatibility
- **trace_id**: 32-character hex string (128-bit) - same across entire request trace
- **span_id**: 16-character hex string (64-bit) - unique per function call
- **parent_span_id**: Links child spans to parent spans

### 2. Cloud Provider Support
- **AWS EC2**: Via IMDSv2 with full instance metadata
- **AWS EKS**: Environment-based with Kubernetes metadata
- **GCP**: Via metadata server
- **Azure**: Via Azure IMDS
- **Local**: Fallback for development

### 3. Kubernetes Integration
- **Basic identification**: Cluster, namespace, pod, container, node names
- **Owner references**: Deployment, ReplicaSet, StatefulSet, DaemonSet, Job, CronJob
- **Labels and annotations**: Pod, namespace, and node labels for filtering and correlation
- **Service discovery**: Automatic detection of services routing to the pod
- **Resource allocation**: CPU/memory requests and limits
- **Container details**: Container ID, pod UID for log correlation
- **Service account**: RBAC identity for security auditing
- **Pod state**: Phase, IPs, and runtime status
- Auto-detected from service account, environment variables, and Kubernetes API

### 4. Resource Metadata
- Runtime information (language, version)
- OS information (type, version)
- SDK information

### 5. Service Identification
- Service name and version
- Environment (dev/staging/prod)

### 6. Severity Levels
- INFO: Normal operation
- ERROR: Exception occurred
- WARN: Anomaly detected
- DEBUG: Verbose tracing
- TRACE: Most detailed

### 7. RFC3339 Timestamps
- Human-readable ISO 8601 format with millisecond precision
- Nanosecond precision timestamp for high-resolution timing

## Configuration

Enable cloud metadata collection:

```bash
export ENABLE_CLOUD_METADATA=true
export ENVIRONMENT=prod
export SERVICE_NAME=my-service
export SERVICE_VERSION=1.0.0
```

For Kubernetes:

```bash
export K8S_CLUSTER_NAME=prod-cluster
export KUBERNETES_SERVICE_HOST=10.96.0.1
export HOSTNAME=$(hostname)  # Pod name
```

## Caching

Cloud metadata is cached to avoid repeated API calls:
- Cache location: `~/.telemetry_metadata.json`
- Loaded once on first telemetry event
- Persists across multiple runs

## Backward Compatibility

The v2 schema maintains backward compatibility:
- `correlation_id` field preserved for legacy systems
- `program_name` in attributes
- All original event types still supported

## Next Steps

See:
- [TELEMETRY_PROMPT.md](TELEMETRY_PROMPT.md) - Full v2 schema specification
- [ARCHITECTURE.md](ARCHITECTURE.md) - System architecture
- [EXAMPLES.md](EXAMPLES.md) - Usage examples
