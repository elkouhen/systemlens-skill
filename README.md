# systemlens-skill

This companion skill installs and verifies CodeQL, creates a CodeQL-backed
SystemLens source index, analyzes the application directly for complementary
architecture facts, imports those facts from a `systemlens-ai-graph-v1` JSON
manifest, and produces an HTML graph.

## Quick start

Install the skill with:

```bash
npx skills add elkouhen/systemlens-skill
```

From the application root, install the CodeQL CLI using the [official CodeQL
CLI setup instructions](https://docs.github.com/en/code-security/how-tos/find-and-fix-code-vulnerabilities/scan-from-the-command-line/set-up-codeql-cli),
put it on `PATH`, and verify it:

```bash
codeql version
```

Then create the source baseline and ask the agent to inspect the source
directly for complementary facts:

```bash
systemlens init
mkdir -p .codeql
codeql database create .codeql/systemlens-java \
  --language=java --source-root=. --build-mode=none
systemlens index --full --call-graph-engine codeql \
  --codeql-database .codeql/systemlens-java
systemlens import-facts architecture.ai-graph.json \
  --namespace direct-analysis --complete
systemlens export microservices --html architecture.html
```

The manifest should contain complementary facts only after indexing. The
indexed snapshot remains the source of truth for modules, endpoints and code
flows. Direct bootstrap import with optional manifest flows remains available
when CodeQL cannot run.

## Seven-step handoff

1. Scope the application and analysis question.
2. Install and verify CodeQL.
3. Create the CodeQL database and index with SystemLens.
4. Read the source directly and collect relative evidence.
5. Create the JSON fact manifest.
6. Import the manifest into a dedicated namespace.
7. Generate and review the HTML export.

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
