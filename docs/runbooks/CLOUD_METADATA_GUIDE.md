# Cloud Metadata Collection Guide

## Overview

Telemetry v2 automatically collects comprehensive cloud and infrastructure metadata from:

- ✅ **AWS EC2** - via IMDSv2 (Instance Metadata Service v2)
- ✅ **AWS ECS** - via task metadata endpoint
- ✅ **AWS EKS** - via environment variables + Kubernetes API
- ✅ **GCP Compute Engine** - via metadata server
- ✅ **Azure VMs** - via Azure IMDS
- ✅ **Kubernetes** - via service account + API
- ✅ **Local** - fallback for development

## Quick Setup

```bash
# Enable cloud metadata collection
export ENABLE_CLOUD_METADATA=true

# Optional: Set service information
export SERVICE_NAME=my-service
export SERVICE_VERSION=1.0.0
export ENVIRONMENT=prod
```

That's it! Metadata is auto-detected based on your environment.

## AWS EC2 Metadata

### How It Works

EC2 metadata is collected via **IMDSv2** (Instance Metadata Service version 2) for enhanced security.

**Step 1: Acquire token**
```http
PUT http://169.254.169.254/latest/api/token
X-aws-ec2-metadata-token-ttl-seconds: 60
```

**Step 2: Query instance identity**
```http
GET http://169.254.169.254/latest/dynamic/instance-identity/document
X-aws-ec2-metadata-token: <token>
```

**Response:**
```json
{
  "accountId": "123456789012",
  "architecture": "x86_64",
  "availabilityZone": "us-east-1a",
  "imageId": "ami-0ff8a91507f77f867",
  "instanceId": "i-0b123456789abcdef",
  "instanceType": "t3.large",
  "region": "us-east-1"
}
```

### Collected Metadata

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

### IAM Permissions

No special IAM permissions required! EC2 metadata is accessible from any instance.

### Troubleshooting

**Problem: Metadata timeouts**

```bash
# Check IMDSv2 is enabled
aws ec2 describe-instances --instance-ids i-xxx --query 'Reservations[0].Instances[0].MetadataOptions'

# Enable IMDSv2 if needed
aws ec2 modify-instance-metadata-options \
    --instance-id i-xxx \
    --http-tokens required \
    --http-put-response-hop-limit 1
```

**Problem: No public IP**

Public IP is optional. If instance has no public IP, field will be null.

## AWS ECS Metadata

### How It Works

ECS metadata is collected via:
1. **Environment variables** - Basic task information
2. **Task metadata endpoint** - Detailed container information

**Environment Variables:**
- `ECS_CONTAINER_METADATA_URI_V4`
- `AWS_EXECUTION_ENV=AWS_ECS_FARGATE` or `AWS_ECS_EC2`
- `AWS_REGION`

**Metadata Endpoint:**
```http
GET $ECS_CONTAINER_METADATA_URI_V4/task
```

### Collected Metadata

```json
{
  "cloud": {
    "provider": "aws",
    "region": "us-east-1",
    "availability_zone": "us-east-1a",
    "account.id": "123456789012",
    "platform": "ecs",
    "metadata_source": "ECS metadata"
  },
  "host": {
    "name": "ip-172-31-45-67.ec2.internal",
    "ip": "172.31.45.67"
  },
  "process": {
    "pid": 1,
    "executable": "/usr/bin/python3",
    "command_line": "python app.py"
  }
}
```

### Task Definition Configuration

```json
{
  "family": "my-service",
  "containerDefinitions": [
    {
      "name": "my-container",
      "image": "myapp:latest",
      "environment": [
        {"name": "ENABLE_CLOUD_METADATA", "value": "true"},
        {"name": "SERVICE_NAME", "value": "my-service"},
        {"name": "SERVICE_VERSION", "value": "1.0.0"},
        {"name": "ENVIRONMENT", "value": "prod"}
      ]
    }
  ]
}
```

## AWS EKS Metadata

### How It Works

EKS combines **Kubernetes** metadata with **AWS** cloud information:

