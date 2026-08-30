# systemlens-skill

Coding-agent skill for `systemlens`, a local Java/Spring architecture explorer based
on source ASTs.

The skill guides an agent through initialization, incremental indexing and
architecture exploration of complex repositories: microservices, HTTP APIs,
Kafka or other message channels, MongoDB or other databases, schemas, modules,
dependencies and topology risks. Unsupported conventions can be completed with
an evidence-based AI graph and reviewed MCP facts. The intended workflow is
iterative: index once with SystemLens, generate focused JSON fact passes, then
re-import them idempotently so newer evidence replaces older AI facts without
duplicating or overwriting source-derived facts.

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
systemlens analyze coverage
systemlens analyze indexing-issues
systemlens analyze audit
```

SystemLens uses local Java/Spring ASTs. No model download, rule pack or external
code-analysis service is required.

## Contents

- [`skills/systemlens/SKILL.md`](skills/systemlens/SKILL.md) — architecture-first workflow.
- [`skills/systemlens/references/settings.md`](skills/systemlens/references/settings.md) —
  project configuration.
- [`skills/systemlens/references/management.md`](skills/systemlens/references/management.md)
  — installation, MCP setup and troubleshooting.

## MCP

Start `systemlens mcp` from an initialized repository. For Codex:

```bash
codex mcp add systemlens -- systemlens mcp
```

## License

[Apache License 2.0](LICENSE).
