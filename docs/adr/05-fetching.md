# 05 Fetching
Status: accepted
## Decisions
- The act of fetching is not part of any sha. The data it returns is hashed. Fetching the same data twice must give the same sha, so fetch timestamps and API response metadata are kept in a sidecar and excluded from the hashed content.
- Fetching is the start of the DAG. It never runs silently. It runs only on request, per the config (for example `--refetch electricity`). New and old vintages are both kept. A normal run uses the vintage the node's input selects.
- A node picks a raw data vintage with the `@` suffix on its input, for example `steel_raw.data@v2026-03`, where the version is a git ref in `raw_data` (09).
- A data revision appears as a changed data node, and the trace stops there.
- Fetchers live in their own `fetchers` repo: one generic fetcher per source (BLS, EIA, BEA), configured per series in the execution repo.
- Raw data is tracked by git in a separate `raw_data` repo, so data commits do not change any fetcher's code sha. Public domain data is committed with citation.
## Mechanics (delegated to the assistant, 2026-10-12)
- A fetch writes the series CSV and a sidecar (`<series>.fetch.yml`: source, series id, request parameters, the fetch time, and the response digest) into the `raw_data` working tree.
- The fetcher never commits. The user commits and tags the new vintage in `raw_data` (for example `v2026-03`). The orchestrator only reads pinned refs.
- A fetch whose data sha equals the latest committed vintage reports "no change" and writes nothing new.
- A failed fetch leaves existing files untouched and exits non-zero.
## History
The first answer was that fetching is inside the DAG; this was refined to the above. The first answer on raw data was "untracked on disk"; reversed to "tracked" so vintages are recoverable.
