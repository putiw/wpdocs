# myPhysio

myPhysio is a physiotherapy platform with a clinician/admin web app, a patient-facing mobile app, shared packages, and Supabase backend services.

## Agent Start Here

For a contextless AI agent, this page is the starting runbook. Use it together with the local gitignored secret bundle:

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
| `secrets/myphysio/info-about-myphysio.txt` | Backend SSH details and Supabase keys copied from the private desktop note. |
| `secrets/myphysio/web.env` | Environment variables for `apps/web/.env`. |
| `secrets/myphysio/mobile.env` | Environment variables for `apps/mobile/.env`. |

Before using or editing secrets, verify they are ignored:

```bash
cd /Users/pw1246/Documents/GitHub/wpdocs
git check-ignore -v secrets/myphysio/info-about-myphysio.txt secrets/myphysio/web.env secrets/myphysio/mobile.env
```

Never paste passwords, service-role keys, JWTs, `.env` values, or SSH credentials into tracked docs, commits, issues, PRs, or chat output.

## What This Is

The project is maintained as a pnpm/Turborepo monorepo:

```text
/Users/pw1246/Documents/GitHub/myPhysio-merge
```

The main parts are:

| Area | Path | Purpose |
|---|---|---|
| Web admin app | `apps/web` | Next.js dashboard for physiotherapists and administrators. |
| Mobile app | `apps/mobile` | Expo/React Native app for patients and physiotherapist-facing mobile workflows. |
| Shared library | `packages/lib` | Shared constants, types, exercise helpers, prescription helpers, and statistics logic. |
| Design system | `packages/design-system` | Shared design tokens/components used across apps. |
| Supabase functions | `supabase/functions` | Reminder-related Edge Functions. |

## Key App Assets

Editable logo/source assets are in the landing page asset folder:

```text
/Users/pw1246/Documents/GitHub/myPhysio-merge/apps/web/landing-page/assets/logo_v3.ai
/Users/pw1246/Documents/GitHub/myPhysio-merge/apps/web/landing-page/assets/logo_v2.1.ai
/Users/pw1246/Documents/GitHub/myPhysio-merge/apps/web/landing-page/assets/logo_v2.ai
/Users/pw1246/Documents/GitHub/myPhysio-merge/apps/web/landing-page/assets/logo_v2.svg
```

Related editable landing-page artwork:

```text
/Users/pw1246/Documents/GitHub/myPhysio-merge/apps/web/landing-page/assets/banner_v2.ai
/Users/pw1246/Documents/GitHub/myPhysio-merge/apps/web/landing-page/assets/banner_v2.svg
/Users/pw1246/Documents/GitHub/myPhysio-merge/apps/web/landing-page/assets/hero_without_logo.ai
/Users/pw1246/Documents/GitHub/myPhysio-merge/apps/web/landing-page/assets/tab.ai
```

Generated mobile/PWA icon targets are in the Expo app:

```text
/Users/pw1246/Documents/GitHub/myPhysio-merge/apps/mobile/assets/images/icon.png
/Users/pw1246/Documents/GitHub/myPhysio-merge/apps/mobile/assets/images/splash-icon.png
/Users/pw1246/Documents/GitHub/myPhysio-merge/apps/mobile/assets/images/favicon-32.png
/Users/pw1246/Documents/GitHub/myPhysio-merge/apps/mobile/pwa-icons/icon-192.png
/Users/pw1246/Documents/GitHub/myPhysio-merge/apps/mobile/pwa-icons/icon-512.png
```

`apps/mobile/app.json` currently points the Expo app icon and Android adaptive foreground image at `./assets/images/icon.png`, the splash image at `./assets/images/splash-icon.png`, and the web favicon at `./assets/images/favicon-32.png`. If changing the brand mark, edit/export from the `.ai` source and regenerate these PNG targets. The in-app header logo is rendered as text in `apps/mobile/components/PhysioHeader.tsx`, not as an image.

An older duplicate source set exists at:

```text
/Users/pw1246/Documents/GitHub/myPhysio-Admin-Frontend/landing-page/assets/
```

As of 2026-07-03, local searches across Documents, Desktop, and Downloads did not find another separate myPhysio logo source file outside the merged repo and older duplicate repo.

## My Role

- Maintained the merged project repo and application structure.
- Added clinician-facing workflows for patients, exercises, prescriptions, logs, and program management.
- Added patient/mobile workflows for assigned exercises, workout tracking, notifications, and app feedback.
- Maintained deployment notes and build scripts for web and mobile delivery.
- Kept private deployment credentials and Supabase secrets out of git-facing documentation.

## Systems Involved

