# Three-minute demo run-of-show

The demo contract should let a reviewer understand the system without narration. It shows the whole loop: healthy system, fault, triage, investigation, human approval, replay, adversarial containment, and evaluation.

## Pacing

The segment timings in the original brief add up to 3 minutes 30 seconds. This proposed cut preserves each beat and totals exactly 3 minutes; adjust after timing a live run.

| Time | Demo action | What the viewer sees |
| --- | --- | --- |
| 0:00–0:10 | Open dashboard on healthy system | Request flow, no active incidents |
| 0:10–0:20 | Trigger `memory-leak` | Chaos action and Grafana RSS/latency changes |
| 0:20–0:45 | Show alert and Tier-1 triage | Investigate decision and noise-filtering histogram |
| 0:45–1:25 | Let supervisor and specialists investigate | Metrics, Logs, Traces, and Runbooks evidence; root cause is unbounded API cache; token/cost counters |
| 1:25–1:50 | Review and approve remediation | Proposed restart, approval request, one-replica blast-radius limit, audit entry |
| 1:50–2:15 | Replay the incident | Offline replay with no LLM or external calls and a fidelity result |
| 2:15–2:30 | Click “Break it” | Prompt-injected log; Aegis denial and audit record |
| 2:30–2:50 | Open evaluation and postmortem | Per-scenario scorecard, CI status, escalation/cost metrics, postmortem draft |
| 2:50–3:00 | Close on architecture and pitch | Safety, replay, and measurement in one system |

## Readiness checklist

- Clean clone can start the stack and load the dashboard.
- `memory-leak` shows correlated telemetry across metrics, logs, and traces.
- Triage's ignore/investigate counts and latency are visible.
- Specialist evidence is linked to the diagnosis and source telemetry.
- Remediation cannot execute before an authorized human approval.
- Approval is scoped and the action is simulated with a one-replica limit.
- Replay uses recorded tool responses and makes no model/provider calls.
- The “Break it” path shows a blocked action and matching audit record.
- Evaluation numbers are measured and labeled with seed/sample size; no target is presented as a result.
- Postmortem is clearly marked as a draft for human review.

## One-sentence close

“Vigil is an autonomous on-call SRE agent that investigates a deterministic production-like environment, constrains every tool call, and proves its behavior through replay and evaluation.”
