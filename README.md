# Moxon antenna kit
by OE1EBG

## Interactive calculator

[![Launch Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/ebirn/Moxon_OE1EBG/feature/interactive-moxon-notebook?labpath=notebooks/moxon_tuning_protocol.ipynb)

An interactive Jupyter notebook for Moxon rectangle dimensions and visualization is available at [`notebooks/interactive_moxon_calculator.ipynb`](notebooks/interactive_moxon_calculator.ipynb).

The notebook includes the theory note from [`notebooks/moxon_theory.md`](notebooks/moxon_theory.md), presets for 2 m and 70 cm, conductor diameter and velocity factor controls, dimension tables, and scaled geometry plots.

## Tuning guide

Structured build and tuning instructions are available in [`TUNING.md`](TUNING.md). The guide starts from the printed T-pieces, pipes, wire, and an antenna analyzer such as a NanoVNA or RigExpert Stick, then walks through iterative Moxon-specific tuning.

The tuning protocol notebook [`notebooks/moxon_tuning_protocol.ipynb`](notebooks/moxon_tuning_protocol.ipynb) provides a form for logging iterative measurements to CSV sessions under `notebooks/data/`. Notebook cell outputs are cleared by the repository hooks, so CSV files are the persistent tuning records and can be saved/restored independently.

## Project descriptor

[`project.yml`](project.yml) describes the project for its pages on [oe1ebg.at](https://oe1ebg.at/): the title and licence, which Markdown files become the theory and build pages, and display names (and optional Printables links) for the exports and bodies in [`.freecad_tools/config.yml`](.freecad_tools/config.yml). It is part of the release bundle, which the site builds from.
