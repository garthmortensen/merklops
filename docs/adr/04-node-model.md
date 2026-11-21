# 04 Node model and model set
Status: accepted (model placeholders in 11)
Decisions:
- A node is either a CSV node (data in or data out) or a compute node. The DAG alternates between them.
- A model has two compute steps (a convention, not an orchestrator requirement, see 11): an estimation node writes a coefficients CSV, and a projection node reads those coefficients and writes a prices CSV.
- Upstream prices CSVs feed downstream models. Raw data nodes sit at the start of the DAG and are produced by the fetchers (05).
- The coefficients CSV holds point estimates and standard errors.
- Draft model set of 10: electricity, diesel_fuel, steel, cement_concrete, lumber, insulation, paint_coatings, labor_wages, equipment (nine, each split into estimation and projection) and cost_index (an aggregator, one repo). The dependencies between them come from a hand-written placeholder passthrough prior (11), which a Leontief or RIMS II matrix can replace later.
- Equipment has no good free source; a proxy series is used and labeled as a placeholder.
