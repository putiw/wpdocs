# AI Agent Context — wpdocs

Give this file to any AI agent when you want it to document a workflow into this site.

---

## What is this repo?

A **MkDocs-based documentation site** (Material theme) containing personal reference documentation for neuroimaging research workflows. It is hosted on Read the Docs and can be served locally with `mkdocs serve` for live-reload development.

**Repository:** `https://github.com/putiw/wpdocs`
**Owner:** Puti (pw1246@nyu.edu) — neuroimaging researcher at NYU

## Purpose

This is a **"how I do things" knowledge base** — step-by-step procedures, code snippets, configuration files, and explanations for every stage of neuroimaging research. The goal is that future-me or any collaborator can reproduce any workflow without re-discovering it from scratch.

## Tech Stack

- **MkDocs** with **Material for MkDocs** theme
- Config: `mkdocs.yml` (nav structure, theme, extensions)
- Docs live in: `docs/` directory
- Markdown extensions enabled: admonitions, code highlighting with copy button, tabbed content, task lists, superfences
- Hosted on **Read the Docs** (`.readthedocs.yaml` in repo root)
- Python dependencies: `requirements.txt`

## Documentation Categories

Tabs follow the neuroimaging pipeline: acquire → manage → process → analyze → visualize.

| Section | Directory | Covers |
|---------|-----------|--------|
| Getting Started | `docs/getting-started/` | Workstation setup, machines, software installation |
| Experiments | `docs/experiments/` | fMRI QC stimulus tasks (Psychtoolbox), scanner iMac tips |
| Data Management | `docs/data-management/` | Data transfer, BIDS conversion, XNAT, organization |
| Preprocessing | `docs/preprocessing/` | fMRIPrep, FreeSurfer, MELD Graph, custom steps |
| Quality Control | `docs/quality-control/` | MRIQC (individual & group), visual QC |
| Analysis | `docs/analysis/` | GLM, RSA, ridge regression, encoding models, connectivity, searchlight, statistics |
| Visualization | `docs/visualization/` | Freeview, surface viz, Blender, figures, 3D brain printing |
| Workflows | `docs/workflows/` | Task-oriented "how do I..." entry points |
| Systems | `docs/systems/` | Access, addresses, mounted paths, and system ownership |
| Infrastructure | `docs/hpc/`, `docs/containers/`, `docs/scripting/` | HPC/Jubail, Docker/Singularity, bash/python scripting |
| Projects | `docs/projects/`, `docs/ccad/` | ARI, CCAD epilepsy (with sub-pages for VPN, pipeline, etc.) |
| Reference | `docs/reference/` | Cheatsheet, troubleshooting, glossary, external resources |

## How to Add Documentation

### Writing a new page within an existing category

1. Create a `.md` file in the appropriate `docs/<category>/` directory
2. Add it to the `nav:` section in `mkdocs.yml` under the correct category
3. Use the writing conventions below

### Creating a new category

1. Create `docs/<new-category>/index.md` with an overview
2. Add subcategory pages as separate `.md` files
3. Add the new section to `nav:` in `mkdocs.yml`

### Writing Conventions

- **Title:** Use a single `# Title` at the top of each page
- **Structure:** Use `## Section` headers to break up the procedure
- **Code blocks:** Always specify the language (```python, ```bash, ```yaml)
- **Admonitions:** Use MkDocs admonitions for callouts:
  - `!!! note "TODO"` — for sections that still need content
  - `!!! tip` — for helpful tips
  - `!!! warning` — for things that can go wrong
  - `!!! example` — for worked examples
- **Code annotations:** Use pymdownx code annotations where helpful
- **Tabs:** Use `=== "Tab Name"` for showing alternatives (e.g., Docker vs Singularity)
- **Prerequisites:** Start procedure pages with a "Prerequisites" section listing what's needed
- **Tone:** Direct, practical, second-person ("Run this command..."). Not academic — this is a personal reference, not a textbook.
- **Privacy:** For public/tracked docs, use de-identified placeholders such as
  `sub-XXXX` or `sub-MRNxxxx` unless Puti explicitly says the page is private.
  Do not include patient names, real MRNs, passwords, tokens, private keys, or
  `.env` values in tracked documentation.

### Example page structure

```markdown
# How to Run fMRIPrep on Jubail

## Prerequisites

- BIDS-validated dataset on `/scratch`
- Singularity image of fMRIPrep (see [Building Images](../containers/building-images.md))
- FreeSurfer license file

## Step 1: Prepare the SLURM script

...code and explanation...

## Step 2: Submit the job

...code and explanation...

## Common Issues

!!! warning "Out of memory"
    If fMRIPrep gets OOM-killed, increase `--mem` to 32G...

## Output Structure

...what to expect in the derivatives folder...
```

## Local Development

```bash
cd /path/to/wpdocs
pip install -r requirements.txt
mkdocs serve
# Site at http://127.0.0.1:8000 with live reload
```

## Big Picture

Puti works across the full neuroimaging pipeline — from data acquisition and management (XNAT), through preprocessing (fMRIPrep, FreeSurfer), analysis (GLM, RSA, ridge regression, encoding models), to visualization (Freeview, Blender, 3D printing) and HPC orchestration (Jubail, Docker/Singularity). This documentation should eventually cover **every repeatable procedure** in that workflow so nothing has to be re-figured-out.

## Instructions for AI Agents

When Puti asks you to document something:

1. **Ask which category** it belongs to (or determine from context)
2. **Write the full procedure** as a markdown file following the conventions above
3. **Save it** to the correct `docs/<category>/` directory
4. **Update `mkdocs.yml`** to add it to the nav
5. **Cross-link** to related pages where relevant
6. **Include runnable code** — not pseudocode. Real commands, real paths (use placeholders like `<subject_id>` where needed)
7. **Document failure modes** — what can go wrong and how to fix it
8. Pages marked with `!!! note "TODO"` are stubs waiting to be filled in — prioritize those when relevant
9. **Keep public docs de-identified** — private identifiers and secrets belong in
   ignored local notes under `secrets/`, not in tracked Markdown.
