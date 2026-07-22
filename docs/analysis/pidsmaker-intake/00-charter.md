# PIDSMaker Fork-Delta Intake — Charter

- **Repo:** `D:\repos\PIDSMaker` — fork `git@github.com:EVR-RDY-Projects/PIDSMaker.git` (origin)
- **Upstream:** `https://github.com/ubc-provenance/PIDSMaker.git` (remote `upstream`, read-only reference)
- **Fork HEAD (main):** `5cff2c4` `chore: scope dependabot manifests`
- **Delta at intake:** 12 commits ahead of `upstream/main`, 3 behind (verified 2026-07-22)
- **Branch for this work:** `docs/brownfield-intake` (cut from `main`)
- **Change surface:** new files under `docs/analysis/pidsmaker-intake/` only. Nothing else modified.

## Scope — FORK-DELTA intake (not a full ADOPT)

This is a **light, delta-focused orientation**, not an exhaustive extraction of the PIDSMaker
codebase. The goal is to answer four questions for the Rampart-PIDS program:

1. **What has the fork changed** relative to upstream (12 ahead), and **what upstream work is the
   fork missing** (3 behind), with relevance to Rampart-PIDS. → `fork-delta.md`
2. **What is PIDSMaker** and its top-level architecture, models/pipelines, the meaning of
   "per-site Word2Vec models" in this codebase, and license/dependency posture. →
   `upstream-orientation.md`
3. **What surface would Rampart's PIDS stage consume** from this framework, per the TAPS corpus. →
   `taps-integration-surface.md`
4. **What are the risks** — supply-chain posture of the fork, behind-upstream drift policy,
   PRC-origin dependency check. → distributed across the above, summarized in `INDEX.md`.

## Explicitly out of scope

- No source-code modification, refactor, or "surgery" (that is Program-Plan Phase 2, WS-2.2).
- No dependency upgrades, no upstream sync (only *assessed*, not performed).
- No running of the framework or datasets.
- No full per-module extraction — architecture is described at directory/entry-point altitude.

## Method / evidence rules

Every claim in these artifacts is backed by `path:line`, a commit SHA, or command output taken
from the real repo on 2026-07-22. Where a corpus document's cited code pointer no longer matches
the current fork, that drift is reported as a finding rather than repeated as fact.

## Constraints observed

- Work only on `docs/brownfield-intake`; push to **origin** (the fork), never to `upstream`.
- Commits carry no AI-attribution trailers; PR body carries no attribution footer.
- Commit identity is `Timothy DeBerry <craig.deberry1@gmail.com>` (the repo's configured, intentional identity).
