# Architecture pass profiles

Use pass profiles to build an architecture model incrementally. Each pass has a
narrow question, a dedicated fact namespace, and a replaceable JSON snapshot.
The model becomes richer after each import while code-derived facts remain
untouched.

## Shared contract

Every profile produces an artifact named:

```text
architecture.ai-<profile>.pass-<NNN>.json
```

The manifest should use the `systemlens-ai-graph-v1` format and include stable
identifiers, source evidence, and provenance:

```json
{
  "format": "systemlens-ai-graph-v1",
  "project": "<repository-name>",
  "generated_by": {
    "agent": "<agent-name>",
    "model": "<model-name>",
    "source_revision": "<commit-or-revision>",
    "pass": "<profile>-<NNN>",
    "namespace": "ai-<profile>"
  },
  "mode": "partial",
  "nodes": [],
  "edges": []
}
```

Use `mode: "partial"` unless the pass inspected the complete profile scope.
`partial` upserts the facts present in the file and preserves other facts in
the same namespace. `complete` declares the file to be the full snapshot for
that namespace and removes facts previously owned by that namespace but absent
from the new snapshot.

Keep node and edge IDs stable across passes. Prefer IDs derived from the
repository identity and logical name, not from line numbers or generated text.
Attach evidence such as file path, line or symbol, configuration key, and a
short observation. Never copy credentials, tokens, or secret values into a
manifest.

Import each artifact explicitly:

```bash
systemlens import-facts architecture.ai-<profile>.pass-<NNN>.json \
  --namespace ai-<profile>
```

Use `--complete` only for a deliberate full-scope snapshot. Review the import
summary (`inserted`, `updated`, and `removed`) and query the graph after every
import. If evidence is ambiguous, emit an unresolved fact with its reason or
omit the fact; do not turn an inference into a confirmed dependency.

## Recommended execution order

Run the profiles in this order:

1. `boundaries` establishes the service, module, external-system, and runtime
   vocabulary.
2. `http`, `messaging`, and `data` can run in parallel once boundaries exist.
3. `deployment` maps logical components to the environments and runtime
   resources that host them.

The default sequence is therefore:

```text
boundaries → (http || messaging || data) → deployment
```

Repeat only the affected profiles after a code or infrastructure change. A
profile may reference entities discovered by an earlier profile, but it must
not silently rewrite another profile's namespace.

## `boundaries`

Namespace: `ai-boundaries`  
Artifact: `architecture.ai-boundaries.pass-<NNN>.json`

### Question

What are the deployable services, applications, modules, libraries, external
systems, and ownership boundaries?

### Inspect

- Repository and build roots (`pom.xml`, `build.gradle`, `package.json`, Go
  modules, workspace files, and equivalent manifests).
- Service directories, executable entry points, Dockerfiles, and compose files.
- Module/package declarations and explicit dependency manifests.
- Repository documentation only as corroborating evidence.

### Emit

- `service`, `module`, `external_system`, and `deployment` nodes where
  justified.
- Containment and dependency relationships. Use an allowed graph edge kind
  such as `provides` with `relation: "contains"` when representing logical
  containment.
- Ownership or responsibility only when it is explicit in code, configuration,
  or repository metadata.

### Exit criteria

Every emitted service has an entry point or build/runtime evidence, a source
path, and a stable identity. Unclear repository folders remain modules or
unresolved observations instead of being promoted to microservices.

## `http`

Namespace: `ai-http`  
Artifact: `architecture.ai-http.pass-<NNN>.json`

### Question

Which synchronous HTTP interfaces exist, and which components call or serve
them?

### Inspect

- Controllers, routers, handlers, middleware, and route registration.
- OpenAPI specifications and generated clients.
- HTTP client usage (`Feign`, `WebClient`, `RestTemplate`, `fetch`, `axios`,
  gRPC-over-HTTP gateways, and equivalents).
- Base URLs, service discovery names, ingress routes, and timeout/retry
  configuration.

### Emit

- API nodes with method, path, version, protocol, and owning component.
- `serves` edges from a service to an API.
- `calls` edges from a caller to an API or target service, including the
  configured destination when it can be proven.
