# Six-Fingers Motor Execution

A block-design motor task where participants flex or tap individual fingers in response to visual cues. Used to map finger-specific somatotopic representations in primary motor and somatosensory cortex.

## Purpose

Drive finger-specific BOLD responses in M1/S1 to verify somatotopic organization. Each finger gets its own regressor in the design matrix, enabling representational similarity analysis (RSA) or multivariate pattern classification of individual finger representations.

## Design

Block design with rest and cue epochs:

- 3 blocks, each containing 5 trials (one per finger in random order)
- Each trial: 12 s rest (neutral hand image) + 12 s execution cue (highlighted finger)
- A final 12 s rest period follows the last block
- Total design duration: **396 s** (6.6 minutes)
- End screen duration: 0 s (no end screen by default)

## Task Types

Selected at experiment start via the BIDS info dialog:

- **Flexing:** participants flex the highlighted finger (palm-up hand images)
- **Tapping:** participants tap the highlighted finger (images are horizontally flipped to show palm-down)

## Scan Parameters

| Parameter | Recommended | Notes |
|-----------|-------------|-------|
| Resolution | 2 mm isotropic | Standard for motor mapping |
| TR | 2 s | Block design, sub-second TR not needed |
| Volumes | 198 (at 2 s TR) | 396 s / 2 s = 198 |

## Finger Order

Within each block, the five fingers (thumb, index, middle, ring, pinky) are presented in a random order determined at experiment start. The order differs across blocks and runs. The actual order is logged in the output files.

## Code Structure

```
six-fingers-execution/
├── main.m                              # Entry point — block/trial loops
├── images/                             # Hand images (hand.png, hand-01..05.png)
├── utils/
│   ├── setup/
│   │   ├── setup_display.m             # Screen/window setup
│   │   ├── setup_param.m              # Parameters + texture preloading
│   │   ├── setup_keyboard.m            # Key mappings
│   │   ├── wait_trigger.m              # Scanner trigger or manual start
│   │   ├── cleanup_experiment.m        # Save data, close devices
│   │   └── get_info.m                  # BIDS subject/session/task info
│   └── stimuli/
│       ├── present_image_epoch.m       # Core: draw image, wait, log timing
│       ├── s1_rest.m                   # Rest epoch (neutral hand)
│       ├── s2_executionCue.m           # Execution cue (highlighted finger)
│       └── s3_endScreen.m             # End screen
```

## Key Implementation Details

All six hand images (one neutral + five finger-highlighted) are loaded and converted to textures in `setup_param.m` before the trigger. A `containers.Map` maps image paths to texture handles for fast lookup.

The core timing happens in `present_image_epoch.m`, which handles every epoch (rest and cue). It follows the Vulkan workaround pattern (see [Psychtoolbox Tips](psychtoolbox-tips.md)):

- `WaitSecs('UntilTime')` + bare `Flip` instead of timed Flip
- `GetSecs` for onset timestamps instead of `vbl`
- `DrawingFinished` to pre-process the draw before flip
- Continuous draw+flip during the 12 s wait to keep the GPU warm

Each epoch lasts 12 seconds. Without continuous flips, the GPU would go cold during every epoch and stall at the start of the next one.

## Running

```matlab
cd six-fingers-execution
main
```

A dialog prompts for subject/session/run and task type (flexing or tapping). In debug mode, press `t` or `5` to trigger.

## Output

- `*_events.tsv` — onset, duration, trial_type (flexing/tapping or rest), finger name, block and trial indices
- `*.mat` — full `pa` struct including `actualTiming` with onset delays and planned vs. actual timing
- `*_dm.csv` — binary design matrix with one column per finger (thumb, index, middle, ring, pinky), one row per second
