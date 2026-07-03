# CCAD Language Mapping Case Workflow

This page documents the cleaned local workflow for a CCAD language-mapping case
without exposing the real subject identifier in public docs.

Local project root:

```text
/Users/pw1246/Desktop/projects/support/CCAD/languageMapping
```

Use the private local project inventory or ignored notes for the real subject ID
and exact case-specific filenames.

## What Was Kept

The project directory was decluttered to keep the reproducible surface-based
language workflow and its final review products.

Kept derivative folders:

```text
derivatives/fmriprep
derivatives/freesurfer
derivatives/surface_alltasks_run1only
derivatives/mask
```

Removed older exploratory outputs included raw-volume GLM attempts, second-run
test outputs, clipped display maps, pial-preview lesion experiments, temporary
work folders, old test-subject derivatives, and stale macOS `.DS_Store` files.

## Final Commands

Run the full final workflow from the local language-mapping project code folder.
The exact case-specific launcher name is intentionally not listed in public
docs; use the private local inventory or search the project code folder for the
current `surface_language_outputs` launcher.

Expected stages:

```bash
# Surface GLM in fsnative
python /Users/pw1246/Desktop/projects/support/CCAD/languageMapping/code/<surface_glm_script>.py

# Combine/project language surface results into T1w review space
/Users/pw1246/Desktop/projects/support/CCAD/languageMapping/code/<combine_surface_to_t1w_script>.sh
/Users/pw1246/Desktop/projects/support/CCAD/languageMapping/code/<project_language_task_surfaces_to_t1w_script>.sh

# Make review mosaics
/Users/pw1246/Desktop/projects/support/CCAD/languageMapping/code/<make_mosaic_script>.sh
```

To open the projected T1w result during projection, use the workflow's
`OPEN_ITKSNAP=1` option if available.

## Inputs

The surface GLM workflow expects fMRIPrep and FreeSurfer outputs:

```text
/Users/pw1246/Desktop/projects/support/CCAD/languageMapping/derivatives/fmriprep/<subject>/<session>
/Users/pw1246/Desktop/projects/support/CCAD/languageMapping/derivatives/freesurfer/<subject>
```

The BIDS raw data remain under:

```text
/Users/pw1246/Desktop/projects/support/CCAD/languageMapping/rawdata/<subject>/<session>
```

## Surface GLM Outputs

Per-task fsnative surface GLM maps are written under:

```text
derivatives/surface_alltasks_run1only/<subject>/<session>/surface
```

The modeled tasks are:

- sentence completion run 1
- silent word generation
- object naming
- antonym generation
- breath hold

The second sentence-completion run was intentionally excluded from the cleaned
final workflow.

Each task should have left/right hemisphere files:

```text
*_hemi-L_space-fsnative_stat-tstat.mgz
*_hemi-R_space-fsnative_stat-tstat.mgz
*_hemi-L_space-fsnative_stat-beta.mgz
*_hemi-R_space-fsnative_stat-beta.mgz
*_desc-languageCandidate_mask.mgz
```

## T1w-Projected Language Response

The final T1w-space volume averages the four language t-stat maps:

- sentence completion run 1
- silent word generation
- object naming
- antonym generation

Breath hold is not included in the averaged language response.

The projection script smooths the mean surface map with 10 mm FWHM on the
FreeSurfer surface, projects it through the cortical ribbon, then transforms it
to fMRIPrep T1w space.

Treat this T1w-averaged map as a visualization/review product derived from
smoothed surface statistics, not as an independent voxelwise GLM.

## Mosaic Figures

MRIQC-style PNG mosaics are written under:

```text
derivatives/surface_alltasks_run1only/<subject>/<session>/figures
```

Expected output:

- 3 averaged all-language mosaics: axial, coronal, sagittal
- 12 per-task mosaics: 4 language tasks x 3 views

Display settings used in the cleaned workflow:

- hot colormap
- range `2..5`
- full overlay opacity
- T1w background dimmed with `--bg-scale 0.72`
- every 3 slices
- 10 columns per row
- FreeSurfer brain mask transformed to T1w space for crop bounds

## Lesion Visualization

The accepted simple lesion visualization is a MATLAB isosurface plot using:

```text
derivatives/surface_alltasks_run1only/<subject>/<session>/volume/<subject>_<session>_space-T1w_desc-freesurferBrain_mask.nii.gz
derivatives/mask/<subject>_<session>_lesion-mask_meldpostop.nii.gz
```

The pial-surface lesion previews and smooth glass-brain experiments were removed
because the original brain-mask isosurface version was preferred.

## Model Notes

The task GLM is intentionally simple and local:

- fMRIPrep fsnative surface BOLD files
- task HRF regressor
- intercept, linear drift, quadratic drift
- six motion regressors: `trans_x`, `trans_y`, `trans_z`, `rot_x`, `rot_y`, `rot_z`
- percent signal change scaling before GLM

For language tasks, candidate masks use:

```text
tstat >= 2.0 and beta >= 0.25
```

For breath hold, the mask uses absolute modulation:

```text
abs(tstat) >= 2.0 and abs(beta) >= 0.25
```
