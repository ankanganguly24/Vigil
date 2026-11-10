# Vigil architecture

## Purpose and boundaries

Vigil is an autonomous on-call SRE agent exercised against a deterministic simulated production environment. It gathers real metrics, logs, and traces; triages alerts; investigates likely incidents; and proposes remediation. A separate policy boundary evaluates every attempted tool call. A human approves consequential actions.

The first release is a single-user capstone demo. It does not connect to a real cloud provider, customer production system, or external incident-management service. The simulated environment supplies known ground truth and repeatable telemetry.

## System flow

1. Seeded workload runs against three services (`api`, `payments`, and `inventory`) with Postgres and Redis dependencies.
2. The telemetry stack stores metrics, structured logs, and OpenTelemetry traces. Chaos scenarios inject a named fault at a recorded offset.
3. Redpanda transports events to a normalizer that converts them to a versioned common event envelope. Raw source data remains queryable in the observability systems.
4. Tier-1 triage applies deterministic rules, then a small classifier where needed. It returns `ignore`, `investigate`, or `escalate` with a reason and latency measurement.
5. An investigation supervisor plans bounded work and delegates to Metrics, Logs, Traces, and Runbooks specialists. Specialists write structured findings to a shared blackboard; they do not directly execute arbitrary actions.
6. A context manager curates anomalous evidence, retrieves similar prior incidents, summarizes specialist findings, and enforces a hard token budget.
7. Every tool call passes through Aegis before execution. Aegis returns `allow`, `deny`, or `require_approval`. The trace recorder stores the request, policy result, bounded response, and chain hashes.
8. The operator dashboard presents the incident, investigation trace, approval queue, audit entries, replay controls, evaluation results, and postmortem draft.

## Component responsibilities

| Component | Responsibility | Initial technology |
| --- | --- | --- |
| Simulated app | API, payments, and inventory service behavior | Containers in Docker Compose; service implementation language remains open |
| Dependencies | Relational data, connection pools, and cache behavior | Postgres 16, Redis 7 |
| Metrics and logs | Store and query metrics and logs | Prometheus/PromQL, Loki/LogQL |
| Traces | Store distributed traces and GenAI spans | Jaeger plus OpenTelemetry GenAI semantic conventions |
| Dashboards | Explore telemetry and Vigil operations | Grafana for telemetry; Next.js for the operator product surface |
| Workload generator | Seeded normal traffic and non-incident windows | Python script with explicit seed |
| Chaos injector | Named faults and adversarial log injection | CLI and `make scenario NAME=...` |
| Ingestion | Event normalization and routing | Redpanda plus normalizer service |
| Tier-1 triage | Filter noise and route alerts cheaply | Go serving rules/classifier with ONNX Runtime |
| Investigation | Delegate evidence collection and synthesize findings | Custom Python supervisor and specialists |
| LLM routing | Choose providers and cost tiers | LiteLLM behind a provider abstraction |
| Aegis policy | Deterministic tool authorization and approval checks | OPA/Rego |
| Firewall and replay | Enforce tool boundary and replay mocked calls | Go service, SHA-256 hash chain, deterministic replay state machine |
| Evaluation store | Store traces and evaluation observations | Self-hosted Langfuse |
| Evaluation runner | Run golden incidents and enforce regression gates | Promptfoo, pytest, and CI workflow |
| Backend gateway | Serve low-latency product API | Go |
| Backend agent service | Serve investigation orchestration and model calls | FastAPI (Python) |
| Operator surface | Incidents, approvals, postmortems, and eval results | Next.js, React, Tailwind, SSE |
| Deployment | Host API, agent service, workers, and database | Render; Docker Compose remains the local development path |

## Tool boundary and replay

Agents call typed tools through a single policy-enforced interface. Tools expose narrow operations such as bounded PromQL/LogQL queries, trace lookup, runbook retrieval, and explicitly scoped remediation. The policy layer validates tool identity, normalized arguments, role, service, environment, approval token, rate limit, and blast radius before dispatch.

Each action is recorded with an input hash, policy version and decision, bounded output or error, timestamp, prior hash, and resulting hash. Replay substitutes recorded tool outputs at the tool boundary and reruns the orchestration state machine without model-provider or external calls. Replay fidelity is checked against the recorded state transitions and final findings.

The trace is tamper-evident rather than an absolute guarantee of tamper-proof storage. A hash chain detects edits or reordering relative to a trusted chain head; external anchoring is outside the initial scope.

## Data and context flow

The event normalizer adds a schema version, event type, source, service, environment, event time, ingest time, trace/incident identifiers when known, labels, and a payload. It preserves source timestamps and marks malformed or incomplete records instead of silently dropping them. Metrics, logs, and traces remain in their native stores and are queried through allowlisted tools.

The investigation layer avoids inserting raw telemetry dumps into model context. It extracts anomalies, groups duplicates, preserves provenance, summarizes at specialist and supervisor boundaries, and retrieves similar labeled incidents. A hard per-incident token budget applies across turns; when exhausted, the system summarizes further or escalates to a human.

## Preferred implementation decisions

- **Go for latency-sensitive boundaries:** use Go for triage serving, the backend gateway, and the firewall/replay service. Measure triage against the <15ms goal and firewall policy-path overhead against the <1ms p99 goal; neither target is a measured result yet.
- **Python for agent iteration:** use a custom Python supervisor and a separate FastAPI agent service. Keep the investigation state machine, evidence handoffs, and retry bounds explicit and replayable.
- **Redpanda:** use its Kafka API for streaming while keeping compatibility with common Kafka tooling and reducing local/CI operations.
- **Rules and ONNX before LLM investigation:** deterministic rules handle known patterns; Go-hosted ONNX inference handles residual fast triage. Only investigate/escalate decisions proceed to model-backed agents.
- **LiteLLM:** route multiple providers by task tier through an abstraction. Record provider, model, version, and cost assumptions with evaluation results.
- **OPA/Rego:** express allow, deny, and approval policies declaratively, then measure the compiled policy path under realistic requests.
- **Langfuse with Promptfoo and pytest:** self-host Langfuse for evaluation and trace storage; use Promptfoo for declarative scenario prompts and pytest for deterministic harness/policy tests and CI gates.
- **Render:** deploy the full-stack demo, workers, and database on Render. Keep Docker Compose as the local reproducible path and document any differences between local and hosted configuration.

## Reliability and failure behavior

- If ingestion is delayed, retain source timestamps and expose pipeline lag; do not label stale evidence as current.
- If a telemetry query fails, record the failure as evidence and retry only within a bounded policy.
- If the policy engine is unavailable or returns an invalid result, fail closed for actions; read-only behavior may also be disabled until policy health is restored.
- If the context or cost budget is reached, summarize, stop further model calls, or escalate. Do not silently exceed the budget.
- If a model provider is unavailable, preserve the trace, mark the investigation incomplete, and offer replay or human review.
- If replay input is incomplete or a hash check fails, stop replay and report the first divergence.

## Open design questions

- Classifier dataset and selection between a gradient-boosted model and a distilled transformer.
- Embedding model and local vector-store choice for similar-incident retrieval.
- Render service topology, database sizing, and deployment constraints.
- Trace retention and redaction policy for any future external or personally identifying fields.

Decisions will be updated here when a small prototype or evaluation provides evidence.
