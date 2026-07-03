# CCAD Contacts and Systems

## Key Contacts

| Person / group | Email | Notes |
|---|---|---|
| Puti CCAD account | `wenp@clevelandclinicabudhabi.ae` | Use for CCAD login and Microsoft Authenticator. |
| Osama Abdullah | `osama.abdullah@nyu.edu`; `AbdullO@ccad.ae` | NYUAD/CCAD collaboration access context. |
| Mohammad Asif Dogar | `DogarM@ccad.ae` | Asif's iMac is `10.157.29.87`. |
| Sam Rogers | `RogersS2@ccad.ae` | CCAD access thread. |
| Rizwana Altaf | `AltafR@ccad.ae` | CCAD access thread. |
| Alok Anand | `AnandA@ccad.ae` | CCAD access thread. |
| Hidayath Ansari | `AnsariH@ccad.ae` | CCAD access thread. |
| DL-NIAdmin | `DL-NIAdmin@ccad.ae` | Neuroimaging/admin support list. |
| Network Security | `NETWORKSECURITY@ccad.ae` | VPN/network routing support. |

## Systems

| System | Address | Account / access | Purpose |
|---|---|---|---|
| FortiClient VPN | `ccadvpn.clevelandclinicabudhabi.ae` | CCAD account | VPN endpoint. |
| VPN profile | `CCAD-ZTNA` | XML profile from IT | Current VPN profile. |
| Asif's iMac | `10.157.29.87` | `imac1` | OsiriX and DICOM export machine. |
| Asif's iMac VNC | `vnc://10.157.29.87` | `imac1` | Remote desktop from macOS Finder. |
| BrainLab station | `CCADBLP050.CCADI.LOCAL` | CCAD account, app permission required | BrainLab access from VDI. |
| Omnissa Horizon | `mydesktop.ccadi.local` | CCAD account | VDI access. |
| VDI pool | `Imaging Desktop` | CCAD account | PACS, SyngoVia, BrainLab access. |

## Storage

| Path | Location | Purpose |
|---|---|---|
| `/Volumes/fMRI` | Asif's iMac / fMRI file share | Shared export/storage location. |
| `/Volumes/fMRI/Epilepsy_Cases/BIDS` | Asif's iMac | CCAD epilepsy BIDS root. |
| `/Volumes/fMRI/Epilepsy_Cases/BIDS/sourcedata` | Asif's iMac | DICOM source folders. |
| `/Volumes/fMRI/Epilepsy_Cases/BIDS/rawdata` | Asif's iMac | BIDS rawdata output. |
| `/Users/pw1246/Library/CloudStorage/Box-Box/projects/CCAD/epilepsy/BIDSepi` | Local Mac / Box | Local synced CCAD epilepsy BIDS root. |
| `/Users/pw1246/Documents/GitHub/pipelines/ccad` | Local Mac | CCAD pipeline scripts/configs. |

## Quick Checks

```bash
ssh imac1@10.157.29.87
```

```bash
open vnc://10.157.29.87
```
