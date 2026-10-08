# Product requirements: direct application analysis skill

The skill helps an agent produce a reviewable architecture graph from source
code without running the SystemLens source indexer.

## Users

- A developer who needs a bounded architecture view before changing an
  application.
- An architect who needs explicit evidence and uncertainty in a shareable
  graph.
- An agent that must hand structured findings to SystemLens without changing
  the application or source-derived index.

## Workflow outcome

Given an application root and a focused question, the skill produces a
versioned JSON manifest with relative source evidence, imports it into a named
SystemLens namespace, and generates an HTML architecture export.

## Scope

The skill covers direct source inspection, fact generation, manifest review,
fact import, and HTML export. It does not run AST extraction, CodeQL, runtime
instrumentation, or Kubernetes discovery through SystemLens.

## Acceptance criteria

- The skill explicitly forbids `systemlens index` in its primary workflow.
- Every confirmed fact has relative evidence or an explicit reason for its
  absence.
- Ambiguous and unresolved facts remain qualified in the manifest.
- The importer can bootstrap an empty compatible schema after `systemlens init`.
- The workflow produces an HTML export containing the imported graph facts.
