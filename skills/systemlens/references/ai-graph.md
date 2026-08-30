# AI-produced graph manifests

Use this format when source conventions are too complex for SystemLens's
deterministic extractors. The manifest is an analysis artifact: it is not a
replacement for indexed source evidence and it must not contain secrets or
absolute machine paths.

## `systemlens-ai-graph-v1`

The file is a replaceable fact set, not an append-only event log. It can be
generated repeatedly as `architecture.ai-graph.pass-NNN.json` while the
repository is analysed in focused passes.

```json
{
  "format": "systemlens-ai-graph-v1",
  "project": "project-name",
  "generated_by": {"agent": "agent-name", "model": "model-id", "source_revision": "commit-or-unknown", "pass": "pass-001", "namespace": "ai-architecture"},
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

### Identity and replacement

Use a deterministic identity for every node and edge. The `id` must remain
unchanged when a later pass revises the same logical fact; changed evidence,
status, confidence, metadata or target value is an update, not a new fact.
Include `generated_by.namespace` so imports can replace only facts owned by
this analysis.

An importer reconciles by `(namespace, fact_type, id)`:

- insert facts that do not exist;
- replace the full stored value for facts with the same identity;
- leave source-derived facts and other namespaces untouched;
- remove stale facts only when the manifest is declared complete for that
  namespace, never when it is a partial pass.

The replacement should be atomic per fact or batch when supported. Otherwise
use list → remove matching AI assertion → add replacement, then read back and
verify. Duplicate semantic facts collapse to the newest imported value.
Record pass, source revision and evidence so the latest value remains
auditable.

Allowed node kinds in the legacy file export are `service`, `external_service`,
`topic`, and `collection`. For the persistent MCP enrichment workflow, prefer
the generic node kinds `data_schema` and `message_channel`, with a `technology`
field such as `postgresql`, `redis`, `rabbitmq`, or `sqs`, and a `metadata`
object for provider-specific identifiers. A data schema can be a MongoDB
collection, SQL table, keyspace, bucket or another persisted data contract; a
message channel can be a Kafka topic, queue, exchange, subscription or stream.
For complex or polyglot repositories, service nodes may include `technology`,
`module`, `runtime` and `deployment` metadata; data/message nodes may include
`namespace`, `database`, `schema`, `table`, `collection`, `queue`, `exchange`,
`consumer_group` or `binding` metadata. Use only fields established by source,
configuration or deployment evidence.
Allowed legacy edge kinds are `http`, `event`, and `data`; MCP enrichment also
accepts relation labels such as `provides`, `calls`, `reads`, `writes`,
`publishes`, and `consumes`.

Every edge must include `status` (`confirmed`, `proposed`, `ambiguous`, or
`unresolved`) and `confidence` (`high`, `medium`, `low`, or `unknown`). Include
`reason` for ambiguous or unresolved claims. Include short relative source
evidence whenever available. If a convention cannot be resolved to one target,
emit an ambiguous/unresolved edge instead of choosing a likely service.

Prefer these edge semantics when enriching a complex repository:
`serves`/`calls` for HTTP, `publishes`/`consumes` for channels, and
`reads`/`writes` for data schemas. Keep the channel or schema as an explicit
node when it is identifiable, so the graph can show fan-in, fan-out and shared
data stores. Add `reason` to proposed claims too when the claim depends on a
repository convention rather than a literal reference.

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
graph, reconcile every node and edge by stable identity. `graph_fact_exists`
may inspect the current value, but must not cause an improved fact to be
skipped. Use an update/upsert operation when available; otherwise remove and
re-add only the matching AI assertion. Review with `list_graph_facts` and read
the merged topology with `architecture_graph`.
