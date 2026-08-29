# AI-produced graph manifests

Use this format when source conventions are too complex for SystemLens's
deterministic extractors. The manifest is an analysis artifact: it is not a
replacement for indexed source evidence and it must not contain secrets or
absolute machine paths.

## `systemlens-ai-graph-v1`

```json
{
  "format": "systemlens-ai-graph-v1",
  "project": "project-name",
  "generated_by": {"agent": "agent-name", "model": "model-id", "source_revision": "commit-or-unknown"},
  "nodes": [
    {"id": "orders", "kind": "service", "name": "orders", "evidence": [{"path": "src/main/java/Orders.java", "start_line": 12, "end_line": 20}]},
    {"id": "orders-created", "kind": "topic", "name": "orders.created"}
  ],
  "edges": [
    {"id": "orders-publishes-created", "source": "orders", "target": "orders", "kind": "event", "channel": "orders.created", "status": "confirmed", "confidence": "high", "message_type": "OrderCreated", "evidence": [{"path": "src/main/java/Orders.java", "start_line": 42, "end_line": 42}]}
  ]
}
```

Allowed node kinds are `service`, `external_service`, `topic`, and
`collection`. A collection must have an `owner` naming a service. Allowed
edge kinds are `http`, `event`, and `data`; HTTP and event edges connect two
services, while data edges connect a service to an owned collection. Event
edges use `channel` for the exact topic or channel expression. Topic nodes are
optional metadata and are derived visually from event channels.

Every edge must include `status` (`confirmed`, `proposed`, `ambiguous`, or
`unresolved`) and `confidence` (`high`, `medium`, `low`, or `unknown`). Include
`reason` for ambiguous or unresolved claims. Include short relative source
evidence whenever available. If a convention cannot be resolved to one target,
emit an ambiguous/unresolved edge instead of choosing a likely service.

Generate it with the code revision and analysis method in `generated_by`. Keep
the graph bounded and use stable IDs. Do not include source contents beyond a
short quote, credentials, tokens, request bodies, or absolute paths.

## Display in SystemLens

```bash
systemlens export microservices \
  --graph architecture.ai-graph.json \
  --html architecture.html \
  --root-path /path/to/checkout
```

SystemLens displays confirmed and proposed claims. Ambiguous and unresolved
claims are kept out of the dependency lines and shown in the Quality panel.
The manifest is read-only input and is not persisted into the SQLite source
index.
