# myPhysio

myPhysio is a physiotherapy platform with a clinician/admin web app, a patient-facing mobile app / PWA, shared packages, and Supabase backend services.

## Agent Start Here

For a contextless AI agent, this page is the starting runbook. Read it fully, then read the [Agent Operations Runbook](#agent-operations-runbook) section before making any change. Use it together with the local gitignored secret bundle:

```text
/Users/pw1246/Documents/GitHub/wpdocs/secrets/myphysio/
```

That folder is intentionally ignored by the `wpdocs` repo through:

```text
/Users/pw1246/Documents/GitHub/wpdocs/.gitignore
```

Local secret files:

| File | Use |
|---|---|
| `secrets/myphysio/README.md` | Local-only index for the secret bundle. |
| `secrets/myphysio/info-about-myphysio.txt` | Backend server details and Supabase keys. |
| `secrets/myphysio/id_ed25519` / `id_ed25519.pub` | SSH keypair for the VPS (password auth is disabled). |
| `secrets/myphysio/ssh-mcp.mjs` | Local MCP server exposing `run_remote` (SSH command execution on the VPS) to AI agents. |
| `secrets/myphysio/web.env` | Environment variables for `apps/web/.env`. |
| `secrets/myphysio/mobile.env` | Environment variables for `apps/mobile/.env`. |

Before using or editing secrets, verify they are ignored:

```bash
cd /Users/pw1246/Documents/GitHub/wpdocs
git check-ignore -v secrets/myphysio/info-about-myphysio.txt secrets/myphysio/id_ed25519
```

Never paste passwords, service-role keys, JWTs, `.env` values, or SSH private keys into tracked docs, commits, issues, PRs, or chat output.

## What This Is

The project is maintained as a pnpm/Turborepo monorepo:

```text
/Users/pw1246/Documents/GitHub/myPhysio-merge
```

The main parts are:

| Area | Path | Purpose |
|---|---|---|
| Web admin app | `apps/web` | Next.js dashboard for physiotherapists and administrators (React 18). |
| Mobile app / PWA | `apps/mobile` | Expo/React Native app for patients and physio mobile workflows (React 19); exported as a web PWA. |
| Shared library | `packages/lib` | Shared constants, types, exercise helpers, prescription helpers, statistics logic. |
| Design system | `packages/design-system` | Shared design tokens/components. |
| Supabase functions | `supabase/functions` | Reminder-related Edge Functions. |

## Live Domains

| Domain | Serves | From |
|---|---|---|
| `myphysio.care` | Public landing page + APK download | `/var/www/myphysio/landing` (static) |
| `admin.myphysio.care` | Next.js admin dashboard | PM2 process `myphysio-web`, port 3000, behind nginx |
| `app.myphysio.care` | **Production** patient PWA (Expo web export) | `/var/www/myphysio-app` → symlink into `/var/www/myphysio-app-releases/<ts>` |
| `dev.myphysio.care` | **Dev/testing** patient PWA | `/var/www/myphysio-app-dev` → symlink into `/var/www/myphysio-app-dev-releases/<ts>` |

All have Let's Encrypt certificates (certbot).

## Branch & Deploy Model

**Branches:** `dev` is the working branch; `main` is production. Ship by fast-forwarding: `git push origin dev:main`.

**Production deploys are pull-based.** There is NO GitHub Actions deploy anymore (the runner→VPS SSH was unreliable; the workflow file was deleted). Instead a cron poller on the VPS checks `origin/main` every minute and self-deploys on change:

| Piece | Location |
|---|---|
| Poller script | `/root/myphysio-deploy-poller.sh` (flock-guarded, runs `deploy.sh`) |
| Cron entry | `* * * * *` in root's crontab |
| Deploy log | `/var/log/myphysio-deploy.log` (look for `deploying <sha>` / `finished rc=0`) |
| Prod checkout | `/var/www/myphysio-monorepo` (branch `main`) |
| Prod deploy script | `deploy.sh` (repo root): pull, pnpm install, build, PM2 reload, expo export, SW build-ID injection, gzip+brotli precompress, atomic symlink switch |

**Dev deploys are manual.** From the dev checkout on the server:

```bash
cd /var/www/myphysio-monorepo-dev   # branch: dev
bash deploy-dev.sh                   # PWA only (no PM2/web-admin steps), ~2 min
```

**Rollback:** releases are kept in `/var/www/myphysio-app-releases/`; point the symlink back at a known-good release:

```bash
ln -sfn /var/www/myphysio-app-releases/<known-good-ts> /var/www/myphysio-app
```

`index.html` and `sw.js` are served no-cache, so clients pick up a rollback on next load.

## Backend Server Access

- Host: see `secrets/myphysio/info-about-myphysio.txt` (root@VPS).
- **Password SSH is disabled** (`/etc/ssh/sshd_config.d/00-hardening.conf`). Auth is via the ed25519 key in the secret bundle.
- Human access: `ssh myphysio` (via `~/.ssh/config` Host entry) or `ssh -i secrets/myphysio/id_ed25519 root@<host>`.
- Agent access: the `myphysio-ssh` local MCP server (`secrets/myphysio/ssh-mcp.mjs`, registered in the Claude desktop config) provides a `run_remote` tool. It prefers the key and needs no interaction.
- Emergency: Hostinger web console (browser-based) if keys are ever lost.
- fail2ban/ufw are NOT enabled; the security posture is key-only SSH.

## Frontend Architecture Notes (PWA)

These were added over time and are easy to break if you don't know they exist:

- **Service worker** (`apps/mobile/public/sw.js`): app-shell precache (build ID + entry bundle injected at deploy time by `deploy.sh`/`deploy-dev.sh` via `sed`), cache-first for `/_expo/`, `/assets/`, `/fonts/`, `/pwa/`, network-first navigations with offline fallback, Supabase video/media caching with Range support, push notifications. Only complete (200) responses are cached — media Range requests return 206 which the Cache API rejects.
- **Snapshot cache** (`apps/mobile/context/GlobalData.tsx`): the last successful fetch is persisted to AsyncStorage; startup hydrates from it instantly while Supabase refreshes in the background (`syncing` state in context). Role/profile state is only committed after a successful fetch — never regress this, it prevents the offline "empty account" bug.
- **Offline write queue / outbox** (`apps/mobile/lib/outbox.ts`): exercise completions and pain levels are enqueued as idempotent upserts, flushed on launch/reconnect/before refresh, and overlaid on both snapshot hydration and fresh fetches so queued writes never visually disappear. Workout logs (`hooks/useWorkoutCounter.ts`) use a `synced` flag with retry-on-reconnect instead.
- **Startup splash**: inline HTML/CSS overlay in `apps/mobile/app/+html.tsx` (logo `public/assets/logo_startup.svg` + shimmer), minimum 800 ms display, hidden by `app/index.tsx` when data is ready (helpers in `components/bootSplash.ts`). On web the overlay is the ONLY splash — `index.tsx` renders `null` beneath it; native uses `components/BrandedSplash.tsx` (Reanimated + `SvgXml`, logo XML in `components/logoStartupXml.ts` — Illustrator CSS classes must stay inlined as attributes).
- **Fonts**: web loads self-hosted woff2 from `apps/mobile/public/fonts/` without blocking first paint; native uses the bundled TTFs. Blocking render on fonts was a major startup cost — don't reintroduce it.
- **Metro React pin** (`apps/mobile/metro.config.js`): `react`/`react-dom` are force-resolved to the mobile app's copies. The monorepo also contains React 18 (`apps/web`), and pnpm's hidden hoist folder (`node_modules/.pnpm/node_modules`) can expose it to Metro depending on install order, silently bundling two Reacts → runtime crash (React error #525, blank screen). **Never remove this pin.** After any build, sanity check: `grep -c '"18\.3\.1"' <entry bundle>` should be 0.
- **nginx**: gzip (`/etc/nginx/conf.d/gzip.conf`) + brotli (`/etc/nginx/conf.d/brotli.conf`, module installed) with `gzip_static`/`brotli_static` serving deploy-time precompressed files. Hashed assets are cached immutable 30d; `index.html` and `sw.js` are no-cache.

## Local Development

Restore local env files from the ignored secret bundle if missing:

```bash
cp /Users/pw1246/Documents/GitHub/wpdocs/secrets/myphysio/web.env /Users/pw1246/Documents/GitHub/myPhysio-merge/apps/web/.env
cp /Users/pw1246/Documents/GitHub/wpdocs/secrets/myphysio/mobile.env /Users/pw1246/Documents/GitHub/myPhysio-merge/apps/mobile/.env
```

Install and run from the monorepo root:

```bash
cd /Users/pw1246/Documents/GitHub/myPhysio-merge
pnpm install
pnpm --filter @myphysio/web dev
pnpm --filter @myphysio/mobile dev
```

Useful checks: `pnpm build`, `pnpm lint`, `pnpm type-check` (or `npx tsc --noEmit` inside `apps/mobile`).

Android APK build automation: `apps/mobile/scripts/build-android.sh` (local EAS builds on the VPS, APK served from `https://myphysio.care/downloads/myphysio.apk`). iOS local-device notes: `apps/mobile/IOS_SETUP.md`.

## Supabase Map

Use the ignored secret bundle for actual keys. Public docs only describe how the app connects.

| Surface | Variables / access |
|---|---|
| Web browser/client | `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY` |
| Web server/API routes | `NEXT_PUBLIC_SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY` where admin access is required |
| Mobile app | `EXPO_PUBLIC_SUPABASE_URL`, `EXPO_PUBLIC_SUPABASE_ANON_KEY` |
| Supabase Edge Functions | `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY` |

Main tables/buckets: `patients`, `physios`, `patient_physios`, `exercices` (**note the spelling** — the exercise library table is `exercices`, not `exercises`), `prescriptions`, `patient_logs`, `body_map_pain_levels`, `exercise_completions`, `workout_logs`, `push_subscriptions`, `physio_exercise_saves`, `physio_follows`; storage bucket `exercise`.

Upsert conflict keys used by the offline outbox: `exercise_completions (patient_id,exercise_code,date)`, `body_map_pain_levels (patient_id,body_part,date)`.

RPC/functions referenced by app code: `create_physio_record`, `invite_patient_by_email`.

Edge Function paths: `supabase/functions/send-reminders/index.ts`, `supabase/functions/send-manual-reminder/index.ts`.

## Agent Operations Runbook

How an AI agent should work on this project. This flow has been used successfully for dozens of changes; follow it unless the user asks otherwise.

### Where to make changes

- The canonical working copy for changes is the **server dev checkout** `/var/www/myphysio-monorepo-dev` (branch `dev`), reached via the `myphysio-ssh` MCP `run_remote` tool. It can push to GitHub (the server has a write deploy key); the local sandbox usually cannot authenticate pushes.
- The local clone `/Users/pw1246/Documents/GitHub/myPhysio-merge` is kept as a mirror: after pushing from the server, run locally `git fetch origin && git reset --hard origin/dev`. Check for uncommitted local user files first (`git status`) — stash or leave them untracked, never discard user work.
- Never commit directly on the prod checkout `/var/www/myphysio-monorepo`; it only pulls `main`.

### How to edit safely over SSH

- For targeted edits, use a Python heredoc with exact-match anchors: `assert old in s` and `assert s.count(old) == 1` before replacing. This catches drift immediately.
- For new files, use quoted heredocs (`<<'EOF'`). Avoid relaying base64 through the chat — it corrupts; if binary/asset transfer is unavoidable, verify with `sha256sum` after writing and prefer plain-text formats.
- Files that exist only on the user's Mac (not pushed) are invisible to the server; process them locally and transfer as verified text.

### How to test

1. Type-check on the server after every change set: `cd /var/www/myphysio-monorepo-dev/apps/mobile && npx tsc --noEmit` — must be 0 errors before committing.
2. Commit on `dev` with a descriptive message, push, then deploy to dev: `bash deploy-dev.sh` (run via `nohup ... &` and poll the log — SSH tool calls time out around 45 s).
3. Verify the deploy: `tail /tmp/deploy-dev.log`, check the release symlink, curl key URLs, and check the built bundle for regressions (e.g., dual-React: `grep -c '"18\.3\.1"' entry-*.js` must be 0).
4. Ask the user to test on `https://dev.myphysio.care` (phone testing matters for PWA/offline features). Do not promote to prod without user sign-off.

### How to deploy to production

```bash
# from the dev checkout, after user approval:
git push origin dev:main        # fast-forward only
# the VPS poller deploys automatically within ~1 minute
tail -f /var/log/myphysio-deploy.log   # wait for "finished rc=0"
```

Then verify prod: bundle hash matches the dev-tested one, `https://app.myphysio.care/` returns 200, brotli serving works, `pm2 ls` shows `myphysio-web` online. If broken, roll back the symlink (see Branch & Deploy Model) first, debug second.

### How to log / observe

| What | Where |
|---|---|
| Prod deploys | `/var/log/myphysio-deploy.log` on the VPS |
| Dev deploys | `/tmp/deploy-dev.log` on the VPS (transient) |
| Admin app runtime | `pm2 logs myphysio-web`, `pm2 ls` |
| nginx | `/var/log/nginx/access.log`, `error.log`; test config with `nginx -t` before reload |
| SSH auth | `journalctl -u ssh` |
| Server health | `uptime`, `df -h /`, `free -h` |

For the user: narrate each server step and its findings in chat — they cannot see `run_remote` contents. Commit messages are the durable changelog; keep them accurate.

### Known gotchas (learned the hard way)

- **Dual React / error #525**: see Metro React pin above. Any `pnpm install` can reshuffle pnpm's hidden hoist; the pin in `metro.config.js` is the defense.
- **`exercices` spelling** in table names and code. `apps/mobile/scripts/verify-supabase.js` wrongly checks `exercises`.
- **Cache API rejects 206** responses — never `cache.put` partial responses in `sw.js`.
- **SW/HTML caching**: hashed assets are immutable-cached; anything not content-hashed that must update (like `sw.js`, `index.html`) needs no-cache headers in the nginx site configs.
- **`pgrep -f` self-matches** the pattern inside the shell command that runs it; use bracket tricks like `pgrep -f "deploy[.]sh"`.
- **Long-running server commands**: MCP SSH calls time out (~45 s); run builds with `nohup ... > log 2>&1 &` and poll.
- **Heredocs append a trailing newline**; `truncate -s -1` when byte-exact output matters.
- Snapshot/outbox invariants: role state only after successful fetch; overlay pending ops on any data that reaches state; snapshot cleared on sign-out.

## Operational Notes

- The dev environment (`dev.myphysio.care`) shares the production Supabase project — test accounts touch real data; use dedicated test users.
- A cert-renewal cron (certbot) handles all domains; the one-off dev-cert watcher script (`/root/dev-cert-watcher.sh`) is defunct and can be ignored.
- Push notifications: `push_subscriptions` table + VAPID; Edge Functions `send-reminders` / `send-manual-reminder`.
- The old app-specific Docker deploy path (`apps/web/deploy.sh`, `auto_deploy.sh`, image `myphysio-admin`) is dead — Docker runs no containers on the VPS. PM2 is the active web path.

## Current Status (July 2026)

- Branches: `dev` (working) and `main` (prod) in `github.com:my-Physio/myPhysio-merge`; stale `feature/email-confirmation` branch deleted (its unmerged email-verification work was reviewed and intentionally dropped).
- Recent shipped work: nginx gzip+brotli with deploy-time precompression; woff2 non-blocking fonts; SW app-shell precache and offline start; instant startup from snapshot cache; offline write queue with sync-on-reconnect; prescribed-media pre-caching; GitHub-style activity heatmap on the workout counter; full-history statistics charts with per-day adherence shading; branded startup splash (min 800 ms) using `logo_startup.svg`; syncing indicator; pull-based deploys; key-only SSH.
- Known open items / ideas: wifi-only or size-capped media pre-caching; offline edit/delete replay for already-synced workout entries; physio-side offline support; Apple Developer account → EAS iOS build + TestFlight pipeline (deferred until UI/UX polish is done).

## Resume / Log Notes

- Built and maintained a full-stack physiotherapy platform spanning web admin, mobile patient experience, shared packages, and Supabase backend services.
- Added clinician workflows for patient management, exercise/program creation, prescription management, patient logs, and analytics.
- Added mobile workflows for exercise guidance, workout counting, statistics, notifications, and physio/patient role support.
- Engineered offline-first PWA architecture: service-worker app-shell and media caching, on-device data snapshots, idempotent offline write queue with reconnect sync.
- Cut PWA payloads ~80% (brotli/gzip precompression, woff2 fonts, non-blocking font loading) and added a branded animated startup experience.
- Replaced push-based CI deploys with a resilient pull-based server deploy pipeline; hardened server access to key-only SSH.
- Handled sensitive deployment material by keeping live credentials in a local gitignored bundle, out of public/project documentation.
