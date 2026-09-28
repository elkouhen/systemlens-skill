# Architecture complexity audit prompt

Use this prompt from the root of an indexed repository when you want to
identify structural complexity in the persisted dependency graph and potential
call graphs.

```text
Audit the architecture complexity of this repository using SystemLens.

The goal is to reveal the parts of the architecture that are hardest to
understand, change, test, or operate. Produce a reviewable Markdown report;
do not modify source code, the persisted SystemLens facts, or the enrichment
layer.

1. Confirm that the current directory is the indexed repository root. Run:

   systemlens doctor --json
   systemlens analyze coverage --json
   systemlens analyze indexing-audit --json

   If the index is missing, stale, partial, or has important unresolved
   evidence, state that limitation before drawing conclusions. Do not silently
   reindex or replace a partial snapshot.

2. Collect the persisted dependency views:

   systemlens projects graph --json
   systemlens export microservices --json
   systemlens analyze audit

   Use the project graph to inspect build dependencies and the microservice
   export to inspect deployable service, API, Topic, and Data relationships.
   Keep these graph scopes separate in the report.

3. Collect the persisted call-graph inventory:

   systemlens flows list --json

   For every flow ID, run:

   systemlens flows show <FLOW_ID> --json

   Also run, when supported by the indexed repository:

   systemlens analyze flows-diagnostic --json
   systemlens analyze microservices orphan-integrations --json

   Inspect the ordered steps, root status, cycle status, reconciliation
   status, confidence, source evidence, external effects, and cross-service
   transitions. Treat every flow as a potential source-backed path, never as
   proof of runtime execution.

4. Compute or identify complexity signals from the persisted outputs. Use
   counts and relative comparisons only when they can be reproduced from the
   collected JSON. Look for:

   - dependency hubs with unusually high fan-in or fan-out;
   - services or projects that bridge otherwise separate groups;
   - dependency cycles, strongly connected groups, and long dependency paths;
   - shared libraries or projects that create broad change propagation;
   - services with many incoming callers, many outgoing calls, or both;
   - isolated services, orphan integrations, and edges without resolvable
     endpoints;
   - call graphs with many steps, branches, joins, cycles, repeated nodes, or
     cross-service hops;
   - flows with partial reconciliation, possible or low-confidence edges,
     ambiguous dispatch, unresolved targets, or missing external effects;
   - several flows converging on the same method, service, Topic, or Data
     resource;
   - duplicated or strongly overlapping flows that make behavior difficult to
     distinguish;
   - boundaries where synchronous HTTP, asynchronous messaging, persistence,
     retries, transactions, or external APIs meet.

5. Rank findings by architectural consequence, not by graph size alone. Use:

   - BLOCKER: a concrete cycle, unresolved critical boundary, or evidence gap
     that prevents a reliable change-impact or execution-path conclusion;
   - HIGH: a likely centralization, coupling, or flow-complexity hotspot that
     can propagate changes or make a critical path difficult to reason about;
   - MEDIUM: a meaningful source-backed complexity signal with bounded impact;
   - LOW: an observation worth monitoring but not an immediate risk.

   Do not assign severity from a guessed threshold. Explain the evidence and
   the reason for the ranking. If no finding meets a level, omit that level.

6. Write `architecture-complexity-audit.md` with this structure:

   # Architecture complexity audit

   ## Executive summary
   State the main complexity hotspots, the most affected graph scope, and the
   most important uncertainty in a short paragraph.

   ## Scope and confidence
   Record the repository name, indexed snapshot status, graph sources, flow
   count, and unresolved or partial evidence. Use relative source paths only.

   ## Findings
   For each finding, include:
   - severity and concise title;
   - graph scope: project dependency, deployable topology, or call graph;
   - observed signal and reproducible count or path when available;
   - affected nodes, flows, methods, APIs, Topics, or Data resources;
   - relative source evidence and the SystemLens command output that supports
     the finding;
   - architectural consequence for change impact, reliability, operability,
     or comprehension;
   - a proportionate investigation or remediation direction;
   - confidence and unresolved questions.

   ## Complexity map
   Summarize the highest fan-in, highest fan-out, longest path, cycles,
   convergence points, and flow hotspots. Distinguish measured values from
   qualitative observations.

   ## Recommended next investigations
   List at most five actions, ordered by expected value. Each action must name
   the graph or source evidence it would clarify.

   ## Limits
   State what static source analysis cannot establish, including runtime
   traffic, actual latency, deployment scaling, feature-flag routing, dynamic
   dispatch, and unindexed infrastructure behavior.

7. Validate the report:

   - every material finding has a relative evidence path or a cited SystemLens
     JSON result;
   - no finding invents a dependency, runtime call, ordering, or business
     impact that the evidence does not establish;
   - dependency graph, deployable topology, and call-graph observations are
     not conflated;
   - partial, potential, ambiguous, and low-confidence evidence remains
     explicitly qualified;
   - no credentials, absolute workstation paths, raw secrets, or full source
     dumps are present;
   - the report is a Markdown review artifact only and no graph facts are
     imported.

   Report the output path, graph scopes inspected, flow count, top findings,
   and unresolved evidence.
```

This prompt produces an analysis report only. It does not create an
`systemlens-ai-graph-v1` manifest, import graph facts, or change the indexed
architecture snapshot.
