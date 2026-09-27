# Architecture enrichment prompt

Copy this prompt into an agent session started from the root of the Java/Spring
repository you want to inspect. Replace the bracketed question and scope.

```text
Use SystemLens to enrich the architecture model for this repository.

Question:
[State one architecture question, for example: Which services own the
checkout API, its synchronous callers, and the data resources it writes?]

Scope:
[Name the services, modules, endpoints, Topics, Data resources, or deployment
files in scope. Name explicit exclusions.] 

Work from the repository root. Establish the deterministic baseline first:

1. Run `systemlens init` only if this repository is not initialized.
2. Run `systemlens doctor` and report blocking or unresolved checks.
3. Run `systemlens index` when the persisted inventory is missing or stale.
4. Inspect the SystemLens inventory before reading source files.

Choose only the focused passes that answer the question. Use `boundaries` for
service and module ownership, `http` for synchronous APIs, `messaging` for
Topics and consumers, `data` for stores and reads or writes, and `deployment`
for runtime resources. Run `boundaries` before dependent passes.

Write one versioned manifest per selected pass at the repository root:
`architecture.ai-<profile>.pass-001.json`.

Every manifest must:

- use `systemlens-ai-graph-v1`;
- use the namespace `ai-<profile>` and `mode: "partial"`;
- include the repository revision, producing agent or model, stable node and
  edge IDs, evidence paths, confidence, provenance, and ambiguity;
- use paths relative to the repository root;
- keep deterministic SystemLens facts separate from complementary facts;
- omit credentials, tokens, connection strings, and secret values.

Do not confirm a relationship from a name, directory, shared type, hostname,
or documentation alone. Mark ambiguous relationships unresolved and explain
what evidence is missing. Do not modify source code, configuration, or
deployment files.

Before importing, validate every manifest and report:

- the question, scope, exclusions, and repository revision;
- the files and symbols inspected;
- facts inserted, updated, or left unresolved;
- evidence, confidence, provenance, and namespace ownership;
- assumptions, alternatives, stop conditions, and remaining blind spots.

After review, import each manifest explicitly:

`systemlens import-facts architecture.ai-<profile>.pass-001.json \
  --namespace ai-<profile>`

Use `--complete` only when the selected pass inspected its full namespace
scope. Re-read the merged graph or export it after every import. End with the
commands run, the import summaries, and the unresolved questions.
```

The pass-specific supplements and JSON contract are in
[`references/pass-profiles.md`](../references/pass-profiles.md) and
[`references/ai-graph.md`](../references/ai-graph.md).
