# 12-week build plan

The weekly ship rule in the project brief is to deploy whatever works each Friday. Keep the demo path and the four depth areas in view; features outside them are candidates for removal if schedule pressure rises.

## Weeks 1–2: Simulated production environment

Build the three services, Postgres, Redis, Docker Compose, telemetry stack, seeded workload generator, and all five chaos scenarios.

**Exit check:** each fault produces inspectable telemetry; `make scenario NAME=...` repeats a scenario from its seed and schedule.

## Weeks 3–4: Ingestion and Tier-1 triage

Add Redpanda, the normalizer and event schema, rules, the lightweight classifier, and alert intake from Prometheus.

**Exit check:** alerts flow through triage with latency measurements; initial targets are at least 60% of alerts filtered before any LLM call and p99 triage under 15ms.

## Weeks 5–6: Evaluation harness before the agent

Build the golden dataset, replay runner, scorecard, and CI gates before the multi-agent investigation layer.

**Exit check:** `make eval` runs the five scenarios and seeds, prints per-scenario and aggregate results, and CI fails on a defined regression.

## Weeks 7–9: Investigation and Aegis

Build the supervisor, Metrics/Logs/Traces/Runbooks/Remediation specialists, shared blackboard, policy engine, trace recorder, and replay engine.

**Exit check:** each named incident produces its expected cause; traces replay offline; the injected prompt attack is blocked and recorded.

## Weeks 10–11: Context and cost

Add evidence curation, hierarchical summaries, similar-incident retrieval, token budgets, bounded memory, model routing, and caching where supported.

**Exit check:** README includes reproducible before/after results for context size, cost, and accuracy without presenting examples as measurements.

## Week 12: Product surface and deployment

Build the dashboard, approval queue, postmortem draft, audit views, public demo deployment, replay action, and adversarial “Break it” flow.

**Exit check:** a new reviewer can complete the demo from the run-of-show with no hidden setup or narration.

## Scope guardrails

Do not add real AWS/GCP/Azure actions, PagerDuty/Slack/Jira integrations, customer data, billing, mobile clients, foundation-model fine-tuning, or additional incident types in this capstone. Preserve evaluation and safety work if the schedule slips; cut work that does not support the demo contract or the four depth areas.
