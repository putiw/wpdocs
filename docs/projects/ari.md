# ARI

ARI is a neuroimaging project with data in XNAT and long-term archive copies in NYU Box.

This project page summarizes the project-level context. For the operational archive details, see [ARI Box Archive](../data-management/ari-box-archive.md).

## What This Is

ARI data lives in XNAT and includes rawdata plus derivatives such as validation outputs, FreeSurfer outputs, and v123seg outputs.

There are two ARI-related XNAT projects involved in the archive workflow:

| XNAT project | Role |
|---|---|
| Main ARI | Primary project. Nightly sync backs up derivatives and validated rawdata. |
| ARI-HFS | Supplemental source. Manual rawdata-only sync can add missing validated files without overwriting main ARI Box files. |

The shared Box target is:

```text
box:archive/ARI/
```

Local Box path:

```text
/Users/pw1246/Library/CloudStorage/Box-Box/projects/archive/ARI
```

## My Role

- Set up and documented the XNAT-to-Box archive workflow.
- Maintained Monai-side scripts for Box token refresh, Box mounting, ARI sync, and HFS rawdata sync.
- Defined validation/staging behavior so incomplete modality groups are skipped rather than partially archived.
- Documented recovery, cron, and troubleshooting procedures.

## Systems Involved

| System | Purpose |
|---|---|
| XNAT | Source archive storage for ARI and ARI-HFS project data. |
| Monai | Remote VM that can see XNAT storage and run archive scripts. |
| rclone | Copies staged data to Box. |
| Box JWT app | Provides service-account access for automated Box sync. Credential details are private. |
| NYU Box | Long-term project archive target. |
| Local Mac | Local Box mirror and documentation/editing environment. |

## Main Workflows

| Workflow | Link |
|---|---|
| XNAT to Box archive | [ARI Box Archive](../data-management/ari-box-archive.md) |
| General data transfer patterns | [Data Transfer & Sync](../data-management/data-transfer.md) |
| Machine/path reference | [Machines and Fileshares](../getting-started/machines-and-fileshares.md) |

## Scripts / Repos

Primary scripts on Monai:

```text
/home/mri/bin/sync_ari_to_box.sh
/home/mri/bin/ari_rawdata_stage.py
/home/mri/bin/sync_hfs_rawdata_to_box.sh
/home/mri/bin/box_token.py
/home/mri/bin/mount_box.sh
```

Private setup note with credentials and sensitive setup details:

```text
/Users/pw1246/Desktop/info/info about archive using box.txt
```

## Operational Notes

Main ARI source:

```text
/mnt/CTP/xnat-main/xnat-data/archive/ari/arc001/
```

ARI-HFS source:

```text
/mnt/CTP/xnat-main/xnat-data/archive/rokerslab_ari-hfs_2024_001/arc001
```

Main ARI sync:

```bash
/home/mri/bin/sync_ari_to_box.sh --dry-run --all
/home/mri/bin/sync_ari_to_box.sh --all
```

HFS rawdata sync:

```bash
/home/mri/bin/sync_hfs_rawdata_to_box.sh --dry-run --all
/home/mri/bin/sync_hfs_rawdata_to_box.sh --all
```

Cron:

```text
Box mount refresh: every 50 minutes
Main ARI sync: daily at 03:00
```

## Current Status

- Main ARI nightly sync is documented as active.
- HFS rawdata sync is documented as manual and should be run after reviewing dry-run output.
- DWI backup logic is documented with the updated accepted categories: v1 DWI-only, v1.5 pilot with matching reverse-b0, and v2 with matching reverse-b0.
- Latest recorded HFS all-session dry-run after the DWI update was on 2026-06-08.
- Public docs intentionally omit Box client secrets, private keys, passphrases, and passwords.

## Resume / Log Notes

- Built and documented an automated XNAT-to-Box archive workflow for ARI neuroimaging data.
- Implemented validation-backed staging logic for multimodal BIDS rawdata and derivatives.
- Maintained Monai-side rclone/JWT automation for scheduled Box sync.
- Designed safeguards for HFS supplemental data ingestion using non-overwriting `--ignore-existing` uploads.
- Produced operational documentation covering sync commands, cron jobs, logs, and troubleshooting.
