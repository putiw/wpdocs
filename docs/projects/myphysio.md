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
| `dev.myphysio.care/admin/login` | **Dev/testing** Next.js admin dashboard | PM2 process `myphysio-web-dev`, port 3001, nginx `/admin` proxy, built with `NEXT_PUBLIC_ADMIN_BASE_PATH=/admin` |

All have Let's Encrypt certificates (certbot).

## Branch & Deploy Model

**Branches:** `dev` is the working branch; `main` is production. Promote to production with a PR from `dev` to `main` so GitHub Actions can run before the VPS poller sees the change. The repo is private; GitHub branch protection/rulesets for a private repo require an upgraded GitHub plan, so there is no enforced `Protect main` ruleset right now. Treat the PR flow as mandatory anyway: do not push directly to `main` or use `git push origin dev:main`.

**Production deploys are pull-based.** GitHub Actions runs CI only; there is no GitHub Actions deploy job anymore (the runner→VPS SSH path was unreliable). Instead a cron poller on the VPS checks `origin/main` every minute and self-deploys on change:

| Piece | Location |
|---|---|
| Poller script | `/root/myphysio-deploy-poller.sh` (flock-guarded, runs `deploy.sh`) |
| Cron entry | `* * * * *` in root's crontab |
| Deploy log | `/var/log/myphysio-deploy.log` (look for `deploying <sha>` / `finished rc=0`) |
| Prod checkout | `/var/www/myphysio-monorepo` (branch `main`) |
| Prod deploy script | `deploy.sh` (repo root): pull, pnpm install, build, PM2 reload, expo export, SW build-ID injection, gzip+brotli precompress, atomic symlink switch |

**Dev PWA deploys are manual.** From the dev checkout on the server:

```bash
cd /var/www/myphysio-monorepo-dev   # branch: dev
bash deploy-dev.sh                   # PWA only (no PM2/web-admin steps), ~2 min
```

**Dev admin deploys are separate** because `deploy-dev.sh` only publishes the patient PWA. The dev admin dashboard is served at `https://dev.myphysio.care/admin/login` by PM2 process `myphysio-web-dev` on port 3001, with nginx proxying `/admin` and `/admin/` to that process. Build/restart it from the server dev checkout with:

```bash
cd /var/www/myphysio-monorepo-dev
git pull origin dev
corepack pnpm install
NEXT_PUBLIC_ADMIN_BASE_PATH=/admin corepack pnpm --filter @myphysio/web run build
cd apps/web
PORT=3001 NEXT_PUBLIC_ADMIN_BASE_PATH=/admin pm2 restart myphysio-web-dev --update-env
```

When validating the admin build from a local Codex sandbox, `NEXT_PUBLIC_ADMIN_BASE_PATH=/admin corepack pnpm --filter @myphysio/web run build` may fail with `EPERM: operation not permitted, open '.../apps/web/.next/trace'`. That is a sandbox filesystem permission issue from Next.js writing build trace/cache files, not a TypeScript or app code error. Re-run the exact same build with escalated filesystem permission, or validate on the server checkout instead. Do not treat the `.next/trace` EPERM by itself as a failed code build.

**Rollback:** releases are kept in `/var/www/myphysio-app-releases/`; point the symlink back at a known-good release:

```bash
ln -sfn /var/www/myphysio-app-releases/<known-good-ts> /var/www/myphysio-app
```

`index.html` and `sw.js` are served no-cache, so clients pick up a rollback on next load.

**Release retention:** production PWA deploys create timestamped releases in `/var/www/myphysio-app-releases` and currently do **not** automatically prune old releases; this gives rollback flexibility but means disk usage should be checked periodically. Dev PWA deploys keep the last 5 releases in `/var/www/myphysio-app-dev-releases` (`deploy-dev.sh` handles that pruning). A prior manual server cleanup also cleared build caches (Gradle, npm, pnpm) and old stray `/var/www` app folders to free disk; if disk pressure recurs, those caches and old production PWA releases are the first places to look (`du -sh ~/.gradle ~/.npm ~/.local/share/pnpm /var/www/myphysio-app-releases 2>/dev/null`).

## GitHub Issue and Project Tracking

Use GitHub issues plus the org-level GitHub Project as the durable development log. This is how future agents should track what was planned, implemented, reviewed, shipped, and which commit/PR did the work.

