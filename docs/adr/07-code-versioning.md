# 07 Code versioning
Status: accepted
Decisions:
- Each compute node has its own repo: one per model estimation and one per model projection. Also separate repos for the orchestrator, `execution` (all model and orchestration configs, `orchestration.yml`, metadata, the generated `graph.yaml`), `fetchers`, `raw_data` and `cost_index`. All belong to the same GitHub project. They are sibling directories on disk, with no git submodules, and paths are set in config.
- `model_lib` is one repo with two modules (estimation helpers, projection helpers), cut out of the model repos for DRY. Model repos stay thin and import it.
- Refs are pinned in config in the execution repo, for example `model_lib: v0.3.1`. Stable pins use semantic-version tags. A branch is allowed but the run is flagged floating. A commit sha is also allowed.
- The orchestrator resolves the ref to a full commit sha before the run. The resolved sha is what is hashed and recorded, whatever kind of ref was written. A moved tag therefore shows up as the same config with a different resolved sha.
- A node's code sha is the resolved commit of its own repo plus the resolved commit of `model_lib`. A library change invalidates every node that uses it (coarse but correct); `git diff <tagA> <tagB>` in the repo shows the delta.
- Normal runs execute from clean worktrees of the resolved shas, each node in its own subprocess with its own import path, so a dirty tree cannot occur and nodes may use different library versions in one run.
- Running uncommitted code is a separate, explicit development mode. It is reported as dirty, recorded with a hash of `git diff HEAD`, and never cached.
- The same pinning applies to the model repos.
Consequence: changing a pin is both a config delta and a code delta, and both are traceable.
History: the first answer was "report dirty or clean, use the commit, whole repo diff"; refined to path-aware repos per node plus clean worktrees. A single commit for the whole system was dropped because any commit would change every node.
## Resolved (delegated to the assistant, 2026-10-12)
Clean worktrees live under the runs directory at `worktrees/<repo>@<sha>`. A worktree for a sha never changes, so it is reused between runs and removed only by an explicit `prune` command.