1. **Kubernetes API** - Pod, namespace, service information
2. **Environment variables** - EKS-specific settings
3. **Service account** - Namespace, cluster info

### Collected Metadata

```json
{
  "cloud": {
    "provider": "aws",
    "region": "us-east-1",
    "platform": "eks",
    "metadata_source": "env"
  },
  "host": {
    "name": "ip-10-0-45-123.ec2.internal",
    "ip": "10.0.45.123"
  },
  "k8s": {
    "cluster.name": "prod-eks-cluster",
    "namespace.name": "default",
    "namespace_labels": {
      "environment": "production",
      "team": "platform"
    },
    "pod.name": "myapp-7b8c9d-xyz",
    "pod.uid": "abc123-def456",
    "pod_labels": {
      "app": "myapp",
      "version": "v1.2.3",
      "tier": "backend"
    },
    "pod_annotations": {
      "prometheus.io/scrape": "true",
      "istio.io/rev": "default"
    },
    "deployment_name": "myapp",
    "replicaset_name": "myapp-7b8c9d",
    "container.name": "myapp",
    "container.id": "abc123def456",
    "service_account_name": "myapp-sa",
    "cpu_request": "500m",
    "cpu_limit": "1000m",
    "memory_request": "512Mi",
    "memory_limit": "1Gi",
    "pod_phase": "Running",
    "pod_ip": "10.244.1.5",
    "host_ip": "10.0.45.123",
    "node.name": "ip-10-0-45-123.ec2.internal",
    "node_labels": {
      "kubernetes.io/arch": "amd64",
      "node.kubernetes.io/instance-type": "m5.xlarge"
    },
    "services": ["myapp", "myapp-internal"]
  }
}
```

### Deployment Configuration

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: default
  labels:
    app: myapp
    version: v1.2.3
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
        version: v1.2.3
        tier: backend
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "9090"
    spec:
      serviceAccountName: myapp-sa
      containers:
      - name: myapp
        image: myapp:latest
        resources:
          requests:
            cpu: "500m"
            memory: "512Mi"
          limits:
            cpu: "1000m"
            memory: "1Gi"
        env:
        - name: ENABLE_CLOUD_METADATA
          value: "true"
        - name: K8S_CLUSTER_NAME
          value: "prod-eks-cluster"
        - name: SERVICE_NAME
          value: "myapp"
        - name: SERVICE_VERSION
          value: "v1.2.3"
        - name: ENVIRONMENT
          value: "prod"
```

### RBAC Configuration

For full Kubernetes API access, create a ClusterRole:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myapp-sa
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods", "namespaces", "services", "nodes"]
  verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: myapp-pod-reader
subjects:
- kind: ServiceAccount
  name: myapp-sa
  namespace: default
roleRef:
  kind: ClusterRole
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

**Note:** If RBAC is too restrictive, telemetry falls back to environment variables gracefully.

### Troubleshooting

**Problem: K8s metadata missing**

```bash
# Check service account
kubectl get serviceaccount myapp-sa

# Check RBAC
kubectl auth can-i get pods --as=system:serviceaccount:default:myapp-sa

# Check pod environment
kubectl exec -it myapp-xxx -- env | grep KUBERNETES
```

**Problem: Labels/annotations missing**

```bash
# Verify pod has labels
kubectl get pod myapp-xxx -o jsonpath='{.metadata.labels}'

# Verify RBAC allows API access
kubectl logs myapp-xxx | grep "K8s API"
```

## GCP Compute Engine Metadata

### How It Works

GCP metadata is collected via the **metadata server**:

```http
GET http://metadata.google.internal/computeMetadata/v1/instance/?recursive=true
Metadata-Flavor: Google
```

### Collected Metadata

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
  },
  "host": {
    "name": "instance-1",
    "ip": "10.128.0.2"
  }
}
```

### Instance Configuration

```bash
# Create instance with metadata
gcloud compute instances create myapp \
    --zone=us-central1-a \
    --machine-type=n1-standard-4 \
    --metadata=SERVICE_NAME=myapp,SERVICE_VERSION=1.0.0,ENVIRONMENT=prod
```

