# CCAD Epilepsy MAP18-Like Pipeline

Last updated: 2026-06-23

This page records the current CCAD epilepsy morphometric MRI workflow. The goal is to approximate the MAP18-style T1 morphometry outputs described by Huppertz and colleagues, while keeping all generated patient-facing results organized in `BIDSepi/derivatives/report`.

## Current Status

| Item | Current state | Path / count |
|---|---|---|
| Patient BIDS-style T1w data | Curated and actively updated | `/Users/pw1246/Library/CloudStorage/Box-Box/projects/CCAD/epilepsy/BIDSepi/rawdata` |
| Raw BIDSepi T1w files | 46 files currently present; report outputs are still actively updated | `sub-*/ses-01/anat/*_T1w.nii.gz` |
| Final report-facing VBM model | Model 5 junction WM=1.0 SD map | `BIDSepi/derivatives/report/sub-*/nifti/*_desc-m5_GWjunction.nii` |
| Final report folder | 44 M5, 43 M3, 44 MELD prediction NIfTIs currently present | `/Users/pw1246/Library/CloudStorage/Box-Box/projects/CCAD/epilepsy/BIDSepi/derivatives/report` |
| Original MAP18 outputs | Copied when available | `report/sub-*/nifti/*_desc-map18_GWjunction.hdr/.img` |
| MELD Graph | 44 report-facing prediction NIfTIs currently present | `BIDSepi/derivatives/meld_graph/predictions_reports` |
| One-subject wrapper | Added for new T1w inputs | `/Users/pw1246/Documents/GitHub/pipelines/ccad/scripts/vbm_run.sh` |

## Final Report Folder

The report folder is the place to share with CCAD doctors.

```text
/Users/pw1246/Library/CloudStorage/Box-Box/projects/CCAD/epilepsy/BIDSepi/derivatives/report/
```

Expected subject layout:

```text
report/
└── sub-MRNxxxx/
    ├── MELD_report_sub-MRNxxxx.pdf                  # if available
    ├── MRNxxxx Report.pptx                          # Amal/CCAD MAP18 report, if available
    └── nifti/
        ├── sub-MRNxxxx_ses-01_space-T1w_desc-map18_GWjunction.hdr
        ├── sub-MRNxxxx_ses-01_space-T1w_desc-map18_GWjunction.img
        ├── sub-MRNxxxx_ses-01_space-T1w_desc-m3_GWjunction.nii
        ├── sub-MRNxxxx_ses-01_space-T1w_desc-m5_GWjunction.nii
        ├── sub-MRNxxxx_ses-01_space-T1w_desc-MELD_prediction.nii.gz
        └── open.command
```

`open.command` launches the available NIfTI/Analyze files in ITK-SNAP for side-by-side review. The final report-facing MAP18-like result is `desc-m5_GWjunction.nii`.

## One-Subject Run Command

For a new BIDS-style or standalone T1w NIfTI-GZ:

```bash
bash /Users/pw1246/Documents/GitHub/pipelines/ccad/scripts/vbm_run.sh /path/to/t1w.nii.gz
```

The wrapper:

1. Stages the input into `BIDSepi/rawdata/sub-*/ses-01/anat/` if it is not already there.
2. Runs SPM12 samp2/regflex segmentation.
3. Computes Model 5 feature maps.
4. Applies the local Model 5 normative statistics.
5. Exports MNI and native-space maps.
6. Copies the native WM=1.0 junction z-map into `BIDSepi/derivatives/report/sub-*/nifti/`.

If the script derives the wrong subject name from a standalone filename, rename the input to BIDS style first, for example:

```text
sub-CASE001_ses-01_T1w.nii.gz
```

## Main Folder Layout

