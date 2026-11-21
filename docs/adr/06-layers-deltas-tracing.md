# 06 Layers, delta reports and tracing
Status: accepted (2026-10-12)
## Layers
Each layer has its own sha:
| Layer | What it is |
|-------|-----------|
| Code | Resolved commit of each compute node's repo and of `model_lib` |
| Config | Per-node slice of the config in the execution repo |
| Data | Fetched vintages and node output CSVs |
| Metadata | YAML edited between runs (10) |
| Structure | Generated `graph.yaml` and `orchestration.yml` (09) |
## Deltas (confirmed 2026-10-12)
- Every delta starts **binary: changed true or false**, per node and per layer, straight from the sha comparison. Equality stays exact, with no tolerance.
- **Text files** (code, config, metadata, `graph.yaml`, `orchestration.yml`): binary, then **% similarity**. Similarity = 1 - differing lines / max(total lines in A, total lines in B), so identical text is 100%. The comparison runs on the canonical text (13), so formatting noise does not count. For code, `git diff AAA BBB` remains the detailed view.
- **CSV files** (data vintages and node outputs): binary, then
  - row counts (rows in A, rows in B, rows that differ, rows added, rows removed), and
  - for each numeric column, the max absolute difference and the max relative difference across matched rows. A non-numeric column reports the count of cells that differ.
- Magnitude affects only the report. It never changes what counts as changed.
## Structure of the hashes
- Each node has an `own` sha (its direct inputs) and a `composite` sha (including upstream). A layer sha is a hash over the nodes' shas in that layer. The total sha is a hash over the layer shas.
## Tracing
- Walk from a changed total sha down to the nodes whose own inputs changed while their upstream did not. Those are the root causes. Each root cause is reported with its layer, its binary changed flag and the delta report for its file type (% similarity for text, row counts and column differences for CSV).
- A change can have a layer as its root cause with no node changed. Example: editing only the description or the region in the metadata file changes the metadata layer sha and the total sha, but no node reads those keys. The trace reports the metadata layer itself as the root cause, with its text delta, and lists no affected nodes.
- A trace that reaches a pinned input continues through the artifact's provenance sidecar in the store (08, 09). The structure layer marks pinned edges, so changing a pin is a structure delta as well as a data delta.
## Resolved (delegated to the assistant, 2026-10-12)
1. CSV row matching: each output's schema in its node config declares its key columns, for example `key: [year]`. Rows are matched on the key. Rows present on only one side count as added or removed.
2. Relative difference with an old value of 0 is reported blank (undefined). The absolute difference is still reported.
