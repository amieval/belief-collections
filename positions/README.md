# Positions

A **position** is a stance with an extraction ledger: a markdown artifact that
states a set of claims and tracks, per claim, whether and where each one has
been asserted into a belief collection. It is the middle layer of a
three-layer pipeline:

```
source (what was said)   ->   position (what we claim)   ->   belief graph (what we assert)
   sources/*.md                 positions/*.md                 <collection>/beliefs.json
        ^                            ^      |
        |   Origin: cites            |      |  belief artifact: document:positions/<file>.md
        +----------------------------+      +--> claim blocks gain **Belief:** [[ids]]
```

Forward is the authoring flow: archive the conversation or document under
`sources/`, state the stance as a position, extract the claims as beliefs
through the write flow (`/assert` -> preflight -> import). Backward is the
audit trail: each belief's `artifact` cites the position, the position's
`Origin` cites the source, and the claim blocks accumulate `[[belief-id]]`
refs as extraction lands.

Reach for a position when a conversation produces a multi-claim stance worth
standing behind (the first one here grew out of a chat about eval
architecture). Single-claim extraction from a repo document stays two-layer -
source to belief, no position needed.

## Format

Parsed by plan-app (`PlanApp.Sources.Positions`); the header block sits in the
first 40 lines.

```markdown
# Position: <title>

**Type:** position
**Status:** active            <- active | superseded | abandoned
**Authored:** YYYY-MM-DD
**Origin:** sources/<file>.md (working session, YYYY-MM-DD)
**Companion plans:** <basename>, ...   <- optional

<free prose: the stance in summary, pointers, target collection>

### Claim: <the claim, one line, written as the belief's claim will read>
**DAG status:** pending       <- the author's declared state (see below)
**Belief:** [[paradigm:a361]] <- added when extraction lands; bare or namespaced ids
**Intended:** <type, kind, dep sketch - free-form authoring notes>
```

One `### Claim:` block per candidate belief. `**Intended:**` is optional and
unparsed - authoring notes that survive as provenance.

## Status is declared; truth is joined live

The `**DAG status:**` line is the author's claim about extraction state
(`pending` -> `asserted`), and it is deliberately *not* trusted as truth: a
hand-maintained status line is a cache, and caches rot (cb:a386). The
`[[belief-id]]` refs are the durable join key. Plan-app's positions surface
resolves every ref against the live graph at render time - a claim renders
with its beliefs' *current* status (active, superseded with successor,
retracted, dangling), and a `MISMATCH` flag when the declared line disagrees
with the graph. Superseding a belief changes the rendered position with zero
edits to this file.

Positions are read across a federated source registry
(`PlanApp.Paths.positions_sources/0`); this directory is the default source.

## The first position

`2026-06-10-eval-architecture-is-agentic-epistemics.md` - eight claims, all
extracted into `paradigm:` (a358-a365), with its source archived at
`../sources/2026-06-10-eval-architecture-is-agentic-epistemics.md`. It is the
worked example of the full pipeline.
