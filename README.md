# systemlens-skill

Coding-agent skill for `systemlens`, a local Java/Spring architecture explorer based
on source ASTs.

`systemlens` is the analysis product (CLI and MCP server). This repository is
`systemlens-skill`: agent guidance that helps use the product conservatively;
it does not define or extend the product's public contract.

The skill guides an agent through initialization, incremental indexing and
architecture exploration of complex repositories: microservices, APIs,
Topics, Data resources, modules,
dependencies and topology risks. Unsupported conventions can be completed with
an evidence-based AI graph and reviewed MCP facts. The intended workflow is
iterative: index once with SystemLens, generate focused JSON fact passes, then
re-import them idempotently so newer evidence replaces older AI facts without
duplicating or overwriting source-derived facts.

Each JSON facts manifest is the durable, reviewable handoff between the agent
and SystemLens. It owns only its namespace and supplements, rather than
changes, the deterministic source inventory.

For complex codebases, the skill provides five focused passes: `boundaries`,
`http`, `messaging`, `data`, and `deployment`. Boundaries runs first; API,
messaging, and Data may then run in parallel; deployment closes the loop by
mapping logical components to runtime resources. Each pass has its own namespace
and replaceable JSON artifact. A separate `flows` review reconstructs selected
business outcomes as evidence-backed potential source flows; it is reported in
Markdown and is not imported as an ordered topology graph.

## Install

```bash
npx skills add elkouhen/systemlens-skill
uv tool install systemlens
```

In the Java/Spring repository to inspect:

```bash
systemlens init
systemlens doctor
systemlens index

# Have the agent write reviewable, relative-evidence facts in JSON.
# Import the facts into the AI architecture namespace.
systemlens import-facts architecture.ai-graph.pass-001.json \
  --namespace ai-architecture
systemlens export microservices --html architecture.html
```

SystemLens uses local Java/Spring ASTs and, when the local CodeQL CLI is
provisioned, CodeQL for bounded interprocedural flow analysis. No model download
or remote code-analysis service is required during indexing.

## Contents

- [`PRD.md`](PRD.md) — product requirements and completion objective for the skill.
- [`BACKLOG.md`](BACKLOG.md) — prioritized architecture-analysis work items.
- [`SKILL.md`](SKILL.md) — architecture-first workflow.
- [`references/pass-profiles.md`](references/pass-profiles.md)
  — boundaries, API, messaging, Data and deployment pass contracts.
- [`references/business-flows.md`](references/business-flows.md) — selecting and
  reporting potential business flows from source evidence.
- [`references/settings.md`](references/settings.md) —
  project configuration.
- [`references/management.md`](references/management.md)
  — installation, MCP setup and troubleshooting.

## MCP

Start `systemlens mcp` from an initialized repository. For Codex:

```bash
codex mcp add systemlens -- systemlens mcp
```

## License

[Apache License 2.0](LICENSE).