| Folder | Purpose |
|---|---|
| `/Users/pw1246/Library/CloudStorage/Box-Box/projects/CCAD/epilepsy/BIDSepi/rawdata` | Patient T1w inputs. |
| `/Users/pw1246/Library/CloudStorage/Box-Box/projects/CCAD/epilepsy/BIDSepi/derivatives/spm12_segment_map18like_samp2_regflex` | Patient SPM12 segmentation outputs used by Models 3-5. |
| `/Users/pw1246/Library/CloudStorage/Box-Box/projects/CCAD/epilepsy/BIDSepi/derivatives/map18_like_model5` | Patient Model 5 feature and z-map outputs. |
| `/Users/pw1246/Library/CloudStorage/Box-Box/projects/CCAD/epilepsy/BIDSnorm/derivatives/map18_like_model5/normative_stats` | Local Model 5 normative mean/SD/effective-mask maps. |
| `/Users/pw1246/Library/CloudStorage/Box-Box/projects/CCAD/epilepsy/BIDSepi/derivatives/report` | Final doctor-facing output folder. |
| `/Users/pw1246/Documents/CCAD` | Working MATLAB, shell, and HPC scripts. |
| `/Users/pw1246/Documents/GitHub/pipelines/ccad/scripts` | Reusable project scripts, including `vbm_run.sh`. |
| `/Users/pw1246/Documents/CCAD/tools/spm12-maca64` | Compiled Apple Silicon SPM12 used locally. |

## Model Definitions

| Model | Purpose | Current use |
|---|---|---|
| Model 1 | Early ARI-only MAP18-like implementation. | Kept as development history only. |
| Model 2 | SPM samp2/regflex segmentation; closer to Amal MAP18 intermediate GM maps. | Kept as development history only. |
| Model 3 | Samp2/regflex plus WM=1.0 junction threshold and no post-convolution hard mask zeroing. ARI normative database. | Kept in report for comparison as `desc-m3_GWjunction.nii`. |
| Model 4 | Model 3-style features with broader controls: ARI + ds004199 HC + first 100 HCPYA. | Replaced in report by Model 5. |
| Model 5 | Model 4-style features with all available controls and MAP18-derived soft/deep effective mask. | Current final report-facing model, `desc-m5_GWjunction.nii`. |

## Model 5 Method Summary

Model 5 uses the same SPM12 segmentation settings selected after comparison with Amal MAP18 outputs:

| Component | Setting |
|---|---|
| SPM version | Local Apple Silicon SPM12 at `/Users/pw1246/Documents/CCAD/tools/spm12-maca64`; r6217-style scripts also used on Jubail for control processing. |
| Normalized grid | `181 x 217 x 181`, `1 mm` MNI grid. |
| SPM deformation settings | `samp = 2`, flexible regularization, `affreg = mni`. |
| Patient native export | Uses inverse deformation `iy_*_T1w.nii` to export z-maps back to subject T1w space. |
| Junction lower threshold | `mean(GM) + 0.5 * SD(GM)`. |
| Junction upper threshold variants | WM=0.5 and WM=1.0 are both generated; the final report uses WM=1.0. |
| Junction convolution | Binary junction image convolved with a `5 x 5 x 5` kernel of ones. |
| Extension image | Normalized GM probability smoothed with `6 mm` FWHM. |
| Thickness image | Binary GM run-length thickness approximation; generated for Model 5 but not currently copied to the report folder. |
| Normative z-score | `(patient_feature - normal_mean) / smoothed_normal_SD`. |
| Final effective mask | MAP18-derived soft/deep MNI mask is applied to the final z-map before native export. |

Model 5 normative controls are:

```text
ARI + ds004199 healthy controls + all HCPYA controls = 1618 controls
```

Normative stats live here:

```text
/Users/pw1246/Library/CloudStorage/Box-Box/projects/CCAD/epilepsy/BIDSnorm/derivatives/map18_like_model5/normative_stats
```

Important Model 5 normative outputs:

```text
Model5_ARIplusDS004199HCplusHCPYAall_junction_wm10_mean.nii
Model5_ARIplusDS004199HCplusHCPYAall_junction_wm10_s6sd.nii
Model5_ARIplusDS004199HCplusHCPYAall_junction_wm10_effective_MAP18softDeepMask.nii
Model5_ARIplusDS004199HCplusHCPYAall_extension_mean.nii
Model5_ARIplusDS004199HCplusHCPYAall_thickness_mean.nii
```

## Key Scripts

