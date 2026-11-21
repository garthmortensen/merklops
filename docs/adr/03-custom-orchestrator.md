# 03 Custom orchestrator, no DVC
Status: accepted
Decision: write our own orchestrator, using networkx for the graph. Canonicalization, per-layer deltas and trace-to-source are the novel parts, and DVC would be a layer to work around.
Rationale: DVC hashes raw bytes and has no layer model, delta metrics or trace report.
Related: cache and invariant check in 13; storage in 08.
