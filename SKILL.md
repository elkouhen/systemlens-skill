---
name: systemlens
description: "Guide evidence-based architecture exploration with SystemLens, preserving the distinction between indexed source facts and reviewable complementary analysis."
---

# SystemLens skill

The purpose of this skill is to enrich a SystemLens analysis with reviewable
AI-produced explanations and complementary findings. SystemLens remains the
source of truth for deterministic architecture facts, indexed flows, source
evidence, and CodeQL results. This skill must never replace that index or
silently turn an inference into an indexed fact.

## Scope and terminology

**SystemLens** is the product: its CLI and MCP server index a repository and
expose source-derived architecture facts. Its installed version and public
documentation are the source of truth for supported commands, options, output,
and data contracts.

**systemlens-skill** is optional agent guidance in this directory. It lets a
person using SystemLens direct an agent to choose an investigation, interpret
evidence conservatively, and prepare reviewable complementary findings. It
exists to enrich the product's analysis with explanations and complementary
findings; it does not extend SystemLens, invent a command, or turn an inference
into a source-derived fact.

The companion `systemlens-observability-lab` is a separate runtime validation
environment. Use it when the question requires deployed Kubernetes behaviour,
telemetry, or Elastic verification; do not present static SystemLens evidence
or AI enrichment as proof of runtime behaviour.

Use generic architecture terms in user-facing work: **APIs**, **Topics**, and
**Data**. Technology-specific terms identify evidence or an extractor only
when relevant to the inspected repository.

## Core behaviour

- Work from the analysed repository root and establish the current indexed
  baseline before making broad architecture claims.
- Use the SystemLens inventory first; inspect source, configuration, contracts,
  and deployment manifests only to answer a defined gap or question.
- Prefer a focused investigation to an unbounded repository map. State the
  question, scope, exclusions, and desired deliverable before extracting facts.
- Require concrete, relative evidence for material claims. Preserve dynamic,
  ambiguous, generated-only, test-only, runtime-only, and out-of-scope findings
  as qualified observations rather than guessed dependencies.
- Keep deterministic SystemLens facts distinct from complementary analysis.
  Complementary facts belong to a dedicated namespace and never overwrite facts
  owned by SystemLens or another producer.
- Treat every generated description, report, and complementary fact as an
  enrichment layer. Preserve the underlying SystemLens result unchanged and
  make the relationship to its source flow or fact explicit.
- Make uncertainty, confidence, provenance, and stop conditions visible.
- Never include credentials, tokens, connection-string secrets, or absolute
  workstation paths in findings, reports, examples, or manifests.
- Write user-facing reports and generated examples in English, while preserving
  exact CLI output, source snippets, and user-provided text when quoted.

## Investigation lifecycle

1. Confirm the repository perimeter and freshness of the SystemLens inventory.
2. Choose the smallest analysis pass that answers the request.
3. Collect evidence and correlate only explicit, uniquely resolvable
   identifiers.
4. Use persisted `systemlens flows` as the baseline for ordered source-flow
   analysis; its CodeQL-derived or source-symbol call chains and Kafka continuations remain
   potential, confidence-qualified evidence rather than runtime traces.
5. When a human-readable explanation is needed, enrich each persisted flow with
   one AI-generated description keyed by flow ID. Store these descriptions in
   `.systemlens/flow-descriptions.json`; they are presentation text, not new
   architecture facts. Follow [the flow-description contract](references/flow-descriptions.md).
6. Produce a reviewable result: a report for ordered source-flow analysis, a
   flow-description enrichment file, or a versioned fact manifest for
   complementary topology.
7. Validate and review the result before any import. Re-read the merged model
   after an import and report what remains unresolved.
8. When an imported fact changes the topology used by persisted flows, run
   `systemlens flows calculate`. It reuses the stored AST and CodeQL snapshot
   and projects the independent enrichment facts into the transient
   reconstruction; it does not re-index source files or overwrite source
   facts.

When `.systemlens/analysis-scope.json` exists, read it before collecting
evidence and apply its selectors to every step of the investigation. Resolve
services, flows, Topics, and Data resources against persisted SystemLens IDs;
never widen an unresolved selector by guessing from a name fragment. Follow
the [analysis scope contract](references/analysis-scope.md).

For an example prompt that explains persisted call graphs, use
[`prompts/enrich-architecture.md`](prompts/enrich-architecture.md). It asks the
agent to inspect each flow and write one evidence-backed description without
changing the indexed facts.

Use a partial snapshot by default. A complete snapshot is appropriate only when
the entire declared scope has been inspected and replacement of missing facts is
intentional.

## Reference routing

Read the relevant SystemLens-specific contract before performing the associated
work; do not reconstruct product behaviour from this entrypoint.

- [settings.md](references/settings.md) — repository perimeter, initialization,
  indexing, and refresh.
- [analysis-rules.md](references/analysis-rules.md) — extraction evidence,
  conservative correlation, and prompt composition.
- [pass-profiles.md](references/pass-profiles.md) — focused boundary, API,
  messaging, Data, source-flow, and deployment investigations.
- [business-flows.md](references/business-flows.md) — potential business-flow
  reports and traversal limits.
- [flow-descriptions.md](references/flow-descriptions.md) — AI-generated
  descriptions for persisted flows and the HTML enrichment contract.
- [analysis-scope.md](references/analysis-scope.md) — reusable selectors for
  targeted flow descriptions, audits, and enrichment passes.
- [ai-graph.md](references/ai-graph.md) — versioned complementary fact manifest
  contract and reconciliation rules.
- [management.md](references/management.md) — installation, MCP setup, refresh,
  and troubleshooting.

When a reference describes a command, option, MCP tool, JSON field, or export
behaviour, verify it against the installed or development SystemLens product
before relying on it. Update the reference alongside a SystemLens contract
change; do not put product-specific details back into this generic entrypoint.
