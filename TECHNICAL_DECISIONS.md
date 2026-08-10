# BoxFetch Technical Decisions

## 1. Separate entitlement from execution authority

A purchased capability is not automatically executable. Commercial entitlement, runtime eligibility, connection authority, human approval, and provider mutation remain distinct states.

## 2. Use incremental scope escalation

OAuth grants are intentionally narrow. When a client needs more authority, the system exposes the next required scope rather than granting a broad superset preemptively.

## 3. Bind human decisions to exact digests

Approvals attach to exact plan identities. If a plan changes, the approval does not survive the change. This prevents stale human intent from authorizing new state.

## 4. Make ambiguous mutations non-retryable by default

External providers can respond ambiguously. Retrying can duplicate an operation. BoxFetch therefore moves uncertain dispatches into reconciliation-required state rather than assuming idempotence at the provider boundary.

## 5. Keep owner and agent projections different

Agents need lifecycle state and next-step authority. They do not need provider secrets, complete plan bodies, full receipts, or owner-only connection metadata. The system exposes only what the actor needs to proceed safely.

## 6. Constrain the runtime structurally

Security is not delegated to package goodwill. The runtime limits executables, hosts, environment variables, filesystem reach, and dynamic code paths at the implementation boundary.

## 7. Treat teardown as an independent risk surface

Deletion can be more destructive than creation. Teardown therefore receives its own plan, approval, dispatch, and verification path rather than inheriting authority from the original apply.

## 8. Keep production proof distinct from code proof

Disposable test environments, local provider adapters, staging checks, and live canaries are separate evidence classes. A green build is not represented as proof that a real provider was safely mutated.
