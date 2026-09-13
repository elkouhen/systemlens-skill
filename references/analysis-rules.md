# SystemLens analysis rules

This document is the operational contract for asking an agent to analyse code
for SystemLens. It describes what can become a fact, what remains unresolved,
and when Strategy1 is allowed. It also defines the minimum evidence needed to
complete a complex repository view with microservices, channels and databases
that are not covered by the built-in Java/Spring extractors.

## Evidence and indexing perimeter

- Start at the repository root and run `systemlens doctor`.
- `systemlens index` analyses files selected by `.systemlens/config.yml`.
- Maven/Gradle test source sets are excluded automatically: `src/test`,
  `src/componentTest`, and module names ending in `Test`.
- Persisted source paths are always relative to the indexed root. Never emit or
  persist absolute workstation, WSL, home-directory, credential or token paths.
- Every endpoint carries role, system, logical value, dynamic flag, framework,
  relative source path, line range, snippet, module and optional qualified name.
- A parse failure is a visible extraction diagnostic; it is not permission to
  guess facts from the file name or surrounding code.
- In a polyglot repository, treat build files, deployment manifests and source
  directories as complementary evidence. A directory or image name alone does
  not prove a deployable service. Prefer an executable entry point, build
  target, container image, deployment object or explicit service registration.
- Generated clients, ORM models and schema migrations are evidence of a
  contract or store, not automatically evidence of a live call or ownership.
  Mark generated/runtime-only relationships separately when their target is not
  visible in source.
- Reindex after source/configuration changes. Use `--full` after broad changes
  or extractor upgrades. A changed Spring configuration, build descriptor,
  extraction profile or topic strategy invalidates dependent facts.

## REST and HTTP extraction

The deterministic extractor recognises Spring MVC/WebFlux mappings, Feign,
RestTemplate/RestClient, WebClient, Spring WebFlux router functions, Spring
Cloud Gateway routes, Spring Data REST repositories, Swagger/OpenAPI contracts,
and exposed actuator endpoints when configured. It records `serve` for an
exposed route and `call` for an outgoing call.

- Resolve class and method mappings together, including inherited prefixes.
- Preserve a dynamic path segment as dynamic; do not substitute a likely value.
- An HTTP route match is only a resource refinement. It does not identify the
  target service.
- An internal REST relation requires one exact normalized target alias from an
  explicit HTTP host, `lb://` service name, configured API domain or enabled
  Strategy1 convention, and that alias must identify exactly one indexed
  service.
- Prefix, suffix, substring and route-name similarity are not target evidence.
- A targetless or ambiguous call remains an endpoint and an indexing issue; it
  must not become a graph edge.
- A configured client may produce a service-level `API` relation when no
  target route is available, but only with explicit configuration evidence.
- External APIs are marked external only when the source explicitly names the
  external service through the supported convention.

## Topic extraction

The current deterministic Topic extractor recognises `@KafkaListener`, KafkaTemplate
`send`/`sendDefault`, `ProducerRecord`, Spring Cloud Stream `StreamBridge`,
Kafka Streams sources/sinks, and compatible low-level Kafka client usage.
It records `produce` and `consume` endpoints and infers a payload type only
from an explicit listener parameter or producer/client generic signature.

- A literal topic or a safely resolved Spring property is concrete.
- A dynamic expression is retained with `topic_dynamic=true`; it is never
  replaced by a topic inferred from a variable name, class name or nearby
  declaration.
- Two dynamic values do not form a Topic relation, even when their expressions
  look similar.
- Topic relations require a matching concrete topic between a producer and a
  consumer. The graph displays the service → topic → service path.
- Missing payload type is unknown, not an inferred DTO.
- Duplicate sites are deduplicated by their stable role/topic/path/line ID.
- Across iterative AI passes, duplicate logical facts are reconciled by stable
  `(namespace, fact_type, id)` identity. The newest imported value replaces the
  previous AI assertion; source-derived facts remain immutable.

## Generic Topics

When enriching beyond deterministic Topic extraction, use `kind=message_channel`
for a concrete topic, queue, exchange, subscription or stream. Supported
evidence includes producer/consumer annotations, client calls, binding
configuration, infrastructure manifests and schema/contract files. Capture the
provider (`kafka`, `rabbitmq`, `sqs`, `sns`, `redis`, `nats` or another explicit
technology), provider scope when known, and the routing identifier in
metadata. Distinguish an exchange from its queues and a topic from a consumer
group; do not collapse them into one node.

- A literal or safely resolved configuration value is concrete.
- A binding with a wildcard, computed routing key or environment-only value is
  unresolved unless the value is available in the indexed repository.
