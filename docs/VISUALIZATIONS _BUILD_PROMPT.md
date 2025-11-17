📝 AI Agent Prompt: Visualize Fine-Grained Source-Instrumented Telemetry

Goal:
Build a visualization system for source-instrumented telemetry JSONL that captures function entry/exit, variable changes, loop iterations, HTTP/DB calls, durations, and environment/cloud metadata (EC2, ECS, Kubernetes). Existing tools like Jaeger cannot visualize this fully because the telemetry includes more granular events than standard distributed tracing.


---

1️⃣ Requirements

1. Input

JSONL file or stream with events using a schema like:




{
  "sequence": 1,
  "timestamp": "2025-10-30T15:04:00.001Z",
  "event_type": "function_entry",
  "function_name": "fetch_data",
  "trace_id": "3f5c21a9af8349b0b4c6d27f56d123ab",
  "span_id": "b7ad6b7169203331",
  "parent_span_id": null,
  "duration_ns": null,
  "attributes": {...},
  "service": {...},
  "environment": "prod",
  "cloud": {...},
  "host": {...},
  "k8s": {...},
  "process": {...},
  "error": {...}
}

2. Storage

Persist events for querying and aggregation. Options: Elasticsearch, ClickHouse, DuckDB, or SQLite for prototype.



3. Data Processing

Parse JSONL into structured in-memory objects.

Correlate events by trace_id and span_id.

Aggregate metrics:

Function durations (mean, p50, p95)

Loop iterations and durations

Variable mutation frequency

HTTP/DB latency per function


Build call graphs (parent-child function relationships)

Generate timeline sequences of events



4. Visualization

Interactive web dashboard using React or similar framework.

Visualizations to include:

Timeline/Gantt chart of function entries/exits, HTTP calls, loops

Directed call graphs weighted by duration

Heatmaps for variable mutation frequency

Tables for HTTP requests, variable changes, errors

Drill-down views for trace/span and variable history


Optional: parallel coordinates or correlation matrices for multi-dimensional telemetry



5. Metadata Display

Include host, cloud, process, and Kubernetes metadata in dashboards for filtering and context.



6. Performance

Must handle high-volume telemetry (thousands of events per second).

Use caching or aggregation to reduce dashboard lag.



7. Deliverables

Full web-based dashboard with:

Backend service ingesting JSONL

Data aggregation logic

Frontend visualizations


Example screenshots or mock data visualizations

Comments explaining mapping between telemetry fields and visualizations





---

2️⃣ Additional Instructions for the AI Agent

Focus on clarity and modularity: separate ingestion, aggregation, and visualization layers.

Use open-source libraries wherever possible:

Charts: D3.js, Recharts, Plotly

Graphs: Cytoscape.js, Vis.js

Backend: Node.js, Python Flask/FastAPI, or Go

Database: DuckDB, ClickHouse, or SQLite


Ensure interactive drill-downs: click on a function → see variable changes, loops, HTTP calls.

Provide a minimal working prototype with synthetic telemetry data.

Optimize for high dimensional telemetry (variable + function + cloud + host + process data).
