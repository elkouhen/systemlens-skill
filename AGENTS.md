# AGENTS.md — Development and validation guide

This repository contains the companion agent skill for SystemLens. Treat the
instructions, examples, and reference documents as executable product
contracts: a documented command or JSON shape must remain compatible with the
supported SystemLens CLI and MCP surface.

## Document map

| Document | Purpose | Read when |
|---|---|---|
| [`README.md`](README.md) | Installation and user-facing entry point | Changing setup or daily usage |
| [`PRD.md`](PRD.md) | Product scope, requirements, and acceptance criteria | Evaluating scope or changing workflows |
| [`SKILL.md`](SKILL.md) | Agent instructions and primary workflow | Changing any skill behaviour |
| [`references/analysis-rules.md`](references/analysis-rules.md) | Deterministic extraction and conservative inference rules | Changing analysis guidance |
| [`references/ai-graph.md`](references/ai-graph.md) | `systemlens-ai-graph-v1` contract | Changing generated topology facts |
| [`references/pass-profiles.md`](references/pass-profiles.md) | Focused analysis-pass contracts | Changing pass sequencing or outputs |
| [`references/settings.md`](references/settings.md) | Project configuration contract | Changing initialization or indexing guidance |
| [`references/management.md`](references/management.md) | Installation, MCP setup, refresh, and troubleshooting | Changing operational guidance |
| [`../systemlens/`](../systemlens/) | CLI, MCP, persistence, and export implementation | Verifying a command or public contract |
| [`../systemlens-observability-lab/`](../systemlens-observability-lab/) | Java/Spring integration fixture | Running cross-repository acceptance checks |

The sibling repositories may be absent outside a multi-repository workspace.
Skip local cross-repository checks when they are unavailable, and report which
compatibility checks were not run.

## Source of truth

Apply requirements in this order: explicit user and platform safety
requirements, the supported SystemLens public specifications, this repository's
PRD, `SKILL.md`, reference documents, then examples.

Do not document commands, flags, MCP tools, JSON fields, or behavior from memory.
Verify them against the current SystemLens source or an installed supported
version. When a SystemLens change affects this skill, update the relevant skill
documents in the coordinated change and identify the minimum compatible
SystemLens version or revision in the handoff.

## Maintenance rules

1. Write user-facing skill documentation and examples in English.
2. Keep `README.md` concise; detailed behavior belongs in `SKILL.md` or the
   relevant reference document.
3. Update `PRD.md` when product scope, quality requirements, or acceptance
   criteria change.
4. Keep relative Markdown links valid and keep every referenced file under the
   skill directory unless the link intentionally names a sibling repository.
5. Preserve the frontmatter fields required by skill consumers: `name` and
   `description` in `SKILL.md`.
6. Use paths relative to the analyzed project root in generated fact examples.
   Never include machine-specific source roots.
7. Never place credentials, tokens, private keys, or unredacted secret values in
   instructions, examples, fixtures, or generated artifacts.
8. Keep deterministic SystemLens facts distinct from AI-produced facts. Retain
   explicit confidence, status, provenance, ambiguity, and namespace ownership.
9. Do not silently add an imported ordered-flow format before SystemLens exposes
   a versioned public contract for it.
10. When a workflow, command, option, MCP tool, configuration field, or data
    contract changes, update every affected example in `README.md`, `SKILL.md`,
    and `references/` in the same pass. Before creating a commit, verify that
    these examples are up to date and validate them against the supported
    SystemLens version; do not commit while an affected example is stale.
11. Make targeted changes and preserve unrelated user edits in all repositories.

## Validation

Before delivering a documentation-only change:

```bash
git diff --check
```

Also verify that `SKILL.md` frontmatter parses, local Markdown links resolve,
and every JSON example is syntactically valid. If a repository validation
script or CI target exists, use it instead of an ad-hoc substitute.

For any command, workflow, MCP, settings, or graph-contract change, run the
cross-repository compatibility check when both sibling repositories are
available. It must use the development checkout of `../systemlens/`, not an
unqualified globally installed executable, and must exercise a temporary copy
of the Java application under
`../systemlens-observability-lab/apps/supermarket-demo/`. At minimum verify:

```bash
SYSTEMLENS_BIN="$(cd ../systemlens && pwd)/.venv/bin/systemlens"
"$SYSTEMLENS_BIN" version
cd <temporary-app-copy>
"$SYSTEMLENS_BIN" doctor
"$SYSTEMLENS_BIN" index
"$SYSTEMLENS_BIN" flows --json
"$SYSTEMLENS_BIN" export microservices --html <temporary-output>
```

The compatibility check must not modify the sibling laboratory checkout or
reuse its persisted `.systemlens/findings.db`. Validate structured output and
confirm that the generated HTML contains graph data. Report unavailable Java,
Maven, `uv`, or browser prerequisites rather than claiming the check passed.

## Review checklist

Before handoff, inspect the relevant perspectives:

- Product: the workflow answers a concrete architecture-analysis task and stays
  within the PRD.
- Contract: every command and JSON field exists in the supported SystemLens
  surface.
- Data quality: evidence, confidence, ambiguity, and fact ownership remain
  explicit.
- Operations and security: the workflow is local-first, avoids secrets, and
  does not mutate the analyzed application unless explicitly requested.
- Compatibility: the skill, the development SystemLens checkout, and the Java
  laboratory fixture were checked together, or the missing prerequisite is
  stated.
