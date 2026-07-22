# Fork Delta — EVR-RDY-Projects/PIDSMaker vs ubc-provenance/PIDSMaker

Verified 2026-07-22 after `git fetch upstream`. `main` = `5cff2c4`; `upstream/main` = `3260273`.

- Ahead: `git log --oneline upstream/main..main` → **12 commits**
- Behind: `git log --oneline main..upstream/main` → **3 commits**

## 1. The 12 ahead commits — all governance/CI, zero source/dependency/license change

Every ahead commit touches **only `.github/`**. Confirmed by `git log --stat upstream/main..main`
— no file outside `.github/` appears in any of the 12 diffstats. No `pidsmaker/**`, no `config/**`,
no `LICENSE`, no `Dockerfile`, no `pyproject.toml` change. The fork adds no code and no dependency.

| SHA | Subject | File(s) touched | What it does |
|-----|---------|-----------------|--------------|
| `7b7e970` | chore: add branch flow governance | `.github/workflows/branch-flow.yml` (+54) | CI gate: PRs to `main` must come from `develop` or `release/*`; PRs to `develop` from `feature/*`,`fix/*`,`docs/*`,`chore/*`,`test/*`,`refactor/*`,`dependabot/*`. |
| `0b495ce` | chore: add dynamic policy ci | `.github/workflows/policy-ci.yml` (+139) | Language-agnostic CI: detects repo shape (node/python/rust/go/docker/specweave), runs hygiene check + build/test/docker-smoke per detected stack. |
| `f37bfe7` | chore: add code ownership | `.github/CODEOWNERS` (+18) | Initial CODEOWNERS. |
| `985f39e` | chore: add dependabot config | `.github/dependabot.yml` (+12) | Dependabot for github-actions ecosystem. |
| `f914f95` | chore: update code owners | `.github/CODEOWNERS` (-18/+1) | Collapses ownership to `* @nybblez0x42697A`. |
| `a9ffae0` | chore: target dependabot at develop | `.github/dependabot.yml` (+13/-1) | `target-branch: develop`; groups minor/patch. |
| `7377882` | chore: harden policy ci yaml | `.github/workflows/policy-ci.yml` (+15/-4) | YAML/robustness hardening of policy-ci. |
| `b1fb01d` | chore: add yaml document marker | `.github/workflows/branch-flow.yml` (+1) | Adds `---` document start. |
| `72b6181` | chore: add dependabot yaml marker | `.github/dependabot.yml` (+1) | Adds `---` document start. |
| `05413f3` | chore: allow dependabot branch flow | `.github/workflows/branch-flow.yml` (+2/-2) | Permits `dependabot/*` heads through branch-flow. |
| `6459019` | chore: ignore npm major updates | `.github/dependabot.yml` (+4) | Ignore npm major bumps. |
| `5cff2c4` | chore: scope dependabot manifests | `.github/dependabot.yml` (-15) | Trims dependabot manifests (HEAD). |

**Assessment.** The fork's ahead-delta is house governance scaffolding only: branch-flow
enforcement, a generic policy-CI, CODEOWNERS, and Dependabot. It introduces **no IP, no
proprietary logic, no dependency, and no license change**. Novelty/publishing posture is
unaffected by these commits. See the NOTICE/license note in `upstream-orientation.md` §5.

### Governance side-effect worth flagging
`branch-flow.yml` (`.github/workflows/branch-flow.yml:31-40`) **rejects any PR to `main` whose head
is not `develop` or `release/*`.** A PR from `docs/brownfield-intake` → `main` will therefore
**fail the branch-flow check**. The governance-compliant path for this intake is
`docs/brownfield-intake` → `develop` → `main`. This is surfaced (not silently worked around) — the
target branch for the intake PR is a decision for the team lead.

## 2. The 3 behind commits — what upstream has that the fork lacks

`git log --stat main..upstream/main`:

| SHA | Subject | Files | Relevance to Rampart-PIDS |
|-----|---------|-------|---------------------------|
| `3260273` | Merge PR #47 from ubc-provenance/dev | (merge) | n/a — merge of the two below. |
| `0797215` | fix velox: removing `x_is_tuple` makes velox actually better in average | `config/default.yml` (+1), `config/velox.yml` (-1) | **Low.** A tuning/correctness fix to the **Velox** system config only. Rampart is **ORTHRUS-derived** (ADR-0008), not Velox, so this does not touch the Rampart detection path. Worth taking on sync for baseline fidelity; not blocking. |
| `1398038` | fix zenodo link | `README.md` (+1/-1) | **None (cosmetic).** DOI link fix in the README. |

**Behind-delta is entirely low/no relevance to the Rampart path** (config tuning for a non-adopted
system + a README link). No upstream **code** fix is being missed.

## 3. Sync assessment (drift policy input)

- **Conflict risk on sync: none mechanical.** The 12 ahead commits touch only `.github/**`; the 3
  behind touch only `config/*.yml` and `README.md`. The path sets are disjoint → a merge of
  `upstream/main` into `main` (or `develop`) applies cleanly with no code conflict.
- **The PIDS Program Plan already directs a sync.** `PIDS-Program-Plan.md:84-85`: *"the fork is 3
  commits behind upstream. Sync before Phase 0 work begins, so the reproduction baseline is
  measured against current upstream."* This intake confirms the sync is cheap and low-risk.
- **Drift-policy options** (for the team lead — options only, not a recommendation):
  1. **Sync-before-Phase-0** (as the plan states): merge `upstream/main` now; re-measure baseline.
  2. **Pin-and-defer:** freeze at current `main`; treat upstream as reference; sync only when a
     specific upstream fix is needed. Trades baseline currency for a stable target during surgery.
  3. **Track-continuously:** periodic upstream merges via a scheduled job. Highest currency,
     highest surface for surprise regressions mid-surgery.
