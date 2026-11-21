# 10 Metadata, the jump-off year and region
Status: accepted (2026-10-12)
## Decisions
- Metadata is a YAML file, edited by hand between runs. It holds facts that cross models.
- Keys so far:
  - `description`: a DAG description.
  - `jump_off_year`: YYYY only (no month or day). Annual data up to and including this year is used to estimate, and projection starts the year after.
  - `region`: the geography the run is for. The user gives one of: national, a state, a county, or a zip code.
- The set of keys stays open for more factors.
- **The entire metadata file is hashed into the metadata layer.** No key is exempt, so there is no "descriptive only" group. Any edit, including to the description, changes the metadata layer sha.
- **The whole file does not enter every node.** A node declares in its config which metadata keys it reads (for example `metadata: [jump_off_year]`), and only those keys enter that node's `input_sha`. Editing the description therefore invalidates no node.
- **Region format:** a level plus a code, `region: {level: state, code: CO}`. Levels: `national`, `state`, `county`, `zip`. Codes are FIPS for state and county and the five-digit code for zip. A run fails on an unknown level or code.
- **Region has no effect on the placeholder models.** No placeholder node reads it, so it is carried and hashed in the metadata layer only. A real model can declare it later.
- **No API is ever needed.** Metadata stays a file. This drops the earlier idea of a metadata API for good.
## History
The jump-off value was first a YYYYMMDD date and was changed to a year, because the data is annual. The metadata was first expected to need an API; it does not.
## Resolved (delegated to the assistant, 2026-10-12)
1. A run copies the metadata file into its run record, so the run is reproducible even if the file was edited afterwards and not committed. Committing is not required (but is encouraged, since git then holds the history).
2. No fallback across geography levels. A missing level is an error. This matters only once a node reads region.
