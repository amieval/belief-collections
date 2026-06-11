# Quickstart

A guided tour of Composable Beliefs against the graph the repo actually ships: `beliefs/beliefs.json`, which is CB's own design expressed as beliefs - the framework describing itself in its own format. By the end you will have queried the graph, read a contract, traced a reasoning chain, and seen how a belief is added and verified.

Belief ids are namespaced (`cb:` here, since this repo ships a single collection), so commands take the full id, e.g. `cb:c053` - though a bare local id (`c053`) also resolves when exactly one belief matches.

> **New here?** The main graph is self-referential - CB reasoning about CB - which is a lot to meet first. For a gentle on-ramp on a familiar domain, walk the `lib:` lending-library collection (`library/README.md`, right next to this tour) before this one, then come back.

> **Where to run these commands:** this tour lives with the teaching material in belief-collections, but the `mix` tasks live in the framework. Run the commands below from the framework checkout (`../composable-beliefs/`); the `cb:` graph is its default, so no `--beliefs` flag is needed.

## 1. Build

```bash
mix deps.get
mix compile
```

The belief shell (`mix bs`) is available immediately after compilation.

## 2. Explore the graph

```bash
mix bs stats
```

The output is the live shape of the graph - primitives, compounds, inferences, and directives, nearly all active. This is not a toy domain: it is the schema, mechanism, and positioning of CB itself, expressed in CB's own format (the framework's CLAUDE.md is itself compiled from these beliefs).

List them, filtering by structural type, status, domain, tag, and more:

```bash
mix bs list                # everything
mix bs list compound       # the 5 composed beliefs
mix bs list contracts      # the contract-grade directives
```

## 3. Read a primitive

```bash
mix bs show cb:a302
```

`cb:a302` is a **primitive** - an atomic claim grounded in sources:

- `claim`: "Assertions are immutable once created - existing fields are never edited; the only mutations are status transitions and append-only evidence entries."
- `evidence`: two `{date, artifact, detail}` entries (a `document:` source and a later `session:` reclassification).
- No `confidence` field. `Support: artifacts=2 evidence=2 deps=0` - structural counts, not a synthesized score.

(The claim reads "Assertions" because beliefs are immutable: it was authored before the vocabulary settled on "belief," and editing it in place is exactly what the model forbids.)

## 4. Read a contract: state machine

```bash
mix bs show cb:c053
mix bs tree cb:c053
```

`cb:c053` (`dag-status-lifecycle`) is a **contract** - a directive with `contract: true`, `rules`, and `invariants`. Its `kind` is `state-machine`: the rules are `{from, to, requires}` transition rows (`active -> superseded | retracted | retired`) and the invariants are the constraints that never bend ("superseded_by is non-null iff status is superseded").

The tree shows the contract resting on one dep:

```
cb:c053 [contract] DAG node status follows a directed transition...
├── cb:a302 [primitive] Assertions are immutable once created...
└── cb:a471 [primitive] The boundary between inference and directive is direction of fit made operational...
```

The primitives state the principles - immutability, and the direction-of-fit test that makes `retired` a directive-only exit; the contract (`cb:c053`) states the enforceable HOW. `CB.Belief.Contract.StateMachine` interprets the rules - valid transitions and per-edge requirements.

## 5. Read a contract: enum

```bash
mix bs show cb:c039
```

`cb:c039` is a `kind: enum-registry` contract: the closed enum of `belief.kind` values. This is the same enum `mix cb.verify.schema` checks the `CB.Belief` struct against - the graph specifies the schema the code must satisfy. `CB.Belief.Contract.Enum` interprets it; its siblings govern artifact schemes (`cb:c043`) and domains (`cb:c041`).

## 6. Read an inference: a conclusion that exceeds its deps

```bash
mix bs show cb:a138
mix bs tree cb:a138
```

`cb:a138` is an **inference** - a belief whose meaning is carried by its `claim` plus its `deps`, and whose claim is licensed to state more than the deps jointly state:

- `claim`: "The contract layer exists because: (1) implicit contracts were scattered ... (a133), (2) contracts must be domain-bound ... (a134), (3) the same representation must govern behavior across substrates ... (a135)."
- `deps`: `cb:a133, cb:a134, cb:a135`

Notice the word doing the work: *because*. The deps record three observed problems; the claim explains why a whole layer of the system exists - a causal explanation no dep states, even jointly. That overreach is exactly what the inference type licenses, and what it pays for: the explanation can be wrong while all three observations stand, and the graph can supersede it without touching them. (A **compound** is the contained sibling: it states exactly what its deps jointly state - "two scorers agree" - and can fail only if a dep fails. The eval collections in this repo are full of them.) To understand *why* an inference is believed, you read its deps. The DAG makes that traceable.

## 7. Supersession and staleness

```bash
mix bs show cb:a374
mix bs stale
```

`cb:a374` has `status: superseded`. Its evidence records why: the original belief proposed materialization as a fifth status value, contradicting the closed status enum (today `cb:c053`); the successor models materialization as an orthogonal field instead. The wrong version is kept, not deleted - provenance survives. Beliefs are never edited in place; change is a status transition, which is what makes drift detectable. `mix bs stale` surfaces beliefs whose deps were superseded or retracted (none, currently).

## 8. Add a belief via preflight

Beliefs are written through a conflict-preflight gate, never edited in place. Write a proposed belief to a temp file:

```json
{
  "id": "cb:a900",
  "type": "primitive",
  "kind": "policy",
  "domain": "system",
  "name": "example-belief",
  "claim": "A quickstart example belief, added to demonstrate the preflight gate",
  "status": "active",
  "created": "2026-06-07"
}
```

Preflight is non-mutating - it only inspects:

```bash
mix cb.preflight --file /tmp/cb-a900.json
```

It reports contract-level conflicts (blocked pending adjudication), schema / non-contract conflicts (need a decision), supportive dep candidates, and neutral matches. If preflight is clean, `mix cb.import --file /tmp/cb-a900.json` writes the belief into `beliefs/beliefs.json`.

## 9. Verify schema integrity

```bash
mix cb.verify.schema
```

Checks the `CB.Belief` struct against the schema contracts in the graph (`cb:c056`, `cb:c039`, `cb:c043`, `cb:c041`, the kind-type table `cb:c057`, subject containment `cb:c058`, and directive grounding `cb:c059`). Drift between the code and the declared schema surfaces here.

## 10. Audit for conflicts

```bash
mix cb.audit.conflicts
```

Surfaces overlapping directives per `cb:c055` - two active directives that share subject scope and could produce contradictory actions.

## 11. Materialize a directive

A directive can be turned into concrete work items via the `/materialize` skill:

```
/materialize cb:a384
```

`cb:a384` ("materialized implications require periodic drift audits", in the wording of its day) is an unmaterialized directive. `/materialize` reads it and its deps, proposes `{object, action}` items for your confirmation, writes them through the configured `CB.Materializer.Sink`, and records the result on the belief's `materialized` field - so "why does this task exist?" stays a graph traversal.

## Next steps

- `../composable-beliefs/docs/belief-graph.md` - design reference: the contract index, query patterns, and pointers into the graph
- `../composable-beliefs/docs/operations.md` - extraction workflow; the durable principles now live in the graph (`mix bs list domain:design`)
- `../composable-beliefs/skills/assert/SKILL.md` - the `/assert` protocol for adding beliefs from artifacts
- `../composable-beliefs/skills/assert-session/SKILL.md` - the `/assert-session` protocol for persisting session knowledge
