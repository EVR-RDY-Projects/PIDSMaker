# TAPS / Rampart-PIDS Integration Surface

What Rampart's PIDS stage would call/consume from this framework, mapped from the TAPS corpus to
the actual fork code. Corpus paths are under `D:\repos\evrrdy-mcp\corpus\docs\products\taps\`.

> **Program status (do not overstate maturity).** Rampart has **zero EVR:RDY PIDS engine code**.
> `PIDS-Program-Plan.md:78` quotes Rampart ADD §15: *"The repository does not contain … PIDS
> engine modules with per-site bundle support."* The fork itself is *"**Exists**, never run — zero
> containers ever started"* (`PIDS-Program-Plan.md:75`). This document describes the **intended**
> consumption surface, not a working integration.

## 1. What Rampart is, relative to PIDSMaker

Per `Rampart/ADD.md:488` (ADR-0008) — *"Rampart Is the PIDS and Normalization Layer."* Rampart
**owns** the PIDS pipeline; PIDSMaker (Orthrus path) is the **engine it derives from**. Rampart's
event-driven pipeline (`ADD.md:363`, `:219`):

`Receiver → Validator → EventStore → Graph → PIDS → Alerts → ASIM → Output → SIEM`

The **PIDS Engine** node (`ADD.md:205,232,250`) is the box that maps onto PIDSMaker's
graph-construction → featurization → GNN training/inference → evaluation stages.

## 2. The consumption surface (Rampart FR → PIDSMaker code)

| Rampart concept (corpus) | Maps to in the fork | Notes |
|---|---|---|
| **Per-site PIDS bundles** — signed, versioned inference artifacts (`ADD.md:106,314`; `RAMPART-FR-016/018`) | Per-run outputs of the pipeline (trained Word2Vec model + GNN weights + threshold), currently cached by task-hash on disk (`docs/docs/pipeline.md:29-37`) | Rampart must add **signing + versioned bundle packaging + rotation** ("three most recent active per site, plus pinned", `ADD.md:314`). PIDSMaker has no bundle-signing today. |
| **Per-site partitioning** (`ADD.md:365`; SP-02 `:502`) | One run == one dataset/"site" (`main.py SYSTEM DATASET`); per-site Word2Vec vocab (`featurization_*`; see `upstream-orientation.md` §4) | The framework is *already* per-site by construction — no cross-site model exists. Aligns with `RAMPART-FR-016`. |
| **Graph constructor** (`ADD.md:110,636`) | `pidsmaker/preprocessing/build_graph_methods/build_default_graphs.py` (`gen_edge_fused_tw` at `:179`) | Rampart's ingest must feed provenance in the shape this stage expects (Postgres `event_table`, see `build_default_graphs.py:250-260`). |
| **PIDS inference / scored subgraph** (`ADD.md:260-261`) | `pidsmaker/tasks/training.py` (train+inference loop) + `detection/evaluation_methods/` (scoring/threshold) | ORTHRUS's headline is **attribution** (alert traceable to processes/files/flows), a Phase-1 DoD (`PIDS-Program-Plan.md`). |
| **Alert generator** combining PIDS + signature (`ADD.md:234`) | Downstream of PIDSMaker; **not in the fork** | Rampart-owned; PIDSMaker supplies the scored subgraph only. |
| **Replay tooling** (`ADD.md:110`; `RAMPART-FR-035/037`) | Reuses graph constructor + engine on archived bundles | Phase-2 testbed work. |

## 3. Featurization / vocabulary surface (the load-bearing detail)

Rampart's per-site model integrity (`RAMPART-NFR-014`, bundle rotation) rests on the **per-site
Word2Vec vocabulary** being site-specific and stable. In the fork this is the featurization stage
(`upstream-orientation.md` §4): `gensim` Word2Vec trained per dataset, schema from
`pidsmaker/utils/dataset_utils.py` (`ntype2id` `:301`, `get_rel2id()` `:326`). The
`EVR:RDY PIDS Researcher` role owns *"vocabulary stability"* and *"bundle versioning"*
(`ADD.md:81`) — both are properties of this stage.

## 4. Finding: the Program Plan's "five landmines" code pointers are STALE vs the current fork

`PIDS-Program-Plan.md:243-256` lists five *"documented, line-precise, settled"* fork-surgery
targets. Two of the five cite files/lines that **no longer exist** in the current fork (`5cff2c4`).
Upstream refactored since the plan was written (framework at v4.0.0). Verified 2026-07-22:

| Landmine (plan) | Plan's pointer | Current reality in fork | Status |
|---|---|---|---|
| #1 Edge collapsing keys on `(src,dst,op)` | `build_orthrus_graphs.py:120-140` | **`build_orthrus_graphs.py` does not exist.** Edge fusion is now `gen_edge_fused_tw()` in `pidsmaker/preprocessing/build_graph_methods/build_default_graphs.py:179`; per-event tuple carries `operation` + `object_id`-like fields at `:266-289`. | **Pointer stale — relocate before surgery.** |
| #5 Schema centralized (`rel2id`, `ntype2id`) | `config.py:622-652` | **Not in `config.py`** (1126 lines; no `rel2id`/`ntype2id` there). Schema now: `ntype2id` dict at `pidsmaker/utils/dataset_utils.py:301`; `get_rel2id(cfg)` at `:326`; consumed via `get_rel2id()` in `build_default_graphs.py:199`, `mimicry.py:151`, `feat_inference.py:106`. | **Pointer stale — relocate before surgery.** |
| #2 Time-window aggregation (15 min default) | (behavioral, no line) | Time-windowing present in `build_default_graphs.py` (`gen_edge_fused_tw`, `:227-236`). | Concept intact; re-verify default. |
| #3 Bucketize high-cardinality fields before Word2Vec | (behavioral) | Featurization tokenization in `featurization_methods/` (see §3). | Concept intact. |
| #4 No upstream tests | "bring our own" | `tests/` dir exists but is the fork/upstream baseline; treat as Phase-0 deliverable. | Re-confirm coverage. |

**Why this matters:** the plan says *"Already diagnosed to the line number … Do not re-discover
these"* (`:126,243`) and budgets WS-2.2 at ~2× leverage **on the assumption the pointers are
exact**. They are not, after the upstream refactor. The *diagnoses* (edge-key semantics, schema
cascade, vocab explosion) remain valid; the **line references must be re-anchored** to
`build_default_graphs.py` and `dataset_utils.py` before surgery, or the "expensive thinking already
paid for" is partially invalidated. This is the single most actionable output of the intake.

## 5. GPL isolation rule (carry forward)
`PIDS-Program-Plan.md:269-271` (ADR-001): *"No GPL library is linked into the PIDSMaker fork."* The
current dependency set (`upstream-orientation.md` §5) is Apache/BSD/MIT-class (torch, gensim,
networkx, sklearn, pandas). No GPL is linked today. Phase-2 OT tooling (OpenPLC, libiec61850,
cpppo, lib60870) must remain **standalone processes**, never imported.
