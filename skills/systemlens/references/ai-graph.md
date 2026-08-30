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
    {"id": "billing", "kind": "service", "name": "billing"},
    {"id": "orders-created", "kind": "topic", "name": "orders.created"}
  ],
  "edges": [
    {"id": "orders-to-billing", "source": "orders", "target": "billing", "kind": "event", "channel": "orders.created", "status": "confirmed", "confidence": "high", "message_type": "OrderCreated", "evidence": [{"path": "src/main/java/Orders.java", "start_line": 42, "end_line": 42}]}
  ]
}
```

Allowed node kinds in the legacy file export are `service`, `external_service`,
`topic`, and `collection`. For the persistent MCP enrichment workflow, prefer
the generic node kinds `data_schema` and `message_channel`, with a `technology`
field such as `postgresql`, `redis`, `rabbitmq`, or `sqs`, and a `metadata`
object for provider-specific identifiers. A data schema can be a MongoDB
collection, SQL table, keyspace, bucket or another persisted data contract; a
message channel can be a Kafka topic, queue, exchange, subscription or stream.
Allowed legacy edge kinds are `http`, `event`, and `data`; MCP enrichment also
accepts relation labels such as `provides`, `calls`, `reads`, `writes`,
`publishes`, and `consumes`.

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

For an AI analysis that should be persisted and visible in the complete MCP
graph, call `graph_fact_exists` before `add_graph_fact` for each confirmed node
and edge instead of relying only on this read-only manifest. Review with
`list_graph_facts` and
read the merged topology with `architecture_graph`.
