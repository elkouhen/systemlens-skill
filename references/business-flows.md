# Business-flow discovery

Use this review workflow to explain a small number of important business
outcomes through the code. It is a source-based investigation, not a runtime
trace or a claim that every step executes in production.

## Preconditions and scope

Run `systemlens doctor` and `systemlens index` from the analyzed repository
before starting. Select two or three flows with the requester. A flow can be
named from a documented capability, an API operation, a message trigger, a
scheduled job, or a batch entry point. If the business meaning is not explicit
in the repository or request, use a neutral technical name and record that the
business outcome is unknown.

For each selected flow, record:

- the requested outcome and why it matters;
- its proven entry point or trigger;
- the repository revision and analysis scope; and
- exclusions, such as other services, runtime configuration, or unsupported
  source languages.

## Procedure

1. List the indexed baseline with `systemlens flows --json`. Select relevant
   potential flows by trigger, module, and externally visible effect.
2. Inspect each selected flow with `systemlens flows show <id> --json`.
   Treat its ordered same-method steps as the deterministic baseline.
3. When a relevant step publishes a Topic, use
   `systemlens topics trace <topic> --json` to explore bounded potential
   service-level continuations. A returned path is still potential and may be
   truncated, cyclic, or conditional.
4. Inspect only the source and configuration needed to continue a path:
   direct method calls, or injected interfaces with exactly one repository-
   evidenced implementation. Record the evidence at both the call site and
   selected target.
5. Stop and mark the path unresolved at reflection, dynamic dispatch, multiple
   bean candidates, computed routes or destinations, environment-only wiring,
   or a boundary outside the inspected scope.
6. Compare the resulting report with the API, messaging, and Data topology
   facts. Correct a topology fact only through its owning pass; do not convert
   ordered steps into graph edges.

## Report format

Create one reviewable Markdown report per pass, for example
`business-flows.pass-001.md`. This is not an importable graph manifest.

```markdown
# Business flows — pass 001

Repository revision: `<revision>`
Scope: `<selected services/modules>`

## Place order

Business outcome: `<user-provided or documented outcome>`
Status: `potential`
Confidence: `medium`
Entry point: `POST /orders` — `src/.../OrderController.java:42`

1. `API entry` — `src/.../OrderController.java:42`
2. `transaction begins` — `src/.../OrderService.java:58`
3. `writes orders` — `src/.../OrderRepository.java:31`
4. `publishes orders.created` — `src/.../OrderService.java:76`

Alternatives and boundaries:
- `<conditional, retry, async, timeout, or compensation evidence>`

Unresolved:
- `<dynamic target or missing evidence and reason>`
```

Every step must state its kind, order, relative evidence path, and line or
symbol. Mark explicit transaction, asynchronous/reactive, retry, timeout,
circuit-breaker, dead-letter, and compensation boundaries. Preserve branch
alternatives and summarize loops; never invent a single linear execution path.
Use `systemlens`, `source-assisted`, or `test-evidenced` provenance for every
conclusion. Tests can illustrate behaviour but do not prove production runtime
execution.

## Exit criteria

The report covers the requested outcomes, separates deterministic and
source-assisted steps, and makes every stop condition visible. It uses
`potential` rather than `confirmed` for source flows, includes uncertainty and
scope limits, and contains no credentials, payload values, or absolute paths.
