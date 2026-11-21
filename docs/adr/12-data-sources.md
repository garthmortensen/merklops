# 12 Data sources
Status: accepted as the candidate list for the real-data phase (2026-10-12). The first data is synthetic fixtures (11), so nothing here blocks the build.
Decided (confirmed 2026-10-12): public data only. No paid or licensed data. BLS, EIA and BEA data are public domain. The candidate list below is the selection for the real-data phase.
Verified to exist in BLS API v1 (first year of data found; latest point seen 2024-M12):
- WPU0543 industrial electric power (1958)
- WPU057303 diesel fuel (1973)
- WPU1017 steel mill products (1947); WPU101707 reinforcing bar (1982)
- WPU132 concrete products (1947); WPU1321 cement (1947)
- WPU0811 softwood lumber (1947)
- WPU1392 thermal insulation (1947)
- WPU0621 paint and allied products (1947)
- WPUIP2300001 net inputs to construction industries (1986)
- PCU532412532412 construction and mining machinery rental and leasing (2003), equipment proxy
- CES2000000003 construction average hourly earnings (2006 only; replaced by CES2000000008, see below)
Not found: WPU1193.
Not yet checked: series titles and units, revision status, the EIA series (API key needed, though bulk files need none), BEA table years and sector mappings.
Note on region (10): BLS PPI series are national. Wage data is available by state and metro area, but county and zip level public cost data is scarce. Region support will depend on which public sources exist at each level.
## Resolved (delegated to the assistant, 2026-10-12)
- Labor: use `CES2000000008` (construction average hourly earnings, verified on the BLS API, 1947 to 2024) instead of `CES2000000003` (from 2006 only). `CIU2020000000000I` (ECI, from 2001) is a possible cross-check.
- Equipment: the placeholder proxy stays `PCU532412532412` (from 2003), labeled as a proxy in its config.
- Electricity and diesel: use the BLS PPI series (`WPU0543`, `WPU057303`) so that all sources are BLS and no API key is needed. EIA stays a later option for finer detail.
- BEA I/O tables: use the annual summary-level Use tables (258, 259) from `industry/`. Table years and sector mappings are confirmed when the matrices are built.
- Region: the first real-data phase is national only.
- Series titles, units and revision status are recorded in each fetcher config when the fetcher is built.
