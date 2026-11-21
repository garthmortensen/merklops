# 09 DAG declaration
Status: accepted (2026-10-12)
## Decisions
- **One config file per node** (not per model), in the `execution` repo: `nodes/<node>.yml`. The node name equals the file name. The orchestrator knows nothing about models, only nodes. There is no `group` field and no group selector.
- **Consumer-declared references** (dbt `ref()` style). A node lists its inputs as references to named upstream outputs, and declares its outputs with file names and schemas. References use logical names, not file paths, so renaming an output file touches only the producer's config.
- **Reference syntax:** a plain string, `node.output`. Any string under `inputs` is a reference by definition. An input may be a list of references.
- **Versioned inputs with `@`:** `node.output@version` pins the input to a version. No suffix means live, produced in the same run.
  - A version always resolves to an immutable content sha. The hash uses the sha, never the label.
  - For raw data, the version is a git ref in `raw_data` (tag, commit, or a branch flagged floating), under the ADR 07 rules. Example: `steel_raw.data@v2026-03`.
  - For derived outputs, the version is a sha prefix or a label from a committed `versions.yml` (`baseline_a: <sha>`). A label is an alias, and moving one is a diffable change.
  - A pinned input is a frozen edge: the upstream node does not need to run. `graph.yaml` marks it as pinned.
  - Pinned and live inputs may mix on one node (for example new projection code with old coefficients).
  - Validation fails if a version does not exist in the store or in `raw_data`. A run record lists every pinned input and its resolved sha.
- **Provenance for pins:** every artifact in the content-addressed store carries a sidecar with the producing node, run id, input sha and commits, so a trace can pass through a pin (08).
- **Validation** (a typo is an error): every input resolves to exactly one producer or declared raw data node; no two nodes produce the same output; the graph is acyclic.
- **`graph.yaml`:** generated, never hand-edited, hashed as its own structural layer. Structure only: nodes (name, kind, output names) and edges, with pinned edges marked. No shas.
- **`orchestration.yml`:** says what to run, holds run flags, paths and defaults, and is hashed. Run flags stay out of node configs so toggling one does not change a node's config hash.
  - **Pins:** the default `model_lib` ref lives here, with an optional per-node override. A node's config slice for hashing includes its effective pin, so a default bump still changes every dependent node.
- **Selectors:** `name` (that node), `name+` (the node and everything downstream), `+name` (the node and everything upstream), and `exclude`. A list is a union. No groups, intersections or wildcards. A selected node whose unselected upstream has no cached output fails with a clear message.
## Sketch
    # execution/nodes/steel_projection.yml
    node: steel_projection
    kind: compute                # data | compute
    code:
      repo: steel_projection
      ref: v0.2.0                # tag, branch (floating) or sha
    inputs:
      coefficients: steel_estimation.coefficients@baseline_a
      history: steel_raw.data@v2026-03
      upstream_prices: [electricity_projection.prices]
    metadata: [jump_off_year]    # only these metadata keys enter this node's input_sha (10)
    outputs:
      prices:
        file: steel_prices.csv
        schema: {year: int, value: float, kind: str}
        key: [year]              # key columns, used to match rows in deltas (06)
    params:
      horizon: 5

    # execution/orchestration.yml
    paths: {repos: ../, store: ../store, runs: ../runs}
    defaults:
      lib: {repo: model_lib, ref: v0.3.1}
    metadata: metadata.yml
    run:
      select: ["electricity+", "+cost_index"]
      exclude: ["lumber_projection"]
## Where things live
`nodes/<node>.yml`, `orchestration.yml`, `versions.yml`, `metadata.yml` and the generated `graph.yaml` are all in the `execution` repo.
## Confirmed (2026-10-12, later)
1. `graph.yaml` is committed in the execution repo, and the orchestrator regenerates it every run and stops if the committed copy differs, so a stale file cannot hide a structure change.
2. A node's `input_sha` does not include `orchestration.yml`, so changing the `run:` selection does not invalidate any node cache. The whole file is hashed into the run's total sha and structure layer.
## Rejected
A central hand-written edge list (it duplicates the inputs actually read and can drift); matching on bare filenames (a typo silently changes the graph); a keyword prefix such as `ref x` (redundant); groups (the user prefers plain names).