| Item | Location |
|---|---|
| Development project | `https://github.com/orgs/my-Physio/projects/2` |
| Project name | `myPhysio Development` |
| Main monorepo issues | `https://github.com/my-Physio/myPhysio-merge/issues` |
| Main monorepo PRs | `https://github.com/my-Physio/myPhysio-merge/pulls` |

Keep the Project at the **organization** level, not repo-only. myPhysio work can span this monorepo, the older mobile/admin repos, Supabase, deployment/server work, APK/iOS work, and docs. Repo issues are still the source of truth for code work; the org Project is the cross-repo board.

### Agent access

Project reads/writes require GitHub's `project` token scope. If project commands fail with `INSUFFICIENT_SCOPES`, ask Puti to run:

```bash
gh auth refresh -h github.com -s project
```

Check current access:

```bash
gh auth status
gh project view 2 --owner my-Physio --format json
```

Expected token scopes include at least `repo`, `read:org`, `workflow`, and `project`.

### Standard workflow for new work

1. Create or reuse a GitHub issue in the repo that owns the work. For this monorepo, default to:

   ```bash
   gh issue create --repo my-Physio/myPhysio-merge \
     --title "<short feature/fix title>" \
     --label enhancement \
     --body "<problem, scope, acceptance criteria, and verification plan>"
   ```

2. Add the issue to the org Project:

   ```bash
   gh project item-add 2 --owner my-Physio \
     --url https://github.com/my-Physio/myPhysio-merge/issues/<issue_number> \
     --format json
   ```

3. Set the Project status:

   - `Backlog`: idea exists, not ready to implement.
   - `Ready`: scope is clear enough to start.
   - `In progress`: an agent/human is actively working.
   - `In review`: code is pushed to `dev` or a PR exists, but it is not production-shipped yet.
   - `Done`: merged/promoted, verified, and no further work remains for that issue.

4. Keep commits descriptive and link the issue from PRs. Use `Refs #<issue_number>` while work is in progress; use `Closes #<issue_number>` only when merging that PR should close the issue.

5. After implementation, update the issue body or comment with:

   - implementation date,
   - branch,
   - commit SHA and commit link,
   - PR link, if one exists,
   - exact verification commands and results,
   - deployment state (`dev only`, `main`, or `production verified`).

### After production promotion

When `dev` is promoted to `main` through a PR merge, wait for the VPS poller to deploy, then verify production. After verification, comment on the issue with the production state and close it:

```bash
gh issue comment <issue_number> --repo my-Physio/myPhysio-merge \
  --body "Promoted to main and production on <YYYY-MM-DD>. main and dev both point to <sha>. Verified app.myphysio.care returned HTTP 200 with Last-Modified: <timestamp>."

gh issue close <issue_number> --repo my-Physio/myPhysio-merge \
  --reason completed \
  --comment "Completed and shipped to production. Tracked in the myPhysio Development project as Done."
```

Set the Project item status to `Done`. If using `gh project item-edit`, first get the item ID from `gh project item-add` or `gh project item-list`. Current useful IDs for Project 2:

| Field / option | ID |
|---|---|
| Project ID | `PVT_kwDODljeoc4BJyT6` |
| Status field | `PVTSSF_lADODljeoc4BJyT6zg52M_I` |
| Status `Backlog` | `f75ad846` |
| Status `Ready` | `08afe404` |
| Status `In progress` | `47fc9ee4` |
| Status `In review` | `4cc61d42` |
| Status `Done` | `98236657` |

Example:

```bash
gh project item-edit \
  --project-id PVT_kwDODljeoc4BJyT6 \
  --id <project_item_id> \
  --field-id PVTSSF_lADODljeoc4BJyT6zg52M_I \
  --single-select-option-id 98236657
```

### Retroactive tracking

If a user asks to document work that was already completed, create a completed issue anyway. Include the shipped commit/PR links and verification notes in the issue body, add it to the Project, set it to `Done`, and close it as completed. This keeps the Project useful as a historical feature ledger, not only a future to-do list.

Example issue body shape:

```markdown
Implemented: 2026-07-04
State: Shipped to production via PR #2, merged 2026-07-04 08:28 UTC.
Production verification: app.myphysio.care returned HTTP 200 on 2026-07-04 08:36 UTC.

Commit: <sha>
Commit link: https://github.com/my-Physio/myPhysio-merge/commit/<sha>
Pull request: https://github.com/my-Physio/myPhysio-merge/pull/<number>

Summary:
- <what changed>

Verification:
- <commands/checks run>
```

