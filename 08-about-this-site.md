# About this site

This field guide explains the connective internals of the LangGraph Python library. It focuses on the parts the official docs do not spend much time on: the version bookkeeping that decides when a node runs, what `compile()` lowers a graph into, why checkpoints carry the fields they do, and how replay, resume, and idempotency work together.

This page frames the rest of the site. The pages below stay at the concept level and use real file paths and symbol names so the code stays easy to trace. The guide keeps its attention on the open source Python library in `libs/langgraph` and `libs/checkpoint*`; it does not try to cover the hosted platform, the SDK packages, or the separate JS and TS port in `langgraphjs`.

## Contents

- [The big picture](/00-the-big-picture.md) — The problem space and the core runtime model.
- [State schemas and graph topology](/01-state-schemas-and-graph-topology.md) — How state definitions and edges shape execution.
- [Compilation](/02-compilation.md) — What `compile()` produces and why the result matters.
- [Version bookkeeping](/03-version-bookkeeping.md) — How `channel_versions` and `versions_seen` drive scheduling.
- [Checkpoints](/04-checkpoints.md) — How the stored state is shaped and why the shape matters.
- [Replay, resume, and idempotency](/05-replay-resume-idempotency.md) — How the runtime avoids double work.
- [Fast-moving notes](/06-fast-moving-notes.md) — Dated notes for areas such as `DeltaChannel` and v3 streaming.
- [Where to look in the code](/07-where-to-look-in-the-code.md) — A compact map of the load-bearing files.
- [About this site](/08-about-this-site.md) — Scope, generation, and correction notes.

Doc Holiday (https://doc.holiday) wrote this site by exploring the LangGraph source repository directly. Each page grounds its claims in actual files and symbols, for example `langgraph/pregel/_algo.py`, `StateGraph`, `Pregel`, `JsonPlusSerializer`, `InMemorySaver`, `channel_versions`, and `versions_seen`.

[GENERATED_FROM: commit SHORT_SHA, DATE — the operator will replace this placeholder; include it verbatim, do NOT substitute a commit or date yourself]

The guide covers only the open source Python library. A separate JS and TS port lives in `langgraphjs`. The codebase changes quickly, so this site captures a snapshot rather than a fixed contract. The official LangGraph docs remain authoritative. Corrections are welcome at `[CONTACT_OR_REPO_LINK placeholder for the operator]`.

Fast moving areas such as the beta `DeltaChannel` and v3 streaming appear only as dated side notes when they affect the surrounding design. The main thread of the guide stays on the stable core: `StateGraph`, `Pregel`, channels, and checkpointers.