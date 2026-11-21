# Architecture Decision Records

Source: the design discussion of 2026-10-11 and 2026-10-12.
Items marked "delegated to the assistant" inside an ADR were resolved by the assistant with sensible defaults at the user's request (2026-10-12); the user may revisit them.
Status key: **accepted** (decided), **superseded**. All 14 are currently accepted.

| # | Title | Status |
|---|-------|--------|
| 01 | Orchestrator first, construction as proof of generality | accepted |
| 02 | Annual PPI targets, 5-year point forecasts | accepted |
| 03 | Custom orchestrator (no DVC) | accepted |
| 04 | Node model and model set | accepted |
| 05 | Fetching: outside the hash, data tracked in raw_data | accepted |
| 06 | Layers, delta reports and tracing | accepted |
| 07 | Code versioning: repos, pinned refs, clean worktrees | accepted |
| 08 | Run records and content-addressed store | accepted (simple) |
| 09 | DAG declaration: references, graph.yaml, orchestration.yml | accepted |
| 10 | Metadata, the jump-off year and region | accepted |
| 11 | Model placeholders, node contract and what the DAG needs | accepted |
| 12 | Data sources | accepted (candidates) |
| 13 | Canonicalization, environment and the invariant check | accepted |
| 14 | Core and domain packs (plug-and-play) | accepted |
