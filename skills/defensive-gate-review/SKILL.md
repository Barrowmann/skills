---
name: defensive-gate-review
description: Review defensive validations, readiness gates, retries, and recovery paths to see whether they block normal workflows or widen failure scope. Use for over-defensive code, overly broad gating, and normal-path reliability reviews on existing call paths or code changes.
---

# Defensive gate reliability review

## Core principle

Defensive mechanisms must contain failures instead of adding failure conditions to normal workflows. Add only the minimum safeguards required by reachable failure modes or explicit correctness and security invariants. Every blocking condition must protect a specific invariant of the current operation and justify its own failure cost.

Auxiliary work such as logging, diagnostics, optimization, bookkeeping, and cleanup must not block unrelated operations. Its failure must not invalidate committed results or report a business failure.

Let `E` be the guarded exception and `G` be a false gate failure on a normal path. The added normal-path failure probability is `P(not E) * P(G | not E)`. When `E` is rare, `P(not E)` is near 1, so normal workflows inherit nearly the full failure probability of `G`.

Deliver a review report by default. Apply fixes, file issues, or change configuration only when authorized.

## Evaluation criteria

Answer four questions for each candidate mechanism:

1. **Necessity.** What invariant breaks if you remove this check? Does a reachable scenario or explicit contract require it?
2. **Scope.** Which requests, resources, or steps depend on this check? Can you isolate failure to the affected object?
3. **Failure semantics.** Does the mechanism separate business failure, unknown outcome, temporary unavailability, and auxiliary failure? Can downstream diagnostic or cleanup failure overwrite a committed result?
4. **Net reliability.** When `E` does not happen, what rejections, waits, dependencies, or recovery failures does the mechanism add? Do prevented failures outweigh that cost?

Confirm findings with reachable failure chains. Treat check counts, abstraction counts, and naming only as search cues. Without failure-rate data, argue from concrete counterexamples and keep probability uncertainty explicit.

## Review process

1. **Set the boundary.** Record the repository, revision, and relevant working-tree changes. Trace user-specified flows for existing code, or follow the diff to affected callers and callees for changes.
2. **Trace the normal path.** Locate the entry point, required dependencies, commit point, and return value. Check how validations, readiness gates, retries, timeouts, recovery hooks, locks, and cleanup alter control flow. Check whether recovery actions such as query, cancel, or release share the same gate.
3. **Build a minimal counterexample.** Hold a valid operation fixed and change only an unrelated object, auxiliary component, or transient state. Trace why it fails, stalls, or misreports, and show how the invariant still holds with a narrower gate.
4. **Verify and attribute.** Use isolated fixtures and public entry points. Separate gate-induced failures from real resource failures. Report statically when the full call chain proves the issue. Review the contract in test assertions instead of relying on pass status.
5. **Propose the minimal fix.** Prefer removing unjustified gates, narrowing dependencies and locks, separating auxiliary outcomes, or fixing ownership and transaction boundaries over adding recovery layers. Any new recovery layer, state machine, queue, or configuration must pass the four evaluation criteria.

## Focus areas

- **Widened failure scope.** One bad record rejects a batch, one failed instance halts the system, or one conflict freezes unrelated resources.
- **Blocked recovery.** Query, cancel, return, or release actions require full readiness checks meant for new work.
- **Hard auxiliary dependencies.** Logging, metrics, alerts, diagnostics, summaries, or cleanup block work or overwrite committed results.
- **Misclassified failures.** A query timeout restarts the process, transient or unknown outcomes report as business failures, or dependency waits exhaust the failure budget.
- **Overbroad validation and invalidation.** Unrelated configuration, comments, history, or unused fields invalidate valid inputs or caches.
- **Unbounded waits and recovery.** Retries or setup block the main loop, timeouts leak child processes or locks, or verification latency immediately stales its own result.
- **Missing ownership and release paths.** Cleanup guesses targets by name, old conflicts act as permanent gates, or resolved conflicts have no clearing path.

## Reporting

Order findings by impact and reachability using project priority conventions, and label evidence strength separately. Include five items in each finding:

- **Location and trigger.** State the exact source location and trigger conditions.
- **Causal chain.** Trace the valid operation, the defensive condition, the added dependency or failure propagation, and the business impact.
- **Invariant and overreach.** Name the concrete protection needed and where the mechanism exceeds it.
- **Evidence.** Mark the finding as `reproduced`, `statically proven`, or `unverified`, and state what you checked.
- **Minimal fix and acceptance.** Confirm that real exceptions stay contained and unrelated failures do not block valid operations.

Keep confirmed findings separate from hypotheses. When a spec requires a gate, separate implementation compliance from the reliability cost of the spec and note the contract impact. Merge duplicate symptoms that share a root cause, and cross-reference them when fix boundaries differ. State coverage limits clearly, and report no finding or an open check when evidence is insufficient.
