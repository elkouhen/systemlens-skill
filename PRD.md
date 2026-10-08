# Product requirements: direct application analysis skill

The skill helps an agent produce a reviewable architecture graph from source
code with a deterministic CodeQL-backed SystemLens source index, then add
evidence-backed direct-analysis facts.

## Users

- A developer who needs a bounded architecture view before changing an
  application.
- An architect who needs explicit evidence and uncertainty in a shareable
  graph.
- An agent that must hand structured findings to SystemLens while preserving a
  deterministic source-derived index.

## Workflow outcome

Given an application root and a focused question, the skill produces a
versioned JSON manifest with relative source evidence, imports it into a named
SystemLens namespace, and generates an HTML architecture export.

## Scope

The skill covers CodeQL setup, source indexing, direct source inspection, fact
generation, manifest review, fact import, and HTML export. It does not cover
runtime instrumentation or Kubernetes discovery through SystemLens.

## Acceptance criteria

- The skill requires a verified CodeQL CLI and a CodeQL-backed `systemlens
  index` in its primary workflow.
- Every confirmed fact has relative evidence or an explicit reason for its
  absence.
- Ambiguous and unresolved facts remain qualified in the manifest.
- The importer can bootstrap an empty compatible schema after `systemlens init`.
- The workflow produces an HTML export containing the imported graph facts.
