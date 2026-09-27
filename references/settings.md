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
temporary source-only database for the whole repository and extends potential
flows across Java method calls. Direct input/output reachability is not bounded
by hop or transition counts. `codeql_max_hops` bounds fallback call depth and
`codeql_max_paths` bounds fallback transitions; index progress reports when that
transition bound is reached, even alongside direct results. Direct pairs and
fallback pairs are combined. Cross-module AST fallback requires qualified
receiver/contract types, compatible signatures and one concrete implementation
through source-declared inheritance; it never selects a same-name method alone.
The subprocess timeout also covers live progress reading.
Set `codeql: false` for an AST-only flow inventory. Unique source-declared
receiver calls can still add low-confidence `method_call` steps. The temporary
database is not persisted and indexing does not download CodeQL packages.

For a one-off fast refresh without changing the project configuration, run
`systemlens index --no-codeql`. It cannot be combined with
`--codeql-database`.

After changing project configuration, run `systemlens index`. Explicit Kafka
manifests can be indexed with `systemlens index --manifest FILE`; they must be
Markdown or JSON files inside the indexed repository.
