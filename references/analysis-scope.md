# Analysis scope contract

Use `systemlens-analysis-scope-v1` when an analysis must cover a declared
subset of the indexed model. Store the file at
`.systemlens/analysis-scope.json` in the analyzed repository. The file is an
instruction for the skill; it is not a SystemLens fact manifest and must not
be imported with `systemlens import-facts`.

## Contract

```json
{
  "format": "systemlens-analysis-scope-v1",
  "purpose": "Explain the reservation flows",
  "deliverable": "flow-descriptions",
  "mode": "include",
  "include": {
    "services": ["inventory-service"],
    "flow_ids": [],
    "protocols": ["http", "kafka"],
    "topics": ["supermarket.stock.depleted"],
    "data_resources": []
  },
  "exclude": {
    "services": [],
    "flow_ids": []
  }
}
```

`purpose` and `deliverable` explain why the scope exists. Supported
`deliverable` values are `flow-descriptions`, `complexity-audit`,
`business-flow-review`, and `topology-enrichment`. `mode` is currently
`include`; it records that the listed selectors define the requested subset.

Each selector is optional. An empty `include` object means the full declared
scope. When several selector fields are present, a candidate must match every
non-empty field; values within one field are alternatives. Exclusions are
applied after inclusion. Service, Topic, and Data selectors must match exact
persisted identifiers or values after they have been resolved from SystemLens
JSON. `flow_ids` always require exact flow IDs.

The selected flow set is the intersection of the requested selectors. A flow
matches a service when its persisted call graph or ordered steps reference the
service. A Topic or Data selector matches when the flow contains that persisted
resource. A protocol selector matches the persisted flow kind. If a selector
cannot be resolved, the agent reports it as unresolved and selects no candidate
for that selector; it must not guess from a name fragment.

## Agent procedure

1. Read and validate `.systemlens/analysis-scope.json` before collecting
   evidence. If it is absent, use the full declared scope.
2. Run `systemlens flows list --json` and the relevant graph inventory commands
   before resolving selectors.
3. Resolve selectors against persisted IDs and record selected and excluded
   counts in the deliverable.
4. Apply the same scope to every subsequent `flows show`, audit calculation,
   report section, or enrichment candidate.
5. Report unresolved selectors and stop conditions. Do not widen the scope
   silently.

The scope limits investigation only. It does not change the indexed snapshot,
remove facts, lower confidence, or turn a selected potential flow into a
runtime trace. Reports and descriptions must state the scope file path and
retain the evidence rules of the selected deliverable.
