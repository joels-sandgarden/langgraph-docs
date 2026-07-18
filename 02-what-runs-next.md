# What Runs Next

This page explains how LangGraph decides which node runs in the next superstep. The [Persistence](https://docs.langchain.com/oss/python/langgraph/persistence) page covers checkpoint storage; this page covers the bookkeeping that turns stored state into the next frontier of work.

The scheduler does not keep a runtime queue for this decision and it does not walk edges from a live graph to discover work. It reads two persisted maps in the checkpoint, `channel_versions[channel_name]` and `versions_seen[node_name][channel_name]`, and it treats a node as due when one of its trigger channels is available and the current channel version is newer than the version recorded for that channel in that node’s `versions_seen` entry.

## The checkpoint is the scheduler state

The important move is to treat checkpoint data as the source of truth for scheduling. `channel_versions` records the current version for each channel, and `versions_seen` records the last version each node has already consumed. That pair is enough to decide whether a trigger channel has changed since the node last ran.

`prepare_next_tasks` in `libs/langgraph/langgraph/pregel/_algo.py` builds the next task set, and `_triggers` performs the version comparison for each candidate node. `updated_channels` and `trigger_to_nodes` narrow the candidate set before that comparison runs, so the scheduler spends time on channels that changed instead of scanning the whole graph.

## Where the check happens

The loop follows a simple sequence: run, consume, write, bump, compare again. A task runs in the current superstep, `apply_writes` records that its trigger channels now count as seen, `BaseChannel.consume()` lets a channel clear or compact anything it needs before the next pass, `BaseChannel.update()` applies the new writes, and `BaseChannel.finish()` closes out the frontier when nothing else remains to trigger.

`apply_writes` in `libs/langgraph/langgraph/pregel/_algo.py` updates `versions_seen` first, then it advances `channel_versions` as channels consume reads and accept writes. `prepare_next_tasks` then reads the new checkpoint on the next superstep and asks the same question again: which trigger channels are available, and which ones now outrank the version each node already saw?

## Example: two nodes over three supersteps

The tokens in this table are opaque order markers. The scheduler compares them; it does not parse them as timestamps or counters.

| Superstep | Node that runs | `channel_versions` | `versions_seen[A]` | `versions_seen[B]` |
| --- | --- | --- | --- | --- |
| 0 | `A` | `start=α`, `work=∅` | `start=α` | `work=∅` |
| 1 | `B` | `start=α`, `work=β` | `start=α`, `work=β` | `work=β` |
| 2 | none | `start=α`, `work=β`, `done=γ` | `start=α`, `work=β` | `work=β`, `done=γ` |

In this graph, `A` reacts to `start` and produces `work`, then `B` reacts to `work` and produces `done`. After `B` runs, no trigger channel advances again, so the frontier goes empty and the run stops.

## Why versions exist instead of a dirty flag

Versions make replay and crash recovery deterministic. The next frontier comes back from the checkpoint, so no in-memory queue needs to survive a crash or a restart. A node that runs without its trigger channel advancing does not run again, because `versions_seen` catches up to the current channel version and the comparison turns false on the next pass.

## Failure modes

`recursion_limit` works as a superstep budget, not as call stack depth. When the loop spends that budget, LangGraph raises `GraphRecursionError`; the user-facing troubleshooting page for [GRAPH_RECURSION_LIMIT](https://docs.langchain.com/oss/python/langgraph/GRAPH_RECURSION_LIMIT) covers the error message and the usual response.

The other stop condition is quiescence. When the frontier goes empty, `prepare_next_tasks` returns nothing, `tick()` marks the run done, and `END` in `libs/langgraph/langgraph/constants.py` names that terminal path. `END` does not introduce a separate scheduler rule; it just names the stop that already happens when nothing remains due.

## Version flow

The same two maps drive every pass through the loop: planning compares them, execution produces writes, and `apply_writes` moves both maps forward for the next comparison.

```mermaid
flowchart LR
  CV[channel_versions store] --> Plan[prepare_next_tasks and _triggers]
  VS[versions_seen store] --> Plan
  Plan --> Run[execute due nodes]
  Run --> Writes[writes]
  Writes --> Apply[apply_writes]
  Apply --> Bump[channel_versions advances]
  Apply --> Seen[versions_seen catches up]
  Bump --> Plan
  Seen --> Plan
```

## Where to look in the code

- `libs/langgraph/langgraph/pregel/_algo.py` — `prepare_next_tasks`, `_triggers`, `apply_writes`
- `libs/langgraph/langgraph/pregel/_loop.py` — `tick`, `after_tick`, `_put_checkpoint`
- `libs/checkpoint/langgraph/checkpoint/base/__init__.py` — `Checkpoint`, `channel_versions`, `versions_seen`, `BaseCheckpointSaver.get_next_version`
- `libs/langgraph/langgraph/channels/base.py` — `consume`, `update`, `finish`
- `libs/langgraph/langgraph/pregel/_checkpoint.py` — `create_checkpoint`, `channels_from_checkpoint`
- `libs/langgraph/langgraph/errors.py` — `GraphRecursionError`
- `libs/langgraph/langgraph/constants.py` — `END`