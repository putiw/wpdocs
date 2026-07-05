# Local Dev Server Ports

Use this convention when multiple agents or projects need local dev servers on
the same laptop. TCP ports are machine-global, so a server in one repo can block
another repo even when their source trees are unrelated.

## Active Registry

Temporary live claims belong in the ignored local file:

```text
/Users/pw1246/Documents/GitHub/wpdocs/secrets/local-dev-ports.md
```

That file is intentionally not committed. Use it as a scratchpad for active
sessions only.

## Workflow

Before starting a local dev server:

1. Check whether the desired port is already listening:

   ```bash
   lsof -nP -iTCP:<port> -sTCP:LISTEN
   ```

2. Check the active registry:

   ```bash
   sed -n '1,200p' /Users/pw1246/Documents/GitHub/wpdocs/secrets/local-dev-ports.md
   ```

3. Claim a free port in the registry with the project, owner, date, and command.
4. Start the dev server with that port explicitly set.
5. Delete the claim when the server is stopped.

If the registry and `lsof` disagree, trust `lsof` for current machine state and
then update the registry.

## Port Ranges

| Range | Intended use |
|---|---|
| `3000-3099` | Next.js and other web app defaults |
| `5100-5198` | Agent feature branches, previews, and sandboxes |
| `5199` | puppet ROM sandbox |
| `8081-8099` | Expo / Metro dev servers |

## Common Defaults

| Tool | Default port | Notes |
|---|---:|---|
| Next.js `next dev` | `3000` | Override with `-p <port>` or `PORT=<port>`. |
| Vite | `5173` | Override with `--port <port>`. |
| Expo / Metro `expo start` | `8081` | Override with `--port <port>`. |
| MkDocs `mkdocs serve` | `8000` | Override with `-a 127.0.0.1:<port>`. |

## Examples

```bash
# Next.js
corepack pnpm --filter @myphysio/web dev -- -p 5101

# Expo / Metro
corepack pnpm --filter @myphysio/mobile dev -- --port 8082

# Vite
npm run dev -- --port 5102
```
