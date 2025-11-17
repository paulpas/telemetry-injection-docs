

🧠 Master Prompt: Cloud-Enriched Telemetry Injector Implementation

> Goal:
Implement an automatic telemetry injector that augments arbitrary source code with runtime instrumentation and cloud metadata enrichment.
The injector must produce JSONL-formatted telemetry events compatible with OpenTelemetry and Elastic Common Schema (ECS), and it must gather contextual data from the runtime environment (e.g., EC2, ECS, EKS, or local Linux host).

All outputs must follow the schema and calculation logic described below.




---

🧩 Schema Definition

Each JSON line is a single event.
Use this exact top-level structure (camelCase or snake_case is acceptable, but keep consistent):

{
  "schema": { "format": "telemetry.v2", "version": "2.0.0" },
  "timestamp": "<RFC3339 timestamp in UTC>",
  "timestamp_ns": "<integer nanoseconds since epoch>",
  "sequence": "<incremental event number per process>",
  "event_type": "<string describing event type>",
  "severity": "<INFO|WARN|ERROR|DEBUG|TRACE>",
  "function_name": "<if applicable>",
  "trace_id": "<32-hex string>",
  "span_id": "<16-hex string>",
  "parent_span_id": "<16-hex string or null>",
  "duration_ns": "<integer or null>",
  "message": "<short human-readable description>",
  "attributes": { "<arbitrary structured key-value pairs>" },
  "service": {
    "name": "<logical service or module name>",
    "version": "<semver or git sha>"
  },
  "environment": "<dev|staging|prod>",
  "resource": {
    "telemetry.sdk.name": "custom-injector",
    "telemetry.sdk.version": "2.0.0",
    "runtime.name": "<python|nodejs|go|rust>",
    "runtime.version": "<detected runtime version>",
    "os.type": "<linux|windows|darwin>",
    "os.description": "<os release string>"
  },
  "cloud": {
    "provider": "<aws|gcp|azure|local>",
    "region": "<region name or null>",
    "availability_zone": "<availability zone or null>",
    "account.id": "<cloud account id or null>",
    "instance.id": "<EC2/ECS instance id or null>",
    "machine.type": "<instance type or VM size>",
    "image.id": "<AMI or image id>",
    "platform": "<ec2|ecs|eks|lambda|vm>",
    "metadata_source": "<how values were obtained: IMDSv2|env|file>"
  },
  "host": {
    "name": "<system hostname>",
    "ip": "<primary private ip>",
    "public_ip": "<public ip if exists>"
  },
  "k8s": {
    "cluster.name": "<cluster name>",
    "namespace.name": "<namespace>",
    "pod.name": "<pod name>",
    "container.name": "<container name>",
    "node.name": "<node host name>"
  },
  "process": {
    "pid": "<process id>",
    "executable": "<binary or interpreter>",
    "command_line": "<command line invocation>"
  },
  "error": {
    "type": "<exception type>",
    "message": "<error message>",
    "stacktrace": "<optional stacktrace>"
  }
}


---

⚙️ Calculation & Data Collection Logic (expressed in English)

The LLM must implement the following logic to populate each field:

1. Timestamp Fields

timestamp: current time in UTC formatted as RFC3339 with millisecond precision.
→ Example: datetime.utcnow().isoformat(timespec="milliseconds") + "Z".

timestamp_ns: integer nanoseconds since Unix epoch.
→ Example: (time.time_ns()).


2. Sequence

Start at 1 for each process.

Increment every time a new telemetry event is emitted.


3. Trace & Span Identifiers

trace_id: 16 random bytes (128-bit) rendered as 32-character lowercase hex string.

span_id: 8 random bytes (64-bit) rendered as 16-character lowercase hex string.

parent_span_id: use the parent’s span_id if the event occurs inside a known parent function, otherwise null.

These IDs persist across function_entry, http_request, http_response, function_exit, etc.


4. Duration

For function_exit events, calculate duration_ns = (exit_time_ns − entry_time_ns).

Derive duration_ms for human readability if desired.


5. Severity

Default to "INFO".

Use "ERROR" when event_type = "exception" or when an error is captured.

Use "DEBUG" for verbose tracing, "WARN" for anomalies.


6. Service & Environment

service.name: inferred from module, repo, or project folder name.

service.version: inferred from Git tag, commit hash, or environment variable SERVICE_VERSION.

environment: environment variable ENVIRONMENT or default "dev".


