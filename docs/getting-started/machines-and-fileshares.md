# Machines and Fileshares

Use this page as the quick reference for machines, SSH targets, and shared storage locations.

## Machines

| Resource | Address / path | How to connect | Notes |
|---|---|---|---|
| Monai | `monai.abudhabi.nyu.edu` | `ssh -p 4410 mri@monai.abudhabi.nyu.edu` | Main Linux machine for lab workflows. Local alias may be `monai`. |
| Jubail | `jubail.abudhabi.nyu.edu` | `jubail` | Local terminal alias for the NYUAD HPC/login environment. |
| Asif's iMac | `10.157.29.87` | `ssh imac1@10.157.29.87` | CCAD OsiriX machine. Use after CCAD VPN is connected. |
| Asif's iMac VNC | `vnc://10.157.29.87` | Finder > Go > Connect to Server | Remote desktop access to OsiriX and the mounted fMRI fileshare. |
| CCAD VDI | `mydesktop.ccadi.local` | Omnissa Horizon Client | Use the `Imaging Desktop` pool. PACS, SyngoVia, and BrainLab should be accessed inside this VDI. |
| XNAT web/API | `10.230.12.52` | Browser / XNAT CLI | Current XNAT endpoint used by local XNAT CLI config. |
| XNAT nodes | see list below | `ssh -p 4410 pw1246@<ip>` | Pipeline/admin nodes. Access depends on VPN/network route. |

## XNAT Nodes

```text
10.230.12.52
10.230.12.53
10.230.12.54
10.230.12.55
10.230.12.58
10.230.12.90
10.230.12.146
10.230.12.121
10.230.12.122
10.230.12.123
10.230.12.132
```

Example:

```bash
ssh -p 4410 pw1246@10.230.12.53
```

## Transfer Commands

Use `-e "ssh -p 4410"` for `rsync` to or from Monai:

```bash
rsync -avh -e "ssh -p 4410" /local/path/ mri@monai.abudhabi.nyu.edu:/remote/path/
```

```bash
rsync -avh -e "ssh -p 4410" mri@monai.abudhabi.nyu.edu:/remote/path/ /local/path/
```

Use `-P 4410` for `scp` to or from Monai:

```bash
scp -P 4410 file.nii.gz mri@monai.abudhabi.nyu.edu:/home/mri/
```

Pull from Asif's iMac after exporting DICOMs from OsiriX:

```bash
rsync -avh imac1@10.157.29.87:/path/on/fMRI/fileshare/ /local/path/
```

## Fileshares and Data Roots

| Purpose | macOS path | Monai / Linux path | Notes |
|---|---|---|---|
| NYU MRI projects fileshare | `/Volumes/MRI/projects` | `/mnt/mri/projects` | Main final destination for most project copies. Requires NYU VPN from macOS. |
| CTP-XNAT fileshare | `/Volumes/CTP-XNAT` | `/mnt/CTP` | XNAT-related storage and pipeline data. |
| XNAT pipeline storage on nodes | n/a | `/mnt/xnat_storage` | Common path on XNAT worker nodes. |
| CCAD fMRI fileshare | n/a | `/Volumes/fMRI` on Asif's iMac | Mounted on Asif's iMac; used for CCAD DICOM export from OsiriX. |
| ARI archive on Box | `/Users/pw1246/Library/CloudStorage/Box-Box/projects/archive/ARI` | n/a | Local Box path for ARI archive data. |

## VPN Context

| Task | Network |
|---|---|
| CCAD VDI, PACS, Asif's iMac, fMRI fileshare | CCAD VPN |
| `/Volumes/MRI/projects`, `/Volumes/CTP-XNAT`, Jubail, Monai, XNAT nodes | NYU / NYUAD network or VPN |

Switch VPNs when moving from CCAD export work to NYU fileshare upload work.
