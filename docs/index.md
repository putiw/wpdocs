# Neuroimaging Workflows & Procedures

Personal reference documentation for neuroimaging research — everything from raw data to publication-ready figures.

## Agent Quick Start

If you are a brand-new AI agent, start with the page the user gave you. If they
only pointed you at this repo, use this map:

| Need | Start here |
|---|---|
| Project runbook, current status, deploy notes, feature history | [Projects](projects/index.md) |
| Machine paths, systems, access conventions, local laptop coordination | [Systems](systems/index.md) |
| Local setup, software, machines, fileshares | [Getting Started](getting-started/index.md) |
| Reusable task procedures | [Workflows](workflows/index.md) and the pipeline sections below |
| Short commands, troubleshooting, glossary | [Reference](reference/index.md) |

For local dev servers, ports are shared across the whole laptop. Check
[Local Dev Server Ports](systems/local-dev-servers.md) before starting one.

## What is this?

A living knowledge base of **how I do things** — step-by-step procedures, code snippets, system context, and project records for neuroimaging research. Built so future-me, collaborators, and AI agents can reproduce workflows without re-discovering them from scratch.

## Why this exists

This site has four jobs:

1. Remember workflows I use often but do not want to re-figure out every time.
2. Give AI coding/research agents enough context to work safely and effectively.
3. Provide shareable instructions for colleagues who ask how to do specific tasks.
4. Preserve project-level evidence of systems, tools, and workflows I have built or supported.

## Quick Navigation

| Section | What's in it |
|---------|-------------|
| [Getting Started](getting-started/index.md) | Local setup, machines, fileshares, software |
| [Experiments](experiments/index.md) | fMRI QC tasks (checkerboard, motion localizer, motor), Psychtoolbox tips |
| [Data Management](data-management/index.md) | Data transfer, BIDS conversion, XNAT |
| [Preprocessing](preprocessing/index.md) | fMRIPrep, FreeSurfer, MELD Graph, custom steps |
| [Quality Control](quality-control/index.md) | MRIQC, visual QC checklists |
| [Analysis](analysis/index.md) | GLM, RSA, ridge regression, encoding models, connectivity, searchlight |
| [Visualization](visualization/index.md) | Freeview, surface viz, Blender, figures, 3D printing |
| [Infrastructure](hpc/index.md) | HPC / Jubail, Docker / Singularity, scripting |
| [Systems](systems/index.md) | Machine paths, access conventions, local laptop coordination |
| [Projects](projects/index.md) | ARI, CCAD epilepsy, myPhysio — project-specific records and workflows |
| [Reference](reference/index.md) | Cheatsheet, troubleshooting, glossary |

## How to use locally

```bash
# Install dependencies
pip install -r requirements.txt

# Serve with live reload
mkdocs serve

# Build static site
mkdocs build
```

The site will be at `http://127.0.0.1:8000` and auto-reloads on any file change.

!!! tip "Contributing new docs"
    See [AI_CONTEXT.md](https://github.com/putiw/wpdocs/blob/main/AI_CONTEXT.md) for a prompt you can give any AI agent to help you document a new workflow into this site.
