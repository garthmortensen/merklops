# 13 Canonicalization, environment and the invariant check
Status: accepted (2026-10-12; carried from the earlier issue discussion and confirmed)
Decisions:
- Canonicalize before hashing. YAML: parse, fill defaults, sort keys, re-emit. CSV: stable column and row order, lossless float formatting (shortest round-trip), ISO dates, `\n` line endings, one NaN representation, and a schema for dtypes (`"001"` versus `1`).
- Config is hashed per node (its own slice), and random seeds live in config.
- Inputs fall in three tiers: declared inputs (code, data, config, metadata, seeds) go into the node's input sha; the environment (Docker image digest, `uv.lock` hash, pinned thread counts such as `OMP_NUM_THREADS=1`) also goes into it; observational data (runtime, memory, unit-test results, CPU model) is recorded but never hashed.
- A table of `input_sha -> output_sha` per node is both the cache and the invariant check. If one input sha ever gives two different output shas, an unhashed input is leaking, and it should move into a hashed tier.
- Fetch nodes are exempt from the invariant in the sense that their output hash is recorded, not predicted (05).
## Resolved (delegated to the assistant, 2026-10-12)
Hash function: SHA-256, written as lowercase hex.
CSV canonical form: UTF-8 without a byte order mark, `\n` line endings, a trailing newline, comma delimiter, a header row, no index column, minimal RFC 4180 quoting, columns in schema order, rows sorted by the declared key columns. Floats use the shortest round-trip `repr` (never rounded). A NaN or missing value is an empty field. Dates are ISO 8601 (`YYYY-MM-DD`; annual data uses the year).
YAML canonical form: parsed with a safe loader, defaults filled in, keys sorted at every level, then emitted in block style with 2-space indentation, using the same float rule. The hash is taken over this canonical text. Comments and key order never change the sha.
Per-node config slice: the node's own file, with the effective defaults from `orchestration.yml` merged in.
Environment fingerprint: collected once per run by the orchestrator and passed to every node. It holds the image digest (from the `IMAGE_DIGEST` environment variable, or `none` outside Docker), the sha of `uv.lock`, and the Python version. The orchestrator sets `OMP_NUM_THREADS=1`, `OPENBLAS_NUM_THREADS=1` and `MKL_NUM_THREADS=1` in every node subprocess and records them. Each node's `input_sha` includes the fingerprint.
Invariant table: one append-only file per node in the store, with lines `input_sha output_sha`. A second, different output sha for a known input sha stops the run and names the node.
