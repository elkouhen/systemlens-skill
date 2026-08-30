---
name: systemlens
description: "Guide iterative architecture analysis with SystemLens, enriching an indexed repository with replaceable JSON facts about microservices, APIs, message topics or queues, databases, schemas, dependencies, and exports."
---

# systemlens Architecture Explorer

Use `systemlens` to query a local, persisted architecture inventory, primarily
for Java/Spring repositories. For polyglot or convention-heavy repositories,
combine the deterministic inventory with the AI graph and MCP enrichment paths
below; do not present unsupported source as if it were deterministically indexed.
Start with the inventory before inspecting implementation files. It describes
services, modules, HTTP APIs, data schemas, message channels and their
evidenced relationships. MongoDB collections and Kafka topics are concrete
technology views of the generic schema/channel vocabulary.

## Documentation language

Write user-facing documentation, generated report text and examples in English.
Preserve exact CLI output, source snippets and user-provided text when quoting
them.

## Ownership

Work from the target repository root. Own the `systemlens` lifecycle for that
project.

- Run `systemlens doctor` before drawing broad architectural conclusions.
- If configuration is missing, run `systemlens init`; never overwrite an existing
  `.systemlens/config.yml`.
- If the index is absent or source/configuration changed, run `systemlens index`
  before querying. Use `systemlens index --full` after a broad refactor or an
  extractor upgrade.
- Treat `systemlens` as static, source-evidenced analysis. Keep dynamic,
  ambiguous and unresolved integrations explicit; do not infer a dependency from
  a coincidental route or a topic name.

## Setup

For a fresh project:

```bash
systemlens init
systemlens doctor
systemlens index
```

The configuration controls the indexed source perimeter. See
[settings.md](references/settings.md) only when the perimeter must change.

## Architecture workflow

Start with the indexed architecture and retrieve source evidence only when needed:

1. `systemlens microservices` — discover services and main integrations.
2. `systemlens microservices show <name>` — inspect one service.
3. `systemlens microservices topics <name>`, `apis <name>` or `mongodb <name>` —
   follow linked objects.
4. `systemlens topics`, `systemlens apis`, `systemlens dtos` or
   `systemlens mongodb` — inspect a shared object from the other direction.
5. `systemlens analyze coverage` and `systemlens analyze indexing-issues` —
   identify inventory limits before relying on a conclusion.
6. `systemlens analyze audit` — assess static topology risks.
7. `systemlens analyze microservices impact <name>` or `path <source> <target>` —
   inspect dependencies and impact paths.

For a complex codebase, build the analysis in passes and keep the result
evidence-based:

1. Establish the repository perimeter and build/deployment units: services,
   modules, applications, Docker/Kubernetes/Helm manifests and configuration
   files. Record the source path that establishes each service boundary.
2. For every service, inventory inbound/outbound HTTP, published/consumed
   channels, and read/write data stores. Separate a logical service from its
   deployable module, database/schema, table/collection, topic/queue/exchange,
   and external dependency.
3. Correlate both directions: producer → channel → consumer, service → API →
   service, and service → data schema. Require a concrete shared identifier
   before creating an edge; retain candidate links as ambiguous when several
   targets match.
4. Run `coverage`, `indexing-issues` and `audit`, then inspect source evidence
   for every important conclusion. Report blind spots (unsupported language,
   generated code, dynamic configuration, missing manifests or runtime-only
   wiring) explicitly.
5. If deterministic extraction is incomplete, create a bounded AI graph and
   optionally persist only reviewed claims through MCP. Keep the deterministic
   snapshot and AI/runtime observations distinguishable in the final report.

The minimum useful deliverable for a complex repository is a service matrix
with service boundary, APIs, channels, data stores, evidence paths and
confidence, plus a list of unresolved relationships. Include technology and
ownership where the repository proves them; never infer ownership from naming
alone.

```bash
systemlens microservices show order-service
systemlens topics consumers orders.created
systemlens apis consumers 'POST /payments'
systemlens mongodb services orders
systemlens modules show shared-domain
systemlens analyze coverage
systemlens analyze indexing-issues
systemlens analyze audit
```

Kafka message types are shown only when explicit in the source. A missing type
is unknown, not an invitation to infer it from a topic name or serializer.

## Iterative AI fact workflow

Use this path to progressively complete a SystemLens model when repository
conventions are not covered by deterministic extractors. The persisted model
combines the SystemLens index with the latest accepted AI facts:

1. Set `PROJECT_ROOT` to the repository being analysed and run the normal
   SystemLens baseline (`doctor`, `init` if needed, then `index`). Keep
   Strategy1 opt-in; do not enable it unless the repository follows the rules
   in [analysis-rules.md](references/analysis-rules.md).
2. On each analysis pass, inspect the relevant source, configuration,
   manifests and previous findings, then produce a JSON fact file such as
   `architecture.ai-graph.pass-001.json` using the
   `systemlens-ai-graph-v1` contract. Use stable IDs and relative evidence
   paths. A later pass may repeat a fact with better evidence or corrected
   metadata; it must retain the same identity for the same logical fact.
3. Validate the JSON before import: IDs are unique, edge endpoints exist,
   statuses/confidence are valid, evidence is relative, and
   ambiguous/unresolved claims include a reason.
4. Import the complete fact file through the SystemLens/MCP enrichment surface
   as an upsert/reconciliation operation. Match existing AI facts by stable
   identity and replace their full value, evidence, status, confidence and
   metadata. Do not append a second copy and do not skip an improved fact just
   because it already exists.
