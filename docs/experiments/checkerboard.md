# Checkerboard HRF Estimation

A short event-related task that flashes a full-field contrast-reversing checkerboard to estimate the hemodynamic response function in V1. Used to compare BOLD sensitivity and temporal resolution across scanners.

## Purpose

Estimate the HRF shape in primary visual cortex. Because the stimulus is simple and the V1 response is large and reliable, differences in the recovered HRF reflect scanner hardware (coil, field strength, shimming) rather than task or subject variability. Running the same design across scanners gives a direct comparison of BOLD sensitivity.

## Design

Event-related with jittered inter-stimulus intervals:

- 27 checkerboard flash events, each 1.0 s
- Contrast-reversing at 8 Hz (flicker)
- ISI drawn uniformly from 4–12 s, adjusted to hit 240 s total
- 12 s initial baseline, 12 s final baseline
- Total design duration: **240 s** (4 minutes)
- Three fixed designs (A, B, C) with different random seeds for ISI sequences

The 240 s duration is divisible by both 0.75 s TR (320 volumes) and 2 s TR (120 volumes).

## Scan Parameters

| Parameter | Recommended | Notes |
|-----------|-------------|-------|
| TR | **0.75 s** | Sub-second TR gives ~6 samples across the HRF rise vs ~2 at 2 s TR. Critical for resolving HRF peak and undershoot. |
| Volumes | 320 (at 0.75 s) or 120 (at 2 s) | |
| Resolution | Standard (2 mm or whatever matches your protocol) | Not resolution-sensitive — V1 activation is large. |

## Attention Task

The fixation cross is always red (no dimming task in the current design). Participants are instructed to maintain fixation.

## Code Structure

```
checkerboard/
├── main.m                              # Entry point
├── utils/
│   ├── setup/
│   │   ├── setup_display.m             # Screen/window setup
│   │   ├── setup_param.m               # Experiment parameters
│   │   ├── setup_keyboard.m            # Key mappings
│   │   ├── load_checkerboard_design.m  # ISI sequences (designs A/B/C)
│   │   ├── wait_trigger.m              # Scanner trigger or manual start
│   │   ├── cleanup_experiment.m        # Save data, close devices
│   │   └── get_info.m                  # BIDS subject/session info
```

## Key Implementation Details

Checkerboard textures (phase 1 and phase 2) are pre-computed and preloaded to VRAM before the trigger. During each event, the code alternates between the two textures at 8 Hz using a frame counter and `framesPerFlickerPhase = round(frameRate / 16)`.

The event loop follows the pattern described in [Psychtoolbox Tips](psychtoolbox-tips.md):

1. ISI: continuous draw+flip of fixation to keep GPU warm
2. Pre-draw first checkerboard frame + `DrawingFinished` before onset
3. `WaitSecs('UntilTime')` + bare `Flip` at onset
4. Lock `eventEndAbs` to actual onset via `GetSecs` (guarantees full 1 s duration)
5. Flicker loop with bare Flips until `eventEndAbs`
6. ISI flip, log actual duration

## Running

```matlab
cd checkerboard
main
```

A dialog prompts for subject ID, session, and run number (BIDS fields). Then a design selection dialog (A, B, or C). In debug mode, press `t` to trigger.

## Output

- `*_events.tsv` — onset, duration, trial_type for each flash
- `*_events.json` — metadata including design ID and seed
- `*.mat` — full `pa` struct with actual timing, planned onsets, ISI sequence
- `*_dm.csv` — planned design with ISI durations and dim schedule

## References

- Glover (1999) *NeuroImage* — Deconvolution of impulse response in event-related BOLD fMRI
- Boynton et al. (1996) *J Neurosci* — Linear systems analysis of fMRI
- Dale & Buckner (1997) *Human Brain Mapping* — Selective averaging of rapidly presented individual trials
