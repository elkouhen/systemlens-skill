---
name: systemlens
description: "Install CodeQL, index an application with CodeQL and SystemLens, analyze it directly for complementary facts, import the facts as JSON, and produce an HTML graph."
---

# SystemLens CodeQL analysis skill

This skill creates a deterministic source baseline with CodeQL and SystemLens,
then analyzes the application directly to produce a reviewable JSON manifest of
complementary architecture facts. It imports that manifest and generates an
HTML architecture export.

## Workflow

Follow these steps in order:

1. Define the application root, analysis question, included services, and
   excluded directories. Treat the application root as the base for all paths.
2. Install the CodeQL CLI if it is not already available, following the
   [official CodeQL CLI setup instructions](https://docs.github.com/en/code-security/how-tos/find-and-fix-code-vulnerabilities/scan-from-the-command-line/set-up-codeql-cli).
   Put the executable on `PATH` and verify it with `codeql version` before
   continuing.
3. Initialize SystemLens and create a CodeQL database for the application:
   ```bash
   systemlens init
   mkdir -p .codeql
   codeql database create .codeql/systemlens-java \
     --language=java --source-root=. --build-mode=none
   ```
4. Index the application with the CodeQL-backed SystemLens engine:
   ```bash
   systemlens index --full --call-graph-engine codeql \
     --codeql-database .codeql/systemlens-java
   ```
   The indexed snapshot is the baseline for source modules, endpoints and
   code flows.
5. Inspect the source directly with repository tools. Read controllers,
   consumers, publishers, clients, persistence adapters, configuration, and
   local contracts only when they answer the defined question.
6. Record one evidence-backed complementary node or edge per fact in a
   `systemlens-ai-graph-v1` JSON manifest. For flow analysis, add the
   indexed SystemLens flows as the baseline and add only direct observations
   that are not already represented there. Do not duplicate indexed endpoints
   or flows in the manifest after indexing. Use relative evidence paths and
   preserve unknown or ambiguous values instead of guessing them.
7. Run `systemlens import-facts manifest.json --namespace direct-analysis
   --complete`. The importer adds the complementary facts to the indexed
   snapshot without replacing CodeQL-derived source facts.
8. Run `systemlens export microservices --html architecture.html` and inspect
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

The manifest contract, including optional direct bootstrap flows, is defined in
[`references/fact-manifest.md`](references/fact-manifest.md). The importer
validates the format and stores complementary facts under the requested
namespace. A direct bootstrap import remains available when CodeQL cannot run,
but it is not the default workflow.

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
  CodeQL-backed source index and the complementary manifest.

For a complete workflow example, see
[`README.md`](README.md) and the step-by-step page on the
[skill site](docs/index.html).
