# Wireframes

Interactive, presentation-only pages with example data from the construction pack. Nothing is saved or really run.

Published privately (index with both pages): https://claude.ai/artifact/KbRiXpkvgPRjnHDeDhSjj2

## Pages
- `index.html`: start page, links to both views.
- `workbench.html`: **model builder**. Baseline | difference | candidate, the DAG below, run history at the foot.
- `story-review.html`: **reviewer**. A finished run as seven chapters, ending in notes and sign-off.

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

## Workbench scenarios to try
Each edit shows a preview of how many nodes will recompute before you run.
- steel drift window: only steel recomputes; `cost_index` moves.
- electricity drift window: fans out into four downstream projections.
- jump-off year: every estimation and projection reruns (a metadata change).
- model_lib v0.3.2: every model reruns but outputs are identical, so the trace stops early (a code change with no data change).
- data vintage v2026-01: two series revised; data and structure layers change (a pin change).
