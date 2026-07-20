---
title: About This Site
url: "docs/about-this-site"
description: "What this field guide is, who it is for, and how Doc Holiday generated it."
---

This field guide explains the connective internals of the LangGraph Python library. It focuses on the layer that decides when nodes run, what `compile()` produces, why checkpoints have their shape, and how replay, resume, and idempotency stay consistent across runs.

This page frames the rest of the site. The pages below stay at the concept level and use real file paths and symbol names so each concept stays traceable. The guide keeps its attention on the open source Python library in `libs/langgraph` and `libs/checkpoint*`; it does not repeat API reference material, step by step tutorials, the hosted platform, the SDK packages, or the separate JS and TS port in `langgraphjs`.

Engineers adopting, embedding, extending, or debugging LangGraph need one mental model for state schemas, graph topology, compiled artifacts, checkpoint records, and execution state.

The official LangGraph docs cover usage, tutorials, and reference material. This site owns the connective internals layer they do not spell out: how those pieces work together across the runtime.

The guide runs from the big picture through invoke flow, scheduling, state compilation, topology, checkpoints, and replay, so each sibling page handles one runtime layer and hands off to the next.

## Contents

- [The big picture](./00-the-big-picture.md) — The problem space and the core runtime model.
- [State schemas and graph topology](./01-state-schemas-and-graph-topology.md) — How state definitions and edges shape execution.
- [Compilation](./02-compilation.md) — What `compile()` produces and why the result matters.
- [Version bookkeeping](./03-version-bookkeeping.md) — How `channel_versions` and `versions_seen` drive scheduling.
- [Checkpoints](./04-checkpoints.md) — How the stored state is shaped and why the shape matters.
- [Replay, resume, and idempotency](./05-replay-resume-idempotency.md) — How the runtime avoids double work.
- [Fast-moving notes](./06-fast-moving-notes.md) — Dated notes for areas such as `DeltaChannel` and v3 streaming.
- [Where to look in the code](./07-where-to-look-in-the-code.md) — A compact map of the load-bearing files.

Doc Holiday (https://doc.holiday) wrote this site by exploring the LangGraph source repository directly. Each page grounds its claims in actual files and symbols, for example `langgraph/pregel/_algo.py`, `StateGraph`, `Pregel`, `JsonPlusSerializer`, `InMemorySaver`, `channel_versions`, and `versions_seen`.

Several terms recur across the site because they carry the runtime story. `channel_versions` tracks version state by channel name. `versions_seen` tracks, by node name, what each node has already consumed. Those two fields sit at the center of scheduling, replay, and idempotency, so the guide returns to them when it explains why a checkpoint looks the way it does.

[GENERATED_FROM: commit SHORT_SHA, DATE — the operator will replace this placeholder; include it verbatim, do NOT substitute a commit or date yourself]

The guide covers only the open source Python library. A separate JS and TS port lives in `langgraphjs`. The codebase changes quickly, so this site captures a snapshot rather than a fixed contract. The official LangGraph docs remain authoritative. Corrections are welcome at `[CONTACT_OR_REPO_LINK placeholder for the operator]`.

This snapshot should be read against the repository, not instead of it. When the code shifts, the pages here should shift with it. The correction token above exists so the operator can update the text without rewriting the whole guide.

Fast moving areas such as the beta `DeltaChannel` and v3 streaming appear only as dated side notes when they affect the surrounding design. The main thread of the guide stays on the stable core: `StateGraph`, `Pregel`, channels, and checkpointers.