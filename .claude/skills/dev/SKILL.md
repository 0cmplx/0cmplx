---
name: dev
description: Start the local 0cmplx development environment. Kills stale processes, clears caches, starts server and web, verifies health, links CLI.
---

## Configuration

| Service | Port | Directory |
|---|---|---|
| Server | 3002 | /Users/Elvis/workspace/0cmplxHq/0cmplx-server |
| Web | 4322 | /Users/Elvis/workspace/0cmplxHq/0cmplx-web |
| CLI | - | /Users/Elvis/workspace/0cmplxHq/0cmplx-cli |
| Docs | 3900 | /Users/Elvis/workspace/0cmplxHq/0cmplx-docs |
| CLI Docs | 3901 | /Users/Elvis/workspace/0cmplxHq/0cmplx-cli-docs |
| Redis | 6390 | Docker (server-redis-1) |

## Important

**Always run locally to test changes before suggesting deploy or ship.** This applies to all repos: server, web, CLI, docs.

## Step 1: Kill stale processes

Kill anything on ports 3002 and 4322 to avoid conflicts:

```bash
lsof -ti:3002 2>/dev/null | xargs kill -9 2>/dev/null
lsof -ti:4322 2>/dev/null | xargs kill -9 2>/dev/null
```

Wait 1 second for ports to free.

## Step 2: Verify Redis

Redis must be running on port 6390 (Docker container):

```bash
docker ps | grep "6390->6379"
```

If not running:
```bash
docker start server-redis-1
```

If no container exists, check the server repo for a docker-compose.yml.

## Step 3: Clear Vite cache

Stale Vite cache causes the web app to serve old code:

```bash
rm -rf /Users/Elvis/workspace/0cmplxHq/0cmplx-web/node_modules/.vite
```

## Step 4: Start server

```bash
cd /Users/Elvis/workspace/0cmplxHq/0cmplx-server && npm run dev &
```

Wait 3 seconds, then verify:
```bash
curl -s http://localhost:3002/health
```

Expected: `{"status":"ok"}`

The server uses `tsx watch` which reads `.env` and `.env.local`. GitHub OAuth credentials are in `.env.local` (gitignored).

## Step 5: Start web

```bash
cd /Users/Elvis/workspace/0cmplxHq/0cmplx-web && pnpm dev &
```

Wait 3 seconds, then verify:
```bash
curl -s -o /dev/null -w "%{http_code}" http://localhost:4322/
```

Expected: `200`

The web app reads `PUBLIC_API_URL` from env, defaults to `http://localhost:3002`.

## Step 6: Start docs (if requested or changed)

Only start docs sites if the user is working on documentation or explicitly requests it.

Docs sites run on separate ports. Kill stale processes first:

```bash
lsof -ti:3900 2>/dev/null | xargs kill -9 2>/dev/null
lsof -ti:3901 2>/dev/null | xargs kill -9 2>/dev/null
```

```bash
cd /Users/Elvis/workspace/0cmplxHq/0cmplx-docs && PORT=3900 npx supadocs dev &
cd /Users/Elvis/workspace/0cmplxHq/0cmplx-cli-docs && PORT=3901 npx supadocs dev &
```

Wait 3 seconds, then verify:
```bash
curl -s -o /dev/null -w "%{http_code}" http://localhost:3900/
curl -s -o /dev/null -w "%{http_code}" http://localhost:3901/
```

Note: supadocs may ignore the PORT env var and default to 3900. If so, start them sequentially and the second will auto-increment to 3901.

## Step 7: Link CLI (if not already linked)

```bash
which 0cmplx || (cd /Users/Elvis/workspace/0cmplxHq/0cmplx-cli && npm run build && npm link)
```

The CLI uses `CMPLX_API_URL` env var. For local dev:
```bash
CMPLX_API_URL=http://localhost:3002 0cmplx
```

For production (default): `0cmplx` (uses https://api.0cmplx.com)

## Step 8: Report

```
0cmplx dev environment ready

  Server:  http://localhost:3002  OK
  Web:     http://localhost:4322  OK
  Redis:   localhost:6390         OK
  CLI:     0cmplx (linked)       OK
  OAuth:   {enabled/disabled}

  CLI local usage:
    CMPLX_API_URL=http://localhost:3002 0cmplx
```

## Gotchas

- **tsx watch does not always pick up new files.** If you add new source files, restart the server manually.
- **Vite cache must be cleared** after code changes to the web app. This skill does it automatically.
- **Port 4322 conflict**: the old ropuppy monorepo may be running on 4322. This skill kills it.
- **CLI env var**: `CMPLX_API_URL` must be set for local dev, otherwise the CLI hits production.
