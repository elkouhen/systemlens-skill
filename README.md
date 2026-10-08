# systemlens-skill

This companion skill analyzes an application directly, writes architecture
facts and optional ordered flows to a `systemlens-ai-graph-v1` JSON manifest,
imports the manifest into SystemLens, and produces an HTML graph.

The workflow does not run `systemlens index`. It is useful when an agent must
inspect a bounded application, preserve its evidence and uncertainty, and
share the result without creating a deterministic source index.

## Quick start

Install the skill with:

```bash
npx skills add elkouhen/systemlens-skill
```

From the application root, ask the agent to inspect the source directly and
write `architecture.ai-graph.json`. Then run:

```bash
systemlens init
systemlens import-facts architecture.ai-graph.json \
  --namespace direct-analysis --complete
systemlens export microservices --html architecture.html
```

`systemlens init` creates configuration only. The import creates the empty
compatible SQLite schema and stores the manifest facts. No source extractor or
CodeQL analysis runs in this workflow.

## Five-step handoff

1. Scope the application and analysis question.
2. Read the source directly and collect relative evidence.
3. Create the JSON fact manifest.
4. Import the manifest into a dedicated namespace.
5. Generate and review the HTML export.

Read [`SKILL.md`](SKILL.md) for the operating rules and
[`references/fact-manifest.md`](references/fact-manifest.md) for the JSON
contract. The [site](https://elkouhen.github.io/systemlens-skill/) illustrates
the workflow with the supermarket POC.

The repository includes the reviewed POC manifest, including three direct
flows, at
[`examples/supermarket-direct-analysis.json`](examples/supermarket-direct-analysis.json).

## Related projects

- [SystemLens](https://github.com/elkouhen/systemlens) validates imports and
  renders the HTML graph.
- [SystemLens observability lab](https://github.com/elkouhen/systemlens-observability-lab)
  provides applications for practice and runtime validation.