### Troubleshooting

**Problem: Metadata server timeout**

```bash
# Test metadata server
curl -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/id

# Check firewall rules allow metadata access
gcloud compute firewall-rules list | grep metadata
```

## Azure VM Metadata

### How It Works

Azure metadata is collected via **Azure IMDS**:

```http
GET http://169.254.169.254/metadata/instance?api-version=2021-02-01
Metadata: true
```

### Collected Metadata

```json
{
  "cloud": {
    "provider": "azure",
    "region": "eastus",
    "instance.id": "12345678-1234-1234-1234-123456789012",
    "machine.type": "Standard_D4s_v3",
    "platform": "vm",
    "metadata_source": "Azure IMDS"
  },
  "host": {
    "name": "vm-myapp-01",
    "ip": "10.0.0.4"
  }
}
```

### VM Configuration

```bash
# Create VM with tags
az vm create \
    --name myapp \
    --resource-group mygroup \
    --image UbuntuLTS \
    --size Standard_D4s_v3 \
    --tags SERVICE_NAME=myapp SERVICE_VERSION=1.0.0 ENVIRONMENT=prod
```

### Troubleshooting

**Problem: IMDS not responding**

```bash
# Test IMDS
curl -H "Metadata:true" \
  "http://169.254.169.254/metadata/instance?api-version=2021-02-01"

# Check VM has managed identity enabled
az vm identity show --name myapp --resource-group mygroup
```

## Kubernetes API Collection

### Pod Metadata

The Kubernetes API provides comprehensive pod information:

**API Request:**
```http
GET https://kubernetes.default.svc/api/v1/namespaces/{namespace}/pods/{pod_name}
Authorization: Bearer <service-account-token>
```

**Collected Fields:**
- Pod UID, labels, annotations
- Owner references (Deployment, ReplicaSet, StatefulSet, etc.)
- Service account name
- Resource requests/limits (CPU, memory)
- Container IDs
- Pod phase, IPs
- Node name

### Namespace Metadata

**API Request:**
```http
GET https://kubernetes.default.svc/api/v1/namespaces/{namespace}
Authorization: Bearer <service-account-token>
```

**Collected Fields:**
- Namespace labels
- Namespace annotations

### Service Discovery

**API Request:**
```http
GET https://kubernetes.default.svc/api/v1/namespaces/{namespace}/services
Authorization: Bearer <service-account-token>
```

**Process:**
1. Query all services in namespace
2. Match service selectors against pod labels
3. Identify which services route to this pod

**Example:**
```yaml
# Service
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp      # Matches pod with label app=myapp
    tier: backend   # Matches pod with label tier=backend
```

If pod has labels `{app: myapp, tier: backend}`, it will be included in `services: ["myapp"]`.

## Metadata Caching

### Cache Location

```bash
~/.telemetry_metadata.json
```

### Cache Structure

```json
{
  "cloud": {
    "provider": "aws",
    "region": "us-east-1",
    ...
  },
  "host": {
    "name": "ip-172-31-23-45.ec2.internal",
    ...
  },
  "k8s": {
    "cluster.name": "prod-cluster",
    ...
  },
  "resource": {
    "runtime.name": "python",
    ...
  },
  "cached_at": "2025-10-30T15:04:00.123Z"
}
```

### Cache Behavior

1. **First telemetry event**: Collect metadata (100-200ms)
2. **Save to cache**: Persist to `~/.telemetry_metadata.json`
3. **Subsequent events**: Load from cache (~1ms)
4. **Cache invalidation**: Manual deletion required

### Cache Management

```bash
# View cached metadata
cat ~/.telemetry_metadata.json | jq

# Clear cache (force refresh)
rm ~/.telemetry_metadata.json

# Disable caching (not recommended)
export DISABLE_METADATA_CACHE=true
```

## Performance Considerations

### Collection Times

