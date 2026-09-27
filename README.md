# systemlens-skill

Companion skill for `systemlens`, a local Java/Spring architecture explorer.

**Website:** [systemlens-skill on GitHub Pages](https://elkouhen.github.io/systemlens-skill/)

## Scope

This repository owns agent guidance for completing SystemLens findings with
reviewable AI explanations and complementary facts. It does not own the
SystemLens CLI, MCP server, deterministic index, or HTML export.

Use [SystemLens](https://github.com/elkouhen/systemlens) for repository
indexing and graph exploration. Use this skill when an agent needs to complete
a bounded finding with evidence. Explaining persisted call graphs is one
example.

## Related projects

- [SystemLens](https://github.com/elkouhen/systemlens) owns indexing, graph
  exploration, and exports.
- [SystemLens observability lab](https://github.com/elkouhen/systemlens-observability-lab)
  owns the runnable observability environment.

The skill adds reviewable AI explanations and complementary findings around a
deterministic SystemLens result. It does not replace indexing, alter source
evidence, or become a second source of truth.

It can describe persisted potential flows in plain language. Its optional
topology passes cover bounded gaps that require reviewable evidence outside
deterministic extraction.

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

For a complete CodeQL analysis, the skill can also generate one AI-written
description per persisted flow. These descriptions are stored separately in
`.systemlens/flow-descriptions.json` and loaded by the HTML export by flow ID;
they enrich presentation without changing indexed facts. See
[`references/flow-descriptions.md`](references/flow-descriptions.md) and the
[`full CodeQL prompt`](prompts/full-index-codeql-flow-descriptions.md).

For a copyable example prompt that explains persisted call graphs, see
[`prompts/enrich-architecture.md`](prompts/enrich-architecture.md).

## Install

```bash
npx skills add elkouhen/systemlens-skill
```

Install SystemLens separately, then follow its product documentation to index
the repository before asking the agent to complete findings:

```bash
uv tool install systemlens
```

See the [SystemLens quick start](https://github.com/elkouhen/systemlens#quick-start)
for product commands.

## Contents

- [`PRD.md`](PRD.md) — product requirements and completion objective for the skill.
- [`BACKLOG.md`](BACKLOG.md) — prioritized architecture-analysis work items.
- [`SKILL.md`](SKILL.md) — architecture-first workflow.
- [`prompts/enrich-architecture.md`](prompts/enrich-architecture.md) — example
  prompt for explaining persisted call graphs.
- [`references/pass-profiles.md`](references/pass-profiles.md)
  — boundaries, API, messaging, Data and deployment pass contracts.
- [`references/business-flows.md`](references/business-flows.md) — selecting and
  reporting potential business flows from source evidence.
- [`references/flow-descriptions.md`](references/flow-descriptions.md) — AI
  descriptions for persisted flows and the HTML enrichment contract.
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
