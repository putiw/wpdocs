# XNAT Pipelines

This page is the starting point for XNAT pipeline and container-service work.
Use it when a pipeline run fails, a wrapper needs to be updated, or a project
resource such as a dcm2bids config needs to be exposed to XNAT.

## Main Places

| Item | Path / address |
|---|---|
| XNAT web/API | `https://xnat.abudhabi.nyu.edu` |
| macOS archive mount | `/Volumes/CTP-XNAT/xnat-main/xnat-data/archive/` |
| Monai archive mount | `/mnt/CTP/xnat-main/xnat-data/archive/` |
| XNAT node storage | `/mnt/xnat_storage` |
| Pipeline definitions | `/Volumes/CTP-XNAT/pipelines/` |
| Local pipeline scripts repo | `/Users/pw1246/Documents/GitHub/pipelines/` |

## Common Pipeline Types

| Pipeline | What to check first |
|---|---|
| `dcm2bids` | Project-level `resources/configs`, selected config file, DICOM scan metadata, BIDS validator output. |
| `fmriprep` | Rawdata resource, FreeSurfer resource shape, FreeSurfer license, Jubail Slurm logs, output resource copyback. |
| Resource upload / refresh | Resource catalog XML, REST file listing, project-vs-session resource URI. |

## Container Run Debugging

From the XNAT UI, capture:

- command ID
- workflow ID
- container ID
- project and experiment/session ID
- Docker image
- mounts
- stdout/stderr log paths

Typical log paths look like:

```text
/data/xnat/archive/CONTAINER_EXEC/<command_id>/LOGS/docker/stdout.log
/data/xnat/archive/CONTAINER_EXEC/<command_id>/LOGS/docker/stderr.log
```

On a node, the same storage is usually visible under:

```text
/mnt/xnat_storage/xnat-main/xnat-data/archive/CONTAINER_EXEC/<command_id>/LOGS/docker/
```

## dcm2bids Config Resources

Project-level dcm2bids configs live under:

```text
/Volumes/CTP-XNAT/xnat-main/xnat-data/archive/<project>/resources/configs/
```

After editing a config JSON, refresh the catalog:

```bash
export XNAT_ALIAS="<alias-token>"
export XNAT_SECRET="<alias-secret>"

/Users/pw1246/Documents/GitHub/pipelines/xnat/refresh_xnat_resource_from_catalog.sh \
  /Volumes/CTP-XNAT/xnat-main/xnat-data/archive/<project>/resources/configs/configs_catalog.xml
```

Use separate config files when protocol variants are genuinely incompatible.
See [dcm2bids Config Files](dcm2bids-config.md).

## fMRIPrep on Jubail

The XNAT fMRIPrep wrapper submits a Slurm script to Jubail and then copies
outputs back into XNAT resources. Start with:

- [fMRIPrep](../preprocessing/fmriprep.md)
- [fMRIPrep on Jubail](../preprocessing/fmriprep-jubail.md)
- [Uploading Resources to XNAT](xnat-upload-resources.md)

Key point: wrapper files are named from the XNAT workflow ID, not the Slurm job
ID. A normal run creates `/home/mri/<workflow_id>.slurm` and
`/scratch/mri/<workflow_id>/`.

If a failed job is named only `.slurm`, suspect an empty `XNAT_WORKFLOW_ID` and
do not trust cleanup until the wrapper safety checks are confirmed.

## Resource Catalog Debugging

If files exist on disk but do not show up in XNAT:

1. Confirm the resource directory has `<resource>_catalog.xml`.
2. Confirm the catalog entries match files on disk.
3. Refresh the catalog with the helper script.
4. Check the REST file endpoint, not only the aggregate `file_count`.

`append` fixes files present on disk but missing from the catalog. It does not
remove stale catalog rows for deleted files; use `delete` or manually prune stale
entries when needed.
