# SystemLens analysis rules

This document is the operational contract for asking an agent to analyse code
for SystemLens. It describes what can become a fact, what remains unresolved,
and when Strategy1 is allowed.

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

## Kafka extraction

The deterministic extractor recognises `@KafkaListener`, KafkaTemplate
`send`/`sendDefault`, `ProducerRecord`, Spring Cloud Stream `StreamBridge`,
Kafka Streams sources/sinks, and compatible low-level Kafka client usage.
It records `produce` and `consume` endpoints and infers a payload type only
from an explicit listener parameter or producer/client generic signature.

- A literal topic or a safely resolved Spring property is concrete.
- A dynamic expression is retained with `topic_dynamic=true`; it is never
  replaced by a topic inferred from a variable name, class name or nearby
  declaration.
- Two dynamic values do not form a Kafka relation, even when their expressions
  look similar.
- Kafka relations require a matching concrete topic between a producer and a
  consumer. The graph displays the service → topic → service path.
- Missing payload type is unknown, not an inferred DTO.
- Duplicate sites are deduplicated by their stable role/topic/path/line ID.

## Strategy1 (explicit opt-in)

Strategy1 is repository-convention support, not a general heuristic. Enable it
only when the repository follows these conventions:

```bash
systemlens index --topic-strategy strategy1
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
- Published operations remain attributed to the module that owns the `.rest`
  declaration, while the contract path remains its own evidence.
- Independently of Strategy1, each build module inventories valid OpenAPI
  YAML/JSON files under its own `src/main/resources/openapi/`, regardless of
  the filename.

Do not enable Strategy1 merely to make an expected edge appear. First inspect
`systemlens analyze indexing-issues --json`, verify that the repository follows
the convention, and retain unresolved output when it does not.

## Modules, MongoDB and manifests

- Maven and Gradle modules are discovered with collision-safe identities;
  artifact display names are not sufficient when duplicates exist.
- MongoDB collections are extracted from `@Document`, repository entity types
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