| System | Purpose |
|---|---|
| Next.js web app | Clinician/admin dashboard, authentication, exercise management, patient management, and program workflows. |
| Expo mobile app | Patient exercise experience, body map/pain views, workout counter, physio/patient role flows, and mobile notifications. |
| Supabase | Auth, PostgreSQL data, storage, client APIs, service-role operations, and Edge Functions. |
| Hostinger/VPS | Backend/server deployment and automated mobile APK build/deployment scripts. |
| Vercel | Optional/referenced deployment target for the web admin app. |
| Public site/downloads | Landing page assets and Android APK download endpoint. |

## Main Workflows

| Workflow | Notes |
|---|---|
| Web admin dashboard | Manage users/patients, exercises, programs, prescriptions, logs, analytics, and profile details. |
| Mobile patient app | Show assigned exercises, body map, program progress, statistics, workout counter, and exercise completion feedback. |
| Physiotherapist mobile views | Role selection, patient list/details, patient logs, program creation/management, exercise creation, and stats. |
| Video/exercise media | Exercise media support includes larger video uploads and optional sound feedback in the mobile flow. |
| Notifications/reminders | Supabase Edge Functions include `send-reminders` and `send-manual-reminder`. |
| Android build delivery | Mobile README records automated Android builds on the VPS and APK serving from `https://myphysio.care/downloads/myphysio.apk`. |

## Local Development

Restore local environment files from the ignored `wpdocs` secret bundle if they are missing:

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

Useful checks:

```bash
pnpm build
pnpm lint
pnpm type-check
```

The mobile app can also be started from `apps/mobile` with:

```bash
npx expo start
```

## Backend Server Access and Deployment

Backend SSH credentials are local-only. Read them from:

```text
/Users/pw1246/Documents/GitHub/wpdocs/secrets/myphysio/info-about-myphysio.txt
```

The original private note is still at:

```text
/Users/pw1246/Desktop/log/info/info about myphysio.txt
```

Expected server checkout used by the GitHub Actions deployment workflow:

```text
/var/www/myphysio-monorepo
```

The workflow in `.github/workflows/deploy.yml` connects with GitHub Actions secrets, then runs:

```bash
cd /var/www/myphysio-monorepo
git pull origin main
bash deploy.sh
```

Root deployment script:

```text
/Users/pw1246/Documents/GitHub/myPhysio-merge/deploy.sh
```

Important deployment facts from the root script:

| Setting | Value |
|---|---|
| Web PM2 process | `myphysio-web` |
| Web port | `3000` |
| Clinician/admin dashboard domain | `admin.myphysio.care` |
| Mobile web/PWA domain | `app.myphysio.care` |
| Mobile release directory | `/var/www/myphysio-app-releases` |
| Mobile live symlink | `/var/www/myphysio-app` |

The root deploy script pulls latest code, installs pnpm dependencies, checks for web/mobile `.env` files, builds the monorepo, reloads or starts the Next.js web app with PM2, exports the Expo web build, injects PWA assets, and updates the mobile live symlink.

Domain routing notes:

| Domain | Serves |
|---|---|
| `myphysio.care` | Landing page. |
| `admin.myphysio.care` | Next.js clinician/admin dashboard. The landing page routes clinician login/signup links here. |
| `app.myphysio.care` | Mobile web/PWA build from the Expo app. |

The `APP_DOMAIN="app.myphysio.care"` variable in the root deploy script refers to the mobile web/PWA domain, not the clinician dashboard.

There is also an older/app-specific web deployment path:

```text
/Users/pw1246/Documents/GitHub/myPhysio-merge/apps/web/deploy.sh
/Users/pw1246/Documents/GitHub/myPhysio-merge/apps/web/auto_deploy.sh
```

That path builds a Docker image named `myphysio-admin`, runs a container named `myphysio-app`, and deploys landing page files to:

```text
/var/www/myphysio/landing
```

Before changing deployment, check which path is actually active on the server instead of assuming PM2 or Docker.

Android APK build automation:

```text
/Users/pw1246/Documents/GitHub/myPhysio-merge/apps/mobile/scripts/build-android.sh
```

That script runs local EAS Android builds on the VPS, writes `myphysio.apk`, updates `build_history.log`, and copies the APK to public download locations, including:

```text
/var/www/myphysio/landing/downloads/myphysio.apk
```

## Scripts / Repos

Project repo:

```text
/Users/pw1246/Documents/GitHub/myPhysio-merge
```

Private setup note:

```text
/Users/pw1246/Desktop/log/info/info about myphysio.txt
```

Local secret bundle for agents:

```text
/Users/pw1246/Documents/GitHub/wpdocs/secrets/myphysio/
```

!!! warning "Private credentials"
    The private setup note and local `wpdocs/secrets/myphysio/` files contain server access details and Supabase secrets. Do not copy passwords, service-role keys, `.env` values, JWTs, or SSH credentials into tracked docs.

