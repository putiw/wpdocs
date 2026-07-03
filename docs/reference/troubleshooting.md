# Troubleshooting

Use this page as a first pass when something fails and you are not sure which
workflow-specific page applies yet.

## First Checks

```bash
pwd
whoami
hostname
df -h
which python
python --version
python -m pip --version
```

For remote/HPC jobs, also check:

```bash
squeue -u "$USER"
sacct -j <job_id> --format=JobID,JobName,State,Elapsed,ExitCode
```

## Common Patterns

| Symptom | Likely cause | Start here |
|---|---|---|
| Command works interactively but not over SSH | Shell startup prints banners or tools are not on noninteractive `PATH` | [Software Installation](../getting-started/software-installation.md) |
| Python package installed but script cannot import it | Installed into the wrong Python/env | [Python Environments](../information/python-environments.md) |
| XNAT resource files are on disk but not visible in browser | Catalog XML or resource refresh issue | [Uploading Resources to XNAT](../data-management/xnat-upload-resources.md) |
| XNAT catalog refresh returns HTTP 500 | Stale catalog rows, bad resource URI, or deleted files still in XML | [Uploading Resources to XNAT](../data-management/xnat-upload-resources.md#stale-catalog-rows) |
| fMRIPrep fails in AFNI `3dTshift` | `SliceTiming` values disagree with declared `RepetitionTime` | [fMRIPrep](../preprocessing/fmriprep.md#common-failures) |
| fMRIPrep reruns FreeSurfer unexpectedly | `--fs-subjects-dir` does not point to the real FreeSurfer subject parent | [fMRIPrep on Jubail](../preprocessing/fmriprep-jubail.md#freesurfer-reuse) |
| Slurm job file is `.slurm` | Wrapper likely had empty workflow ID | [fMRIPrep on Jubail](../preprocessing/fmriprep-jubail.md#identifier-map) |
| Slurm submit succeeds but output retrieval fails | Missing stdout/stderr or failed rsync | [Job Submission](../hpc/job-submission.md) |
| BIDS fieldmaps are ignored | Wrong/missing `IntendedFor`, `B0FieldIdentifier`, or `B0FieldSource` | [dcm2bids Config Files](../data-management/dcm2bids-config.md) |

## Disk and Quota

Before debugging software, check whether the relevant filesystem is full:

```bash
df -h /home "$PWD"
```

On Jubail/XNAT wrapper jobs, check both home and scratch:

```bash
ssh -p 4410 mri@jubail.abudhabi.nyu.edu "df -h /home/mri /scratch/mri"
```

A full home directory can prevent Slurm scripts or stdout/stderr files from
being created, which then creates misleading downstream failures.

## Secrets and Identifiers

Do not paste tokens, passwords, private keys, `.env` values, patient names, or
real MRNs into public docs, commits, issues, PRs, or chat output. Use local
ignored notes under:

```text
/Users/pw1246/Documents/GitHub/wpdocs/secrets/
```