| Source | First Call | Cached |
|--------|-----------|---------|
| **AWS EC2 IMDSv2** | 50-100ms | ~1ms |
| **GCP Metadata** | 50-100ms | ~1ms |
| **Azure IMDS** | 50-100ms | ~1ms |
| **K8s API** | 100-200ms | ~1ms |
| **Local Detection** | <1ms | ~1ms |

### Best Practices

1. **Always enable caching** (default behavior)
2. **Collect once per process** (happens automatically)
3. **Use environment variables** for static metadata (SERVICE_NAME, etc.)
4. **Pre-warm cache** in containers:

```dockerfile
# Dockerfile
RUN python -c "from src.cloud_metadata_collector import MetadataCollector; MetadataCollector().collect()"
```

## Security Considerations

### AWS IMDSv2

IMDSv2 requires token-based authentication:

```python
# Acquire token
token_response = requests.put(
    'http://169.254.169.254/latest/api/token',
    headers={'X-aws-ec2-metadata-token-ttl-seconds': '60'}
)
token = token_response.text

# Use token for requests
requests.get(
    'http://169.254.169.254/latest/dynamic/instance-identity/document',
    headers={'X-aws-ec2-metadata-token': token}
)
```

### Kubernetes RBAC

Restrict API access using RBAC:

```yaml
# Minimal permissions
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get"]
  resourceNames: ["myapp-xxx"]  # Restrict to specific pod
```

### Sensitive Data

Telemetry automatically excludes:
- Secrets
- Passwords
- API keys
- Tokens

Labels and annotations are included but should not contain sensitive data.

## Troubleshooting Guide

### Problem: No cloud metadata collected

**Diagnosis:**
```bash
# Check if enabled
echo $ENABLE_CLOUD_METADATA

# Check cache
ls ~/.telemetry_metadata.json

# Test manually
python3 << EOF
from src.cloud_metadata_collector import MetadataCollector
collector = MetadataCollector()
metadata = collector.collect()
print(metadata)
EOF
```

**Solution:**
```bash
export ENABLE_CLOUD_METADATA=true
rm ~/.telemetry_metadata.json  # Clear old cache
python myapp.py
```

### Problem: Wrong cloud provider detected

**Diagnosis:**
Check detection order:
1. EC2 (checks for http://169.254.169.254)
2. GCP (checks for http://metadata.google.internal)
3. Azure (checks for Azure IMDS)
4. Kubernetes (checks for KUBERNETES_SERVICE_HOST)
5. Local (fallback)

**Solution:**
Provider detection is automatic and based on environment. Cannot be overridden.

### Problem: Kubernetes API access denied

**Diagnosis:**
```bash
# Check RBAC
kubectl auth can-i get pods --as=system:serviceaccount:default:myapp-sa

# Check service account exists
kubectl get serviceaccount myapp-sa

# Check logs
kubectl logs myapp-xxx | grep -i "k8s\|rbac\|permission"
```

**Solution:**
```bash
# Apply RBAC configuration (see above)
kubectl apply -f rbac.yaml

# Or disable K8s API collection (falls back to env vars)
export DISABLE_K8S_API=true
```

### Problem: Slow metadata collection

**Diagnosis:**
```bash
# Check if cache is being used
ls -lh ~/.telemetry_metadata.json

# Time collection
time python3 -c "from src.cloud_metadata_collector import MetadataCollector; MetadataCollector().collect()"
```

**Solution:**
```bash
# Ensure caching is enabled (default)
unset DISABLE_METADATA_CACHE

# Pre-collect in container startup
python3 -c "from src.cloud_metadata_collector import MetadataCollector; MetadataCollector().collect()"
```

## See Also

- [TELEMETRY_V2_REFERENCE.md](TELEMETRY_V2_REFERENCE.md) - Complete schema reference
- [TELEMETRY_V2_USAGE.md](TELEMETRY_V2_USAGE.md) - Usage guide
- [TELEMETRY_V2_EXAMPLES.md](../TELEMETRY_V2_EXAMPLES.md) - Real-world examples
- [cloud_metadata_collector.py](../../src/cloud_metadata_collector.py) - Implementation