Top-level monorepo commands:

```bash
pnpm install
pnpm dev
pnpm build
pnpm lint
pnpm type-check
```

Package-specific development commands:

```bash
pnpm --filter @myphysio/web dev
pnpm --filter @myphysio/mobile dev
```

Web app package:

```text
@myphysio/web
```

Mobile app package:

```text
@myphysio/mobile
```

## Operational Notes

Environment variables are required for both apps. Keep them in local deployment environments, Vercel settings, Supabase settings, or VPS secrets, not in git.

Common public client variables:

```text
NEXT_PUBLIC_SUPABASE_URL
NEXT_PUBLIC_SUPABASE_ANON_KEY
EXPO_PUBLIC_SUPABASE_URL
EXPO_PUBLIC_SUPABASE_ANON_KEY
```

Sensitive server-side variables:

```text
SUPABASE_SERVICE_ROLE_KEY
```

Supabase Edge Functions use Deno environment variables:

```text
SUPABASE_URL
SUPABASE_SERVICE_ROLE_KEY
```

Edge Function paths:

```text
/Users/pw1246/Documents/GitHub/myPhysio-merge/supabase/functions/send-reminders/index.ts
/Users/pw1246/Documents/GitHub/myPhysio-merge/supabase/functions/send-manual-reminder/index.ts
```

The public Supabase project host appears in app code and env files, but the current canonical values should be read from the ignored secret bundle before debugging deployments.

## Supabase Map

Use the ignored secret bundle for actual keys. The public docs should only describe how the app connects and which data surfaces exist.

| Surface | Variables / access |
|---|---|
| Web browser/client | `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY` |
| Web server/API routes | `NEXT_PUBLIC_SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY` where admin access is required |
| Mobile app | `EXPO_PUBLIC_SUPABASE_URL`, `EXPO_PUBLIC_SUPABASE_ANON_KEY` |
| Supabase Edge Functions | `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY` |

Server/admin Supabase usage appears in:

```text
apps/web/app/actions/program-actions.ts
apps/web/app/api/users/invite/route.ts
apps/web/app/api/admin/create-physio/route.ts
apps/web/app/api/video/compress/route.ts
supabase/functions/send-reminders/index.ts
supabase/functions/send-manual-reminder/index.ts
```

Main Supabase tables and buckets seen in the current code:

| Name | Type | Notes |
|---|---|---|
| `patients` | table | Patient profile records. |
| `physios` | table | Physiotherapist profile records and activation/connect-code state. |
| `patient_physios` | table | Patient-physio relationship records. |
| `exercices` | table | Exercise library table; note the spelling in code. |
| `prescriptions` | table | Assigned programs/prescriptions. |
| `patient_logs` | table | Patient log records shown in web/mobile physio views. |
| `body_map_pain_levels` | table | Body map pain tracking data. |
| `exercise_completions` | table | Exercise completion tracking. |
| `workout_logs` | table | Workout counter tracking. |
| `push_subscriptions` | table | Push notification subscriptions. |
| `physio_exercise_saves` | table | Saved exercise relationships for physiotherapists. |
| `physio_follows` | table | Hub/follow relationship records. |
| `exercise` | storage bucket | Exercise media storage. |

RPC/functions referenced by app code include:

```text
create_physio_record
invite_patient_by_email
```

Be careful with `apps/mobile/scripts/verify-supabase.js`: it currently checks an `exercises` table, while the active app code primarily uses `exercices`.

Mobile iOS setup has extra local-device build requirements in:

```text
/Users/pw1246/Documents/GitHub/myPhysio-merge/apps/mobile/IOS_SETUP.md
```

Web/Vercel deployment notes are in:

```text
/Users/pw1246/Documents/GitHub/myPhysio-merge/apps/web/VERCEL_DEPLOYMENT.md
```

## Current Status

- The actual code is in `/Users/pw1246/Documents/GitHub/myPhysio-merge`.
- The private operational note records the server and Supabase setup, but its secrets are intentionally excluded from this page.
- Recent local notes record added video sound, higher video upload limits, notifications, workout counter tracking, and patient-role views for physiotherapists.
- The repo has separate web, mobile, shared library, design-system, and Supabase function areas.

## Resume / Log Notes

- Built and maintained a full-stack physiotherapy platform spanning web admin, mobile patient experience, shared packages, and Supabase backend services.
- Added clinician workflows for patient management, exercise/program creation, prescription management, patient logs, and analytics.
- Added mobile workflows for exercise guidance, workout counting, statistics, notifications, and physio/patient role support.
- Maintained deployment documentation for web hosting, Android APK generation, iOS local builds, and private server operations.
- Handled sensitive deployment material by keeping live credentials in private notes and out of public/project documentation.
