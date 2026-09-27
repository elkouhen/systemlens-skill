# AI flow descriptions

SystemLens can display a concise AI-generated description for each persisted
potential code flow. This description is a presentation enrichment. It does
not modify the indexed flow, its CodeQL evidence, its confidence, or its
potential status.

## Workflow

Run a complete index first, then list the persisted flows as JSON. For each
flow, inspect its ordered steps and source locations with
`systemlens flows show <FLOW_ID> --json`. Use the flow steps and available
method-call evidence to write one concise sentence in the language requested
by the user. The sentence should explain the trigger, the originating service,
the main operation, and the downstream effect when those facts are present.

Do not infer runtime execution, delivery guarantees, branch ordering, database
writes, or a target service that the indexed flow does not establish. Keep
potential and partially reconciled wording explicit. A description may explain
the sequence represented by the persisted steps, but it must not turn a
potential source flow into a confirmed runtime trace.

Store the result in `.systemlens/flow-descriptions.json`:

```json
{
  "format": "systemlens-flow-descriptions-v1",
  "repository": "<repository name>",
  "generated_by": "systemlens flows list",
  "flows": [
    {
      "id": "<FLOW_ID>",
      "description": "<one concise, evidence-based sentence>"
    }
  ]
}
```

The `id` must match a persisted flow ID. Include one entry for every flow,
avoid duplicate IDs and leave no description empty. Do not put absolute source
paths, credentials, tokens, or payload secrets in the file.

The HTML export loads this file automatically when it is present. Descriptions
are matched by flow ID and shown in the flow cards. Missing or invalid
descriptions are ignored without changing the indexed architecture snapshot.

For the complete CodeQL indexing and enrichment procedure, use
[`../prompts/full-index-codeql-flow-descriptions.md`](../prompts/full-index-codeql-flow-descriptions.md).