| Step | Script |
|---|---|
| One-subject VBM/M5 report wrapper | `/Users/pw1246/Documents/GitHub/pipelines/ccad/scripts/vbm_run.sh` |
| Patient SPM samp2/regflex segmentation | `/Users/pw1246/Documents/CCAD/run_spm12_segment_ccad_map18like_samp2_regflex_subjects.m` |
| Patient SPM parallel wrapper | `/Users/pw1246/Documents/CCAD/run_spm12_segment_ccad_map18like_samp2_regflex_parallel.sh` |
| Model 5 patient feature generation | `/Users/pw1246/Documents/CCAD/run_model5_one_patient_feature.sh` |
| Model 5 feature MATLAB implementation | `/Users/pw1246/Documents/CCAD/jubail_model5_features_subject.m` |
| Model 5 patient z/native export | `/Users/pw1246/Documents/CCAD/run_model5_patient_z_and_native.m` |
| Model 5 patient batch wrapper | `/Users/pw1246/Documents/CCAD/run_model5_patient_z_parallel.sh` |
| Finalize Model 5 from Jubail/local outputs | `/Users/pw1246/Documents/CCAD/finalize_model5_locally.sh` |
| Replace report M4 with M5 | `/Users/pw1246/Documents/CCAD/update_report_replace_m4_with_m5.sh` |
| Doctor-facing 3D/flat cluster visualization example | `/Users/pw1246/Documents/CCAD/plot_native_3d_glassbrain_<case_id>.m` |

## Paper Step Mapping

| MAP18 paper step | Local status |
|---|---|
| High-resolution T1 input | Implemented through BIDSepi rawdata curation. Use caution for external 2D/non-MPRAGE scans. |
| Bias correction and spatial normalization | Implemented with SPM12 unified segmentation and explicit 1 mm MNI writing. |
| GM/WM/CSF segmentation | Implemented with SPM12 `c1/c2/c3` and `wc1/wc2/wc3` outputs. |
| Junction binary image | Implemented from normalized bias-corrected T1 and GM/WM intensity thresholds. |
| Junction convolution | Implemented with a `5 x 5 x 5` kernel. |
| Junction z-score | Implemented for Model 5 using 1618-control normative statistics and smoothed SD. |
| Extension z-score | Implemented in Model 5, but not part of the current doctor-facing report export. |
| Thickness z-score | Implemented in Model 5 as a run-length approximation, but not part of the current doctor-facing report export. |
| Native/report-space outputs | Implemented for Model 5 with inverse deformation back to T1w space. |

## Cautions

- The pipeline assumes a T1-weighted 3D acquisition broadly comparable to the normative controls. Thick-slice 2D scans and synthetic reconstructions can be processed for exploratory review, but should not be interpreted diagnostically.
- For a thick-slice 2D T1 case, use language such as: “This exploratory, non-validated result was generated from a thick-slice 2D T1 scan; apparent findings may reflect interpolation and partial-volume artifacts and should not be interpreted diagnostically.”
- Siemens `SPACE` is a sequence name. If a scan is clinically usable but not MPRAGE, describe it as a “3D isotropic T1-weighted inversion-recovery SPACE acquisition” rather than assuming all clinicians will treat it as equivalent to MPRAGE.
- `desc-m5_GWjunction.nii` is the current local MAP18-like output; it is not the proprietary MAP18 software output.

## Verification Commands

Count report-facing Model 5 maps:

```bash
find /Users/pw1246/Library/CloudStorage/Box-Box/projects/CCAD/epilepsy/BIDSepi/derivatives/report -path '*/nifti/*_desc-m5_GWjunction.nii' -type f | wc -l
```

Count report-facing M3 maps:

```bash
find /Users/pw1246/Library/CloudStorage/Box-Box/projects/CCAD/epilepsy/BIDSepi/derivatives/report -path '*/nifti/*_desc-m3_GWjunction.nii' -type f | wc -l
```

Count report-facing MELD prediction maps:

```bash
find /Users/pw1246/Library/CloudStorage/Box-Box/projects/CCAD/epilepsy/BIDSepi/derivatives/report -path '*/nifti/*_desc-MELD_prediction.nii.gz' -type f | wc -l
```

Check Model 5 normative stats:

```bash
ls /Users/pw1246/Library/CloudStorage/Box-Box/projects/CCAD/epilepsy/BIDSnorm/derivatives/map18_like_model5/normative_stats/Model5_ARIplusDS004199HCplusHCPYAall_*_stats_summary.txt
```

## Related Pages

- [CCAD Epilepsy BIDS Conversion](epilepsy-bids.md)
- [MELD Graph](../preprocessing/meld-graph.md)
- [CCAD Contacts and Systems](server-paths.md)
