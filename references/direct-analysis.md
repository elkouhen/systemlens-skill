# Direct analysis procedure

## Scope the question

Write down the application root, the architecture question, the services or
resources in scope, and the directories to exclude. Exclude generated output,
build directories, vendored dependencies, and secrets unless the question
requires them.

## Read source evidence

Inspect the smallest set of files that can answer the question. For a service
graph, start with application entry points, HTTP controllers and clients,
Kafka listeners and publishers, persistence adapters, configuration, and local
OpenAPI or AsyncAPI contracts.

Record the path and line for each fact while reading. Keep a direct source
observation separate from a conclusion that requires correlation across files.

## Correlate conservatively

Join facts only when service names, routes, topics, or resource names match
explicitly. Preserve multiple candidates as `ambiguous` instead of choosing
one. Preserve dynamic expressions as expressions instead of substituting a
guessed value.

## Build and review the manifest

Use stable IDs based on the role and identity of each node or edge. Check that
every edge references existing node IDs, every evidence path is relative, and
every status and confidence value is valid. Review the JSON before importing it.

## Import and export

Run `systemlens init` once in the application root. Then import the manifest
into a dedicated namespace and generate the HTML export. Do not run
`systemlens index` as part of this procedure. If the question later requires a
deterministic source inventory, treat that as a separate workflow and namespace.
