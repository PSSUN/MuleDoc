# PrismSNV User Documentation (Sphinx + Read the Docs + Shibuya)

This repository contains the standalone PrismSNV user documentation project built with:

- Sphinx
- Read the Docs
- Shibuya theme

## Local Preview

```bash
python -m venv .venv
. .venv/Scripts/activate
pip install -r docs/requirements.txt
sphinx-build -b html docs docs/_build/html
```

After a successful build, open: `docs/_build/html/index.html`

If you changed `index.md` toctree structure (added/reordered pages), run a full forced rebuild to avoid stale sidebar navigation:

```bash
sphinx-build -E -a -b html docs docs/_build/html
```

## Read the Docs

The project root already includes `.readthedocs.yaml`, so it is ready for RTD builds.
