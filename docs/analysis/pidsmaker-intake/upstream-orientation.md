# Upstream Orientation — PIDSMaker (baseline, light intake)

Altitude: directories, entry points, and the concepts Rampart-PIDS depends on. Not a full
per-module extraction. All pointers verified against the fork tree at `5cff2c4` on 2026-07-22.

## 1. What PIDSMaker is

PIDSMaker (`pyproject.toml:6-8`, version **4.0.0**, *"Intrusion detection framework based on deep
learning"*) is a research framework from UBC's provenance group for **building and benchmarking
provenance-based intrusion detection systems (PIDSs)** using deep-learning / GNN architectures.
`README.md`: *"The first framework designed to build and experiment with provenance-based intrusion
detection systems (PIDSs) using deep learning architectures … a single codebase to run most recent
state-of-the-art systems and easily customize them."*

It re-implements 8 published PIDSs behind one pipeline and CLI, and ships pre-processed provenance
datasets (DARPA TC E3/E5, DARPA OpTC, ATLAS/Carbanak EDR) for reproducible evaluation.

**Supported systems** (`README.md` table): Velox, **Orthrus**, R-Caid, Flash, Kairos, Magic,
NodLink, ThreaTrace. Each is defined by a YAML config in `config/` (`config/orthrus.yml`,
`config/kairos.yml`, `config/velox.yml`, …). Rampart adopts the **Orthrus** system (per
ADR-0008 / PIDS Program Plan).

## 2. Top-level architecture

```
pidsmaker/
├── main.py                 # THE single entry point (main.py:1-10 docstring)
├── config/
│   ├── config.py           # argument schema for all YAML options (1126 lines)
│   └── pipeline.py         # pipeline wiring / validation
├── preprocessing/          # stage 1-2: parse provenance + build graphs
│   ├── build_graph_methods/ (build_default_graphs.py, build_magic_graphs.py)
│   └── transformation_methods/
├── featurization/          # stage 3: text-embedding featurization
│   ├── featurization_methods/ (word2vec, doc2vec, fasttext, flash, trw, alacarte)
│   └── feat_inference_methods/
├── encoders/  decoders/  objectives/     # stage 5 GNN model pieces
├── detection/
│   ├── training_methods/                 # stage 5 training loop
│   └── evaluation_methods/               # stage 6 metrics/plots (node/edge eval)
├── triage/tracing_methods/               # stage 7 attack tracing
├── tasks/                  # per-stage orchestration modules (see pipeline below)
├── experiments/            # tuning (W&B sweeps) + uncertainty quantification
└── utils/                  # dataset_utils.py (schema), data_utils, utils
```

Other top-level: `config/` (per-system YAML), `Ground_Truth/`, `dataset_preprocessing/`,
`docs/` (mkdocs site), `Dockerfile` + `compose-*.yml` (Postgres + `pids` container),
`download_datasets.sh`, `entrypoint.sh`, `tests/`.

### Entry point and pipeline stages
`pidsmaker/main.py` is the *only* entry point (`docs/docs/pipeline.md:14`,
`main.py` docstring). Invocation: `python pidsmaker/main.py SYSTEM DATASET [--overrides]`
(`README.md` "Basic usage"). Under the hood it runs an **8-task** sequential pipeline
(`main.py:8-9`): `construction → transformation → featurization → feat_inference → batching →
training → evaluation → triage`, orchestrated by the `pidsmaker/tasks/*.py` modules
(`docs/docs/pipeline.md:16-27`). Each task is content-hashed on its args; outputs are cached by
hash so unchanged stages are skipped on re-run (`pipeline.md:29-37`). Featurization dispatch:
`pidsmaker/tasks/featurization.py:17` reads `cfg.featurization.used_method` and dispatches (e.g.
`featurization.py:27-28` → `featurization_word2vec.main(cfg)`).

## 3. Models / pipelines that exist

- **8 PIDS "systems"**, each a YAML in `config/` inheriting from `config/default.yml`; a run
  selects one system + one dataset. Systems differ by featurization method, encoder/decoder,
  objective, and evaluation/threshold method — all swappable via CLI or YAML (`README.md`
  "Customize existing systems").
- **Featurization methods** (`pidsmaker/featurization/featurization_methods/`):
  `featurization_word2vec.py`, `featurization_doc2vec.py`, `featurization_fasttext.py`,
  `featurization_flash.py`, `featurization_trw.py` (temporal random walk), `featurization_alacarte.py`.
- **Tuning + uncertainty** experiment harnesses under `pidsmaker/experiments/` (W&B grid sweeps;
  MC-dropout / deep-ensemble / hyperparameter-sensitivity — `main.py:3-6`).

## 4. "Per-site Word2Vec models" — what it maps to here

A **"site"** in this codebase = a **dataset**, i.e. one DARPA TC / OpTC host environment
(`CADETS_E3`, `THEIA_E3`, `CLEARSCOPE_E3`, `FIVEDIRECTIONS_E3`, `TRACE_E3`, the `_E5` variants,
`optc_h051/h201/h501`, `ATLASV2_EDR`, `CARBANAKV2_EDR` — `README.md` "Supported Datasets";
`download_datasets.sh:17-32`). A run is bound to exactly one dataset: `python main.py SYSTEM DATASET`.

**"Per-site Word2Vec"** therefore maps to: the **featurization stage (stage 3) trains a Word2Vec
embedding model on the node-label text corpus of the one dataset being run**, and persists it as a
per-run artifact keyed by the task hash. Concretely, Word2Vec is trained via `gensim.models.Word2Vec`:
- `featurization/featurization_methods/featurization_flash.py:100-107` — trains and saves
  `word2vec_model_final.model`.
- `featurization/featurization_methods/featurization_trw.py:35-72` — `train_word2vec()` saves
  `trw_word2vec.model`; `:75-80` `update_word2vec()` loads/continues it.
- `featurization/featurization_methods/featurization_alacarte.py:16,487,501` — trains or
  `Word2Vec.load()`s a model for À-la-carte embedding inference **without retraining** (`:5`).
- Config schema: `pidsmaker/config/config.py:542-554` (`word2vec` args: `alpha`, `window_size`,
  `min_count`, `use_skip_gram`, `epochs`, `negative`, `decline_rate`, …); defaults in
  `config/default.yml:35-43`; Orthrus selects it at `config/orthrus.yml:23-24`
  (`used_method: word2vec`).

This is exactly the property the PIDS Program Plan relies on: *"Per-site training is mandatory.
Word2Vec vocabulary is site-specific"* (`PIDS-Program-Plan.md:184-185`). Each deployment/site
trains its own vocabulary and threshold on its own benign baseline; there is no shared cross-site
model. The graph **schema** (node/edge types) that feeds featurization now lives in
`pidsmaker/utils/dataset_utils.py` (`ntype2id` dict at `:301`, `get_rel2id(cfg)` at `:326`) — see
`taps-integration-surface.md` for the drift note vs the Program Plan's stale pointer.

## 5. License and dependency stack

### License — Apache-2.0, intact; no NOTICE required
`LICENSE` is the full **Apache License 2.0** text (`LICENSE:1-3`). Upstream has **no `NOTICE`
file** (`git ls-tree -r upstream/main | grep -i notice` → none), and the fork adds none. Apache-2.0
§4(d) requires propagating a NOTICE **only if one exists**; since upstream ships none, the fork has
**no NOTICE obligation** and is compliant. The 12 fork commits do not touch `LICENSE` (see
`fork-delta.md`). `pyproject.toml` does not declare a license classifier; the authoritative grant
is the `LICENSE` file.

> Downstream note for any redistribution of a *modified* fork: Apache-2.0 §4(b) requires marking
> changed files as modified. The current fork changes only `.github/**` (non-source); if/when
> Rampart modifies `pidsmaker/**`, add change-notices to those files. Not an issue today.

### Dependencies — pinned in the Dockerfile (no `requirements.txt`/`environment.yml`)
There is **no `requirements.txt`, `environment.yml`, or `setup.py`**; dependencies are declared and
version-pinned inline in `Dockerfile`. Core stack: Python 3.9 (conda), `torch==1.13.1+cu117`
(+torchvision/torchaudio), `torch_geometric==2.5.3` + PyG extensions, `gensim==4.3.1` (Word2Vec),
`scikit-learn==1.2.0`, `networkx==2.8.7`, `numpy==1.26.4`, `scipy==1.10.1`, `pandas==2.2.2`,
`igraph==0.11.5`, `nltk==3.8.1`, `wandb==0.24.1`, `psycopg2` (Postgres backend), `graphviz`,
`gdown==5.2.0`, plus dev tools (`pytest`, `pre-commit`, `mkdocs-material`).

### Supply-chain sources — all Western/standard; no PRC-origin dependency found
Checked `Dockerfile`, `download_datasets.sh`, and configs for package indexes and download hosts:
- PyPI (default `pip`), `download.pytorch.org/whl/cu117`, `data.pyg.org/whl`,
  `repo.anaconda.com` (Anaconda3-2023.03), `deb.nodesource.com` (Node 20, for Claude Code),
  `github.com/tianon/gosu` (privilege-drop helper). **No PRC-hosted mirror, no alternate
  `index-url`, no non-Western package source** appears anywhere.
- Datasets download from Google Drive (`www.googleapis.com/drive/v3`, `download_datasets.sh:41-66`)
  and are **US DARPA-origin** corpora (Transparent Computing E3/E5, OpTC) plus ATLAS/Carbanak.
  Requires a caller-supplied Google OAuth `ACCESS_TOKEN`. No PRC data source.

**Minor supply-chain observations (upstream Dockerfile choices, not fork-introduced):**
- NodeSource apt repo is added with `deb [trusted=yes] …` (`Dockerfile`), which **disables APT GPG
  signature verification** for that source.
- `gosu` is fetched via `wget` from GitHub releases and smoke-tested (`gosu --version`) but **not
  SHA256-pinned**.
- The base image uses the **Anaconda `defaults` channel** (`conda install`/`conda create`). For a
  *commercial* deployment, Anaconda Inc.'s current ToS on the `defaults` channel can carry
  licensing/paid-use implications — a legal-review item, not a code issue. (Flagged; not verified
  against current Anaconda ToS text here — "I have not verified this" against the live ToS.)

None of the above is PRC-related; all are standard hardening/licensing hygiene notes for the
eventual production build.
