# Evaluation design

Evaluation is a first-class project component. The goal is to test incident handling against labeled, repeatable inputs and make regressions visible in CI. A live demo alone is not evidence of quality.

## Golden incident set

The minimum set contains all five scenarios with at least three seeds each (15 labeled incidents total):

1. `memory-leak` — unbounded API cache.
2. `db-pool` — connections held by the payments slow-query path.
3. `redis-evict` — inventory working set exceeds Redis memory.
4. `deploy-500` — bad API code path behind a feature flag.
5. `payment-timeout` — slow mocked external payment provider.

Each record includes a scenario version, seed, command schedule, expected root cause, expected evidence, and expected remediation class. Add healthy or benign-alert windows to measure false positives; incident-only examples cannot supply that denominator.

## Metrics and initial gates

| Metric | Definition | Initial gate |
| --- | --- | ---: |
| Root-cause accuracy | Correct labeled causes / evaluated incidents | ≥80% |
| Time to diagnosis | Alert received to correct cause recorded | <90s |
| Remediation success | Correct outcome / attempted remediations | ≥70% |
| False-positive rate | Non-incidents sent to investigation / evaluated non-incidents | ≤5% |
| Tokens per incident | Mean input plus output tokens across model calls | ≤25K |
| Cost per incident | Mean provider cost for all model calls in an incident | ≤$0.50 |
| Human-escalation rate | Incidents escalated / all evaluated incidents | ≤30% |
| Policy overhead | p99 time for policy evaluation | <1ms |
| Adversarial containment | Unauthorized actions blocked / defined adversarial attempts | 100% |

These are targets from the initial requirements. They are not measured results. Publish sample size, seeds, versions, and run conditions next to each score so numbers can be reproduced.

## Runner behavior

`make eval` is planned to:

1. Validate scenario files and dataset labels.
2. Start or connect to the deterministic simulated environment.
3. Run each scenario and healthy window through ingestion, triage, investigation, policy enforcement, and replay recording.
4. Score root cause, expected evidence, remediation class, latency, false positives, token use, cost, escalation, and policy timing.
5. Write machine-readable results and a concise per-scenario and aggregate scorecard.
6. Compare results with the accepted baseline and return a non-zero exit status when a regression gate fails.

The runner must report partial failures and missing evidence rather than silently excluding unsuccessful cases.

## CI regression policy

After a baseline is accepted, CI should fail when any tracked metric regresses by more than 10% relative to that baseline, subject to explicit metric direction and minimum sample-size rules. Absolute gates also apply to the listed targets. Baseline updates should include a reason and new result artifact; do not automatically move the baseline to make a failing build green.

Deterministic code and policy tests run on every change. Model-dependent evaluations should record model/provider and prompt versions, use controlled seeds, and be designed to manage provider variance and cost. If provider nondeterminism prevents a stable hard gate, CI can use replay for deterministic scoring and a scheduled or manually triggered live-model evaluation for drift and quality tracking.

## Four depth-area comparisons

### Context engineering

Compare naive raw telemetry context with hierarchical specialist summaries, similar-incident retrieval, anomaly curation, bounded working memory, and a hard token budget. Report tokens, root-cause accuracy, and evidence coverage. Example figures in the project brief are illustrative only.

### Automated evaluation

Report per-scenario results and the aggregate scorecard, with sample sizes and regression history. Include both incident and healthy windows.

### Safety

Run prompt-injection, data-exfiltration, privilege-escalation, and runaway-loop attempts. Report attempts, policy outcomes, blocked actions, false denies, and p50/p95/p99 policy overhead.

### Cost optimization

Compare baseline routing and context against tiered routing, prompt caching where supported, compression, and per-incident budgets. Report dollars and tokens per incident by tier alongside accuracy and escalation rates.

## Result publication template

Replace this section with measured output when the runner exists. Do not fill with target values presented as outcomes.

| Scenario | Seeds | Root cause | Diagnosis time | Remediation | Tokens | Cost | Escalated |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| `memory-leak` | Pending | Pending | Pending | Pending | Pending | Pending | Pending |
| `db-pool` | Pending | Pending | Pending | Pending | Pending | Pending | Pending |
| `redis-evict` | Pending | Pending | Pending | Pending | Pending | Pending | Pending |
| `deploy-500` | Pending | Pending | Pending | Pending | Pending | Pending | Pending |
| `payment-timeout` | Pending | Pending | Pending | Pending | Pending | Pending | Pending |
| **Aggregate** | **Pending** | **Pending** | **Pending** | **Pending** | **Pending** | **Pending** | **Pending** |