- A shared payload schema can be linked to a channel, but matching field names
  or serializer classes do not prove producer/consumer compatibility.
- Runtime-observed messaging destinations remain observations and must not be
  promoted to confirmed static Topics without source or configuration evidence.

## Data resources

For Data resources not covered by the deterministic MongoDB extractor, use
`kind=data_schema` and `technology` such as `postgresql`, `mysql`, `oracle`,
`sqlserver`, `redis`, `elasticsearch`, `s3` or another explicit provider.
Represent the narrowest proven Data resource: database/schema/table or view,
collection, index, keyspace, bucket or search index. Record migrations,
entities/ORM mappings, repository/DAO queries, client configuration and
deployment-managed stores as separate evidence when applicable.

- A migration proves that a Data resource is declared; a query or repository
  proves access; neither alone proves which service owns the data.
- Create `reads`/`writes` relations only when the access site and resource can
  be tied to the same concrete identifier. A database URL without a database
  or schema is a store hint, not a Data-level edge.
- Treat shared databases as shared dependencies, not service-to-service calls.
  Flag cross-service writes, undocumented ownership, destructive migrations and
  incompatible Data changes as review items when evidence supports them.
- Keep secrets, connection strings with credentials and raw SQL result data out
  of manifests and reports; redact values while preserving the property name
  and relative evidence path.

## Correlation and quality

Use stable identities based on normalized coordinates (service/module,
provider, namespace and resource identifier). Correlate only on explicit host,
route, binding, topic/queue, schema/table/collection or manifest references.
For every inferred or enriched edge record the evidence path, line when
available, confidence, and a short rationale. If more than one target remains
possible, emit `ambiguous`; if the identifier is dynamic or absent, emit
`unresolved`. Never turn a common class, table or topic suffix into a confirmed
relationship.

## Prompt composition for code extraction

Use a focused prompt per pass, not a generic request to "map the architecture".
State the requested scope, profile namespace, repository revision, and whether
the output is a `partial` or a deliberate `complete` snapshot. Begin with the
indexed inventory and ask the agent to inspect only the gaps that matter to the
question.

The prompt must require the agent to:

- collect the narrowest concrete identifier for each candidate fact before
  correlating it with another component;
- seek complementary evidence across code, configuration, contracts, and
  deployment manifests when available, while recording which evidence type
  supports which part of the claim;
- distinguish declared resources, configured bindings, implemented access
  sites, generated artifacts, tests, and deployment definitions;
- record an explicit stop condition for dynamic values, several matching
  targets, runtime-only wiring, uninspected repositories, or unavailable
  environments;
- produce only stable IDs, relative paths, safe configuration references,
  confidence, status, and a concise rationale; and
- validate the JSON and review the import result before claiming that the graph
  has been updated.

Ask for a relationship only when an explicit identifier joins both sides. A
second corroborating observation strengthens confidence but does not turn a
dynamic or many-to-one match into a resolved edge. Conversely, a lack of a
second observation is not a reason to hide a concrete access site: emit the
resource or endpoint with the appropriate qualified status.

Do not use source shape as a substitute for semantics. In particular, a
migration does not prove present ownership, a generated client does not prove a
live call, a DTO does not prove a message contract, a hostname does not prove a
service target, and a local development manifest does not prove production
deployment. Keep those observations separately reviewable.

## Potential source flows

`systemlens flows` materializes conservative same-method Java paths and, by
default when local CodeQL is available, bounded interprocedural Java call
chains. An HTTP or Topic entry point is followed by source-ordered HTTP calls,
Topic publications, and Data reads/writes in the same parsed method; CodeQL
adds `method_call` steps only between indexed source methods. Concrete Kafka
publications can continue into concrete downstream message-entry flows, with a
bounded number of asynchronous hops. Cyclic call chains and Kafka continuations
are retained with `status: cycle` rather than hidden. Every flow remains
`potential`: a branch may prevent an effect from running.

An assisted analysis may continue through direct calls or an injected
interface only when repository wiring resolves exactly one implementation.
Keep conditional branches as alternatives, summarize loops, make transaction
and async/reactive boundaries explicit, and terminate at reflection, dynamic
dispatch, multiple candidates, or runtime-only routing. Every transition needs
relative evidence for both the call site and selected target. Tests may support
a separate `test-evidenced` path but do not prove production execution.

These flows describe possible behaviour visible in source, not runtime traces,
frequency, latency, or guaranteed execution. Do not flatten their ordered or
branched steps into ordinary AI graph edges while the graph contract has no
versioned sequence semantics.

