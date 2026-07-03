# Psychtoolbox on the Scanner iMac

Hard-won lessons from debugging stimulus timing on macOS with Psychtoolbox-3. Everything here applies to the scanner-room iMac running macOS with `skipSyncTests = 1`. Some of it generalizes to other macOS setups; almost none of it applies to Linux.

## The PsychVulkanCore Problem

This is the single most important thing to understand. On macOS, Psychtoolbox uses a Vulkan display backend (`PsychVulkanCore`) for stimulus presentation. This backend has a broken timed-presentation path that causes two cascading problems:

1. **Timed Flips stall.** `Screen('Flip', window, when)` — where `when` is a target timestamp — goes through the Vulkan timed-presentation code, which frequently fails with: `PsychVulkanCore-ERROR: PsychPresent(1): Failed to retrieve visual stimulus onset timestamp! Timed out.` Each timeout stalls the experiment for several seconds.

2. **`vbl` timestamps are unreliable.** The `vbl` return value from `Screen('Flip')` comes from the Vulkan backend. When the backend is misbehaving, `vbl` can be `NaN`, zero, wildly wrong, or simply delayed.

!!! warning "These are not intermittent glitches"
    On the scanner iMac with `skipSyncTests = 1`, PsychVulkanCore errors are the norm, not the exception. Every experiment must be written to tolerate them.

## The Three Rules

Every QC experiment in our repo follows these three rules to work around PsychVulkanCore:

### 1. Never use timed Flip — use WaitSecs + bare Flip

```matlab
% BAD — triggers Vulkan timed-presentation path, will stall
Screen('Flip', window, targetTime - 0.5 * ifi);

% GOOD — CPU-based wait, then immediate flip at next vsync
WaitSecs('UntilTime', targetTime - 0.5 * ifi);
Screen('Flip', window);
```

`WaitSecs('UntilTime')` is a high-resolution CPU-clock wait. It does not touch the Vulkan backend. The subsequent bare `Screen('Flip', window)` just flips at the next vertical blank — no timed scheduling, no Vulkan timestamp retrieval.

### 2. Never use `vbl` for timing — use GetSecs

```matlab
% BAD — vbl is unreliable with broken Vulkan backend
vbl = Screen('Flip', window);
nextFlipTime = vbl + stimDuration;

% GOOD — GetSecs reads the CPU clock, always reliable
Screen('Flip', window);
t = GetSecs;
stimEndTime = t + stimDuration;
```

Use `GetSecs` immediately after every `Screen('Flip')` call for any timing decision. The `vbl` return value is only useful on systems with a working display backend (i.e., Linux).

