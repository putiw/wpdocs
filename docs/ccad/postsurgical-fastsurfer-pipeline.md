# Post-Surgical Reconstruction Pipeline (MELD-PostOp + FastSurfer)

Last updated: 2026-07-02

This page documents the two-part Jubail pipeline that auto-segments a
post-surgical resection cavity and runs cortical surface reconstruction with
lesion inpainting for a de-identified CCAD language-mapping case.

**Why this is needed:** The patient has a surgical resection cavity that
confuses standard FreeSurfer surface reconstruction. The pipeline first
generates a binary lesion/cavity mask using MELD-PostOp (an nnU-Net model
trained specifically on post-operative T1w MRI), then passes that mask to
FastSurfer's `--lesion_mask` flag, which inpaints the cavity before running
surface reconstruction.

---

## Overview

| Stage | Tool | What it does |
|---|---|---|
| A | MELD-PostOp | nnU-Net inference → binary lesion/cavity mask |
| B | FastSurfer + LIT | Inpaints cavity, runs cortical reconstruction |

Both stages run inside a single SLURM job on Jubail.

---

## One-Time Manual Setup

These steps only need to be done once. The batch script handles everything
else automatically on first run.

### 1. Download MELD-PostOp model weights

The model weights cannot be downloaded automatically (no stable direct link).

1. Open the Figshare page in a browser (see MELD-PostOp documentation for the link)
2. Download `MELD-PostOp.zip` (~2.3 GB)
3. Copy it to Jubail:

```bash
scp MELD-PostOp.zip pw1246@jubail.abudhabi.nyu.edu:/scratch/pw1246/software/MELD-PostOp/
```

The zip contains both the run scripts (`bin/`) and the model weights
(`model/Dataset001_MELDPostOp_v1.0.0/`). The batch script unzips it
automatically on first run.

### 2. License files

Both licenses must be present before running:

```text
/scratch/pw1246/software/licenses/meldpostop_license.txt
/scratch/pw1246/software/licenses/freesurfer_license.txt
```

The FreeSurfer license is the standard `license.txt` from
`freesurfer.net/registration.html`. The MELD-PostOp license is obtained from
the MELD-PostOp authors.

---

## Batch Script

```text
/scratch/pw1246/MRI/CCAD/jubail_pipeline/run_full_pipeline.sh
```

### SLURM settings

```bash
#SBATCH --job-name=avm_pipeline
#SBATCH --partition=nvidia
#SBATCH --gres=gpu:v100:1        # V100 nodes have free capacity; see GPU note below
#SBATCH --cpus-per-task=8
#SBATCH --mem=64G
#SBATCH --time=06:00:00
```

!!! warning "GPU compute capability"
    PyTorch pip wheels from 2.5+ dropped support for V100 (compute capability
    7.0). The script pins `torch==2.4.1` from the `cu118` index, which is the
    last release that compiled CC 7.0 kernels into its pip wheels. If you
    switch to an A100 node (`--gres=gpu:a100:1`), you can remove the torch
    pin and use any recent PyTorch version.

### Paths hardcoded in the script

| Variable | Value |
|---|---|
| `PROJECT_ROOT` | `/scratch/pw1246/MRI/CCAD/languageMapping` |
| `SUBJECT` | `sub-CASEPOSTOP` |
| `SES` | `ses-01` |
| `T1W` | `rawdata/sub-CASEPOSTOP/ses-01/anat/sub-CASEPOSTOP_ses-01_T1w.nii.gz` |
| `MELDPOSTOP_DIR` | `/scratch/pw1246/software/MELD-PostOp` |
| `SIF_IMAGE` | `/scratch/pw1246/software/fastsurfer-gpu.sif` |

---

## Running the Pipeline

```bash
sbatch /scratch/pw1246/MRI/CCAD/jubail_pipeline/run_full_pipeline.sh
```

Monitor progress:

```bash
squeue -u pw1246
tail -f /scratch/pw1246/MRI/CCAD/languageMapping/derivatives/pipeline_<jobid>.log
```

### Expected runtime

| Stage | Duration |
|---|---|
| Conda env + pip install (first run only) | ~10 min |
| MELD-PostOp unzip (first run only) | ~3 min |
| MELD-PostOp inference | ~1 min |
| Singularity image build (first run only) | ~10–30 min |
| FastSurfer + LIT cortical reconstruction | ~1–3 hours |

