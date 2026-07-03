# CCAD AVM Language Mapping Pilot

This note documents the single-subject pilot workflow used for CCAD AVM
language mapping data in:

```text
/Users/pw1246/Desktop/projects/support/CCAD/languageMapping
```

The current pilot subject is `sub-test20260630`, session `ses-01`.

## DICOM to BIDS Conversion

The pilot DICOM conversion files are kept next to the local project code:

```text
/Users/pw1246/Desktop/projects/support/CCAD/languageMapping/code/language_mapping_dcm2bids_config.json
/Users/pw1246/Desktop/projects/support/CCAD/languageMapping/code/convert_language_mapping_dicoms_to_bids.py
/Users/pw1246/Desktop/projects/support/CCAD/languageMapping/code/scan_inventory_Localizers_20260630.md
```

Run the conversion and sync rawdata back locally with:

```bash
/Users/pw1246/Desktop/projects/support/CCAD/languageMapping/code/convert_language_mapping_dicoms_to_bids.py --convert
```

By default the script runs remote `dcm2bids` on Asif's iMac and then syncs:

```text
remote: /Volumes/fMRI/Projects/languageMapping/rawdata/
local:  /Users/pw1246/Desktop/projects/support/CCAD/languageMapping/rawdata/
```

Use `--no-sync` only when you intentionally want to leave converted rawdata on
the CCAD-side machine.

## Scan Selection Notes

The scanner exported useful original acquisitions plus scanner-derived outputs.
The default config should include the original T1w, BOLD, and fieldmap series and
exclude derived `MoCoSeries`, `EvaSeries_GLM`, `Mean_&_t-Maps`, `Design`, and
`PhoenixZIPReport` outputs.

Key decisions from the scan inventory:

- Use T1 series 9 as the canonical `T1w`; series 10 is the normalized duplicate.
- Use BOLD series 11 and 17 for breath-hold and language.
- Use the SE AP/PA topup pair for both BOLD runs. Matrix, FOV/pixel spacing,
  slice thickness/spacing, orientation, first slice position, and 32-slice
  prescription match both BOLD runs.
- The two GRE fieldmap sets are not identical. Series 2/3 and 7/8 share the same
  GRE protocol and matrix, but have different first-slice positions and timing.
  Use the later 7/8 pair by default because it is closer to the usable functional
  acquisitions.
- Remote tools are present but not necessarily in a plain noninteractive SSH
  `PATH`: `dcm2niix=/Users/imac1/bin/dcm2niix` and
  `dcm2bids=/Users/imac1/opt/anaconda3/bin/dcm2bids`.

## Goal

The working clinical question is whether the sentence-completion fMRI task
identifies left-hemisphere language cortex in native anatomy, and whether the
result can be inspected both on the native FreeSurfer surface and in subject
T1w space for AVM/tumor-adjacent planning review.

For this pilot, the most useful display is the native-surface language result
smoothed on cortex, then projected into T1w space. The direct voxelwise T1w GLM
map was noisier, so it is kept as a secondary product.

## Main MATLAB entry point

Use this launcher to reopen the two main views:

```matlab
cd('/Users/pw1246/Desktop/projects/support/CCAD/languageMapping/code/lm_bh_pilot')
view_ccad_avm_language_pilot
```

This opens:

- Freeview: left `lh.inflated` fsnative surface with sentence-completion
  t-stat overlay, smoothed 10 mm and displayed at `t = 4..8`.
- ITK-SNAP: subject T1w with the surface-smoothed LH language result projected
  into T1w space, also clipped/displayed at `t = 4..8`, plus a `t >= 4` mask.

Open just one view if needed:

```matlab
view_ccad_avm_language_pilot('surface')
view_ccad_avm_language_pilot('t1w')
```

## Recompute Surface-to-T1w Projection

The surface-to-volume projection is reproducible with:

```bash
/Users/pw1246/Desktop/projects/support/CCAD/languageMapping/code/lm_bh_pilot/project_language_surface_to_t1w.sh
```

The script smooths each fsnative hemisphere by 10 mm FWHM using FreeSurfer,
fills the FreeSurfer cortical ribbon, transforms the result into fMRIPrep T1w
space, and writes both bilateral and left-hemisphere-only outputs.

## Key Outputs

Native surface language t-stat:

```text
/Users/pw1246/Desktop/projects/support/CCAD/languageMapping/derivatives/lm_bh_pilot/sub-test20260630/ses-01/surface/sub-test20260630_ses-01_task-language_hemi-L_space-fsnative_stat-tstat.mgz
```

Left-hemisphere surface-projected T1w display map:

```text
/Users/pw1246/Desktop/projects/support/CCAD/languageMapping/derivatives/lm_bh_pilot/sub-test20260630/ses-01/volume/sub-test20260630_ses-01_task-language_hemi-L_space-T1w_desc-fromSurfaceSmooth10TstatRange4to8_stat-tstat.nii.gz
```

Left-hemisphere surface-projected T1w mask:

```text
/Users/pw1246/Desktop/projects/support/CCAD/languageMapping/derivatives/lm_bh_pilot/sub-test20260630/ses-01/volume/sub-test20260630_ses-01_task-language_hemi-L_space-T1w_desc-fromSurfaceSmooth10TstatGte4_mask.nii.gz
```

Surface screenshots:

```text
/Users/pw1246/Desktop/projects/support/CCAD/languageMapping/derivatives/language/*smooth10_range4to8.png
```

## Notes

- The task model is sentence completion versus rest/off.
- The display threshold used for the cleaned review products is `t >= 4`, with
  the overlay display capped at `t = 8`.
- The surface-to-T1w map is a visualization/review product derived from the
  smoothed fsnative surface statistic. It should not be treated as an
  independent voxelwise GLM estimate.
- Template tract/DSI Studio outputs were removed from this pilot folder. Future
  tract review should use patient diffusion data when available.
