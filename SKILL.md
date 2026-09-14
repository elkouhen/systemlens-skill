---
name: systemlens
description: "Guide evidence-based architecture exploration with SystemLens, preserving the distinction between indexed source facts and reviewable complementary analysis."
---

# SystemLens skill

## Scope and terminology

**SystemLens** is the product: its CLI and MCP server index a repository and
expose source-derived architecture facts. Its installed version and public
documentation are the source of truth for supported commands, options, output,
and data contracts.

**systemlens-skill** is optional agent guidance in this directory. It lets a
person using SystemLens direct an agent to choose an investigation, interpret
evidence conservatively, and prepare reviewable complementary findings. It
enriches the separate graph-fact layer; it does not extend SystemLens, invent a
command, or turn an inference into a source-derived fact.

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
   analysis; its CodeQL-derived call chains and Kafka continuations remain
   potential, confidence-qualified evidence rather than runtime traces.
5. Produce a reviewable result: a report for ordered source-flow analysis, or
   a versioned fact manifest for complementary topology.
6. Validate and review the result before any import. Re-read the merged model
   after an import and report what remains unresolved.

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
- [ai-graph.md](references/ai-graph.md) — versioned complementary fact manifest
  contract and reconciliation rules.
- [management.md](references/management.md) — installation, MCP setup, refresh,
  and troubleshooting.

When a reference describes a command, option, MCP tool, JSON field, or export
behaviour, verify it against the installed or development SystemLens product
before relying on it. Update the reference alongside a SystemLens contract
change; do not put product-specific details back into this generic entrypoint.
