# Shrimp

Shrimp is a calm, landscape-first aquarium breeding game built with Phaser, TypeScript, and Vite. Its design target is a soothing, low-pressure loop: watch, feed, breed, catch, move, sell, and decorate ornamental-shrimp tanks.

## Agent Start Here

Read this page before changing code, gameplay, deployment, or art. The source of truth remains the code and tests; this page is the durable operational and product handoff.

| Item | Location |
|---|---|
| App repository | `/Users/pw1246/Documents/Codex/2026-07-06/i/work/shrimp-tanks-phaser` |
| GitHub | `https://github.com/putiw/shrimp` |
| Production | `https://fish.myphysio.care/` |
| Hand-drawn source art | `/Users/pw1246/Documents/Claude/Projects/shrimpTank` |
| Production build output | `/Users/pw1246/Documents/Codex/2026-07-06/i/outputs/shrimp-tanks` |

Current production and `main` state as of 2026-07-12:

- `6eef247` delivered the aquascape shop/storage redesign, per-instance decor, up to six tanks, Magical Tank 6, Magic Mushroom, authored decor collision/rest data, audio/UI work, death presentation, and movement fixes.
- `9bc803d` restored native mobile drawer scrolling, made taps immediate, and predecoded shop thumbnails.
- `a628f6f` removed the redundant Owned inventory mode. The inventory control now cycles **All ↔ Stock**.
- The production baseline passes **17 test files / 85 tests**. TypeScript and the production build pass.
- The deployed build at `fish.myphysio.care` was verified with matching build hashes, accessible artwork, working phone-sized one-tap/scroll behavior, and no browser console errors.

### Non-production stability branch (2026-07-12)

