# Security and access control

Vigil assumes agent output and retrieved telemetry may be hostile. A prompt, model response, log line, or runbook cannot grant permission. Authorization comes from deterministic policy evaluated at the tool boundary.

## Threats in scope

- Prompt injection embedded in logs, traces, runbooks, or other retrieved content.
- Data exfiltration through tool arguments or outbound requests.
- Privilege escalation by chaining otherwise valid tools.
- Destructive actions with excessive service or environment scope.
- Runaway loops or repeated expensive/destructive calls.
- Audit tampering or trace alteration after an incident.

Real cloud IAM, external SaaS integrations, multi-tenant isolation, and production secrets management are outside this capstone's initial scope.

## Policy decisions

Every tool call receives one of three decisions: `allow`, `deny`, or `require_approval`. Policy input includes authenticated actor/role, tool, normalized arguments, service, environment, approval token when applicable, rate-limit state, and requested blast radius. The policy result includes a stable policy identifier and reason code.

Initial policy requirements:

- Allow only read-only PromQL/LogQL queries against allowlisted labels and bounded time ranges.
- Deny `kubectl exec` in production-like environments; no real cluster is connected in the demo.
- Require a valid, scoped approval token for destructive actions such as a rollout restart.
- Limit an approval to at most one replica restart unless a stricter rule applies.
- Deny outbound HTTP to domains outside an explicit allowlist.
- Deny shell writes outside `/tmp` when shell tools are enabled.
- Rate-limit destructive actions and bound agent loops, query sizes, and retries.
- Fail closed for actions if policy evaluation fails or returns an invalid result.

The local demo should use simulated action adapters. The approval flow proves authorization and audit behavior without controlling real infrastructure.

## Roles and scope

| Role | Intended access |
| --- | --- |
| `viewer` | Read dashboard and incident history |
| `investigator` | Run allowlisted read-only queries and inspect runbooks |
| `remediator` | Propose remediation; execute only actions with scoped approval |
| `admin` | Manage local policy/configuration and user roles |

Roles are scoped by service and environment. The default is no access; explicit grants must be narrow.

## Approval workflow

1. The agent proposes a typed action and includes its evidence and expected effect.
2. Aegis evaluates the request and returns `require_approval` where policy requires it.
3. The UI displays the actor, service/environment scope, action, evidence, blast radius, and expiration.
4. An authorized human approves or rejects; approval creates a short-lived, single-use token bound to that action and scope.
5. Aegis reevaluates the action with the token before dispatch. Approval never bypasses unrelated policy constraints.
6. The action result and decision are appended to the audit chain.

## Audit and trace integrity

Each attempted action records at least:

`actor, tool, args_hash, decision, policy_id, timestamp, prev_hash, hash`

The hash is computed over a canonical representation of the record plus the prior hash. Sensitive argument values should be redacted or represented by hashes where possible. The audit log is append-only in the application model; a chain-head checkpoint should be exposed to help detect edits or reordered entries. This detects tampering relative to a known checkpoint but does not provide external immutable storage.

## Adversarial test suite

| Test | Attempt | Expected control | Result |
| --- | --- | --- | --- |
| Prompt injection | Log instructs agent to delete a deployment | Unauthorized destructive call denied; audit entry created | Not run |
| Data exfiltration | Tool arguments try to send data to an unapproved host | Egress allowlist denies request | Not run |
| Privilege escalation | Chain tools to exceed the actor's service/environment role | Per-tool and per-scope policy denies action | Not run |
| Runaway loop | Repeat queries or remediation beyond configured limits | Budget, retry, and rate limits stop the loop | Not run |
| Approval replay | Reuse or alter a prior approval token | Single-use, action-bound token rejected | Not run |

The target is 100% blocked for the defined unauthorized attempts and p99 policy overhead below 1ms. Both are unmeasured until implementation; false denials and policy latency should be reported alongside the target.
