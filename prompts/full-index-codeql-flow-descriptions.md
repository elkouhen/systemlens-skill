# Full CodeQL indexing and flow descriptions

Use this prompt with an agent operating from the root of an indexed Java
repository.

If `.systemlens/analysis-scope.json` exists, read and validate it first. Apply
the scope to the flow IDs before writing descriptions, and report selected,
excluded, and unresolved selectors. Do not generate descriptions outside the
declared scope.

```text
Run a complete SystemLens index for the current repository and use the existing
CodeQL Java database for method-call analysis.

Inputs:
- Repository root: <REPOSITORY_ROOT>
- CodeQL database: <CODEQL_DATABASE_PATH>

Procedure:

1. Confirm that <REPOSITORY_ROOT> is the current working directory and that
   <CODEQL_DATABASE_PATH> exists.
2. Confirm that the CodeQL database was created from the same repository
   revision and source-root layout. If this cannot be confirmed, stop and
   report the risk.
3. Run the full index with CodeQL:

   uv run systemlens index --full \
     --call-graph-engine codeql \
     --codeql-database <CODEQL_DATABASE_PATH> \
     --codeql-progress

4. Treat a failed index as a blocking error. Do not silently fall back to
   AST-only flows, and do not use --no-codeql.
5. Export the persisted flows without inspecting or reproducing CodeQL
   evidence outside the SystemLens flow output:

   uv run systemlens flows list --json

   Resolve `.systemlens/analysis-scope.json` against the complete flow list.
   Record the selected, excluded, and unresolved selectors. Inspect each
   selected flow with:

   uv run systemlens flows show <FLOW_ID> --json

6. For every selected flow, write exactly one concise sentence in the
   requested language. Include, when supported by the flow:
   - the trigger, such as an HTTP route, Kafka topic, or scheduled task;
   - the originating service and the relevant downstream service or effect;
   - the main operation, such as an HTTP call, Kafka publication, or data
     access.
7. Ground every sentence in the ordered flow steps and their source locations.
   Use method-call steps already present in the flow output when explaining the
   operation. Do not infer runtime execution, ordering across branches,
   delivery guarantees, database writes, or a target service that the flow does
   not establish.
8. Preserve the uncertainty of potential or partially reconciled flows. Use
   terms such as "potential", "may", or "partially reconciled" when required.
9. Write `.systemlens/flow-descriptions.json` with this format:

   {
     "format": "systemlens-flow-descriptions-v1",
     "repository": "<repository name>",
     "generated_by": "systemlens flows list",
     "flows": [
       {"id": "<FLOW_ID>", "description": "<one sentence>"}
     ]
   }

10. Validate that the file contains one entry for every selected flow ID, no
    duplicate IDs, no empty descriptions, and no absolute machine-specific
    source paths. Do not create descriptions for excluded flows.
    The file is an AI enrichment layer. It must not replace or edit the
    persisted SystemLens facts.
11. Run the HTML export and confirm that the flow cards contain the imported
    descriptions:

    uv run systemlens export microservices --html <OUTPUT_HTML_PATH>

12. Report the index command, the flow count, the enrichment path, the export
    path, and any unresolved evidence or validation warning. Do not claim that
    CodeQL was used unless the index completed with the supplied database.
```

The prompt creates presentation text only. It does not import ordered flow
facts, replace source evidence, or turn a potential flow into a runtime claim.
