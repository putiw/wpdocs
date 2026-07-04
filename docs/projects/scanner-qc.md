# Scanner QC

Scanner QC is a local neuroimaging project for comparing scanner/protocol behavior using simple fMRI tasks and fMRIPrep-derived outputs.

## Agent Start Here

Project root:

```text
/Users/pw1246/Library/CloudStorage/Box-Box/projects/scannerQC
```

Main inventory workbook:

```text
/Users/pw1246/Library/CloudStorage/Box-Box/projects/scannerQC/code/Scans.xlsx
```

Use the workbook as the scan/run summary, but verify against BIDS-style filenames before changing task or run labels. The task and run fields should match the actual entities in fMRIPrep outputs such as:

```text
derivatives/fmriprep/sub-*/ses-*/func/*_space-T1w_desc-preproc_bold.nii.gz
```

!!! warning "Do not invent missing acquisition fields"
    fMRIPrep derivative JSON files usually preserve `RepetitionTime` and `TaskName`, but may not preserve `EchoTime`, `MultibandAccelerationFactor`, or original acquisition voxel geometry. If raw BOLD sidecars are missing locally, leave TE/MB blank and mark the source/limitation instead of writing guessed values.

## What This Is

The project collects scanner/protocol comparison tasks and their analyses:

| Task family | Purpose |
|---|---|
| Finger flex/tap | Motor/somatosensory mapping and resolution checks. |
| Checkerboard / HRF / TR QC | Visual HRF and tSNR comparison across TR/protocol conditions. |
| Breathhold | Vascular/reactivity-style QC analysis. |

The local tree is BIDS-like, with rawdata, fMRIPrep outputs, FreeSurfer outputs, task event files, and MATLAB analysis code in one Box-backed project folder.

## Systems Involved

| System | Purpose |
|---|---|
| Local Box project folder | Main working copy and shared project storage. |
| fMRIPrep derivatives | Primary available preprocessed BOLD outputs for most sessions. |
| FreeSurfer derivatives | Surface geometry and labels used by MATLAB surface analyses. |
| MATLAB | Main analysis environment for checkerboard, breathhold, flex/tap, and finger-map workflows. |
| FreeSurfer command-line tools | Header inspection and surface/MGH conversion helpers. |
| XNAT/Jubail | Source preprocessing context for some fMRIPrep outputs. Rawdata may not always be rsynced back locally. |

## Project Layout

```text
scannerQC/
  code/
    Scans.xlsx
    bh/
    checkerboard/
    exp/
    fingermap/
    flex/
  rawdata/
  derivatives/
    fmriprep/
    freesurfer/
    checkerboard/
    flex/
    tap/
    bh/
    breathhold/
    scannerqc_analyses/
```

Important code folders:

| Path | Use |
|---|---|
| `code/Scans.xlsx` | Human-readable scan inventory: task, run, design, TR, TE, slices, MB, voxel size, source notes. |
| `code/checkerboard/` | Checkerboard/HRF/TR-QC analysis scripts and V1/tSNR outputs. |
| `code/fingermap/` | Finger execution GLM, finger preference maps, and RSA analysis. |
| `code/flex/` | Flex/tap GLM and map inspection scripts. |
| `code/bh/` | Breathhold analysis scripts. |
| `code/exp/` | Event/design files copied from task code or regenerated for analysis. |

## Scan Inventory Workbook

`Scans.xlsx` should document at least:

- `sub`
- `ses`
- `task`
- `run`
- `design`
- `TR (ms)`
- `TE (ms)`
- `slices`
- `MB`
- `voxel size (mm)`
- `metadata source`
- `notes`

Use units in the header. Keep `sub`, `ses`, and `run` displayed with leading zeros.

### Current Task / Run Pattern

The fMRIPrep filenames currently define this task/run structure:

| Session | Task labels | Run pattern | Notes |
|---|---|---|---|
| `ses-01` | `flex`, `tap` | runs `01`-`03` for each task | Motor/finger task runs. |
| `ses-02` | `breathhold30`, `breathhold31`, `breathhold39` | no BIDS `run-` entity | Leave run blank if the filename has no run entity. |
| `ses-03` | `hrf` | runs `01`-`06` | Checkerboard HRF; odd runs are 0.75 s TR and even runs are 2 s TR. |
| `ses-04` | `flex` | runs `01`-`03` | Include run 03 if present in fMRIPrep. |
| `ses-05` | `trqc750`, `trqc2000` | `trqc750` runs `01`-`03`; `trqc2000` runs `04`-`06` | Do not relabel these as `hrf`; use the actual task entities. |
| `ses-06` | `flex16` | runs `01`-`03` | Higher-resolution flex run set. |

