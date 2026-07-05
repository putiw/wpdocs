# wpdocs

Personal reference documentation for systems, projects, workflows, and research
operations. This repo is meant to help future humans and AI agents find the
right operational context quickly instead of re-deriving it.

## Fast Start For Agents

1. If the user gives you a specific page, read that page first and treat it as
   the runbook for that context.
2. If you are working on a named project, start in `docs/projects/`.
3. If you need machine paths, server ownership, ports, access conventions, or
   local coordination rules, start in `docs/systems/`.
4. If you need a reusable procedure, look in the workflow or pipeline section:
   `docs/workflows/`, `docs/data-management/`, `docs/preprocessing/`,
   `docs/analysis/`, `docs/visualization/`, `docs/hpc/`, `docs/containers/`,
   or `docs/scripting/`.
5. Keep secrets, tokens, keys, `.env` values, patient identifiers, and live
   temporary state out of tracked docs. Local-only material belongs under
   `secrets/`, which is ignored by git.

For documentation-writing conventions, read `AI_CONTEXT.md`.

## Where To Look

| Need | Start here |
|---|---|
| Project-specific runbooks and history | `docs/projects/` |
| Server paths, local laptop conventions, ports, access notes | `docs/systems/` |
| Local setup and software installation | `docs/getting-started/` |
| XNAT, BIDS, transfers, archives | `docs/data-management/` |
| fMRIPrep, FreeSurfer, MRIQC, custom preprocessing | `docs/preprocessing/`, `docs/quality-control/` |
| GLM, RSA, encoding models, stats, searchlight | `docs/analysis/` |
| Freeview, surface visualization, Blender, figures | `docs/visualization/` |
| Jubail, Docker, Singularity, scripts | `docs/hpc/`, `docs/containers/`, `docs/scripting/` |
| Short commands and troubleshooting | `docs/reference/` |

## Important Local Conventions

- Local dev-server ports are laptop-wide. Before starting a server, check
  `docs/systems/local-dev-servers.md` and the ignored live registry at
  `secrets/local-dev-ports.md`.
- Project pages can point to ignored secret bundles, but tracked docs must not
  contain the secret values themselves.
- Use absolute paths in operational runbooks when the path is machine-specific
  and important.

## Current High-Value Pages

| Page | Why it matters |
|---|---|
| `docs/projects/myphysio.md` | Full myPhysio agent runbook: repo, deploys, Supabase, server access, known gotchas. |
| `docs/systems/local-dev-servers.md` | Laptop-wide dev-server port coordination for parallel agents/projects. |
| `docs/projects/ari.md` | ARI archive and Box/XNAT support notes. |
| `docs/projects/ccad-epilepsy.md` | CCAD epilepsy project overview and links to access/pipeline pages. |

## Run The Docs Locally

```bash
cd /Users/pw1246/Documents/GitHub/wpdocs
pip install -r requirements.txt
mkdocs serve
```

The local site is served at `http://127.0.0.1:8000`.

To validate a build:

```bash
mkdocs build --strict
```
