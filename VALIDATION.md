# Agent Authorization & Controlled Execution Validation

The system uses separate evidence layers because protocol correctness, authorization correctness, execution safety, and real-provider behavior are different claims.

## Pull-request quality gate

The private repository has one stable required `quality` status. It aggregates static analysis, type checking, linting, the deterministic runtime proof, unit and security-core test shards, documentation controls, and the production build. A skipped or cancelled dependency does not count as a pass.

The local equivalent runs the same major layers sequentially rather than treating CI as the only source of truth.

## Explicit validation layers

The repository separates four major test classes:

1. **Unit** for application behavior outside the security roots.
2. **Security-core** for money, wallet, transaction, provider, auth, OAuth, and MCP behavior.
3. **Operations** for generators, static boundaries, operator scripts, and acceptance tooling.
4. **Disposable PostgreSQL proofs** for lifecycle, persistence, concurrency, and behaviors that a store double cannot prove.

That separation matters because a fast mocked test should not be reported as evidence of a provider call or a transactional database invariant.

## Hosted MCP surface

The current hosted catalog contains **15 tools**:

- **8 acquisition-oriented tools**
- **7 controlled-run tools**

The seven run tools require authorities outside the default acquisition grant. TypeScript and Python client proof harnesses exercise MCP interoperability independently rather than relying only on the system's own internal caller.

## OAuth and authority evidence

The proof surface covers authorization-code and PKCE behavior, concurrent OAuth operations, scope narrowing, incremental scope step-up, protected-session binding, registration and session abuse ceilings, revocation and rotation, and owner-versus-agent authority.

The scope ladder exposes only the next missing run authority. It does not automatically walk a newly connected client toward provider mutation or teardown.

## Standalone runtime proof

Every pull request builds the distributable Node 22 ESM runtime artifact independently from the source tree.

The proof derives the actual dependency closure from the build output, rejects undeclared or missing closure entries, scans distributed bytes for source maps, local absolute paths, environment-secret values, and private-plane tokens, and then runs the artifact against all **six delivered reference packages**.

The proof also exercises refusal cases, including tampered packages, incompatible runtime ranges, unapproved apply, missing or foreign approved-plan digests, and unapproved teardown.

## Controlled-run mutation and recovery matrix

The current safety register contains **161 registered cases** and **161 machine-mapped cases**, with no unmapped entries in the current evidence record. The matrix distinguishes several levels instead of publishing one misleading coverage percentage:

- registered cases
- machine-mapped cases
- cases requiring direct observation
- cases with direct observation satisfied
- cases still awaiting direct observation
- cases where provider-call count was actually observed

At the current private-repository evidence head, **108 cases require direct observation**, **44 satisfy that requirement**, and **64 remain explicitly outstanding**. Outstanding evidence is therefore visible rather than silently converted into "covered."

## Why some proofs require real PostgreSQL

One pagination defect existed specifically because an orchestrator's `limit + 1` look-ahead disagreed with the store's own clamp. A fake store would return whatever the test asked for and miss the problem.

The regression is therefore exercised against PostgreSQL with **51 seeded owner runs**: a maximum-size page must emit a cursor, following it must return the fifty-first run exactly once, and no caller may bypass the public maximum with an unbounded request.

This is representative of the validation philosophy: use a test double only when the property survives the substitution.

## Mutation testing of safety properties

A manual source-mutation campaign removes one safety property at a time and names the proof expected to fail. The campaign restores the target byte-for-byte after every mutation and stops if a mutated safety property survives.

That asks a stronger question than "is the suite green?" It asks whether the suite would notice if a claimed control disappeared.

## Provider evidence

Provider-facing work is divided into deterministic adapter proof, direct provider-call observation, and explicit live-canary evidence. An internally reviewed adapter can be code-ready without implying that a real production provider mutation has been performed.

Ambiguous dispatched mutations are exercised as a distinct lifecycle outcome. The expected response is to stop forward mutation and require reconciliation, not to retry automatically.

## Claim boundary

The public case study supports the claim that the system has a working controlled-execution architecture with scoped agent access, human approval, spending and entitlement controls, idempotent lifecycle state, constrained runtime behavior, and layered verification.

It does not claim arbitrary third-party code execution, universal provider coverage, or that every integration has completed a real production canary. The evidence register explicitly carries outstanding direct-observation work where that stronger proof does not yet exist.
