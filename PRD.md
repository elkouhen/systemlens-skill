# Product Requirements Document: SystemLens architecture completion skill

## 1. Product objective

The primary objective of this skill is to **complete SystemLens's static
architecture analysis**.

SystemLens remains the source of deterministic, code-derived facts. The skill
adds the architectural facts that static extraction cannot reliably determine,
such as implicit service boundaries, HTTP relationships, messaging flows,
data ownership, and deployment bindings. These additions must be evidence-based,
reviewable, and incrementally mergeable into the SystemLens model.

The skill must never silently replace, delete, or blur the distinction between
deterministic SystemLens facts and AI-produced complementary facts.

## 2. Problem statement

Static analysis can identify many constructs directly from source code and
configuration, but complex repositories commonly contain gaps caused by:

- dynamic service discovery and environment-specific configuration;
- generated clients, shared contracts, and framework conventions;
- topics, queues, exchanges, and payload contracts spread across modules;
- database ownership and cross-service access patterns;
- deployment topology expressed outside application source code;
- relationships that require correlating several independent pieces of evidence.

The skill provides a controlled workflow for closing these gaps without
turning assumptions into confirmed architecture facts.

## 3. Users and use cases

### Primary user

An architect, developer, or reviewer who needs a usable architecture model for
a complex codebase and wants to understand both what the code proves and what
requires architectural interpretation.

### Core use cases

1. Index a repository once with SystemLens and establish the deterministic
   baseline.
2. Run one or more focused architecture passes over the same repository.
3. Produce JSON facts with source evidence and stable identities.
4. Import each pass into SystemLens and update existing complementary facts
   rather than creating duplicates.
5. Repeat a pass after code or configuration changes and reconcile the model.
6. Inspect unresolved questions and blind spots instead of hiding uncertainty.

## 4. Product principles

- **Static-first:** always inspect and preserve the SystemLens static baseline.
- **Complement, do not overwrite:** AI facts complete missing relationships;
  they do not mutate source-owned facts.
- **Evidence over inference:** every important fact includes a file, symbol,
  configuration key, manifest, or other traceable observation.
- **Idempotent iteration:** re-importing a fact with the same stable identity
  updates its value and evidence instead of duplicating it.
- **Scoped ownership:** each pass writes only to its own namespace.
- **Explicit uncertainty:** ambiguous or unresolved relationships remain
  labelled as such.
- **Secret-safe output:** manifests contain references to secret/config keys,
  never credentials or secret values.

## 5. Functional requirements

### FR-1 — Establish a deterministic baseline

The skill must guide the user or agent through `systemlens init`, `doctor`,
`index`, coverage checks, indexing diagnostics, and static audit before
complementary facts are generated.

### FR-2 — Support focused architecture passes

The skill must support these independently runnable passes:

| Pass | Namespace | Primary output |
| --- | --- | --- |
| Boundaries | `ai-boundaries` | services, modules, external systems, logical ownership boundaries |
| HTTP | `ai-http` | APIs, routes, synchronous callers and servers |
| Messaging | `ai-messaging` | topics, queues, exchanges, publishers, consumers, delivery metadata |
| Data | `ai-data` | stores, schemas, tables, collections, reads, writes, ownership observations |
| Deployment | `ai-deployment` | workloads, environments, runtime resources, configuration bindings |

The default order is:

```text
boundaries → (http || messaging || data) → deployment
```

The skill may select a subset when the user's question is narrower.

### FR-3 — Generate a versioned fact artifact

Each pass must produce a JSON artifact named:

```text
architecture.ai-<profile>.pass-<NNN>.json
```

The artifact must identify the repository revision, pass, namespace, producing
agent/model, stable node and edge identities, and source evidence.

### FR-4 — Reconcile facts iteratively

The workflow must import every artifact explicitly through SystemLens's graph
fact import path. The import must upsert matching identities so that the newest
reviewed value and evidence replace the older complementary value.

`partial` imports preserve other facts in the same namespace. `complete`
imports are allowed only when the pass inspected the full namespace scope and
may remove facts absent from that complete snapshot.

### FR-5 — Preserve fact provenance

The resulting graph and reports must distinguish deterministic SystemLens facts
from complementary AI facts. A later pass may reference entities from another
namespace but must not silently rewrite that namespace.

### FR-6 — Provide reviewable handoffs

After every pass, the skill must report files inspected, facts inserted,
updated, or removed, unresolved questions, and the current revision. The agent
must query or export the graph before starting the next dependent pass.

## 6. Quality requirements

- No confirmed dependency may be emitted without concrete supporting evidence.
- Stable IDs must survive repeated passes and source-line movement where the
  logical entity is unchanged.
- A pass must distinguish current configuration from historical migrations or
  generated artifacts.
- Environment-specific deployment facts must remain environment-specific.
- Missing or unsupported extraction coverage must be stated in the report.
- JSON artifacts must validate against the supported SystemLens graph contract.
- The skill documentation must describe the five pass profiles and their
  namespaces.

## 7. Out of scope

- Replacing or weakening SystemLens's deterministic extraction rules.
- Treating AI hypotheses as runtime telemetry or production truth.
- Automatically changing source code, infrastructure, or deployment systems.
- Storing credentials, tokens, or other secret material in graph facts.
- Claiming complete topology when the relevant repository or environment scope
  was not inspected.

## 8. Acceptance criteria

The feature is complete when an architect can:

1. Run SystemLens static indexing and obtain a baseline model.
2. Execute the boundaries, HTTP, messaging, data, and deployment passes either
   sequentially or with the documented parallelization.
3. Import the resulting JSON artifacts into their dedicated namespaces.
4. Re-run one pass with a changed fact and observe an update, not a duplicate.
5. Confirm that static SystemLens facts remain present and distinguishable.
6. Inspect evidence and unresolved items for every major complementary fact.
7. Rebuild a coherent current model after several iterations without manually
   merging JSON files.

## 9. Success measures

- Fewer important architecture relationships remain unexplained after the
  focused passes.
- Repeated imports produce no duplicate facts for stable identities.
- Reviewers can trace complementary facts back to repository evidence.
- Static facts and complementary facts remain distinguishable in exports and
  reports.
- Architecture updates require rerunning only the affected pass profiles.
