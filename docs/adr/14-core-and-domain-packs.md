# 14 Core and domain packs (plug-and-play)
Status: accepted (2026-11-21). Refines 01: the product is an industry-agnostic core; construction becomes the first domain pack.

## Principle
The core knows only: **node, artifact, format, reference, source, metadata, layer, run**. It never knows a time axis, a unit, a currency, a geography or a model type. Everything domain-shaped arrives through a small number of declared extension points, each with a versioned contract. Adding a domain must need no core change: a new pack, a new project, nothing else.

## Three tiers
| Tier | What it is | Example |
|------|-----------|---------|
| **Core** | Hashing, canonicalization framework, DAG build and validation, cache and invariant table, store, run records, layers, deltas, tracing, selectors, the `check-pack` conformance kit | `merkle_core` |
| **Pack** | A versioned, self-describing bundle for one domain: node templates, sources, formats, comparators, metadata schema, fixtures | `pack-construction`, `pack-toy` |
| **Project** | An execution repo that uses one or more packs: node configs, `orchestration.yml`, `metadata.yml`, `versions.yml`, pins | today's `execution` repo |

Today's ADRs map onto this: 03, 06, 08, 09, 13 are core; 02, 04, 10 (keys), 11 (maths, schemas), 12 are the construction pack; 05 and 07 split (mechanism in core, BLS/EIA/BEA and model repos in the pack).

## Extension points (the only things a pack can supply)
1. **Compute runners**: the node contract of 11, unchanged and language-agnostic (subprocess). Python and R both qualify. The contract gets a version and a concrete calling convention: `run --config slice.yml --inputs inputs.json --out <dir>`, where `inputs.json` maps each declared input name to a resolved path (or a list of paths).
2. **Sources** (fetchers): the same subprocess contract with `kind: source`. Output is recorded, not predicted (05, 13). Writes the artifact plus the excluded fetch sidecar.
3. **Formats**: `canonicalize(path, schema) -> bytes` and `validate(path, schema)`. The core ships `csv`, `yaml`, `json` and `bytes` (raw sha, no canonical form). A pack may add one, for example `parquet`, `geojson` or a model binary.
4. **Comparators**: `compare(a, b, schema) -> delta report`. The report only; equality is always the binary sha (06), so a comparator can never change what counts as changed. Core ships text similarity and the CSV row/column report; a pack may add one, for example a % change of a final forecast, or a map diff.
5. **Metadata schema**: a JSON Schema for the keys the pack's nodes may read (for example `jump_off_year`, `region`). The core validates `metadata.yml` against the union of the schemas of the packs in use and hashes it as today (10). The core has no built-in keys.
6. **Node templates**: a generator that expands a short declaration into ordinary `nodes/<node>.yml` files (for example "a model = estimation + projection"). Expansion happens before validation, and the expanded files are what get hashed and committed, like `graph.yaml` (09). The core never sees a template.

What a pack can **not** supply: new layers, a change to the hash function, the cache, the trace algorithm or equality. Those stay core so that every domain gets identical guarantees.

## Pack manifest
    # pack.yml
    name: construction
    version: 0.1.0
    core: ">=1.0,<2.0"           # core API range
    node_contract: 1
    sources:     [bls, eia, bea]              # each a runner repo/ref or entry point
    formats:     []                            # uses the core csv/yaml
    comparators: [forecast_pct_change]
    metadata_schema: metadata.schema.json
    templates:   [estimation_projection_model]
    fixtures:    fixtures/                      # synthetic data + expected shas for check-pack

## Rules that keep it honest
- **A pack is code, so it is pinned and hashed.** The project pins each pack like `model_lib` (07): `packs: {construction: v0.1.0}`. The resolved commit enters the code layer.
- **Formats and comparators are hash-relevant in a subtle way.** A format's canonicalizer decides an artifact's sha. Its version therefore enters the `input_sha` of every node whose outputs or inputs use it. Bumping a canonicalizer is a visible code delta across those nodes, never a silent sha change. Comparators affect only reports and are recorded, not hashed.
- **Namespacing.** Node and output names are `pack/node.output` once more than one pack is in a project, so two packs can both have a `prices` node. With one pack the prefix may be omitted.
- **Cross-pack edges** are ordinary references. A pack exports outputs by declaring their format and schema; a consumer in another pack depends only on that, never on the producer's internals.
- **No reach-around.** A runner reads only `inputs.json`, its config slice and the declared metadata keys (11). `check-pack` enforces it.

## Conformance kit (`check-pack`)
Shipped by the core; the gate for "plug-and-play":
- manifest validates, core range satisfied, every referenced runner and schema exists;
- each runner run twice on the pack fixtures gives byte-identical outputs (determinism);
- writes only declared outputs; outputs validate against their schemas and formats;
- canonicalizing an already canonical artifact is a no-op (idempotence);
- the 9 acceptance scenarios of 11 pass on the pack's fixtures, generalized: rerun = all cache hits, one fixture value changed = trace names that node, and so on.

## Proof of generality (replaces the test in 01)
- `pack-toy` ships inside the core repo: two or three trivial nodes (for example sum and scale over a JSON or CSV), used by core tests. It must not look like construction.
- `pack-construction` is the first real pack.
- A second, structurally different pack is the acceptance test of this ADR: the core must run it with no core change. To decide (below).

## Decided (2026-11-21)
1. **Second domain: ACA risk adjustment** (HHS-HCC style: enrollment and diagnoses map to HCCs, coefficients give member risk scores, aggregated into plan-level scores and state transfers). It differs from construction in the right ways: member-level not annual series, scoring not forecasting, published model coefficients as a versioned input.
2. **Packs are git repos with pinned refs** (07). No Python package distribution.
3. **Formats and comparators run in-process as Python plugins; runners and sources run as subprocesses.**
4. **One pack per project for the prototype**, namespacing designed in but unused.
5. **Layers are fixed at five in the core.**

## Consequences of the ACA pack
- All project data is synthetic (confirmed 2026-11-21), so there are no privacy constraints: member-level data may live in `raw_data` like any other data, and the UI may show rows freely.
- Volumes suggest a columnar format (parquet) as a pack format plugin.
