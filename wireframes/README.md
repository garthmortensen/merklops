# Wireframes

Interactive, presentation-only pages with example data from the construction pack. Nothing is saved or really run.

Published privately (index with both pages): https://claude.ai/artifact/KbRiXpkvgPRjnHDeDhSjj2

## Pages
- `index.html`: start page, links to both views.
- `workbench.html`: **run workbench** (model builder). Baseline | difference | candidate, the DAG below, run history at the foot.
- `delta_setter.html`: **variation of the workbench**, aligned to the ADRs. Every setting sits in the file it lives in (`metadata.yml`, `orchestration.yml`, `nodes/<node>.yml`) as a setting | baseline | candidate row. Covers per-node code refs (tag, floating branch, dev working tree, resolved shas) and `model_lib` default plus overrides (07, 09); `@` vintage and `@baseline_a` coefficient pins (05, 09); metadata keys and which nodes read them (10); selectors (09); the environment fingerprint (13); the run plan YAML; and run-record file names (08).
- `dag_view.html`: **Merkle DAG**. The 29 nodes and 53 edges as an SVG with arrows from producer to consumer. Each edge is labelled with the sha it carries (pinned edges dashed); each node shows a hash per layer (C K D M E) plus its input sha (cache key) and output sha. A Merkle root strip shows the six layer shas and the total sha. Scenarios switch the candidate; an inspector shows own, input, output and composite shas and the incoming and outgoing edges.
- `delta_diff.html`: **Merkle DAG diff**, drill-down from lineage to node to layer. The DAG view plus a side pane: select a node, then a layer pill. Code → git diff (red/green) of the node repo and `model_lib`; config → canonical YAML diff of `nodes/<node>.yml`; data → row-level CSV diff matched on the key (changed, added, removed rows, Δ % per numeric cell, max abs/rel per column); metadata and environment → text diffs. Merkle-root layers open run-level diffs (`metadata.yml`, environment, `graph.yaml` + `orchestration.yml`). Upstream/downstream buttons walk the trace. Deep links: `delta_diff.html#scn=run123&node=steel_estimation&layer=config`; the story page links in.
- `results_dashboard.html`: **results dashboards** for leadership, downstream of the Merkle DAG. Pick a signed-off baseline and up to three runs; one dashboard per model (cost_index overview with a contribution chart, steel with its drift estimate, electricity with downstream exposure, a standard template for the rest); absolute, Δ and Δ % modes; table view; provenance shas linking into the Merkle DAG diff. A structure-change banner appears when runs have different structure shas.
- `audit_review.html`: **release audit** for managers. Guardrail findings computed from run records (wrong code version, out-of-range parameter, wrong vintage, stale upstream, misconfigured selection, environment drift), each resolvable by accept-with-reason or return-to-owner (failures cannot be waived); intent vs effect per owner; regression tie-out against production; hash-bound owner sign-offs; a release gate. "Simulate owners' fixes" shows the happy path.
- `workflow.html`: **model lifecycle workflow**. The year mapped (features, reparameterization, data refresh, cascades, audit), error classes and their guardrails, ten recommendations (cycle manifest, run tiers, typed parameters, sweeps, declared intent, data calendar, automatic cascades, hash-bound sign-offs, release CI, CODEOWNERS), five click-through flows linking the wireframes, and two new screens: cycle board and data calendar.
- `ai_opportunities.html`: **AI on the Merkle DAG** (exploration). What Dagster+ AI, MLflow 3 and Airflow's common-ai provider ship (Nov 2026); principles (numbers from tools, every claim cites a hash, deterministic gates, agents propose and people sign); 12 LLM use cases by persona with a value/effort map; an "ask the lineage" mock with tool calls; six agents and an autonomy ladder; an MCP tool surface; a phased start; risks and controls.
- `story-review.html`: **run lineage story** (reviewer). A finished run as seven chapters, ending in notes and sign-off.

The page files are written for the artifact publisher, which adds the `<!doctype html>` skeleton. For a multi-page publish, standalone copies with the skeleton are generated into the scratchpad.