- Evidence for authentication, timeout, retry, or circuit-breaking policies
  when explicitly configured.

### Exit criteria

Each API has a concrete route/spec/client reference. A hostname alone is not
enough to identify a service call; mark the target unresolved when service
discovery or configuration cannot be resolved.

## `messaging`

Namespace: `ai-messaging`  
Artifact: `architecture.ai-messaging.pass-<NNN>.json`

### Question

Which asynchronous channels exist, who publishes and consumes them, and what
delivery contract do they imply?

### Inspect

- Kafka, RabbitMQ, SQS/SNS, NATS, Redis Streams/Pub/Sub, JMS, and equivalent
  producers and consumers.
- Topic, queue, exchange, routing-key, subscription, consumer-group, and
  dead-letter configuration.
- Event schemas, serializers, message envelopes, retry policies, and ordering
  or idempotency settings.

### Emit

- `message_channel` nodes with broker, channel name, protocol, schema, and
  delivery metadata when available.
- `publishes` edges from producers to channels.
- `consumes` edges from channels to consumers.
- Separate facts for dead-letter channels and retry channels when they are
  configured resources, not merely inferred behavior.

### Exit criteria

The channel name and its evidence are recorded. Publisher/consumer direction
is explicit, and a shared payload type is not presented as a guaranteed event
schema unless serialization or a schema registry proves it.

## `data`

Namespace: `ai-data`  
Artifact: `architecture.ai-data.pass-<NNN>.json`

### Question

Which databases, schemas, tables, collections, buckets, and caches exist, and
which components read or write them?

### Inspect

- Datasource configuration, connection names, migration files, ORM mappings,
  repositories, queries, and transaction boundaries.
- PostgreSQL/MySQL and other SQL schemas, MongoDB/Document stores, Redis,
  Elasticsearch, object storage, and embedded databases.
- Backup, retention, encryption, replica, and ownership configuration where
  present.

### Emit

- `data_store` nodes for physical stores and `data_schema` nodes for logical
  schemas, tables, collections, indexes, buckets, or keyspaces.
- `reads` and `writes` edges with operation or access evidence.
- Ownership, source-of-truth, replication, and shared-database observations
  only when explicit or clearly labelled as hypotheses.
- Migration/version evidence to distinguish current schema from historical
  definitions.

### Exit criteria

Every read/write edge points to a concrete query, repository, mapping, or
configuration source. Connection strings are represented by safe references
such as a variable name, never by their secret value.

## `deployment`

Namespace: `ai-deployment`  
Artifact: `architecture.ai-deployment.pass-<NNN>.json`

### Question

Where do logical components run, and which infrastructure/configuration binds
them to runtime environments?

### Inspect

- Dockerfiles, Compose, Kubernetes manifests, Helm charts, Kustomize, and
  serverless definitions.
- Terraform/Pulumi/CloudFormation and CI/CD deployment workflows.
- Deployments, StatefulSets, Jobs, Services, Ingresses, queues, databases,
  config maps, secret references, probes, replicas, resources, and network
  policies.
- Environment overlays and image/tag references.

### Emit

- `deployment`, `runtime`, `external_system`, and infrastructure resource
  nodes where the evidence supports them.
- Deployment-to-component and runtime-to-resource relationships. Use an
  allowed edge kind such as `provides` with a precise `relation` value such as
  `deployed_as`, `uses`, or `exposes`.
- Environment, namespace, region, image, replica, scaling, health-check, and
  configuration-reference facts.
- Secret/config references by name and scope, never material values.

### Exit criteria

Each deployment relationship links a logical component to a concrete manifest,
workload, or pipeline target. Environment overlays are kept distinct, and a
local Compose dependency is not claimed to be the production topology without
production evidence.

## Handoff after a pass

Return a short pass report containing the profile, revision, files inspected,
facts added or updated, unresolved questions, and the import summary. Then
query or export the current graph before starting the next profile. This makes
each iteration reviewable and lets a later pass correct a prior value by
upserting the same stable fact identity.