Active development is continuing on `codex/v-next-stability`. This branch is **not production, has not been merged or deployed, and must not be described as live**. Its current automated suite passes **270/270 tests**. Reviewer stress validation also passed with 300 shrimp in an inactive tank plus 300 shrimp in the active tank. The local production build passes with five HTML entries and 66 offline-shell assets. Draft pull request [#14](https://github.com/putiw/shrimp/pull/14) is open. GitHub Actions `verify` passed at current PR head `b8db7c0` on [run 29185886116](https://github.com/putiw/shrimp/actions/runs/29185886116).

Implemented commits on the branch:

| Commit | Result |
|---|---|
| `2da1abb` | Adds pinned CI/release validation for generated assets, TypeScript, tests, production packaging, offline references, and packaged files. |
| `139cce1` | Restores compact status/progression feedback and corrects the selling/aquascape unlock copy. |
| `e660518` | Requires confirmation before destructive Start Over. |
| `8bbf50a` | Derives newborn lifecycle and behavior physiology from inherited DNA. |
| `9a0b491` | Consumes density-based breeding rolls once per elapsed interval instead of rerolling every frame. |
| `7028260` | Adds autosave/automatic resume with Web Locks single-writer coordination, bounded storage operations, malformed-save fallback, scene-generation guards, and transactional Start Over rollback. |
| `9eedecb` | Aligns starter-shrimp physiology with their final DNA. |
| `85b8134` | Replaces repeated mate-scout scans with linear per-update indexing. |
| `491d29f` | Adds bounded ordinary-shrimp separation steering to reduce visual stacking without rigid collision. |
| `d65f941` | Cleans up retained scene/runtime objects across load, reset, and long sessions. |
| `36ae46b` | Normalizes continuous damping, eating, mating, and related simulation effects by elapsed time. |
| `1809e37` | Replaces continuous menu polling with event-driven UI synchronization. |
| `9d1a574` | Adds deterministic seeded simulation soak/invariant coverage. |
| `bda1022` | Increases the CI test timeout so the deterministic soak suite can complete reliably; the subsequent `verify` run passed. |
| `0a6a14f` | Runs inactive-tank biological clocks at full speed and uses fair bounded movement scheduling instead of making inactive colonies biologically slower. |
| `a8fb98d` | Replaces the fixed 9.5-second gate with readiness-based startup dismissal and allows the presentation to be skipped once safe visual readiness is reached. |
| `715fcc5` | Defines legendary-parent offspring as fully independent new inheritance outcomes: any color, visible grade G5+, without inheriting the legendary marker. |
| `bb76c19` | Redesigns mate scouting so a bounded 2–4 eligible males scout and target the nearest eligible female. |
| `5669162` | Preserves valid mate-scout intent through movement recovery instead of silently discarding the courtship state. |
| `b8db7c0` | Detects swept mate-scout arrivals so a fast movement step cannot skip across the destination and stall pairing. |

The remaining stability/product queue is intentionally not folded into those implementation claims:

- decide the hidden-decor-stat policy tracked in GitHub issue #11;
- measure art/audio/WebGL memory, CPU, thermals, and battery on physical iPhones;
- make product decisions for audio controls, accessibility, lineage/inbreeding, and the longer-term rare legendary discovery path;
- complete browser-level mobile E2E coverage and later native XCUITest work in issue #13;
- choose minimum iOS, iPhone/iPad scope, bundle identifier, signing team, and wrapper approach for issue #12.

Do not revive the old terminology “cull tank” in player-facing UI. Tanks are displayed as **Tank 1, Tank 2, … Tank 6**. Internal identifiers such as `main` and `cull` remain implementation details.

### Repository safety

At the time of the 2026-07-12 read-only review, the game repo had these preserved untracked local items:

```text
.codex-shrimp-doc.md
GENETICS_INTEGRATION_NOTE.md
dist-local/
```

They were not created, changed, staged, or removed by the review agents. Do not reset, clean, or overwrite user work. Do not edit the source-art directory unless Puti explicitly asks.

## Current Player Experience

### Tanks and progression

- A new game owns Tank 1 and Tank 2. New tanks are purchased from the aquascape shop, up to six total.
- Tank purchase prices rise in 300 AED steps. Tank 6 is the Magical Tank.
- Shrimp in the Magical Tank are frozen biologically: they do not breed, age, or die. Movement and feeding continue.
- The direct shrimp-selling action unlocks after 20 transfers into Tank 2.
- The aquascape shop permanently unlocks once the player reaches 30 AED.
- With three or more tanks, caught shrimp go into a bucket and can then be transferred to a chosen tank.
- Catching is guaranteed at 40 or more shrimp in the active tank. Below 40, a shrimp may escape, but an escaping shrimp remains catchable.

### Aquascape shop and decor

- `sell.png`/`shop.png` are the direct sell-shrimp action. `decor.png` opens the combined shop and storage drawer.
- The wallet appears while selling and while the aquascape drawer is open.
- The compact drawer uses an inventory toggle (**All ↔ Stock**), a type toggle, and a global Remove mode.
- Players cannot resell purchased decor. Removing an item returns that same instance to storage.
- Each tank can hold at most 20 placed decor instances; storage is unlimited.
- V1 decor transforms are translation within the valid placement zone and horizontal flip only.
- Sand decor uses authored `*b.png` contact/collision masks. Disconnected base regions and transparent passages are intentional.
- Authored `*r.png` paths are preferred resting surfaces. Both adult and young shrimp may use them.
- A shrimp must never switch in front of/behind an object while its sprite overlaps the visible decor, regardless of the smaller base collision mask.
- Built-in tall grass is visual-only. It is not a rest surface, collision object, or traversal route.
- Floating plants are pass-through decor constrained to the waterline.
- Newly placed sand decor chooses a comparatively open position and plays a short water-drop/wobble animation. Floating plants skip the drop.

Current decor is data-driven in `src/game/decor.ts`. The catalog includes eight rocks, two driftwood pieces, three bamboo pieces, Sunken Ship, Elephant Statue, Ancient Pantheon, Guardian Lion, floating plants, and Magic Mushroom.

Decor effects are deliberately hidden from players. Flavor text may hint at preferences, but the actual effects should not be exposed. Repeated effects use capped/diminishing stacking and apply only while placed in a tank.

Important audit note: the environment system calculates `breedingComfortBonus`, but the current breeding path does not consume it. Treat that as a confirmed incomplete mechanic, not as a working player benefit.

### Magic Mushroom

- The Magic Mushroom costs 666 AED and its catalog description is exactly **“Enlightens a shrimp”**.
- An unused mushroom must be placed onto a shrimp. That shrimp becomes immortal, stays at prime age, and can breed normally forever.
- Used and unused mushrooms appear separately in inventory. A used mushroom remains ordinary movable/storable decor but cannot enlighten a second shrimp.
- An enlightened shrimp uses the authored mushroom-eye overlays, alternating at randomized 1–5 second intervals.

### Lifecycle and presentation

- Shrimp have sex, maturity, pregnancy, inherited DNA, individual body traits, chronological age, and scheduled death.
- Aged death has its own top-layer authored dead-leg/eye frame. The shrimp flips vertically, sinks with damped lateral sway, rests on the sand for one second, then fades.
- Immortal shrimp and shrimp in the Magical Tank do not enter the aged-death presentation.
- Open-water shrimp always animate swimming legs. A stuck monitor redirects a shrimp that makes no progress in open water for roughly 2.4 seconds.
- Fast-swim front/upper leg alternation runs at 1.5× its former animation rate without changing movement speed.

### Saving

- There are two manual browser save slots.
- IndexedDB (`shrimp-aquarium-saves`) is primary; localStorage keys `shrimp-save-v1` and `shrimp-save-v1-slot-2` are compatibility backups.
- Saves include simulation state and authoritative decor ownership/placement. Old hardscape layout data is migration input only.
- Loading shifts timestamps so elapsed real-world time is not simulated offline.
- **Production `a628f6f` has no autosave or automatic resume.** Closing or background-terminating production can lose progress since the last manual save.
- **Production Start Over has no confirmation step.**
- The non-production `codex/v-next-stability` branch addresses both gaps in `7028260` and `e660518`, including multi-tab ownership and reset rollback safeguards. Do not describe those protections as deployed until the branch completes review and release.

## Architecture

| Path | Responsibility |
|---|---|
| `src/game/simulation.ts` | Simulation, lifecycle, movement, feeding, breeding, transfers, economy, tanks, save migration, decor effects |
| `src/game/genetics.ts` | 88-base genome, phenotype, inheritance, mutation, breeding profiles |
| `src/game/decor.ts` | Decor catalog, prices, effects, inventory helpers, 20-per-tank rule |
| `src/game/storage.ts` | IndexedDB save slots and localStorage backup |
| `src/game/types.ts` | Persistent and runtime data model |
| `src/scenes/TankScene.ts` | Phaser rendering, input, sprite assembly, depth routing, aquascape interaction |
| `src/rendering/ShrimpColorPipeline.ts` | WebGL shrimp coloration |
| `src/game/audio.ts` | Ambient, catch, push, sell, purchase, placement, and removal sounds |
| `src/ui/controls.ts` | Main DOM controls and tank switching |
| `src/ui/aquascape.ts` | Shop/storage drawer, filters, item cards, placement toolbar |
| `scripts/generate-tank-assets.mjs` | Generated decor textures, overlap masks, base collision data, and rest paths |
| `public/assets/` | Runtime art/audio |
| `public/sw.js` | Service worker and release cache |

The main renderer is fixed at 1864×860, WebGL, and 30 FPS. Phaser scales the landscape composition into the available viewport. High population and decoded texture/audio memory are therefore important mobile constraints.

## Genetics and Breeding

The runtime source of truth is `src/game/genetics.ts`; `public/dna-breeding-lab.html` imports the same helpers for inspection.

The current genome is 88 ATGC-style bases:

| Segment | Bases | Meaning |
|---|---:|---|
| Pigment | 36 | Hue, saturation, lightness |
| Shell | 16 | Shell density/opacity and grade potential |
| Shade | 12 | Brightness/darkness |
| Pattern | 4 | Solid/rili expression |
| Vigor | 4 | Fertility, fecundity, embryo survival, stability |
| Body | 16 | Movement, size, lifespan, activity, catch/food traits |

Current model constants that are easy to misdocument:

- Hue tolerance: 10°
- Hue penalty: 100%
- Low-saturation penalty weight: 24%
- Vigor weight: 55%
- Grade-fertility weight: 65%
- Rili threshold: 3 `C` alleles
- Child-wide hue mutation: **10° standard deviation**
- Lightness rare-mutation multiplier: **2× base chance**
- Same-grade coherent shell drift: 5% two grades lower, 30% one lower, 40% same, 19% one higher, 6% two higher

Do not copy old notes that say the child hue mutation is 5°. Confirm constants in source before documenting future tuning.

### Confirmed breeding audit findings and implementation status (2026-07-12)

These are code-review findings, not speculative balance opinions:

1. Production constructs newborn physiology before installing inherited DNA. Development commit `8bbf50a` fixes normal newborns, and `9eedecb` applies the same final-DNA rule to starter shrimp.
2. Production rerolls crowded-tank breeding probability every frame while a pair is eligible. Development commit `9a0b491` consumes the roll once per elapsed interval.
3. `breedingComfortBonus` is aggregated from decor but is not applied to breeding.
4. Production has several damping, eating, and mating transitions tied to update count. Development commit `36ae46b` normalizes the reviewed continuous effects by elapsed time.
5. Production mating candidate scans can approach O(n²) at high populations. Development commit `85b8134` indexes mate-scout eligibility once per update.
6. Production biological clocks and behavior timers are not uniformly advanced between active and inactive tanks. Development commit `0a6a14f` applies full biological clocks and fair bounded movement scheduling across inactive tanks.
7. Legendary-parent inheritance previously had conflicting paths. Development commit `715fcc5` makes each offspring an independent any-color, G5+ non-legendary result. This resolves the parent-offspring contract but does not complete the future rare legendary-discovery design in issue #8.

Keep gameplay recommendations separate from those confirmed defects. Possible later design work includes lineage/diversity pressure, outside suppliers, and a legendary rainbow path; those remain product choices in GitHub issues #7 and #8.

## Movement Design

Movement intentionally combines slow surface grazing, fast water-column travel, food pursuit, floater resting, mate scouting, mating, touch pushing, and decor-aware depth routing.

Important invariants:

- Shrimp may travel through the visible water column and across shallow, middle, and deep sand.
- A water-column shrimp must never appear fully idle; swimmer legs continue at a gentle minimum rate.
- Procedural tall grass is draw-only and cannot support shrimp.
- Surface targets come only from real sand, authored decor rest paths, or floating-plant roots.
- Front/back depth changes require a valid route and cannot occur while the sprite overlaps decor.
- Pushing a group should produce varied, slightly overlapping sounds, not a long serialized queue.
- The simulation has an open-water no-progress recovery, but it is a safety net rather than a substitute for correct target/state logic.

Production review concern: ordinary shrimp do not have a lightweight overlap-separation pass and can visually stack outside mating. Development commit `491d29f` adds bounded separation steering; it remains non-production until the branch is reviewed and released.

Development mate-scout behavior is deliberately bounded: `bb76c19` selects 2–4 eligible male scouts and sends each toward the nearest eligible female. `5669162` preserves that intent through recovery, while `b8db7c0` treats a swept path across the arrival zone as arrival. Together these address long-lived courtship stalls without returning to all-pairs scouting.

## Art Workflow

Puti’s source folders deliberately include the misspelling `shirmp/`. Do not rename source folders or files without updating the copy/generation pipeline.

| Source | Runtime destination |
|---|---|
| `shirmp/` | `public/assets/shrimp/` |
| `decor/` | generated/cropped runtime decor under `public/assets/tank/` |
| `tank/` | `public/assets/tank/` |
| `icon/` | `public/assets/icon/` |
| audio source files | `public/assets/audio/` after conversion/copy |

Rules:

- Preserve authored transparent PNG registration. Related visible, `b`, and `r` files share the same source canvas.
- `*b.png` is the base-only physical/contact mask; `*r.png` authors preferred resting paths.
- Keep transparent passages open. Some decor has multiple disconnected sand-contact regions.
- Shrimp sprite parts must retain a common canvas/alignment.
- Avoid accidental white matte pixels around transparent art.
- After adding/changing decor source files, run the generator and its check; do not hand-edit generated metadata.
- Before deployment, confirm public directories are traversable and artwork files are readable by the web server.

## Local Development and Verification

```bash
cd /Users/pw1246/Documents/Codex/2026-07-06/i/work/shrimp-tanks-phaser
npm install
npm test
npm run build
npm run dev
```

Useful commands:

```bash
npm run check:tank-assets
npm run generate:tank-assets
npm run preview
```

Useful developer pages:

| Page | Purpose |
|---|---|
| `/shrimp-gallery.html` | Appearance gallery |
| `/shrimp-opacity-test.html` | Grade/opacity tuning |
| `/dna-breeding-lab.html` | Shared-runtime DNA and inheritance lab |
| `/movement-lab.html` | Movement and leg-animation lab |
| `/mating-dance-lab.html` | Mating sequence lab |
| `/lifespan-render-lab.html` | Lifecycle/death rendering checks |

For a clean local PWA session, use a save namespace or `?sw-clean=1`. Local dev/preview also unregisters service workers and clears caches.

Minimum release verification:

1. `npm test` passes all 85 current tests.
2. `npm run build` passes TypeScript, generated-asset validation, Vite, and service-worker update.
3. Test one-tap controls and drawer scrolling at a phone-sized landscape viewport.
4. Load a fresh game and a migrated save.
5. Inspect browser console for errors.
6. Verify representative art, manifest, and service worker return HTTP 200.
7. Compare live `index.html` and `sw.js` hashes to the verified build.

## Performance Review Notes

Confirmed or strongly supported production risks from the 2026-07-12 review:

- Mating candidate work can scale quadratically in the active tank; development commit `85b8134` replaces the reviewed scout scans with linear indexing.
- Some simulation behavior is frame-rate dependent; development commit `36ae46b` normalizes the reviewed continuous effects.
- Production uses a fixed 9.5-second startup gate; development commit `a8fb98d` dismisses on verified visual readiness and supports safe skipping.
- Mobile memory pressure is likely from decoded art, generated textures/thumbnails, WebGL texture residency, and audio buffers; measure on physical iPhones before choosing reductions.
- Production menu synchronization continuously polls; development commit `1809e37` makes it event-driven.
- Retained scene objects can grow across reset/load cycles; development commit `d65f941` adds runtime cleanup.

Measure before tuning. Use Safari Web Inspector Timelines on a physical iPhone and test representative populations, including crowded tanks and repeated shop opening. Record FPS/frame time, main-thread CPU, JS heap/GC, WebGL/decoded image memory where observable, audio behavior, thermals, and recovery after backgrounding.

## Product and UI Audit Priorities

### Production defects with non-production fixes in review

1. Autosave/auto-resume and safe lifecycle handling are implemented on the development branch in `7028260`.
2. Start Over confirmation is implemented in `e660518`.
3. Compact progression status and corrected unlock copy are implemented in `139cce1`.

These remain production defects until the development branch passes final review and is deliberately deployed.

### Recommendations requiring product decisions

- Add a quiet first-session tutorial for feed, catch/transfer, sell, shop/storage, and saves.
- Define success/long-term goals without turning the calm aquarium into a task list.
- Decide whether hidden decor effects should remain entirely undisclosed or receive consistent qualitative hints.
- Create performance budgets for target iPhones and dense-tank populations.

## App Store Readiness

The current production game is a web PWA, not an App Store-ready iOS project. The largest missing pieces are:

- no native iOS/Xcode wrapper, signing, bundle identifier, provisioning, or archive pipeline;
- no verified native lifecycle integration for save/resume, audio interruption/backgrounding, safe areas, orientation, and low-memory recovery;
- production lacks autosave and Start Over confirmation; reviewed web-side fixes exist on the non-production stability branch but native lifecycle integration is still required;
- no App Store product metadata, screenshots, native icon/launch-screen set, support URL, or privacy-policy page;
- no durable asset-rights/license ledger for user-created and third-party fonts/audio/art;
- no completed privacy-data inventory, App Privacy answers, or privacy-manifest audit for the chosen native wrapper and dependencies;
- incomplete accessibility audit (touch target size, contrast, text scaling/legibility, nonvisual labels, reduced motion/audio controls);
- no documented device matrix covering current minimum iOS, older/slower supported iPhones, interruptions, offline launch, install/update, storage pressure, and long sessions.

Apple requirements change. Before submission, verify current rules from Apple’s official documentation rather than treating this handoff as policy advice. Do not promise reviewer acceptance.

Recommended sequence:

1. Fix persistence/destructive-action safety and the missing progression status.
2. Profile and stabilize the PWA on physical target iPhones.
3. Choose a native wrapper strategy and make a proof-of-concept build.
4. Complete lifecycle/audio/safe-area/accessibility work.
5. Build the privacy, support, rights, icon, launch, screenshot, and metadata package.
6. Run TestFlight device QA before App Review.

## Deployment Runbook

The game is a static release served from:

```text
https://fish.myphysio.care/
```

Known server layout:

| Item | Value |
|---|---|
| Live symlink | `/var/www/myphysio-app-fish` |
| Versioned releases | `/var/www/myphysio-app-fish-releases` |

The older hosting handoff is at:

```text
/Users/pw1246/Documents/GitHub/myPhysio-merge/docs/pwa-hosting-handoff.md
```

Use existing approved SSH/SCP patterns. Never record credentials, private keys, or tokens here. Deploy a versioned directory, verify it, then atomically change the live symlink. Ensure macOS AppleDouble files are excluded when archiving (`COPYFILE_DISABLE=1`).

Post-deployment checks:

```bash
curl -I https://fish.myphysio.care/
curl -I https://fish.myphysio.care/sw.js
curl -I https://fish.myphysio.care/manifest.webmanifest
```

Also confirm representative artwork, cache headers, service-worker cache revision, full-file accessibility, browser console, and byte-for-byte hashes for `index.html` and `sw.js`.

## GitHub Planning

Before creating work, inspect existing issues at `https://github.com/putiw/shrimp/issues` and avoid duplicate implementation trackers.

- Issues #1–#5 are historical core-mechanic trackers whose acceptance scope has been delivered.
- Issue #6 remains relevant because wellbeing/decor infrastructure is only partially connected to gameplay.
- Issues #7 and #8 are later product/design work, not confirmed defects.
- Issue #9 records the aquascape redesign and production rollout.
- The consolidated 2026-07-12 audit issue is the durable queue for confirmed technical/product risks and App Store readiness.
- Issue #11 tracks the unresolved hidden-decor-stat decision.
- Issue #12 tracks the iOS wrapper and external publishing decisions.
- Issue #13 tracks browser/mobile release automation, soak coverage, and future native XCUITest work. CI and seeded soak coverage have landed on the non-production stability branch, but browser-level mobile E2E and native coverage remain open.
- Draft PR #14 contains the stability branch. Its local build and current-head CI verification pass. It remains unmerged and undeployed.

Issue #8 remains open because `715fcc5` defines what legendary parents produce; it does not implement the complete extremely rare path by which a normal breeding population first discovers a legendary rainbow shrimp.

Label recommendations as recommendations. Do not present unmeasured performance suspicions, balance preferences, or possible Apple reviewer concerns as confirmed bugs.

## Durable Design Constraints

- Keep the tank view calm, landscape-first, and icon-led.
- Prefer soft, short, non-sharp sounds; prevent long serialized sound queues.
- Do not expose exact hidden decor effects.
- Do not add offline biological simulation without an explicit product decision.
- Preserve front/back spatial coherence around decor.
- Preserve user art and existing save migration.
- Do not call Tank 2 the “cull tank” in player-facing text.
- Validate generated decor assets and file permissions before every release.
