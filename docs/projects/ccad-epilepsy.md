# CCAD Epilepsy

CCAD Epilepsy captures the workflow for accessing CCAD clinical systems, exporting imaging data, and converting epilepsy DICOMs into BIDS.

## What This Is

The CCAD workflow starts inside the CCAD network and ends on local/NYU storage.

High-level flow:

```text
CCAD VPN -> Omnissa Horizon VDI -> PACS -> OsiriX on Asif's iMac -> fMRI fileshare -> local Mac -> NYU MRI fileshare
```

Epilepsy data exported from CCAD can then be converted to BIDS using the CCAD pipeline script.

## My Role

- Documented CCAD VPN and VDI access from macOS.
- Documented the PACS-to-OsiriX-to-fileshare data export workflow.
- Identified key systems, paths, contacts, and machines.
- Documented the CCAD epilepsy BIDS conversion script and expected source/target locations.

## Systems Involved

| System | Address / path | Purpose |
|---|---|---|
| FortiClient VPN | `ccadvpn.clevelandclinicabudhabi.ae` | CCAD network access. |
| Omnissa Horizon | `mydesktop.ccadi.local` | VDI access. |
| VDI pool | `Imaging Desktop` | PACS, SyngoVia, BrainLab access. |
| PACS | inside CCAD VDI | Source of clinical imaging studies. |
| Asif's iMac | `10.157.29.87` | OsiriX and DICOM export machine. |
| Asif's iMac VNC | `vnc://10.157.29.87` | Remote desktop access. |
| fMRI fileshare | `/Volumes/fMRI` on Asif's iMac | Intermediate export/storage target. |
| NYU MRI fileshare | `/Volumes/MRI/projects` | Final project storage after switching to NYU VPN. |

## Main Workflows

| Workflow | Link |
|---|---|
| VPN and VDI access | [CCAD VPN and VDI Access](../ccad/vpn-access.md) |
| Data export from CCAD | [Getting Data from CCAD](../ccad/data-from-ccad.md) |
| Epilepsy BIDS conversion | [CCAD Epilepsy BIDS Conversion](../ccad/epilepsy-bids.md) |
| MAP18-like morphometry pipeline | [CCAD Epilepsy MAP18-Like Pipeline](../ccad/epilepsy-map18-pipeline.md) |
| Contacts and systems | [CCAD Contacts and Systems](../ccad/server-paths.md) |
| Transfer patterns | [Data Transfer & Sync](../data-management/data-transfer.md) |

## Scripts / Repos

CCAD epilepsy conversion script:

```text
/Users/pw1246/Documents/GitHub/pipelines/ccad/scripts/convert_missing_dicoms_to_bids.py
```

Config:

```text
/Users/pw1246/Documents/GitHub/pipelines/ccad/configs/ccad_dcm2bids_config.json
```

Remote BIDS root on Asif's iMac:

```text
/Volumes/fMRI/Epilepsy_Cases/BIDS
```

## Operational Notes

Use CCAD VPN while working with:

```text
mydesktop.ccadi.local
10.157.29.87
vnc://10.157.29.87
```

Use NYU VPN when copying final data into:

```text
/Volumes/MRI/projects
```

Pull exported DICOMs from Asif's iMac:

```bash
rsync -avh imac1@10.157.29.87:/path/on/fMRI/fileshare/ /local/target/folder/
```

Do not document passwords, patient identifiers, or VPN profile passcodes in public docs.

## Current Status

- CCAD VPN/VDI access steps are documented.
- PACS-to-OsiriX export workflow is documented.
- BrainLab page exists but access is not yet fully established.
- Epilepsy BIDS conversion script is documented with current path defaults.

## Resume / Log Notes

- Established and documented a CCAD clinical imaging data access workflow spanning VPN, VDI, PACS, OsiriX, and NYU storage.
- Documented secure handling of clinical imaging exports without exposing credentials or patient identifiers.
- Built project-specific operational notes for CCAD epilepsy BIDS conversion.
- Coordinated practical system knowledge across CCAD IT, clinical workstations, and NYU research storage.
