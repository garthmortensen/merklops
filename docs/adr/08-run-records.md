# 08 Run records and content-addressed store
Status: accepted (deliberately simple)
## Decisions
- One YAML file per run, named like `YYYYMMDD_HHMMSS_run_123.yaml`, all in the same directory, append-only, not committed to git.
- Outputs and data vintages are kept in a content-addressed store, so two runs can be compared, and row-level deltas computed, without git.
- Every artifact in the store carries a provenance sidecar (producing node, run id, input sha, commits), so a trace can pass through a pinned input (09).
- No backup, committed manifest or database for the prototype. Raw data is recoverable from the `raw_data` repo (05), and code from the pinned repos.
- Accepted consequence: a lost run directory loses the history of past runs on that machine. Revisit if a second machine or person needs to see run history.
## Layout and contents (delegated to the assistant, 2026-10-12)
- Store layout: `store/<first two hex chars>/<sha>.csv` (or `.yml` for text artifacts) plus `<sha>.prov.yml` for provenance. Per-node invariant tables sit in `store/invariant/<node>.txt` (13).
- A run record holds: run id and start time, the resolved sha of every node (own and composite), the five layer shas and the total sha, the resolved commit of every repo, the effective pins and any floating refs, the dirty flag (development mode only), every pinned input with its resolved sha, the metadata file contents (10), the selection and exclusions, the environment fingerprint (13), per-node status (ran, cache hit, failed, skipped) and per-node runtime and memory (recorded, never hashed).