5. Scope replacement to facts managed by the current JSON producer or
   namespace. Never delete or overwrite source-derived SystemLens facts, and
   never delete unrelated AI facts from earlier passes. If the available MCP
   surface only supports add/remove, use a bounded list → remove matching AI
   assertion → add replacement sequence, then verify the result.
6. Read the merged result with `architecture_graph`, review the import summary
   and `list_graph_facts`, then export the current model if needed:
   SystemLens's HTML renderer:

   ```bash
   cd "$PROJECT_ROOT"
   systemlens import-facts architecture.ai-graph.pass-001.json \
     --namespace ai-architecture
   systemlens export microservices \
     --graph architecture.ai-graph.json \
     --html architecture.html \
     --root-path "$PROJECT_ROOT"
   ```

7. Open `architecture.html`. Confirmed and proposed relations are drawn;
   ambiguous and unresolved claims are available in the Quality panel. The
   HTML graph is read-only; importing the JSON is what updates the persisted
   enrichment layer.

An agent can use this prompt as a bounded starting point:

> Analyse the code and configuration under this directory. First identify
> all deployable services/modules, external systems, event topics or queues,
> HTTP routes, database schemas/tables/collections and their read/write sites.
> Then
> write `architecture.ai-graph.json` in `systemlens-ai-graph-v1` format. Keep
> all evidence paths relative to the directory, include short reasons for
> ambiguous or unresolved relationships, and do not guess dynamic targets.
> Give the file a stable pass name, validate it, import it as a replacement of
> the facts in its producer namespace, verify the merged graph, and finally run
> `systemlens export microservices --graph architecture.ai-graph.json --html
> architecture.html --root-path .` when an HTML handoff is needed.

Use `--json` when a downstream step needs structured results. The MCP surface
supports the same workflow through `index_repository`, `import_graph_facts` and
`architecture_graph`. The returned graph includes legacy API, MongoDB
collection and Kafka associations as well as generic schema/channel facts.

For the complete extraction contract, including supported Java/Spring forms,
dynamic-value handling, exact REST target resolution, Kafka matching,
module/OpenAPI attribution, MongoDB evidence and Strategy1 conventions, read
[analysis-rules.md](references/analysis-rules.md). Never enable Strategy1 just
to force an expected edge; verify the repository convention first.

When the repository follows the documented `getTopics()` and
`${kafka.topics.*.name}` conventions, use the explicit strategy:

```bash
systemlens index --topic-strategy strategy1
```

It may derive convention-based Kafka and configured HTTP dependencies. Treat
these as convention-derived facts and inspect their source evidence. Do not
enable this strategy solely to make an expected relation appear.

For the Strategy1 OpenAPI publication convention, a declaration at
`src/main/resources/openapi/xxx.rest` can publish valid same-named contracts
anywhere in the repository. It also publishes every valid YAML or JSON OpenAPI
contract under `model-xxx/src/main/resources/openapi/`; contract file names do
not need to match `xxx`. Reindex with `--full --topic-strategy strategy1` after
adding or moving a declaration or shared contract.

Every build module independently inventories all valid YAML or JSON OpenAPI
documents under its own `src/main/resources/openapi/` directory. Contract file
names do not need to be `openapi.*` or `swagger.*`.

## Exports and MCP

```bash
systemlens export microservices --html architecture.html
systemlens export microservices --c4 likec4-project
systemlens export modules --html module-dependencies.html
systemlens mcp
```

The HTML export includes persisted MCP graph facts. After enriching a graph,
rerun the export command to display added `data_schema` and
`message_channel` nodes and their relations.

When deterministic extraction cannot resolve repository-specific conventions,
have the analysis agent produce a `systemlens-ai-graph-v1` manifest. Render it
temporarily when reviewing it, or import it to the enrichment layer when the
facts have been validated:

```bash
systemlens export microservices --graph architecture.ai-graph.json --html architecture.html --root-path /path/to/checkout
```

The manifest must keep source evidence relative, include confidence and
`confirmed`/`proposed`/`ambiguous`/`unresolved` status on every claim, and give
an explicit reason for unresolved claims. SystemLens draws confirmed and
proposed claims and puts ambiguous or unresolved claims in the Quality panel;
it never turns an AI guess into persisted source topology; persistence occurs
only through explicit `import-facts`/`import_graph_facts`. See
[ai-graph.md](references/ai-graph.md) for the full contract and example.

Exports consume the persisted snapshot; refresh the index deliberately when
source changes must be reflected. Use `--root-path` only to resolve local source
links at HTML export time.

The MCP server is a control surface for the two-layer workflow: call
`index_repository` first, then reconcile JSON facts into the enrichment layer
using `fact_type=node` or `fact_type=edge`. Reconciliation is idempotent: the
same stable identity updates the existing AI assertion instead of creating a
duplicate. Use `architecture_graph` for the merged generic graph.
`remove_graph_fact` is limited to matching AI assertions during replacement or
explicit cleanup; it never deletes source-derived facts.

For a data resource, use `kind=data_schema` and set `technology` to
`mongodb`, `postgresql`, `redis`, `s3`, or another provider. For messaging,
use `kind=message_channel` and set `technology` to `kafka`, `rabbitmq`, `sqs`,
or another provider. Put provider-specific identifiers in `metadata`, for
example `{database, schema, table}` or `{exchange, queue}`. Use edge kinds
such as `provides`, `calls`, `reads`, `writes`, `publishes`, and `consumes`.

## References

- [settings.md](references/settings.md): project configuration.
- [analysis-rules.md](references/analysis-rules.md): deterministic extraction,
  conservative resolution and Strategy1 rules.
- [ai-graph.md](references/ai-graph.md): versioned AI-produced graph manifests.
- [management.md](references/management.md): installation, refresh and
  troubleshooting.
