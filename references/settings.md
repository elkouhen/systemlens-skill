# systemlens settings

`systemlens init` creates the project configuration consumed by `systemlens index`:

```yaml
include:
  - "**/*"
exclude:
  - ".git/**"
  - ".venv/**"
  - "node_modules/**"
  - ".systemlens/**"
min_severity: INFO
root_path: .
analysis:
  strategy: default
  codeql: true
  codeql_max_hops: 12
  codeql_max_paths: 10000
```

`include` and `exclude` define the source perimeter. Maven/Gradle test source
sets are excluded automatically. `min_severity` remains accepted for index
compatibility but does not change AST endpoint extraction.

`analysis.codeql` enables the default local interprocedural analysis. When the
CodeQL CLI and its Java pack are provisioned locally, SystemLens creates a
temporary source-only database and extends potential flows across Java method
calls. `codeql_max_hops` bounds call depth and `codeql_max_paths` bounds explored
call transitions; index progress reports when that transition bound is reached.
Set `codeql: false` only when an AST-only flow inventory is intended. The
temporary database is not persisted and indexing does not download CodeQL
packages.

For a one-off fast refresh without changing the project configuration, run
`systemlens index --no-codeql`. It cannot be combined with
`--codeql-database`.

After changing project configuration, run `systemlens index`. Explicit Kafka
manifests can be indexed with `systemlens index --manifest FILE`; they must be
Markdown or JSON files inside the indexed repository.
