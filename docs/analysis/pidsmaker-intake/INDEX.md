# PIDSMaker Fork-Delta Intake — Index

**Scope: light FORK-DELTA intake, not a full ADOPT.** This set orients the Rampart-PIDS program on
the fork's delta from upstream, the framework baseline, the Rampart consumption surface, and risks.
It does **not** extract every module, modify any source, or perform an upstream sync — those are
program-plan work items, only *assessed* here. Every claim is cited to `path:line` or a commit SHA,
verified against the fork at `5cff2c4` on 2026-07-22.

## Artifacts
| File | Contents |
|---|---|
| `00-charter.md` | Scope, boundaries, method, constraints. |
| `fork-delta.md` | All 12 ahead commits (governance/CI only) + 3 behind commits (low relevance) + sync/drift-policy options. |
| `upstream-orientation.md` | What PIDSMaker is, architecture + entry point, models/pipeline, "per-site Word2Vec" mapping, Apache-2.0/NOTICE + dependency & supply-chain posture. |
| `taps-integration-surface.md` | Rampart FR → fork-code map; featurization/vocabulary surface; STALE-landmine finding; GPL-isolation rule. |

## Key findings (verdict first)

1. **Fork ahead-delta = 12 commits, 100% governance/CI** (`.github/**` only): branch-flow,
   policy-CI, CODEOWNERS, Dependabot. **No source, dependency, or license change.** IP/novelty
   posture unaffected. (`fork-delta.md` §1)

2. **Fork behind-delta = 3 commits, low/no relevance to Rampart** (a Velox config tuning fix +
   a README DOI link). No upstream **code** fix is missed on the ORTHRUS path.
   Sync is **mechanically conflict-free** (disjoint paths) and the Program Plan already directs it.
   (`fork-delta.md` §2-3)

3. **License clean:** Apache-2.0 intact; upstream ships **no NOTICE**, so the fork has **no NOTICE
   obligation** and is compliant. (`upstream-orientation.md` §5)

4. **Supply chain: no PRC-origin dependency or data source found.** All package indexes/hosts are
   PyPI / pytorch.org / pyg / Anaconda / NodeSource / GitHub; datasets are US DARPA-origin via
   Google Drive. Minor upstream-Dockerfile hygiene notes: NodeSource `[trusted=yes]` (GPG off),
   unpinned `gosu` binary, Anaconda `defaults`-channel commercial-ToS question. (`upstream-orientation.md` §5)

5. **ACTION — the Program Plan's "five landmine" code pointers are STALE.** `build_orthrus_graphs.py`
   and `config.py:622-652` no longer exist; the edge-fusion and schema logic moved to
   `build_default_graphs.py:179` and `utils/dataset_utils.py:301/326`. The *diagnoses* hold, but
   the line references must be **re-anchored before WS-2.2 surgery**. (`taps-integration-surface.md` §4)

6. **Governance side-effect:** the fork's own `branch-flow.yml` **rejects PRs to `main` not from
   `develop`/`release/*`**, so this intake's `docs/brownfield-intake → main` PR will fail that
   check. Compliant path is via `develop`. Surfaced for the team lead's routing decision.
   (`fork-delta.md` §1)

## Status
Intake complete. No source modified. Only files under `docs/analysis/pidsmaker-intake/` added.
