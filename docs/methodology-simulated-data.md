# Why Vigil uses simulated production data

Vigil uses a controlled workload and named chaos scenarios to build and evaluate incident response. The workload is synthetic; the infrastructure and telemetry interfaces are real software used by production teams.

## What is real and what is simulated

| Part | Treatment |
| --- | --- |
| API, payments, and inventory services | Simulated application behavior in containers |
| Traffic | Synthetic, seeded, and repeatable |
| Faults | Injected by a first-class chaos CLI with known labels |
| Datastores | Real Postgres and Redis instances |
| Metrics and queries | Prometheus and real PromQL |
| Logs and queries | Loki and real LogQL |
| Traces | OpenTelemetry spans stored and inspected in Jaeger |
| Event transport | Redpanda using the Kafka API |
| Customer production data | Not used |

## Why simulation is the right evaluation method

### Labeled ground truth

An injected fault has a known cause and onset. Each scenario record contains the scenario name, seed, command schedule, expected root cause, expected evidence, and remediation class. The evaluator can compare the agent's diagnosis to this label.

### Repeatable inputs

The same seed and schedule should reproduce the workload and fault window. This makes regression tests meaningful and enables offline replay of a recorded agent session with mocked tool responses.

### Safe adversarial testing

The environment can include malicious instructions in log text and test whether they cross the tool boundary. The agent is allowed to observe untrusted content; only the deterministic policy layer can authorize actions.

### Reproducible demonstration

A reviewer can start the stack and trigger a named scenario without access to private company systems or data. Every scenario has evidence designed to be inspectable in the telemetry tools.

## Realistic telemetry requirements

Simulation must not mean clean toy data. The generators should create:

- Duplicate records and events arriving out of order.
- Missing optional fields and occasional malformed log lines.
- High-cardinality labels and background noise.
- Correlated metrics, logs, and spans with trace and incident identifiers where available.
- Healthy windows and benign alerts to support false-positive measurement.
- Consistent timestamps, fixed seeds, and recorded scenario schedules.

The chaos injector is a product surface: each named scenario must be documented and callable from a clean clone. A scenario file is the source of truth for the seed, event schedule, expected cause, evidence, and remediation class.

## Reproducibility contract

1. Every workload and chaos run accepts and records a seed.
2. `make scenario NAME=<name>` replays the defined telemetry window for that scenario.
3. Scenario configuration is versioned with the code and includes expected labels.
4. The agent session trace records tool calls, policy results, and tool outputs needed for replay.
5. `make replay INCIDENT_ID=<id>` replays against recorded tool responses without an LLM or external dependency.
6. An evaluator reports the scenario version, seed, agent/prompt version, policy version, model routing, and run timestamp with each result.

## Honest boundary

Vigil does not claim to operate against real cloud providers, real PagerDuty/Slack/Jira services, or customer production data. A production deployment would require additional integrations, operational controls, privacy review, and evaluation against appropriate real-world environments.

Vigil is also not bootstrapped by operating itself in production. Its replay engine and evaluation harness can be used to debug Vigil during development, which demonstrates the same development workflow intended for future users without implying production use.
