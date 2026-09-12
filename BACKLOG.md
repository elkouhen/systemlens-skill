# Analysis backlog

This backlog turns the architecture-analysis axes into reviewable work items.
Start with the baseline and the system map; then select the focused passes that
answer the current architecture question. Findings must retain evidence,
confidence, status, ambiguity, and their owning namespace.

## Now — establish the architecture baseline

| ID | Task | Expected outcome | Priority |
| --- | --- | --- | --- |
| BL-01 | Validate the analysis workspace with `systemlens init`, `doctor`, and `index`. | A current deterministic code inventory and any indexing gaps. | P0 |
| BL-02 | Map modules, layers, and dependency directions. | A boundaries-pass artifact identifying services, modules, external systems, and questionable boundaries. | P0 |
| BL-03 | Identify central components, cycles, and cross-cutting dependencies. | A ranked list of coupling risks, each with source evidence and impact. | P0 |
| BL-04 | Reconstruct the two or three most important business flows. | A source-flow review from entry point through application logic, data access, and outbound calls. | P0 |

## Next — inspect critical technical boundaries

| ID | Task | Expected outcome | Priority |
| --- | --- | --- | --- |
| BL-05 | Inventory HTTP endpoints and their synchronous callers. | An HTTP-pass artifact covering routes, callers, servers, contracts, and unclear ownership. | P1 |
| BL-06 | Map asynchronous integrations. | A messaging-pass artifact covering channels, publishers, consumers, payload clues, and delivery semantics. | P1 |
| BL-07 | Analyze data stores and ownership. | A data-pass artifact covering stores, schemas or tables, reads, writes, transactions, and cross-boundary access. | P1 |
| BL-08 | Review third-party integrations and failure boundaries. | An evidence-backed inventory of external APIs, clients, timeouts, retries, fallbacks, and idempotency concerns. | P1 |
| BL-09 | Map deployment and configuration bindings. | A deployment-pass artifact relating logical components to workloads, environments, resources, and secret references only. | P1 |

## Improve — assess quality attributes and change risk

| ID | Task | Expected outcome | Priority |
| --- | --- | --- | --- |
| BL-10 | Assess error handling, partial failures, concurrency, and recovery paths. | A robustness risk register with affected flows and concrete evidence. | P2 |
| BL-11 | Review authentication, authorization, input validation, secret handling, and sensitive-data exposure. | A security review backlog; do not include secret values in findings. | P2 |
| BL-12 | Identify performance risks in critical flows. | Candidate N+1 queries, expensive access patterns, fan-out chains, blocking work, and cache gaps. | P2 |
| BL-13 | Assess observability along critical flows. | Gaps in structured logs, metrics, traces, correlation, alerting, and failure diagnosis. | P2 |
| BL-14 | Review test coverage and confidence for risky flows. | Missing unit, integration, contract, and regression scenarios, prioritized by risk. | P2 |
| BL-15 | Identify maintainability and evolution risks. | Hotspots for duplication, oversized units, unclear extension points, API compatibility, and migration risk. | P2 |

## Definition of done for every item

- Scope and the repository revision are recorded.
- Findings distinguish deterministic SystemLens facts from AI-produced facts.
- Every confirmed architecture fact has relative-path evidence and an explicit
  confidence and status; unresolved ambiguity remains visible.
- Affected topology passes are emitted as versioned JSON artifacts and imported
  only into their owning namespaces.
- The handoff lists inspected files, added or updated facts, open questions,
  and recommended follow-up work.

## Suggested execution order

```text
BL-01 → BL-02 → (BL-03 || BL-04) → (BL-05 || BL-06 || BL-07) → BL-08 → BL-09
      → (BL-10 || BL-11 || BL-12 || BL-13 || BL-14 || BL-15)
```