7. Resource Metadata

Detect runtime with system libraries:

runtime.name = language or interpreter name.

runtime.version = output of python --version, node -v, etc.


os.type and os.description: from platform.system() and platform.release().


8. Cloud Metadata

Try EC2 metadata first (IMDSv2):

Acquire token:
PUT http://169.254.169.254/latest/api/token with X-aws-ec2-metadata-token-ttl-seconds: 60.

Use token to query:
GET http://169.254.169.254/latest/dynamic/instance-identity/document

Extract:

region → "region"

instanceId → "instance.id"

accountId → "account.id"

instanceType → "machine.type"

imageId → "image.id"

availabilityZone → "availability_zone"



If IMDS fails, check ECS or EKS environment variables:

ECS_CONTAINER_METADATA_URI, KUBERNETES_SERVICE_HOST, AWS_REGION, etc.


Fallback: "provider": "local" with "metadata_source": "none".


9. Host Info

host.name: from socket.gethostname().

host.ip: primary private IP from socket.gethostbyname(hostname).

host.public_ip: external IP by calling a metadata or public IP service if allowed.


10. Kubernetes Context

Populate when /var/run/secrets/kubernetes.io/serviceaccount/namespace exists:

cluster.name from env K8S_CLUSTER_NAME.

namespace.name from service account file.

pod.name from env HOSTNAME.

container.name from runtime container metadata.



11. Process Info

pid = current process id (os.getpid()).

executable = path of binary or interpreter (sys.executable).

command_line = full command line arguments (sys.argv).


12. Error Fields

When an exception occurs, capture:

error.type = exception class name.

error.message = str(e).

error.stacktrace = formatted traceback.



13. Attributes

Key-value pairs specific to the event type.
Example for HTTP:

{
  "http.method": "GET",
  "http.url": "https://api.example.com/data",
  "http.status_code": 200,
  "http.user_agent": "MyApp/1.0"
}

Example for DB call:

{
  "db.system": "postgresql",
  "db.statement": "SELECT * FROM users WHERE id=1"
}



---

🧩 Example event sequence (expected output)

{"schema":{"format":"telemetry.v2"},"timestamp":"2025-10-30T15:04:00.001Z","timestamp_ns":1730291040123456789,"sequence":1,"event_type":"function_entry","function_name":"fetch_data","trace_id":"3f5c21a9af8349b0b4c6d27f56d123ab","span_id":"b7ad6b7169203331","service":{"name":"data-fetcher","version":"1.0.0"},"environment":"prod","resource":{"runtime.name":"python","runtime.version":"3.11.5","os.type":"linux","os.description":"Ubuntu 22.04"},"cloud":{"provider":"aws","region":"us-east-1","availability_zone":"us-east-1a","account.id":"123456789012","instance.id":"i-0b123456789abcdef","machine.type":"t3.large","image.id":"ami-0ff8a91507f77f867","metadata_source":"imds"},"host":{"name":"ip-172-31-23-45.ec2.internal","ip":"172.31.23.45","public_ip":"54.91.123.45"},"process":{"pid":3421,"executable":"/usr/bin/python3","command_line":"python app.py"}}


---

✅ Implementation Requirements for the LLM

When you feed this prompt into an LLM:

1. It should generate code in the requested language (e.g., Python, Node.js, Go, Rust).


2. The code must:

Automatically gather metadata according to the logic above.

Inject telemetry event creation calls into existing code functions.

Maintain correlation between function entries/exits and HTTP calls.

Emit events to stdout or to a specified file in JSONL format.



3. Include configuration options (env vars or CLI args) for:

OUTPUT_PATH

SERVICE_NAME

ENVIRONMENT

ENABLE_CLOUD_METADATA

COLLECT_STACKTRACE_ON_ERROR





---

🧰 Optional Extensions

The LLM may optionally:

Implement a local cache of metadata (e.g., ~/.telemetry_metadata.json) so repeated metadata calls are avoided.

Offer a sampling mechanism (sample_rate field) to reduce event volume.

Support exporting to OpenTelemetry Collector via OTLP/gRPC or HTTP.

Provide pluggable filters/redactors for sensitive fields.



---

🧩 Deliverable expectation

The LLM’s output must include:

1. A complete source file implementing the above schema and data collection.


2. Example output events from a simple instrumented function.


3. Comments explaining which part of the schema each section of code corresponds to.