## Backend Server Access

- Host: see `secrets/myphysio/info-about-myphysio.txt` (root@VPS).
- **Password SSH is disabled** (`/etc/ssh/sshd_config.d/00-hardening.conf`). Auth is via the ed25519 key in the secret bundle.
- Human access: `ssh myphysio` (via `~/.ssh/config` Host entry) or `ssh -i secrets/myphysio/id_ed25519 root@<host>`.
- Agent access: the `myphysio-ssh` local MCP server (`secrets/myphysio/ssh-mcp.mjs`, registered in the Claude desktop config) provides a `run_remote` tool. It prefers the key and needs no interaction.
- Emergency: Hostinger web console (browser-based) if keys are ever lost.
- fail2ban/ufw are NOT enabled; the security posture is key-only SSH.
- Supabase CLI access: the VPS root account has `SUPABASE_ACCESS_TOKEN` exported in `/root/.bashrc`. The value is intentionally not documented. Because `.bashrc` returns early for non-interactive shells, agent SSH commands that need Supabase auth should first run `PS1=codex; source ~/.bashrc >/dev/null 2>&1` in the remote bash session, then run `npx supabase ...`. Never print or paste the token value.

## Frontend Architecture Notes (PWA)

These were added over time and are easy to break if you don't know they exist:

- **Service worker** (`apps/mobile/public/sw.js`): app-shell precache (build ID + entry bundle injected at deploy time by `deploy.sh`/`deploy-dev.sh` via `sed`), cache-first for `/_expo/`, `/assets/`, `/fonts/`, `/pwa/`, network-first navigations with offline fallback, Supabase video/media caching with Range support, push notifications. Only complete (200) responses are cached — media Range requests return 206 which the Cache API rejects.
- **Snapshot cache** (`apps/mobile/context/GlobalData.tsx`): the last successful fetch is persisted to AsyncStorage; startup hydrates from it instantly while Supabase refreshes in the background (`syncing` state in context). Role/profile state is only committed after a successful fetch — never regress this, it prevents the offline "empty account" bug. Startup query batches are intentionally wrapped with an 8 s timeout so a stalled cold-launch Supabase request cannot pin the app on the loading screen forever; the catch/finally path must still release `loading` without committing half-fetched role/profile state.
- **Offline write queue / outbox** (`apps/mobile/lib/outbox.ts`): exercise completions and pain levels are enqueued as idempotent upserts, flushed on launch/reconnect/before refresh, and overlaid on both snapshot hydration and fresh fetches so queued writes never visually disappear. Workout logs (`hooks/useWorkoutCounter.ts`) use a `synced` flag with retry-on-reconnect instead.
- **Startup splash**: inline HTML/CSS overlay in `apps/mobile/app/+html.tsx` (logo `public/assets/logo_startup.svg` + shimmer), minimum 800 ms display, hidden by `app/index.tsx` when data is ready (helpers in `components/bootSplash.ts`). `app/index.tsx` also has a 10 s hard failsafe so the route gate cannot be terminal even if global loading gets stuck. On web the overlay is the ONLY splash — `index.tsx` renders `null` beneath it; native uses `components/BrandedSplash.tsx` (Reanimated + `SvgXml`, logo XML in `components/logoStartupXml.ts` — Illustrator CSS classes must stay inlined as attributes).
- **Supabase client fetch timeout** (`apps/mobile/lib/supabase.ts`): Supabase REST/auth/client requests use a custom `fetch` with a 15 s `AbortController` timeout. This bounds hung API calls while staying longer than the 8 s startup budget. Patient media playback uses public URLs/browser fetch/service-worker paths, not this Supabase client fetch.
- **Exercise deck filter** (`apps/mobile/lib/exerciseDeck.ts`): the patient home deck filters prescriptions with `p.patient_visible !== false && p.status !== 'cancelled'`. Do **not** revert this to `status === 'active'`. The intent is that date-expired programs (stored status still `'active'`) remain on the patient's deck unless the physio explicitly hides them via `patient_visible = false`. Cancelling a program always hides it. The physio's derived "Completed" display label (date past end) is web-admin-only and never written back to the DB.
- **Exercise rationale / "Why this helps"**: patient exercise pages only show this card when editable data provides a non-empty rationale (`exercices.benefit_rationale` or a per-prescription `benefitRationale` override). There is intentionally no bundled exercise-code fallback map; blank rationale means the card is hidden.
- **Fonts**: web loads self-hosted woff2 from `apps/mobile/public/fonts/` without blocking first paint; native uses the bundled TTFs. Blocking render on fonts was a major startup cost — don't reintroduce it.
- **Metro React pin** (`apps/mobile/metro.config.js`): `react`/`react-dom` are force-resolved to the mobile app's copies. The monorepo also contains React 18 (`apps/web`), and pnpm's hidden hoist folder (`node_modules/.pnpm/node_modules`) can expose it to Metro depending on install order, silently bundling two Reacts → runtime crash (React error #525, blank screen). **Never remove this pin.** After any build, sanity check: `grep -c '"18\.3\.1"' <entry bundle>` should be 0.
- **nginx**: gzip (`/etc/nginx/conf.d/gzip.conf`) + brotli (`/etc/nginx/conf.d/brotli.conf`, module installed) with `gzip_static`/`brotli_static` serving deploy-time precompressed files. Hashed assets are cached immutable 30d; `index.html` and `sw.js` are no-cache.

## Local Development

Restore local env files from the ignored secret bundle if missing:

```bash
cp /Users/pw1246/Documents/GitHub/wpdocs/secrets/myphysio/web.env /Users/pw1246/Documents/GitHub/myPhysio-merge/apps/web/.env
cp /Users/pw1246/Documents/GitHub/wpdocs/secrets/myphysio/mobile.env /Users/pw1246/Documents/GitHub/myPhysio-merge/apps/mobile/.env
```

Local dev-server ports are shared across the whole laptop, not just this repo.
Before starting `next dev`, `expo start`, or any other dev server, check the
laptop-wide convention in [Local Dev Server Ports](../systems/local-dev-servers.md)
and the ignored live registry:

```text
/Users/pw1246/Documents/GitHub/wpdocs/secrets/local-dev-ports.md
```

Then verify the port with `lsof -nP -iTCP:<port> -sTCP:LISTEN`, claim a free
port in that registry, and start the server with the port explicitly set. The
usual framework defaults are `3000` for the web/admin Next.js app and `8081` for
the mobile Expo/Metro server, but those may already be occupied by another local
project.

Install and run from the monorepo root:

```bash
cd /Users/pw1246/Documents/GitHub/myPhysio-merge
pnpm install
pnpm --filter @myphysio/web dev
pnpm --filter @myphysio/mobile dev
```

If you claimed alternate ports:

```bash
corepack pnpm --filter @myphysio/web dev -- -p <web_port>
corepack pnpm --filter @myphysio/mobile dev -- --port <mobile_port>
```

Useful checks: use `corepack pnpm` because the repo pins `pnpm@8.15.0`; a newer global pnpm against the v8 lockfile can misbehave. Current repo-health checks are:

```bash
corepack pnpm install
corepack pnpm type-check
corepack pnpm lint
corepack pnpm --filter @myphysio/mobile test -- --runInBand
corepack pnpm --filter @myphysio/web build
```

`corepack pnpm type-check` now runs real package-level tasks for web, mobile, and shared packages. `corepack pnpm lint` runs web lint and shared-package TypeScript checks; mobile ESLint has a larger existing baseline and is intentionally a documented no-op until that baseline is addressed. `apps/web` TypeScript and Next lint are now enforced during `next build`.

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

Notable columns added in July 2026:
- `prescriptions.patient_visible boolean NOT NULL DEFAULT true` — physio-controlled flag; when `false`, the prescription's exercises are hidden from the patient's home deck regardless of `status`. Set from the web admin program form or program history Eye/EyeOff toggle. Read by the mobile app on each sync.
- `patient_physios.notes text` — physio-private free-text note per patient connection (nullable). Displayed and inline-edited under the patient name in the web admin Users table.

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

1. Type-check on the server after every change set: `cd /var/www/myphysio-monorepo-dev && corepack pnpm type-check` — must be 0 errors before committing. For mobile-only work, also run `corepack pnpm --filter @myphysio/mobile test -- --runInBand`.
2. Commit on `dev` with a descriptive message, push, then deploy the relevant dev target: patient PWA uses `bash deploy-dev.sh` (run via `nohup ... &` and poll the log); admin dashboard uses the `NEXT_PUBLIC_ADMIN_BASE_PATH=/admin` build + `myphysio-web-dev` PM2 restart shown above.
3. Verify the deploy: for PWA, `tail /tmp/deploy-dev.log`, check the release symlink, curl key URLs, and check the built bundle for regressions (e.g., dual-React: `grep -c '"18\.3\.1"' entry-*.js` must be 0). For admin, curl `https://dev.myphysio.care/admin/login`, confirm `/admin/_next/...` assets return 200, and check `pm2 ls` shows `myphysio-web-dev` online.
4. Ask the user to test on `https://dev.myphysio.care` for PWA work or `https://dev.myphysio.care/admin/login` for admin work. Do not promote to prod without user sign-off.

### How to deploy to production

```bash
# from the dev checkout, after user approval:
gh pr create --base main --head dev --repo my-Physio/myPhysio-merge \
  --title "Promote dev to main" \
  --body "Promote the tested dev branch to production."
gh pr checks --watch
gh pr merge --merge
# the VPS poller deploys automatically within ~1 minute
tail -f /var/log/myphysio-deploy.log   # wait for "finished rc=0"
```

Then verify prod: `https://admin.myphysio.care/login` and `https://app.myphysio.care/` return 200, `/var/www/myphysio-app` points at the new timestamped release, brotli/static serving works, `pm2 ls` shows `myphysio-web` online, and the Expo web entry bundle still has `grep -c '"18\.3\.1"' entry-*.js` equal to 0. If broken, roll back the symlink (see Branch & Deploy Model) first, debug second. Do not merge the promotion PR until CI is green.

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
- The old app-specific Docker/npm deploy path (`apps/web/Dockerfile`, `apps/web/deploy.sh`, `apps/web/auto_deploy.sh`, image `myphysio-admin`) is dead — Docker runs no containers on the VPS. PM2 via the root monorepo `deploy.sh` is the active web path.

## Current Status (July 2026)

- Branches: `dev` (working) and `main` (prod) in `github.com:my-Physio/myPhysio-merge`; stale `feature/email-confirmation` branch deleted (its unmerged email-verification work was reviewed and intentionally dropped).
- Recent shipped work (production): nginx gzip+brotli with deploy-time precompression; woff2 non-blocking fonts; SW app-shell precache and offline start; instant startup from snapshot cache with bounded startup fetches; offline write queue with sync-on-reconnect; prescribed-media pre-caching; GitHub-style activity heatmap on the workout counter; full-history statistics charts with per-day adherence shading; branded startup splash (min 800 ms, 10 s failsafe) using `logo_startup.svg`; syncing indicator; pull-based deploys; key-only SSH; repo-health scripts and CI checks; PR-based production promotion; YouTube exercise video links with automatic thumbnails; duplicate YouTube link support; Manage Program library panel scrolling; exercise creation modal styling/height alignment.
- Current in-progress app work: Default Program admin tab (Issue #30) unless the project board says otherwise.
- Known open items / ideas: patient-side toggle to show/hide individual programs on their own home screen (Issue 4b — needs `patient_program_preferences` table or similar, deferred); wifi-only or size-capped media pre-caching; offline edit/delete replay for already-synced workout entries; physio-side offline support; Apple Developer account → EAS iOS build + TestFlight pipeline (deferred until UI/UX polish is done).

## Resume / Log Notes

- Built and maintained a full-stack physiotherapy platform spanning web admin, mobile patient experience, shared packages, and Supabase backend services.
- Added clinician workflows for patient management, exercise/program creation, prescription management, patient logs, and analytics.
- Added mobile workflows for exercise guidance, workout counting, statistics, notifications, and physio/patient role support.
- Engineered offline-first PWA architecture: service-worker app-shell and media caching, on-device data snapshots, idempotent offline write queue with reconnect sync.
- Cut PWA payloads ~80% (brotli/gzip precompression, woff2 fonts, non-blocking font loading) and added a branded animated startup experience.
- Replaced push-based CI deploys with a resilient pull-based server deploy pipeline; hardened server access to key-only SSH.
- Handled sensitive deployment material by keeping live credentials in a local gitignored bundle, out of public/project documentation.