!!! note "Is this really okay?"
    On a working system, `VBLTimestamp` is more precise than `GetSecs` because it uses beamposition correction to back-calculate exactly when the buffer swap occurred — compensating for OS scheduling jitter (typically <1ms). `FlipTimestamp` (Flip's third return value) is essentially the same as calling `GetSecs` right after Flip ([Mario Kleiner confirms this](https://github.com/Psychtoolbox-3/Psychtoolbox-3/wiki/FAQ:-Explanation-of-Flip-Timestamps)). So the precision we lose by using `GetSecs` instead of `VBLTimestamp` is at most ~1ms. For fMRI — where the HRF is 30s wide, TRs are 0.75–2s, and standard analysis assumes onset accuracy of ~100ms — this is completely negligible. Where VBL precision *does* matter is psychophysics (e.g., presenting a stimulus for exactly 2 frames). That's not what these QC tasks do.

### 3. Keep the GPU pipeline warm — never let it idle

This is the subtlest issue. When the GPU goes idle for more than a few seconds (no `Screen('Flip')` calls), the Vulkan pipeline goes "cold." The *next* `Screen('Flip')` — even a bare one — triggers a Vulkan timeout as the backend tries to re-establish frame timing.

```matlab
% BAD — GPU idles for the entire ISI, next flip will stall
Screen('Flip', window);  % show fixation
WaitSecs(isiDuration);   % GPU cold for seconds
Screen('Flip', window);  % THIS WILL STALL

% GOOD — continuous flips keep the pipeline alive
while GetSecs < nextOnsetTime - 0.5 * ifi
    Screen('FillRect', window, bgColor);
    draw_fixation(window, params);
    Screen('Flip', window);
end
% GPU is warm, next flip is immediate
```

This applies everywhere the GPU might sit idle: inter-stimulus intervals, baseline periods, wait-for-trigger screens, and any pause longer than ~1 second.

!!! tip "The one-sentence explanation"
    On macOS, Psychtoolbox's Vulkan display backend loses track of frame timing whenever the GPU sits idle for a few seconds, so we keep redrawing every frame during idle periods to prevent the GPU pipeline from going cold and stalling the next stimulus frame.

## Pre-drawing and DrawingFinished

When a stimulus must appear at a precise time, draw it into the back buffer *before* the onset time, then tell the GPU to start processing:

```matlab
% During ISI, before stimulus onset:
Screen('DrawTexture', window, stimTexture);
draw_fixation(window, params);
Screen('DrawingFinished', window);  % GPU starts processing NOW

% At onset time:
WaitSecs('UntilTime', onsetTime - 0.5 * ifi);
Screen('Flip', window);  % near-instant, drawing already done
```

`Screen('DrawingFinished')` tells the GPU to begin rasterizing immediately rather than deferring all work until `Flip`. This reduces the time between `Flip` and the actual buffer swap.

## Locking Duration to Actual Onset

If the first flip is late (due to a residual Vulkan hiccup), you want the stimulus to still last its full planned duration. Lock the end time to the *actual* onset, not the *planned* onset:

```matlab
% BAD — if onset is late, stimulus is cut short
eventEndTime = plannedOnset + stimDuration;

% GOOD — full duration guaranteed regardless of onset delay
Screen('Flip', window);
t = GetSecs;
eventEndTime = t + stimDuration;
```

## KbQueue and ListenChar Conflict

`ListenChar(2)` suppresses keyboard echo to the MATLAB command window by internally using `GetChar`'s keyboard queue on the default device. If you also call `KbQueueCreate()` on the default device, they fight over the same queue.

Fix: always pass `-1` (all keyboards) to KbQueue, and create the queue *before* calling `ListenChar(2)`:

```matlab
KbQueueCreate(-1);   % -1 = all keyboards, won't conflict
KbQueueStart(-1);
ListenChar(2);       % safe — uses default device, KbQueue uses -1
```

Every KbQueue call in the experiment — `KbQueueCheck`, `KbQueueStop`, `KbQueueRelease` — must also pass `-1`.

## Timing Reporting

Capture `totalExperimentTime` *before* the end screen and cleanup. Otherwise the reported time includes the "Done" screen wait and Psychtoolbox teardown:

```matlab
% After last stimulus/baseline:
pa.totalExperimentTime = GetSecs - experimentStartTime;

% End screen (NOT included in reported time)
DrawFormattedText(window, 'Done', 'center', 'center', white);
Screen('Flip', window);
WaitSecs(endScreenDuration);

% Cleanup (also NOT included)
cleanup_experiment(VP, pa, kb, experimentStartTime);
```

In `cleanup_experiment.m`, only compute the time if it wasn't already set:

```matlab
if ~isfield(pa, 'totalExperimentTime') || isempty(pa.totalExperimentTime)
    pa.totalExperimentTime = GetSecs - experimentStartTime;
end
```

## Texture Management

Pre-compute and preload all textures before the experiment starts — never create textures inside the stimulus loop:

```matlab
% BEFORE trigger:
tex1 = Screen('MakeTexture', window, imageData1);
tex2 = Screen('MakeTexture', window, imageData2);
Screen('PreloadTextures', window, [tex1, tex2]);

% During experiment: just DrawTexture, no creation overhead
Screen('DrawTexture', window, tex1);
```

`Screen('PreloadTextures')` uploads textures to VRAM so the first `DrawTexture` call doesn't stall.

## Debug Configuration Pattern

All QC experiments use a `debugConfig` struct that controls scanner vs. laptop behavior:

```matlab
debugConfig.enabled = 1;           % 1 = debug mode
debugConfig.useVPixx = 0;          % 1 = VPixx/Datapixx hardware
debugConfig.fullscreen = 1;        % 0 = windowed for debugging
debugConfig.skipSyncTests = 1;     % required on macOS
debugConfig.displayMode = 2;       % 1 = NYUAD lab, 2 = laptop
debugConfig.manualTrigger = 1;     % 1 = keyboard 't', 0 = scanner TTL
debugConfig.buttonbox = 0;         % 1 = button box, 0 = keyboard
```

For scanner runs: set `fullscreen = 1`, `manualTrigger = 0`, `useVPixx = 1`, `displayMode = 1`.

## Wait-for-Trigger

The `wait_trigger` function continuously redraws and flips the "Waiting for Trigger..." screen every frame. This keeps the GPU warm so the very first stimulus frame after trigger doesn't stall. It accepts either a manual keyboard trigger (`t` or `5` key) or a scanner TTL via VPixx DIN bit 14.

## Common Pitfalls

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| Timed Flip | Multi-second stalls, Vulkan timeout errors | WaitSecs + bare Flip |
| Using `vbl` for timing | Wrong durations, drifting onsets | GetSecs after every Flip |
| GPU idle during ISI | First stimulus frame freezes | Continuous flips during waits |
| `KbQueueCreate()` without `-1` | "keyboard queue already in use by GetChar()" | Pass `-1` to all KbQueue calls |
| `totalExperimentTime` after end screen | Reported time includes 3-5 s end screen | Capture before end screen |
| Creating textures in the loop | Frame drops on first presentation | Pre-compute all textures before trigger |
| `WaitSecs(0.001)` polling | GPU goes cold between polls | Replace with draw+flip loop |

## Making Durations Divisible by Multiple TRs

If you want the same experiment to work with different TR values (e.g., 0.75 s and 2 s), make the total duration a multiple of their LCM. For 0.75 and 2, LCM = 6, so the duration must be a multiple of 6. Baselines should also be divisible by both TRs so they align to the TR grid.

Example: the checkerboard task uses 240 s total with 12 s baselines (240/0.75 = 320 volumes, 240/2 = 120 volumes, 12/0.75 = 16 TRs, 12/2 = 6 TRs).
