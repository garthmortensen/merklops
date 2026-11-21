# 11 Model placeholders, node contract and what the DAG needs from them
Status: accepted (2026-10-12). The model internals are placeholders; the node contract is the part that matters.

## Principle
The product is the orchestrator (01). The 10 models exist to exercise it. Real models (for example ones built on BEA RIMS II multipliers or I/O tables) will replace them later. Replacing a model must need no orchestrator change: a new repo or tag, a new config, and possibly a changed edge (a structure delta). So this ADR fixes the contract and keeps the internals deliberately trivial.

## Node contract (applies to every compute node, any model)
- Invoked by the orchestrator as a subprocess (07) with: the node's config slice, the resolved input file paths (one per declared input), and an output directory.
- Reads only declared inputs. No network, no wall clock, no environment reads that are not hashed.
- Deterministic: the same hashed inputs give byte-identical output. Randomness is not used by the placeholders; a `seed` config key is reserved for later models.
- Writes exactly the declared outputs (files named in its config), each as a canonical CSV (13) with a dtype schema sidecar, and nothing else in the output directory that affects results.
- Exit code 0 means success. Any other code means failure.
- Failure handling: a failed node marks the run record, its downstream nodes are skipped (not run on stale data), and independent branches still run.

## Node types and what each reads
| Node | Reads | Writes |
|------|-------|--------|
| data node (from a fetcher, 05) | nothing in the DAG | `<series>.csv` (`year,value`) |
| `<m>_estimation` | own history CSV, passthrough prior CSV, jump-off year | `<m>_coefficients.csv` |
| `<m>_projection` | `<m>_coefficients.csv`, own history CSV, upstream `prices` CSVs, jump-off year | `<m>_prices.csv` |
| `cost_index` | every model's `prices` CSV, weights from config | `cost_index.csv` |

## Placeholder schemas (long format, rows sorted by key)
- history / data: `year,value`
- passthrough prior: `from_model,to_model,coefficient`
- coefficients: `term,estimate,std_error` (terms: `drift`, then `beta_<upstream>` for each upstream)
- prices: `year,value,kind` where kind is `actual` or `forecast`
- cost_index: `year,value,kind`; the weights are written alongside as `cost_index_weights.csv` (`model,weight`) so they are a diffable CSV node
A producer's schema is part of its own config and is hashed, so changing a schema is a visible delta.

## Placeholder maths (swappable)
- Estimation: `drift` = mean annual log change of the model's own series over data up to the jump-off year, with its standard error. Each `beta_<upstream>` is taken directly from the passthrough prior, with an empty std_error (no fitting).
- Projection: from the last actual year, `dlog p_t = drift + sum(beta_i * dlog upstream_i,t)`, compounded forward 5 years. Models with no upstream use drift only.
- `cost_index`: weighted mean of the model price indices, each rebased to 100 at a base year in config. Weights are hand-set in config (equal by default).
- The passthrough prior starts as a hand-written placeholder CSV, a CSV node in the DAG. It later becomes the Leontief or RIMS II output built in `industry/`, with no change to the nodes that read it.

## What this gives the DAG (the reason for the shape)
- Fan-out: electricity feeds four models. Fan-in: `cost_index` reads all the others. Diamond paths: diesel reaches `cost_index` through cement, lumber and equipment. These are the cases the trace and cache must handle.
- Two edge kinds: a model-internal chain (estimation, coefficients, projection) and cross-model price edges.
- Each node type touches a different set of layers (06):
  - data node: data (the fetch config is in the config layer)
  - estimation and projection: code, config, data, metadata (the jump-off year)
  - cost_index: code, config (weights, base year), data
  - whole DAG: structure
- Metadata read: only the jump-off year, and only by estimation and projection nodes. The jump-off year bounds the data an estimation node uses, and projection starts the year after it (10).

## Scenarios the placeholders must support (acceptance tests for the orchestrator)
1. Re-run unchanged: every node is a cache hit.
2. Refetch gives identical data (only the timestamp differs): same data sha, no downstream change.
3. One data value revised: the trace names that data node, and the delta report shows the row and the max absolute and relative difference.
4. One config value changed (for example the drift window or a weight): the trace names that node's config slice.
5. Library tag bumped (`model_lib`): all dependent nodes change, and the trace names the code layer with the git diff.
6. Edge added or removed: a structure delta in `graph.yaml`.
7. Jump-off date changed: only estimation and projection nodes change.
8. A node fails: downstream skipped, independent branches complete, run record shows it.
9. A model swapped for another with the same contract: only that node's code and config change.

## Out of scope for the prototype
Forecast accuracy, backtesting, distributions (P10/P50/P90), model selection. The coefficients schema keeps `std_error` so quantiles can be added later without a breaking change.

## Confirmed (2026-10-12)
- The placeholder maths and the CSV schemas above are accepted.
- The first data is synthetic fixture CSVs: deterministic, and no fetcher is needed to start. Real BLS pulls (12) come once the fetcher works.
- The estimation and projection split is a convention for these placeholders, not an orchestrator requirement. A real model, such as one built on RIMS II multipliers, may be one node or several.

## Decided (2026-10-12, later)
- `cost_index` uses equal weights.
- The base year is the current year, written as a literal in config (2026) and never computed from the clock, so the same inputs always give the same output.

## Decided (2026-10-12, region)
- Region has no effect on the placeholder models. No `location_factors` node is added.

## Resolved (delegated to the assistant, 2026-10-12)
Synthetic fixtures, kept in the `raw_data` repo under `synthetic/`, reproducible from a generator committed with them:
- One series per data input: electricity, diesel_fuel, steel, cement_concrete, lumber, insulation, paint_coatings, labor_wages and equipment. Each has 40 annual observations, years 1986 to 2025, in the `year,value` schema.
- Shape: a log random walk with drift, `dlog v_t = drift + noise`, with `drift` between 0.02 and 0.04 and noise standard deviation 0.03, starting at 100 in 1986.
- Each series has its own fixed integer seed (1 to 9, in the order above). The generator, the seeds and the parameters are written in a sidecar `generator.yml`, which is hashed with the data.
- The jump-off year is 2025, so projection covers 2026 to 2030. Base year 2026 (a forecast year) for `cost_index`.
- The passthrough prior is a hand-written `passthrough.csv` with `from_model,to_model,coefficient`: electricity to steel 0.25, cement_concrete 0.15, insulation 0.20, paint_coatings 0.10; diesel_fuel to cement_concrete 0.10, lumber 0.15, equipment 0.20; labor_wages to equipment 0.30. All values are placeholders.
