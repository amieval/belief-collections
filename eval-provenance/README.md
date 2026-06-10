# sdl: silent-data-loss-v1 - the eval-provenance worked example

One published eval in miniature: two scorer observations (`sdl:a1`, `sdl:a2`) carrying
`eval:` artifact URIs and the six-subject convention, a cross-ruler agreement compound
(`sdl:a3`), a model-version-scoped verdict (`sdl:a006`), and the routing guidance that
rests on it (`sdl:a007`).

The collection borrows its vocabulary from the `method:` base collection
(`depends_on: ["method"]`). Its original local artifact-scheme enum (`sdl:c1`) is
superseded by the shared `method:c1` - a cross-namespace supersession, kept in the
graph as the worked demonstration of how a collection moves from improvised local
vocabulary to the shared base. `sdl:a4`/`sdl:a5` (kind `policy`, predating the shared
kind enum) are likewise superseded by `sdl:a006`/`sdl:a007` (kinds `verdict` and
`guidance` per `method:c2`).

## Known methodology violations - deliberate

This example **fails two of the `method:` methodology contracts on purpose**, so it
doubles as the demonstration of what a method-check failure looks like
(`mix cb.verify.collection sdl` reports them once the method-check pass is present):

- **m-runs (`method:c7`)**: the verdict `sdl:a006` cites one run (`run3`); the house
  minimum is three distinct runs. A real finding at this evidence level is not a
  weaker verdict - it is not a verdict; it would be authored as an observation or
  exploratory guidance.
- **m-judge-validation (`method:c8`)**: `sdl:a2` is scored by `ruler/llm-judge-vanilla`
  and no `judge-validation` record exists for that judge on this eval.

A teaching collection that visibly fails the house methodology, with the failure named
here, teaches both the mechanism and the culture. For a compliant example, see the
`toy:` collection (`../toy-eval/`).
