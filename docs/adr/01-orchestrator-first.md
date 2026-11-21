# 01 Orchestrator first
Status: accepted
Decision: the generic orchestrator (hashing, caching, tracing, delta reports) is the first deliverable. Construction costs is the proof of generality. The models stay deliberately simple.
Success test: a change in one input traces cleanly to the final `cost_index` delta.
Rationale: the novel part of the idea is the traceability, not the forecast quality.