## Decided (2026-11-21)
- **Purpose:** set up, execute and compare runs. Compare two runs is the hero.
- **Personas:** model builders use the **workbench**; reviewers use the **story**. The question-led home page was dropped.
- **Tone:** pivots around the user, down to earth, plain language first, hashes one click away.
- **Every run has a baseline** chosen at setup (default: the last signed-off run), so every run is born comparable.
- **Local:** served by the orchestrator on the user's machine, single user.
- **Domain-neutral screens** with a pack-supplied domain summary (construction: `cost_index`; ACA: plan risk score / transfers).
- **Run plans:** a setup made in the UI is a YAML file in the runs directory, outside git, canonicalized and hashed like the files it overrides. The trace labels such a change "from run plan". Promoting it into the execution repo is a manual git step.
- **v1 never touches git:** no commits, no code edits, no ref moves.
- **Reviewer notes and sign-off:** shown in the UI, presentation only for now.
- **No privacy constraints:** all data is synthetic.

## Layer scope (decided 2026-11-21)
- **Global, one set for all models:** Metadata (one file: jump-off year, region, description) and Structure (`graph.yaml` + `orchestration.yml`, generated from the pins).
- **Per model:** Code (the model's repo ref and its `model_lib` pin), Config, Data (its vintage).
- The workbench puts global settings on top and a tab per model below. DAG nodes show C K D chips; M and S show once in a global band above the DAG.

## Workbench scenarios to try
Each edit shows a preview of how many nodes will recompute before you run.
- Every node shows a strip of five layer chips (C K D M S); click a node for per-layer shas and deltas, click a layer tile to filter the DAG.
- any model's drift window (all 9 are editable): only that model recomputes; `cost_index` moves.
- electricity drift window: fans out into four downstream projections.
- jump-off year: every estimation and projection reruns (a metadata change).
- model_lib v0.3.2: every model reruns but outputs are identical, so the trace stops early (a code change with no data change).
- data vintage v2026-01: two series revised; data and structure layers change (a pin change).

## Proposed: a sixth layer, Environment (2026-11-21, in `delta_setter.html` only)
System-level factors that can change results: image digest, Python, `uv.lock` (software dependencies), thread pins, database server version, ODBC driver, and `odbc.ini` (canonicalized, secrets excluded). Like metadata, the set is global but a node hashes only the parts it uses: every compute node uses the runtime; nodes that declare `env: [db]` also use the database parts (in the wireframe, `labor_wages_estimation` and `equipment_estimation`).
**Conflicts with the ADRs:** ADR 14 fixes the layers at five in the core, and ADR 06 lists five. ADR 13 puts the environment fingerprint into every node's input sha without a layer of its own. Adopting this needs those three updated.

## Language (2026-11-21)
Headings use the data-structure terms: Merkle DAG, Merkle root, content-addressed, hash chaining, lineage (parents and children), diff, cache hit, append-only run log. Body text stays plain.

## Proposed: results store, dashboards and structural change (2026-11-21)
**Data flow.** DAG run → run record → *publish* (a sink outside the hash) → results store → dashboards. Dashboards never read the DAG.
- **Exported outputs are the contract.** A node marks an output `export: true` with a `schema_version`. Dashboards bind to `name.output@schema` (wildcards allowed: `*_projection.prices@v1`), not to node internals.
- **Results store.** Append-only, tidy rows keyed by run: `run_id, output, schema_version, key cols, value cols` plus a runs table (date, status, total sha, structure sha, output shas). It is built from the content-addressed store, so every number keeps its output sha.
- **Dashboards are pack artifacts.** One spec per model (`dashboards/<model>.yml`), versioned and pinned like code, naming the outputs and schema versions it supports. A model without one gets the standard template.
- **Leadership reads signed-off runs by default.** The baseline must be a signed-off run.

**Structural change (models added, removed, edges rewired) as a workflow.**
1. **Change as code.** Edit `nodes/*.yml` (or run the pack template) on a branch of the execution repo. Never hand-edit `graph.yaml`.
2. **Validate.** Regenerate `graph.yaml`; check references resolve, outputs are unique and the graph is acyclic (ADR 09); run `check-pack` scenarios.
3. **Structure diff in review.** The PR shows a graph diff: nodes added or removed, edges added or removed, pins changed, and the *impact set* (every descendant of a changed edge, from the graph). Exported outputs whose inputs changed are listed as "definition changed".
4. **Contract check.** Any exported output removed, renamed or re-schema'd must bump its `schema_version`; dashboards that bind to it fail the check until their spec supports the new version. A rename can declare `renamed_from:` so history lines up.
5. **Run with cache.** Merge, run: only the impact set recomputes (unchanged subgraphs are cache hits). The new run carries a new structure sha.
6. **Compare across structures.** Dashboards and diffs match outputs by logical name; outputs present in one run only show as added or removed; an aggregate whose inputs changed is decomposed into data deltas plus a "definition change" residual.
7. **Re-baseline.** Sign off the first run on the new structure so later runs compare like for like.
