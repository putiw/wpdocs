# fMRIPrep

fMRIPrep is the default preprocessing workflow for BIDS-formatted fMRI data. It
handles anatomical preprocessing, motion correction, susceptibility distortion
correction when fieldmaps are available, registration, confound extraction, and
standard output reports.

Use the containerized workflow whenever possible. Do not try to maintain a
hand-installed fMRIPrep Python environment unless you have a specific reason.

## Inputs

Expected BIDS layout:

```text
rawdata/
  sub-0248/
    ses-01/
      anat/
      func/
      fmap/
```

Required before a full run:

- BIDS-valid rawdata.
- FreeSurfer license file.
- Enough work space for intermediate files.
- Correct fieldmap metadata if distortion correction is expected:
  `PhaseEncodingDirection`, `TotalReadoutTime`, and either `IntendedFor` or
  `B0FieldIdentifier` / `B0FieldSource`.

## Basic Command

Docker-style local pattern:

```bash
docker run --rm -it \
  -v /path/to/rawdata:/data:ro \
  -v /path/to/derivatives:/out \
  -v /path/to/work:/work \
  -v /path/to/license.txt:/license.txt:ro \
  nipreps/fmriprep:<version> \
  /data /out participant \
  --participant-label 0248 \
  --fs-license-file /license.txt \
  --work-dir /work \
  --output-spaces MNI152NLin2009cAsym T1w fsnative fsaverage
```

On HPC, use Singularity/Apptainer with equivalent bind mounts. See
[fMRIPrep on Jubail](fmriprep-jubail.md) for XNAT/Jubail-specific wrapper notes.

## Output Spaces

Common choices:

| Space | Use |
|---|---|
| `T1w` | Native anatomical-space volumetric analyses and inspection. |
| `MNI152NLin2009cAsym` | Standard-space group analyses. |
| `fsnative` | Subject-native surface GLM and clinical/native surface review. |
| `fsaverage` | Surface group analyses and easier cross-subject comparison. |

For single-subject clinical/localizer work, include `T1w` and `fsnative`. For
group analysis, include a standard MNI space and usually `fsaverage`.

## Reusing FreeSurfer

If `recon-all` already exists, pass the parent `SUBJECTS_DIR`:

```bash
--fs-subjects-dir /path/to/freesurfer
```

That directory should contain:

```text
/path/to/freesurfer/sub-0248/scripts/recon-all.done
```

If the subject directory is nested inside an XNAT resource wrapper, fMRIPrep may
not find it and can run a new reconstruction under a different subject/session
name. See [Uploading Resources to XNAT](../data-management/xnat-upload-resources.md#freesurfer-reuse-with-fmriprep)
and [fMRIPrep on Jubail](fmriprep-jubail.md#freesurfer-reuse).

## Check the Report

Open the generated HTML report for each subject/session before trusting outputs:

```text
derivatives/fmriprep/sub-0248.html
```

Check:

- T1w skull stripping and tissue segmentation.
- BOLD-to-T1w registration.
- Susceptibility distortion correction direction and quality.
- Confound plots and framewise displacement.
- Any warnings about missing fieldmaps, reused FreeSurfer, or ignored metadata.

## Common Failures

| Symptom | Likely cause | Fix |
|---|---|---|
| `3dTshift` complains about illegal values in a timing file | `SliceTiming` values are larger than the declared `RepetitionTime` | Inspect source dcm2niix JSON and fix the dcm2bids config; do not force an incorrect TR. |
| Fieldmaps are ignored | Missing/wrong `IntendedFor`, `B0FieldIdentifier`, or `B0FieldSource` | Fix the dcm2bids config and rerun conversion. |
| fMRIPrep runs a new FreeSurfer subject despite an existing resource | `--fs-subjects-dir` does not point to the actual parent of `sub-XXXX/` | Unwrap/copy the resource so `sub-XXXX` is directly inside `SUBJECTS_DIR`. |
| HTML report exists but outputs are missing | Wrapper or file-copy failure after processing | Check pipeline stdout/stderr and required `rsync` transfers. |