### Metadata Source Rules

Prefer sources in this order:

1. Raw BOLD JSON and raw BOLD NIfTI header.
2. fMRIPrep derivative JSON for `TaskName` and `RepetitionTime`.
3. `mri_info` on fMRIPrep NIfTI only for fallback dimensions/TR/voxel geometry when rawdata is not local.

Useful checks:

```bash
PROJECT=/Users/pw1246/Library/CloudStorage/Box-Box/projects/scannerQC

find "$PROJECT/derivatives/fmriprep" \
  -path '*/func/*_space-T1w_desc-preproc_bold.nii.gz' \
  -print | sort

/Applications/freesurfer/7.4.1/bin/mri_info --tr <bold.nii.gz>
/Applications/freesurfer/7.4.1/bin/mri_info --dim <bold.nii.gz>
/Applications/freesurfer/7.4.1/bin/mri_info --res <bold.nii.gz>
```

!!! warning "Voxel size caveat"
    `space-T1w_desc-preproc_bold.nii.gz` is a processed fMRIPrep output. Its voxel size and slice count are useful fallback documentation for what is locally available, but they may not equal the original acquisition geometry. Use raw BOLD NIfTI/header values when rawdata is available.

## Main Workflows

### Checkerboard / HRF / TR QC

Run from MATLAB:

```matlab
cd('/Users/pw1246/Library/CloudStorage/Box-Box/projects/scannerQC/code/checkerboard')
s0_make_checkerboard_events
s1_fit_checkerboard_hrf
s2_plot_tsnr_figure
s3_compare_top5_tsnr_vertices
s4_make_significance_maps
s5_plot_mriqc_comparisons
s6_compare_all_conditions_tsnr
view_trqc_live
view_checkerboard_results
view_checkerboard_surface
```

Outputs go to:

```text
derivatives/checkerboard/
```

Related experiment documentation:

- [Checkerboard HRF](../experiments/checkerboard.md)

### Finger / Flex / Tap Analyses

Run the finger execution pipeline from MATLAB:

```matlab
cd('/Users/pw1246/Library/CloudStorage/Box-Box/projects/scannerQC/code/fingermap')
s0_setup_paths
s1_fit_execution_glm
s2_make_finger_maps
s3_compute_rsa
s4_plot_rsa_figure
view_finger_maps
view_rsa_results
```

The shared config auto-detects whether the BIDS/fMRIPrep tree is directly under the project root or under `data/`.

Related experiment documentation:

- [Six-Fingers Motor Execution](../experiments/six-fingers-execution.md)

### Breathhold

Run from MATLAB:

```matlab
cd('/Users/pw1246/Library/CloudStorage/Box-Box/projects/scannerQC/code/bh')
s1_timeseries
s2_glm
view_bh
```

Outputs are under:

```text
derivatives/bh/
derivatives/breathhold/
```

## Operational Notes

- Keep rawdata, derivatives, and event files synchronized enough that analysis scripts can discover matching subject/session/task/run entities.
- When rawdata is not fully local, use fMRIPrep outputs for task/run inventory and TR, but record missing TE/MB explicitly.
- Prefer task/run labels parsed from filenames over memory or spreadsheet values.
- Do not hard-code one subject/session in reusable scripts unless the script is intentionally a one-off check.
- Keep sensitive subject identifiers, MRNs, scanner console notes, and credentials out of tracked docs.

## Current Status

- `Scans.xlsx` is the current scan inventory workbook.
- The workbook should include the corrected task names for TR QC runs: `trqc750` and `trqc2000`.
- fMRIPrep outputs are available locally for several sessions; rawdata may be incomplete.
- TE and MB are only reliable when a matching raw BOLD sidecar is available locally.
- MATLAB analysis folders are organized by task family and write outputs under `derivatives/`.

## Resume / Log Notes

- Built a scanner QC project structure combining stimulus-task outputs, fMRIPrep/FreeSurfer derivatives, and MATLAB analyses.
- Maintained checkerboard HRF and tSNR comparisons for TR/protocol QC.
- Maintained finger-map/flex/tap analysis code for somatotopic mapping and RSA.
- Corrected and documented a scan inventory workflow that validates task/run labels against BIDS-style fMRIPrep filenames and records metadata provenance.
