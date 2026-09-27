# Call-graph description prompt

Use this prompt from the root of an indexed Java repository when SystemLens has
persisted potential call graphs and you want a concise explanation for each one.

```text
Use SystemLens to explain the persisted call graphs for this repository.

1. Confirm that the repository is initialized and that its index is current.
2. Run `systemlens flows list --json` and record every persisted flow ID.
3. Run `systemlens flows show <FLOW_ID> --json` for each flow.
4. Write exactly one concise sentence per flow in the requested language.
5. Ground each sentence in the ordered flow steps and source locations.

Each sentence should include the trigger, the originating component, and the
main downstream operation when the flow establishes them. Preserve terms such
as potential or partially reconciled when the flow is uncertain. Do not infer
runtime execution, ordering across branches, delivery guarantees, database
writes, or a target that the flow does not establish.

Write the presentation enrichment to `.systemlens/flow-descriptions.json`:

```json
{
  "format": "systemlens-flow-descriptions-v1",
  "repository": "<repository-name>",
  "generated_by": "systemlens flows list",
  "flows": [
    {"id": "<FLOW_ID>", "description": "<one sentence>"}
  ]
}
```

Validate one non-empty description for every flow ID, with no duplicate IDs or
absolute workstation paths. Do not modify persisted SystemLens facts. Export
the graph after writing the file and report the flow count, output path, and
unresolved evidence.
```

For the complete indexing and CodeQL procedure, see
[`full-index-codeql-flow-descriptions.md`](full-index-codeql-flow-descriptions.md).