Subsequent runs skip the first-run steps (env exists, weights unzipped,
Singularity image built) and go straight to inference and reconstruction.

---

## Outputs

### MELD-PostOp lesion mask

```text
/scratch/pw1246/MRI/CCAD/languageMapping/derivatives/meld-postop/
  sub-CASEPOSTOP_ses-01_lesion-mask_meldpostop.nii.gz
```

Local copy (pulled after the pipeline completes):

```text
/Users/pw1246/Desktop/projects/support/CCAD/languageMapping/derivatives/mask/
  sub-CASEPOSTOP_ses-01_lesion-mask_meldpostop.nii.gz
```

To pull the mask locally:

```bash
scp "pw1246@jubail.abudhabi.nyu.edu:/scratch/pw1246/MRI/CCAD/languageMapping/derivatives/meld-postop/sub-CASEPOSTOP_ses-01_lesion-mask_meldpostop.nii.gz" \
    /Users/pw1246/Desktop/projects/support/CCAD/languageMapping/derivatives/mask/
```

### FastSurfer outputs

```text
/scratch/pw1246/MRI/CCAD/languageMapping/derivatives/fastsurfer-lit/
  sub-CASEPOSTOP_ses-01/
```

Key files to check after the run:

```text
mri/inpainted.lit.nii.gz          # cavity-inpainted T1w used for reconstruction
stats/lesion_impact_summary.yaml   # automated lesion impact report
surf/lh.pial, surf/rh.pial         # cortical surfaces
```

---

## QC Checklist

Before trusting any downstream surface-based GLM or atlas ROI stats:

- [ ] Open `sub-CASEPOSTOP_ses-01_lesion-mask_meldpostop.nii.gz` overlaid on
  the T1w in freeview or ITK-SNAP and confirm the mask covers the resection
  cavity and nothing else
- [ ] Check `stats/lesion_impact_summary.yaml` for the parcels affected
- [ ] Inspect `mri/inpainted.lit.nii.gz` — the cavity should look filled with
  plausible tissue signal
- [ ] Spot-check pial surfaces in freeview; the boundary near the cavity
  should follow the cortex rather than dipping into the resection hole

---

## Troubleshooting

### git clone fails into non-empty directory (exit 128)

The MELD-PostOp zip must be staged in `$MELDPOSTOP_DIR` before the first run.
An older version of the script tried to `git clone` into that directory, which
fails when the zip is already there. The current script bypasses git entirely
and unzips instead.

### `CUDA error: no kernel image is available for execution on the device`

The job landed on a V100 node and the installed PyTorch version does not
include CC 7.0 kernels. PyTorch pip wheels 2.5+ dropped CC 7.0 support.
The script pins `torch==2.4.1` from the `cu118` index to fix this. If this
error reappears, confirm the conda env has torch 2.4.x:

```bash
/scratch/pw1246/conda-envs/meldpostop/bin/python -c "import torch; print(torch.__version__)"
```

If it shows 2.5+, remove the env and resubmit:

```bash
conda env remove -n meldpostop -y
sbatch /scratch/pw1246/MRI/CCAD/jubail_pipeline/run_full_pipeline.sh
```

### `MKL_INTERFACE_LAYER: unbound variable` during conda deactivate

The cluster's MKL activation script uses `$MKL_INTERFACE_LAYER` without
checking if it is set. With `set -euo pipefail` active, this kills the job.
The script wraps `conda deactivate` as:

```bash
set +u; conda deactivate; set -u
```

### Job stuck in queue with A100 request

A100 nodes (`cn*`) can have long queues during peak hours. V100 nodes (`dn*`)
typically have free capacity but require the torch 2.4.1 pin described above.
Check current availability with:

```bash
sinfo -p nvidia --states=idle,mix -o '%N %G %t'
```

---

## Software Versions (as of 2026-07-02)

| Software | Version |
|---|---|
| MELD-PostOp model | Dataset001_MELDPostOp_v1.0.0 |
| nnunetv2 | 2.8.1 |
| torch | 2.4.1+cu118 |
| Python | 3.12 |
| FastSurfer | latest (via `docker://deepmi/fastsurfer:latest`) |
| CUDA module | 11.8.0 |
| Jubail partition | nvidia (V100 nodes `dn*`) |
