# AGENTS.md

This file provides guidance to AI coding agents when working with this repository.

## What this repo is

The OE1EBG Moxon antenna kit (2 m / 70 cm, for SOTA): a FreeCAD model of the 3D-printed parts (`Moxon_OE1EBG.FCStd`), Jupyter notebooks (interactive dimension calculator, tuning log), and the written guides (`TUNING.md`, `notebooks/moxon_theory.md`). Licence CC BY-SA 4.0. The website [oe1ebg.at](https://oe1ebg.at/) builds its Moxon pages from a pinned **release** of this repo (see "Release bundle" below), so what a release contains is an interface, not an implementation detail.

## Commands

Two separate environments:

- **Notebooks / Python** (`pyproject.toml`, `uv.lock`): `uv sync`, then `uv run ...`. `just notebook [path]` opens JupyterLab (default `notebooks/moxon_tuning_protocol.ipynb`).
- **FreeCAD exports** (`.venv` made by `just init`, which installs `freecad-tools` pinned in the `Justfile`; needs FreeCAD installed): `just export-list`, `just export <name>`, `just export-all`. Output goes to `prints/` (3MF plates, STLs) and `docs/` (drawings, BOM CSVs), both git-ignored. The `freecad_tools_version` in the `Justfile`, `pyproject.toml` and `.pre-commit-config.yaml` should move together.
- **Static dashboard** (what the website embeds): `just voici` builds `notebooks/` into `_output/`, `just voici-serve` serves it. The kernel is pinned to `jupyterlite-pyodide-kernel==0.7.0` to match Voici's bundled JS; a newer kernel breaks the app shell with webpack "Unsatisfied version" errors.
- **Run a notebook headless** (what CI does): `uv run jupyter nbconvert --to notebook --execute --ExecutePreprocessor.timeout=60 --output-dir "$(mktemp -d)" notebooks/<name>.ipynb`.
- **Hooks**: `pre-commit install` (commit and push types). They clear notebook outputs and execute each notebook; `freecad-export-manual` is a manual-stage hook. Notebook outputs are never committed, and `notebooks/data/*.csv` are the persistent tuning records.

There is no test suite; the notebook dry run is the check.

## Notebooks

- `interactive_moxon_calculator.ipynb` holds the model (`moxon_dimensions`, NEC deck export) and the widget UI; `moxon_tuning_protocol.ipynb` logs measurements to CSV sessions in `notebooks/data/`. The calculator notebook is also rendered by Voici/JupyterLite in the browser (Pyodide), so code must work there too: the `%pip install` cell is for that, and download links must work under static rendering (see recent commits on the NEC export link).
- Cells tagged `hide-input` are hidden in the dashboard; the static reference cells are hidden in Voici as well.
- `notebooks/moxon_theory.md` and `TUNING.md` are plain Markdown that the website stages as pages: keep links relative and resolvable inside the repo, and images under `img/`.

## CAD export config

`.freecad_tools/config.yml` defines the exports (print plates). Each lists bodies by FreeCAD **label** with position/rotation, the 3MF `output`, `keep_stl` + `stl_output_dir` (STLs are named `<stl_output_dir>/<export>_<body label>.stl`, spaces become `_`, repeated bodies get `_2`, ...), and a BOM. The same part has different labels in different exports (`Feed001`/`Feed`, `Cover001`/`Cover`); `project.yml` maps each label to a display name.

## Release bundle and the website contract

- `project.yml` is the project descriptor read by `oe1ebg/oe1ebg_at` (`scripts/stage_projects.py`): title, licence, the Markdown sources for the theory/build pages, and display names (optional `printables:` links) per export and body label. The site merges it under its own `projects/moxon.yml`, where the site's values win and the release pin lives. A new body label or export in the config needs a name here.
- A release tarball must contain `dist/release-metadata.txt` (`release_tag=`, `commit=`), `LICENSE.txt`, `project.yml`, `.freecad_tools/config.yml`, the Markdown, `notebooks/`, `img/`, and the generated `prints/` and `docs/`. These come from `bundle_paths` / `generated_paths` in `.github/workflows/tag.yml`; add anything the site should see there.
- Try changes against the site before releasing: in `oe1ebg_at`, `just stage-projects --local moxon=../SOTA_Moxon` (uses this checkout, and `prints/` after `just export-all`), then `just preview`.
- Releasing: bump `version` in `pyproject.toml` and `uv.lock`, merge to `main`, push an annotated `vX.Y.Z` tag at the verified `main` commit (CI must be green); `tag.yml` builds the release via `ebirn/freecad_tools`' reusable workflow (`@main`). `nightly.yml` publishes a rolling nightly with a smaller bundle. Only tag when asked.

## CI (`.github/workflows/commit.yml`)

- A `changes` job (`dorny/paths-filter`) gates two jobs: the 3MF dry-run (CAD files, `.freecad_tools/`, `INSTRUCTIONS.md`) and the notebook dry-run (`notebooks/`, `TUNING.md`, `pyproject.toml`, `uv.lock`). `main` runs everything. Add new inputs to the matching filter.
- Notebook execution stalls intermittently (roughly 1 in 5 runs, a different cell each time, kernel idle) until the cell timeout expires. That is why CI uses a 60 s cell timeout with up to 4 attempts per notebook rather than a long timeout; no version pin of ipykernel/pyzmq/Python was found to fix it. Don't raise the timeout, and don't read a single stall as a notebook bug. The pre-commit hook has no retry, so a rerun is normal there too.
- Actions are pinned to full commit SHAs with `# vX.Y.Z` comments.
