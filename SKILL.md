---
name: systemlens
description: "Analyze an application directly, generate evidence-backed architecture facts as JSON, import them into SystemLens, and produce an HTML graph without indexing the source code with SystemLens."
---

# SystemLens direct-analysis skill

This skill analyzes an application directly and produces a reviewable JSON
manifest of architecture facts. It then imports that manifest into SystemLens
and generates an HTML architecture export.

The skill does not run `systemlens index`. SystemLens stores the imported facts
as an enrichment namespace. The source code remains outside the SystemLens
indexing pipeline for this workflow.

## Workflow

Follow these steps in order:

1. Define the application root, analysis question, included services, and
   excluded directories. Treat the application root as the base for all paths.
2. Inspect the source directly with repository tools. Read controllers,
   consumers, publishers, clients, persistence adapters, configuration, and
   local contracts only when they answer the defined question.
3. Record one evidence-backed node or edge per fact in a
   `systemlens-ai-graph-v1` JSON manifest. For flow analysis, add the
   manifest's `endpoints` and ordered `flows` arrays. Use relative evidence
   paths and preserve unknown or ambiguous values instead of guessing them.
4. Run `systemlens init` in the application root. This creates configuration;
   it does not analyze source files.
5. Run `systemlens import-facts manifest.json --namespace direct-analysis
   --complete`. The importer creates the empty compatible SQLite schema when
   the repository has not been indexed and writes only the manifest facts.
6. Run `systemlens export microservices --html architecture.html` and inspect
   the generated graph and its evidence.

## Analysis rules

- Use the smallest source scope that answers the question.
- Treat source annotations, method calls, configuration, and local API or
  AsyncAPI contracts as evidence with different confidence levels.
- Keep node and edge IDs stable across revisions of the same analysis pass.
- Use `confirmed` only when the source evidence supports the relation.
- Use `proposed` for a plausible relation that needs review.
- Use `ambiguous` or `unresolved` with a `reason` when the source does not
  identify one target or channel.
- Never invent a concrete topic, route, collection, service, payload type, or
  source location.
- Keep evidence paths relative to the analyzed application root.
- Never include credentials, tokens, private keys, absolute workstation paths,
  or full secret-bearing configuration values.
- Do not describe static source evidence as runtime observation.

## Manifest and import boundaries

The manifest contract, including optional direct flows, is defined in
[`references/fact-manifest.md`](references/fact-manifest.md). The importer
validates the format, stores facts under the requested namespace, and keeps
source-derived tables empty when no index has been run.

Facts from separate analyses must use separate namespaces. Use `--complete`
only when the manifest represents the complete current snapshot of that
namespace. A partial import leaves other facts in the namespace unchanged.

## Output review

Before handing off the HTML export, verify that:

- every confirmed relation has evidence;
- unresolved and ambiguous facts remain visible as qualified issues;
- the manifest contains no absolute path or secret value;
- the HTML file contains graph data;
- the rendered topology matches the manifest rather than an unrequested
  source index.

For a complete workflow example, see
[`README.md`](README.md) and the step-by-step page on the
[skill site](docs/index.html).
