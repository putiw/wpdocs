# CCAD Epilepsy — Access & Workflows

Practical notes for CCAD access and data movement. These pages support the [CCAD Epilepsy project](../projects/ccad-epilepsy.md).

## Pages

- [VPN & VDI Access](vpn-access.md) — Connecting to the CCAD network
- [Getting Data from CCAD](data-from-ccad.md) — PACS export, OsiriX, rsync from Asif's iMac
- [BrainLab](brainlab.md) — Clinical neuronavigation system notes
- [Epilepsy BIDS Conversion](epilepsy-bids.md) — DICOM to BIDS for epilepsy patients
- [MAP18 Pipeline](epilepsy-map18-pipeline.md) — T1 morphometry (VBM junction maps, MELD predictions)
- [Contacts & Systems](server-paths.md) — Server paths, IP addresses, key contacts

## Rules

- Keep credentials, VPN profile files, passcodes, and patient identifiers out of git.
- Keep sensitive source files in `/Users/pw1246/Documents/GitHub/wpdocs/secrets/`.
- Put only practical, reusable steps in the public docs.
