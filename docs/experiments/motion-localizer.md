# Motion Localizer (MT+)

A block-design random dot kinematogram task that alternates moving and static dot fields to localize area MT+ (V5). Used as a functional localizer before experiments that target motion-sensitive cortex.

## Purpose

Identify the MT+ complex by contrasting BOLD responses to coherent motion vs. static dots. The motion > static contrast reliably activates bilateral MT+ in individual subjects, providing subject-specific ROIs for subsequent analyses.

## Design

Block design with alternating motion and baseline blocks:

- 24 blocks total: 12 motion, 12 baseline (alternating)
- Each block: **15 s**
- Motion types cycle every motion block: outward → inward → clockwise → counter-clockwise (repeats 3 times)
- Baseline condition (selected at experiment start): **static** (dots frozen) or **flickering** (dots randomly repositioned on death, eliminates motion aftereffect)
- Total design duration: **360 s** (6 minutes)
- 5 s end screen (not included in reported time)

## Scan Parameters

| Parameter | Recommended | Notes |
|-----------|-------------|-------|
| Resolution | **2 mm** isotropic | Good SNR for reliable MT+ localization. Only go 1.2 mm if your main experiment also runs at 1.2 mm. |
| TR | 2 s | Block design has plenty of power; sub-second TR not needed. |
| Volumes | 180 (at 2 s TR) | 360 s / 2 s = 180 |

## Dot Field Parameters

| Parameter | Value |
|-----------|-------|
| Number of dots | 250 |
| Dot diameter | 0.2 deg |
| Aperture radius | 10 deg |
| Speed | 12 deg/s |
| Dot lifetime | 0.5 s (limited lifetime) |
| Dot colors | Half white, half black (high contrast) |

Limited-lifetime dots are used to prevent local motion streaks that could drive V1 orientation-selective neurons rather than MT+ motion-selective neurons.

## Attention Task

The fixation cross alternates between green and red at random intervals (2–12 s uniform). Participants press a button on each color change. This maintains attention and fixation while being orthogonal to the motion/static contrast.

## Code Structure

```
motion-localizer/
├── main.m                      # Entry point — block loop, trigger, cleanup
├── utils/
│   ├── setup/
│   │   ├── setup_display.m     # Screen/window setup
│   │   ├── setup_param.m       # Dot field, timing, motion parameters
│   │   ├── setup_keyboard.m    # Key mappings
│   │   ├── wait_trigger.m      # Scanner trigger or manual start
│   │   ├── cleanup_experiment.m
│   │   └── get_info.m          # BIDS subject/session info
```

## Key Implementation Details

The motion localizer naturally keeps the GPU pipeline warm because it flips every frame within each block (drawing 250 dots per frame). There is no ISI gap between blocks — the for loop transitions immediately from one block's while loop to the next.

A brief GPU warmup loop runs right after the trigger to ensure the very first block's first frame doesn't stall. The wait-for-trigger screen also flips continuously (see [Psychtoolbox Tips](psychtoolbox-tips.md)).

Motion computation: outward and inward motion are yoked — outward is computed live and stored in `dotMat`, then inward replays the same positions in reverse. This ensures the inward condition has identical spatial statistics to outward. Clockwise and counter-clockwise are computed independently via angular velocity.

## Baseline: Static vs. Flickering

Choose at the start dialog:

- **Static:** dots freeze in place during baseline blocks. Simpler, but residual motion aftereffect from the preceding motion block can contaminate the baseline response.
- **Flickering:** dots still follow the limited-lifetime cycle (re-randomized on death) but don't move. This eliminates motion aftereffect while keeping luminance transients matched. Recommended when motion aftereffect is a concern.

## Running

```matlab
cd motion-localizer
main
```

A dialog prompts for subject/session/run, then a second dialog asks for baseline condition (Static or Flickering). In debug mode, press `t` to trigger.

## Output

- `*_events.tsv` — onset, duration, trial_type (outward/inward/clockwise/counterclockwise/static/flickering) for each block
- `*_events.json` — metadata sidecar
- `*.mat` — full `pa` struct
- `*_dm.csv` — binary design matrix (motion vs. baseline, one row per second)
