# MICrONs Tutorial

Build for the tutorial.microns-explorer.org documentation. Website built with [Quarto](https://quarto.org/).

## Development environment

This project uses [uv](https://docs.astral.sh/uv/) for Python environment and package management. Dependencies are declared in `pyproject.toml` and pinned in `uv.lock`.

To create a local virtual environment (`.venv`, git-ignored) with all required packages:

```bash
uv sync
```

Render the site locally (executing the notebooks) with:

```bash
uv run quarto render tutorial_book
```

The core analysis packages include `caveclient`, `cloud-volume`, `meshparty`, `pcg-skel`, `imageryclient`, `skeleton-plot`, `nglui`, and `standard-transform`; see `pyproject.toml` for the full list.

## Issues
We welcome bug reports and questions. Please post an informative issue on the GitHub issue tracker.

## Other resources
See:
* [microns-explorer.org](https://www.microns-explorer.org/) for overview of the project
* [Cubic millimeter](https://www.microns-explorer.org/cortical-mm3) for release details on the data resource


Contributions by: Bethanny Danskin, Ben Pedigo, Casey Schneider-Mizell, Forrest Collman, Leila Elabbady, Sven Dorkenwald
  
