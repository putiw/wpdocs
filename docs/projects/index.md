# Projects

Project pages collect work by context: what the project is, what systems it touches, what workflows exist, what scripts or infrastructure support it, and what can be summarized later for logs, reports, or job applications.

Use this section when the question is: *what did I build, maintain, or support for this project?*

Use the pipeline tabs (Data Management, Preprocessing, Analysis, etc.) when the question is: *how do I run this specific command or workflow?*

## Active Projects

| Project | What it captures |
|---|---|
| [myPhysio](myphysio.md) | Physiotherapy web/mobile platform, Supabase backend, deployment notes, mobile APK workflow, and feature history. |
| [ARI](ari.md) | XNAT archive structure, Box backup, Monai scripts, validation/staging logic, and data management support. |
| [CCAD Epilepsy](ccad-epilepsy.md) | CCAD access, PACS/OsiriX data export, epilepsy BIDS conversion, MAP18 pipeline, MELD Graph. Has its own sub-pages for VPN, BrainLab, and pipeline details. |

## Project Backlog

These are project/support areas that should get pages as source material accumulates.

| Area | Possible page focus |
|---|---|
| Laminate | Startup/support workflows, reporting, data organization, scripts, deployment notes. |
| ITH | Tooling, clinical/research support workflows, data handling, scripts. |
| LesionView | Viewer purpose, users, deployment, data requirements, clinical support notes. |
| CVSView | Viewer purpose, users, deployment, data requirements, clinical support notes. |
| SixthFinger | Research project workflows, data location, preprocessing/analysis notes, deliverables. |
| Clinician support | Recurring request types, data export, de-identification, delivery patterns. |
| NYUAD researcher support | Common workflows, XNAT help, fMRIPrep/MRIQC/Jubail support, data transfer. |

## Project Page Template

```md
# Project Name

## What This Is

## My Role

## Systems Involved

## Main Workflows

## Scripts / Repos

## Operational Notes

## Current Status

## Resume / Log Notes
```

## How To Use This Section

- Link out to pipeline pages instead of duplicating every command.
- Keep project-specific context here.
- Keep credentials and patient identifiers out of git.
- Add short resume/log bullets while the work is fresh.
