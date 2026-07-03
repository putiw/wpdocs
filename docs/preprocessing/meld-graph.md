# MELD Graph — FCD Detection for CCAD Epilepsy

MELD Graph (Multi-centre Epilepsy Lesion Detection) is a surface-based Graph Neural Network pipeline for automated detection of focal cortical dysplasias (FCDs) from structural MRI. It is an alternative to MAP18 that does not require MATLAB or a proprietary license.

- **GitHub:** <https://github.com/MELDProject/meld_graph>
- **Paper:** [Ripart et al., 2025 JAMA Neurology](https://jamanetwork.com/journals/jamaneurology/fullarticle/2830410)
- **Docs:** <https://meld-graph.readthedocs.io>

## Where things live

| Item | Path |
|------|------|
| Patient T1w input (BIDS) | `Box/projects/CCAD/epilepsy/BIDSepi/rawdata/` |
| MELD PDF reports (Box) | `Box/projects/CCAD/epilepsy/BIDSepi/derivatives/meld_graph/predictions_reports/` |
| Utility scripts (local Mac) | `/Users/pw1246/Desktop/projects/support/CCAD/map18/scripts/` |
| MELD data on monai | `/home/mri/Documents/MRI/CCAD/map18/meld_data/` |
| MELD install on monai | `/home/mri/software/meld_graph/` |
| FreeSurfer 7.2 on monai | `/home/mri/software/freesurfer/` |
| SSH to monai | `ssh -p 4410 mri@monai.abudhabi.nyu.edu` |

Current report-facing MELD outputs are in Box at:
```
/Users/pw1246/Library/CloudStorage/Box-Box/projects/CCAD/epilepsy/BIDSepi/derivatives/meld_graph/predictions_reports/
```

As of the latest local Box check, the consolidated report folder contains 44
MELD prediction NIfTIs.

## Quick-start: running a new patient

Two scripts handle the entire workflow from a local T1 file to a PDF report in Box.

### Step 1 — Send T1 to monai and launch pipeline

```bash
bash /Users/pw1246/Desktop/projects/support/CCAD/map18/scripts/meld_run.sh /path/to/T1.nii.gz [subject_id]
```

If the filename starts with `sub-XXX_`, the subject ID is auto-derived:

```bash
# ID derived automatically
bash meld_run.sh /path/to/sub-CASE001_ses-01_T1w.nii.gz

# Or provide explicitly for arbitrary filenames
bash meld_run.sh /path/to/Patient_T1.nii.gz sub-CASE001
```

The script:

1. Creates the BIDS directory structure on monai under `meld_data/input/`
2. Rsyncs the T1 (renamed to BIDS convention) to monai
3. Ensures `meld_bids_config.json` is present on monai
4. Writes a pipeline script on monai and launches it in a detached `screen` session

FreeSurfer `recon-all` takes **4–5 hours**. The screen session runs the full pipeline end-to-end.

### Step 2 — Check and fetch results

```bash
bash /Users/pw1246/Desktop/projects/support/CCAD/map18/scripts/meld_fetch.sh sub-CASE001
```

If the PDF is not ready, the script reports whether the screen is still alive and shows the last log lines.

If the PDF is ready, it rsyncs the full report folder to Box and shows cluster detection status.

## Running a batch of subjects

For larger batches, send data and launch the pipeline directly on monai.

### Transfer data to monai

```bash
# Loop subject-by-subject — do NOT pass multiple paths to a single rsync call
for SUBJ in sub-CASE001 sub-CASE002 sub-CASE003; do
  rsync -az --progress -e "ssh -p 4410" \
    "/Users/pw1246/Library/CloudStorage/Box-Box/projects/CCAD/epilepsy/BIDSepi/rawdata/${SUBJ}/" \
    "mri@monai.abudhabi.nyu.edu:/home/mri/Documents/MRI/CCAD/map18/meld_data/input/${SUBJ}/"
done

# Also send the BIDS config
rsync -az -e "ssh -p 4410" \
  "/Users/pw1246/Library/CloudStorage/Box-Box/projects/CCAD/epilepsy/BIDSepi/rawdata/meld_bids_config.json" \
  "mri@monai.abudhabi.nyu.edu:/home/mri/Documents/MRI/CCAD/map18/meld_data/input/"
```

!!! warning "Loop, not a single rsync call"
    Passing many paths to one rsync with shell expansion fails silently (exit code 23). Loop subject-by-subject.

### Create subjects list and launch on monai

SSH to monai, then:

```bash
# Create subjects list
cat > /home/mri/Documents/MRI/CCAD/map18/meld_data/input/subjects_list_batch.txt << 'EOF'
sub-CASE001
sub-CASE002
sub-CASE003
EOF

# Write pipeline script
cat > /tmp/run_meld_batch.sh << 'SCRIPT'
#!/bin/bash
export FREESURFER_HOME=/home/mri/software/freesurfer
source $FREESURFER_HOME/SetUpFreeSurfer.sh
export MELD_LICENSE=/home/mri/software/meld_graph/meld_license.txt
export PATH=/opt/miniconda3/envs/meld_graph/bin:$PATH
cd /home/mri/software/meld_graph
/opt/miniconda3/envs/meld_graph/bin/python scripts/new_patient_pipeline/new_pt_pipeline.py \
    -ids /home/mri/Documents/MRI/CCAD/map18/meld_data/input/subjects_list_batch.txt \
    --parallelise \
    2>&1 | tee /home/mri/Documents/MRI/CCAD/map18/meld_data/meld_run_batch.log
SCRIPT

chmod +x /tmp/run_meld_batch.sh
screen -dmS meld_batch bash /tmp/run_meld_batch.sh
screen -ls   # confirm it's running
```

The log file will appear empty during recon-all due to Python stdout buffering in screen — that is normal.

### Pull all results to Box after completion

```bash
rsync -avz --progress -e "ssh -p 4410" \
  "mri@monai.abudhabi.nyu.edu:/home/mri/Documents/MRI/CCAD/map18/meld_data/output/predictions_reports/" \
  "/Users/pw1246/Library/CloudStorage/Box-Box/projects/CCAD/epilepsy/BIDSepi/derivatives/meld_graph/predictions_reports/"
```

## Monitoring a running batch

**Is the screen still running?**

```bash
ssh -p 4410 mri@monai.abudhabi.nyu.edu "screen -ls"
```

**How many recon-alls are active right now?**

```bash
ssh -p 4410 mri@monai.abudhabi.nyu.edu "ps aux | grep recon-all | grep -v grep | wc -l"
```

**Is a specific subject making progress?**

Check log modification timestamps rather than CPU% (recon-all shows 0% CPU during I/O phases even when it is actively running):

```bash
ssh -p 4410 mri@monai.abudhabi.nyu.edu \
  "ls -lt /home/mri/Documents/MRI/CCAD/map18/meld_data/output/fs_outputs/sub-CASE001/scripts/ | head -5"
```

Files modified within the last few minutes confirm active progress.

**How many PDFs are done?**

```bash
ssh -p 4410 mri@monai.abudhabi.nyu.edu \
  "find /home/mri/Documents/MRI/CCAD/map18/meld_data/output/predictions_reports -name '*.pdf' | wc -l"
```

!!! warning "Count PDFs, not folders"
    Each subject folder contains `predictions/` (NIfTI overlays) and `reports/` (PDF). The `reports/` folder is created before the PDF is written. Count `*.pdf` files to confirm completion, not folder count.

## Pipeline stages and timing

| Stage | Duration | Notes |
|-------|----------|-------|
| FreeSurfer `recon-all` | ~4–5 hours per subject | All subjects in the batch run in parallel |
| Feature extraction | ~15–30 min | After all recon-alls complete |
| GNN prediction | ~30–60 min | Runs as a single batch for all subjects |
| PDF generation | Included in prediction | All PDFs appear together at the end |

!!! info "`--parallelise` means all subjects simultaneously"
    The `--parallelise` flag starts a separate recon-all process for every subject at once. On monai (128 CPUs, 502 GB RAM) this is fine for batches of 30+. A batch of 33 subjects takes ~5–6 hours wall time.

## Key requirements and gotchas

### FreeSurfer version

MELD Graph requires **FreeSurfer 7.2.0** exactly. It is incompatible with 7.3+. FreeSurfer 7.2 is installed at `/home/mri/software/freesurfer` on monai alongside any other versions.

### BIDS config must declare the session

`meld_bids_config.json` must include `"session": "01"` for both T1 and FLAIR entries. Without this, the pipeline cannot find input files even if they exist on disk.

```json
{
  "T1": {
    "session": "01",
    "datatype": "anat",
    "suffix": "T1w"
  },
  "FLAIR": {
    "session": "01",
    "datatype": "anat",
    "suffix": "FLAIR"
  }
}
```

### FSL PATH hijack on monai

FSL at `/home/mri/fsl/bin` appears early in `PATH` and overrides the conda Python. Export the conda bin path **after** sourcing `SetUpFreeSurfer.sh`:

```bash
export FREESURFER_HOME=/home/mri/software/freesurfer
source $FREESURFER_HOME/SetUpFreeSurfer.sh
export PATH=/opt/miniconda3/envs/meld_graph/bin:$PATH   # must come after FreeSurfer source
```

Always use the full Python path rather than relying on `python` in `PATH`.

### Pipeline skips existing recon-all outputs

If `fs_outputs/<subject_id>/` already exists on monai, the pipeline skips recon-all and goes straight to feature extraction. This means re-running to get a report takes ~30–60 min instead of 5+ hours.

### Batch prediction waits for all recon-alls

If one subject takes longer than the others, prediction waits for it. All PDFs appear together at the end, never incrementally.

## Output structure

MELD outputs are stored in two places under `BIDSepi/derivatives/`:

### 1. `meld_graph/predictions_reports/` — full pipeline outputs

This is the canonical MELD output folder, populated directly by `meld_fetch.sh`.

```
meld_graph/
└── predictions_reports/
    └── sub-CASE001/
        ├── predictions/
        │   ├── lh.prediction.nii.gz              # Left hemisphere surface prediction
        │   ├── rh.prediction.nii.gz              # Right hemisphere surface prediction
        │   └── prediction.nii.gz                 # Whole-brain prediction volume
        └── reports/
            ├── MELD_report_sub-CASE001.pdf      # Main PDF report
            ├── inflatbrain_sub-CASE001.png       # Inflated brain thumbnail
            └── info_clusters_sub-CASE001.csv    # Cluster table (empty = no clusters detected)
```

The PDF shows: predicted FCD clusters on inflated brain surface, clusters overlaid on native T1w in axial/coronal/sagittal views, saliency maps, and cluster statistics.

The CSV has one row per detected cluster. If it contains only a header line (or is empty), no clusters were detected.

### 2. `report/` — shared report export folder

This folder consolidates report-facing outputs from multiple pipelines (MAP18, MAP18-like, and MELD) into one place for review or sharing. It is not MELD-specific.

```
report/
├── README.txt
├── report_manifest.csv
└── sub-CASE001/
    ├── MELD_report_sub-CASE001.pdf           # MELD PDF (copied from meld_graph/)
    ├── CASE001 Report.pptx                   # Amal MAP18 report, if available
    └── nifti/
        ├── sub-CASE001_ses-01_space-T1w_desc-MELD_prediction.nii.gz    # MELD prediction NIfTI
        ├── sub-CASE001_ses-01_space-T1w_desc-m3_GWjunction.nii         # MAP18-like junction map
        ├── sub-CASE001_ses-01_space-T1w_desc-m5_GWjunction.nii         # Current final MAP18-like junction map
        ├── sub-CASE001_ses-01_space-T1w_desc-map18_GWjunction.hdr      # Original MAP18 (if available)
        └── open.command                                                 # macOS double-click to open in ITK-SNAP
```

`desc-m3` and `desc-m5` refer to local MAP18-like model variants (see the MAP18-like pipeline page for definitions). `desc-m5` is the current final report-facing MAP18-like result; `desc-m3` is kept for comparison. Not every subject will have all file types — the folder is populated as outputs become available from each pipeline.

!!! note "Where to look for a subject's results"
    - To get the MELD PDF only: `report/sub-SUBJECTID/MELD_report_*.pdf`
    - To get all MELD outputs (NIfTIs + PDF + cluster CSV): `meld_graph/predictions_reports/sub-SUBJECTID/`
    - To see combined outputs from all pipelines side-by-side: `report/sub-SUBJECTID/nifti/`

## Processed Subjects

Do not list patient names or real MRNs in public docs. To see which cases are
currently processed, count the report outputs or consult the private project
inventory.

Report-facing MELD prediction count:

```bash
find /Users/pw1246/Library/CloudStorage/Box-Box/projects/CCAD/epilepsy/BIDSepi/derivatives/report \
  -path '*/nifti/*_desc-MELD_prediction.nii.gz' -type f | wc -l
```

FreeSurfer `fs_outputs/` for processed subjects remain on monai and do not need
to be re-run unless the input T1w changes or the pipeline needs a fresh
reconstruction.

## monai setup reference

Full install details are in:
```
/Users/pw1246/Desktop/projects/support/CCAD/map18/scripts/install_meld_monai.sh
```

Key locations on monai:

| Item | Path |
|------|------|
| Conda environment | `/opt/miniconda3/envs/meld_graph/` |
| Python binary | `/opt/miniconda3/envs/meld_graph/bin/python` |
| MELD source | `/home/mri/software/meld_graph/` |
| FreeSurfer 7.2.0 | `/home/mri/software/freesurfer/` |
| MELD license | `/home/mri/software/meld_graph/meld_license.txt` |
| FreeSurfer license | `/home/mri/software/freesurfer/license.txt` |
| MELD data root | `/home/mri/Documents/MRI/CCAD/map18/meld_data/` |
| BIDS input | `/home/mri/Documents/MRI/CCAD/map18/meld_data/input/` |
| Reports output | `/home/mri/Documents/MRI/CCAD/map18/meld_data/output/predictions_reports/` |
| FreeSurfer outputs | `/home/mri/Documents/MRI/CCAD/map18/meld_data/output/fs_outputs/` |

Monai specs: RHEL 8, 128 CPUs, 502 GB RAM.

## References

- Ripart et al., 2025. Detection of epileptogenic focal cortical dysplasia using graph neural networks. *JAMA Neurology*.
- Spitzer, Ripart et al., 2022. Interpretable surface-based detection of focal cortical dysplasias: a MELD study. *Brain*, 145(11), 3859–3871.
- [MELD Project website](https://meldproject.github.io/)
- [MELD Graph documentation](https://meld-graph.readthedocs.io)
