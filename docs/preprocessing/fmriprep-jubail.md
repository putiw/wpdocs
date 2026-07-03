# fMRIPrep on Jubail

This page records the XNAT/Jubail fMRIPrep workflow and the failure modes that
have come up while running scanner-QC data through:

```text
/Volumes/CTP-XNAT/pipelines/fmriprep
```

Use this page together with [XNAT Pipelines](../data-management/xnat-pipelines.md)
and [Uploading Resources to XNAT](../data-management/xnat-upload-resources.md).

## Identifier Map

The wrapper creates Jubail files from the XNAT workflow ID before Slurm assigns a
job ID.

Example:

```text
XNAT_WORKFLOW_ID=2037399
/home/mri/2037399.slurm
/home/mri/slurm-2037399.out
/home/mri/slurm-2037399.err
/scratch/mri/2037399/
```

Slurm then assigns a separate job ID such as:

```text
16403349_1
```

Do not expect the script file to be named `16403349_1.slurm`. The Slurm job ID
does not exist yet when the wrapper writes `/home/mri/<workflow_id>.slurm`.

If Slurm accounting shows a command named only:

```text
.slurm
```

that means the wrapper likely had an empty `XNAT_WORKFLOW_ID`. Treat that as a
serious wrapper bug, because cleanup paths can become ambiguous.

## Required Safety Checks

The wrapper should fail before submitting or cleaning up if required identifiers
are missing.

Minimum checks:

- `XNAT_WORKFLOW_ID` must be present and numeric.
- `send_script()` must confirm the Slurm script actually copied to
  `/home/mri/<workflow_id>.slurm`.
- `submit_job()` must parse `Submitted batch job <id>` and fail if no job ID is
  returned.
- result retrieval must fail if required `rsync` steps fail.
- cleanup must refuse to delete `/scratch/mri`, `/scratch/mri/`,
  `/scratch/<user>`, or any path that is not exactly
  `/scratch/<user>/<workflow_id>`.

The intended cleanup target after a successful run is only:

```text
/scratch/mri/<workflow_id>
```

It should never remove the shared `/scratch/mri` directory itself.

## FreeSurfer Reuse

To reuse an existing `recon-all` output, the fMRIPrep container needs a real
FreeSurfer `SUBJECTS_DIR` containing `sub-XXXX/` directly under the directory
passed to `--fs-subjects-dir`.

For an XNAT resource, the archive path typically looks like:

```text
/Volumes/CTP-XNAT/xnat-main/xnat-data/archive/<project>/arc001/<session>/RESOURCES/freesurfer/
  freesurfer_catalog.xml
  sub-0248/
  fsaverage/
  fsaverage6/
```

Inside the container, avoid leaving the downloaded resource wrapped like:

```text
/app/freesurfer/<session>/resources/freesurfer/files/sub-0248
```

The wrapper should unwrap or copy the true `SUBJECTS_DIR` so fMRIPrep sees:

```text
/app/freesurfer/sub-0248
```

If fMRIPrep creates a second FreeSurfer directory named after the XNAT session
label, it usually means it did not find the expected `sub-XXXX` subject directory
at the effective `--fs-subjects-dir` path.

## Slurm / Home Directory Failures

When `/home/mri` is full, Slurm script creation and stdout/stderr retrieval can
fail in confusing ways. Check:

```bash
ssh -p 4410 mri@jubail.abudhabi.nyu.edu "df -h /home/mri /scratch/mri"
```

The wrapper should stop immediately if it cannot copy:

```text
/home/mri/<workflow_id>.slurm
/home/mri/slurm-<workflow_id>.out
/home/mri/slurm-<workflow_id>.err
```

Do not continue to cleanup after a failed submit or failed output retrieval
unless the workflow ID and cleanup path have both passed the safety checks above.

## TR and SliceTiming Failures

An fMRIPrep failure during slice-timing correction can come from a bad dcm2bids
config, not from fMRIPrep itself.

Observed pattern:

```text
AFNI 3dTshift: Illegal value 1.385 in tpattern file
```

Cause:

```text
RepetitionTime: 1
SliceTiming values up to ~1.78
```

The config had forced `RepetitionTime` to `1`, while `dcm2niix` reported the
actual TR as `2`. AFNI correctly rejected slice timings larger than the declared
TR.

Fix pattern:

1. Run `dcm2niix` on a representative source scan and inspect the generated JSON.
2. Do not force `RepetitionTime` in the config unless you have verified it from
   source metadata.
3. If `SliceTiming` is valid for the scanner-derived TR, correct the TR rather
   than suppressing slice-timing correction.
4. Re-run dcm2bids and BIDS validation before rerunning fMRIPrep.

## Queue Debugging

For a pending Jubail job, check the script and scheduler state directly:

```bash
ssh -p 4410 mri@jubail.abudhabi.nyu.edu
scontrol show job <slurm_job_id>
sacct -j <slurm_job_id> --format=JobID,JobName,State,Start,End,Elapsed,ExitCode
sbatch --test-only /home/mri/<workflow_id>.slurm
```

If a wrapper or account default injects a feature constraint, compare it against
idle nodes:

```bash
sinfo -p compute -N -h -o "%N %t %c %m %f" | egrep 'idle|mix'
```

Reducing walltime or CPU count can sometimes help backfill, but do not change
resources until you understand whether the pending reason is priority, feature
constraint, reservation, or invalid script syntax.
