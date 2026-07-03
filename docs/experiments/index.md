# fMRI QC Experiments

Stimulus presentation tasks used for scanner quality control and functional localizers. All experiments run in MATLAB with Psychtoolbox-3 on the scanner iMac.

These are not research experiments — they are short, standardized tasks designed to verify scanner BOLD sensitivity, temporal resolution, and functional localization before committing to longer protocols.

## Experiments

| Task | Design | Duration | What it tests |
|------|--------|----------|---------------|
| [Checkerboard HRF](checkerboard.md) | Event-related, 27 × 1 s flashes | 240 s (4 min) | V1 BOLD sensitivity and HRF shape |
| [Motion Localizer](motion-localizer.md) | Block, 24 × 15 s blocks | 360 s (6 min) | MT+ localization (moving vs. static dots) |
| [Six-Fingers Execution](six-fingers-execution.md) | Block, 3 blocks × 5 fingers | ~396 s (6.6 min) | Motor cortex somatotopy |

## Before you run anything

Read [Psychtoolbox on the Scanner iMac](psychtoolbox-tips.md) first. It covers the PsychVulkanCore timing issues on macOS and the workarounds baked into all three experiments. Without understanding those, the code will look unnecessarily complicated.

## Code location

All experiment code lives in the lab documentation repo:

```
brainimaging-lab-documentation/experiments/FMRI/qc/
├── checkerboard/
├── motion-localizer/
└── six-fingers-execution/
```

Each experiment follows the same template structure: `main.m` calls shared utilities in `utils/setup/` (display, parameters, keyboard, trigger, cleanup) and stimulus functions in `utils/stimuli/`.

## Output

Every experiment saves BIDS-compatible outputs:

- `*_events.tsv` — onset, duration, trial_type for each event/block
- `*_events.json` — metadata sidecar
- `*.mat` — full parameter struct (`pa`) including actual timing
- `*_dm.csv` — design matrix (where applicable)