## Strategy1 (explicit opt-in)

Strategy1 is repository-convention support, not a general heuristic. Enable it
only when the repository follows these conventions:

```bash
systemlens index --strategy strategy1
```

The selected profile is persisted and must be consistent across a federated
workspace. Strategy1 adds the following facts:

### Kafka topic conventions

- `getTopics().getXxx()` produces the physical topic `XXX` normalized to
  `SCREAMING_SNAKE_CASE` (camel-case boundaries split; hyphens become
  underscores).
- A method whose name starts with `envoyerMessageKafka` and has at least two
  arguments is treated as a producer. This includes
  `envoyerMessageKafkaRequest` and `envoyerMessageKafkaReply`.
- Its first argument is resolved through the same topic resolver, including
  `kafkaProperties.getTopics().getXxx()`; its second argument is used for the
  explicit payload type when the Java signature makes that type available.
- A `@KafkaListener` annotation containing a key shaped like
  `${kafka.topics.xxx.<property>}` consumes the normalized topic `XXX`.
- Strategy1 replaces the standard Kafka endpoint at the same role, file and
  line when it covers that site, preventing duplicate facts.
- `retour_<request-topic>` is used only by the separate request/reply analysis
  to derive a high-confidence candidate when both sides are present; it is not
  a license to connect arbitrary topics.

### Configured REST clients

- A class recognized as a `Rest*Config*` configuration can expose a logical
  client domain through `getRest().get("domain")`.
- A helper named `create*ClientApi` can identify a domain from its first
  argument when it is a literal, a domain constant such as `DOMAIN_FOO`, or a
  supported `getKey()`/`getUriPath(..., DOMAIN_FOO.getKey())` expression.
- A domain constant is normalized to the corresponding kebab-case service
  name. URLs and path strings are not treated as service identities by this
  convention.
- A configured client factory must resolve to one domain; multiple domains in
  one bean remain ambiguous.
- An uppercase underscore constant in a recognized REST configuration can
  identify a same-named logical service in kebab-case.
- Explicit external-service properties may emit an external dependency, but
  only when the source names it with the supported marker.
- These configuration relations initially have a dynamic route (`ANY
  <dynamic>`). They are matched to a concrete served route only when HTTP
  method compatibility is known; otherwise the service-level API relation is
  retained with its configuration evidence.
- `getXxxServiceUrl()` identifies `xxx-service` only under Strategy1.

### Strategy1 OpenAPI publication

- A production `src/main/resources/openapi/xxx.rest` file is a publication
  declaration, not an OpenAPI document.
- It selects valid same-named `xxx.yaml`, `xxx.yml` or `xxx.json` contracts
  anywhere in the indexed repository.
- It also selects every valid YAML/JSON OpenAPI document below
  `model-xxx/src/main/resources/openapi/`.
- Candidates must parse as OpenAPI and contain a `paths` object. Invalid
  candidates are ignored, not fabricated into APIs.
- Published operations remain attributed to the project that owns the `.rest`
  declaration, while the contract path remains its own evidence.
- Independently of Strategy1, each build project inventories valid OpenAPI
  YAML/JSON files under its own `src/main/resources/openapi/`, regardless of
  the filename.

Do not enable Strategy1 merely to make an expected edge appear. First inspect
`systemlens analyze indexing-issues --json`, verify that the repository follows
the convention, and retain unresolved output when it does not.

## Modules, Data, and manifests

- Maven and Gradle modules are discovered with collision-safe identities;
  artifact display names are not sufficient when duplicates exist.
- MongoDB Data is extracted from `@Document`, repository entity types
  and unambiguous `MongoTemplate` type arguments. Ambiguous class/collection
  matches remain unresolved.
- Mongo persistence metadata includes root and conservatively resolved nested
  project classes. External or ambiguous field types remain text.
- Markdown topic tables and compatible JSON flow manifests are explicit
  `source=manifest` evidence. They supplement AST extraction and never justify
  guessing a missing producer or consumer.

## AI-generated convention analysis

When conventions remain too complex, produce a
`systemlens-ai-graph-v1` manifest using [ai-graph.md](ai-graph.md). Keep its
claims separate from the SQLite source inventory:

- `confirmed` and `proposed` claims can be rendered;
- `ambiguous` and `unresolved` claims must include a reason and remain visible
  in the Quality panel rather than becoming dependencies;
- evidence paths remain relative and claims retain confidence;
- source analysis must not include secrets, raw payloads or absolute paths.

This is the safe extension point for an AI analysis of repository-specific
conventions. It complements deterministic extraction; it does not silently
override it.
