# Fact manifest contract

The direct-analysis handoff uses `systemlens-ai-graph-v1`. The manifest is a
reviewable snapshot of facts produced by the skill, not a source index.

## Required shape

```json
{
  "format": "systemlens-ai-graph-v1",
  "project": "application-name",
  "generated_by": {
    "agent": "direct-source-analysis",
    "model": "model-name",
    "source_revision": "revision-or-working-tree",
    "pass": "direct-001",
    "namespace": "direct-analysis"
  },
  "mode": "partial",
  "nodes": [],
  "edges": []
}
```

## Nodes

Supported node kinds include `service`, `external_service`, `topic`,
`collection`, `data_schema`, and `message_channel`. Every node has a stable
`id`, a display `name`, and an `evidence` list when source evidence exists.
Collections include an `owner` service name.

## Edges

An edge has a stable `id`, `source`, `target`, `kind`, `relation`, `status`,
`confidence`, and `evidence`. Event edges carry `channel` with the exact topic
or channel expression. HTTP edges carry `channel` or `label` with the exact
route or route expression. Ambiguous and unresolved edges require `reason`.

Use `confirmed` when the source supports the relation directly. Use `proposed`
when the relation is plausible but needs review. Use `ambiguous` or
`unresolved` when the source does not identify a single target or channel.

## Evidence rules

Evidence paths are relative to the analyzed application root. Line numbers are
one-based. Short quotes are optional and must not contain secrets. Absolute
paths, Windows paths, credentials, tokens, and unredacted configuration values
are invalid.

## Import semantics

`systemlens import-facts` stores nodes and edges under the manifest namespace.
The importer preserves source-derived facts separately. A `partial` manifest
adds or updates only its listed facts. A `complete` manifest removes stale
facts from its namespace when imported with `--complete`.

## Optional direct flows

The same manifest may include `endpoints` and `flows` for source-evidenced
causal analysis. An endpoint identifies a service, integration system, role,
channel, relative source path, and line range. A flow identifies its service,
method, status, confidence, reason, and ordered steps. Each step may reference
an endpoint ID.

When these arrays are present, `systemlens import-facts` persists them in the
empty repository snapshot. The importer rejects this operation when indexed
source endpoints or modules already exist. The HTML export can then expose the
selected flow and call-tree views without running source indexing.
