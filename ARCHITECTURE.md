# BoxFetch Architecture

## Architectural objective

BoxFetch separates commercial entitlement, agent authority, human approval, execution capability, provider mutation, and audit evidence. No one token, UI action, or tool call implicitly carries all of those powers.

The README gives the high-level lifecycle. This document focuses on the authority boundaries underneath it.

## Authority matrix

| Concern | Primary authority | What it does not imply |
|---|---|---|
| Authentication | Valid user or OAuth session | Purchase entitlement or run authority |
| Marketplace acquisition | Captured purchase / entitlement | Provider credential or execution approval |
| Agent discovery | Tool visibility under current grant | Permission to call a hidden or gated action |
| Run preparation | `boxfetch:runs` | Apply or teardown authority |
| Run inspection and apply | `boxfetch:runs.execute` plus exact owner approval | Teardown authority |
| Teardown | `boxfetch:runs.teardown` plus separate owner decision | General provider access |
| Provider credential | Owner-controlled connection state | Permission for an agent to approve a plan |
| Human approval | Exact sealed plan digest | Automatic dispatch |
| Runtime capability grant | Implementation-specific process/network/filesystem authority | Arbitrary package execution |

The system is designed around the absence of transitive authority. A valid fact in one row does not silently satisfy another.

## Runtime planes

BoxFetch keeps identity, product state, and external provider authority separate.

- **Identity plane:** users, authentication, and approved shared security operations.
- **Product plane:** marketplace, credits, purchases, entitlements, agents, OAuth/MCP grants, controlled runs, and operational evidence.
- **Provider plane:** external mutation behind implementation-specific adapters and capability grants.

The product and identity database authorities are also separated. A generic connection string is not accepted as a fallback for either plane.

## Agent boundary

Protected MCP sessions expose only tools authorized by the current grant, except for a narrow scope-ladder mechanism that can reveal the next missing run authority without attaching that authority to the call.

The current hosted catalog contains eight acquisition-oriented tools and seven controlled-run tools. A newly connected acquisition client does not receive controlled-run authority by default.

The agent-facing run projection is narrower than the owner-facing view. It omits provider connection labels, full plan bodies, receipt bodies, and secret-bearing state. The projection is sufficient to describe lifecycle state and the next action without turning observability into a credential channel.

## Owner boundary

The human owner controls provider connection creation or selection, the declared change, plan approval or decline, explicit apply, cancellation, teardown approval or decline, and explicit teardown execution.

Approval and execution remain separate actions. A human approving a plan does not automatically dispatch a provider mutation.

The owner decision is bound to the exact plan digest re-read inside the transition lock. If another process re-plans the run first, the stale decision does not inherit authority over the replacement plan.

## Controlled execution runtime

The runtime accepts only known implementations with explicit capability grants. Important constraints include:

- default-deny executable access
- implementation-scoped HTTPS hosts
- redirect re-authorization against the same host list
- rooted and symlink-contained filesystem access
- allowlisted child-process environment
- exact-value and pattern-based secret redaction from surfaced output and exceptions
- no arbitrary package-supplied shell
- no dynamic execution of arbitrary package JavaScript

A package can be commercially deliverable without being executable by the hosted runtime. Execution support is an independently reviewed capability.

## Commercial and spending controls

The product plane includes an internal credit ledger, purchases, entitlements, agent acquisition permissions, and spending limits. These are durable product controls rather than prompt instructions.

Commercial entitlement answers whether an owner or agent has acquired an asset. It does not answer whether a provider mutation is authorized, which credential may be used, or whether a human has approved the exact plan.

## Concurrency and replay

State transitions are transactional and mutations require idempotency keys. Approval decisions bind to exact digests. Concurrent re-planning cannot inherit an older approval.

The safety model also distinguishes a local refusal from a mutation that may have reached the provider. Once dispatch creates ambiguity, the system enters reconciliation rather than assuming a retry is harmless.

## Reconciliation

Reconciliation is a first-class state because distributed systems can know that a request was dispatched without knowing whether the remote side committed it.

In that state:

- forward mutation is withheld
- retry is not offered as the default action
- owner-facing UI emphasizes ambiguity
- provider-call evidence and durable state remain available for investigation
- teardown or cancellation is not fabricated as successful unless supported by evidence

## Teardown

Teardown has its own planning and approval lifecycle because deletion is not merely the inverse of creation. The system binds teardown decisions to exact reviewed state and preserves ambiguity when provider evidence is insufficient.

This is particularly important for partially successful external operations, where deleting the wrong resource can be worse than leaving an uncertain resource for explicit reconciliation.
