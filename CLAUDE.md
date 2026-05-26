# 0cmplx

Research and experimental platform for testing AI agents, MCP servers, and tool integrations.

## Repositories

### Open source

| Repo | URL | Purpose |
|---|---|---|
| 0cmplx | github.com/0cmplx/0cmplx | Central governance hub (this repo) |
| cli | github.com/0cmplx/cli | @0cmplx/cli: thin client to 0cmplx cloud |
| docs | github.com/0cmplx/docs | Documentation site (@supaproxy/supadocs) |

### Proprietary

| Repo | URL | Purpose |
|---|---|---|
| engine | github.com/0cmplx/engine | Core engine (Pyodide, parsers, bridge, traps) |
| server | github.com/0cmplx/server | Hono API server + MCP protocol |
| web | github.com/0cmplx/web | Astro 6 + React 19 dashboard |

## Local directories

```
/Users/Elvis/workspace/0cmplxHq/
  0cmplx-hub/       this repo
  0cmplx-engine/    core engine (private)
  0cmplx-cli/       @0cmplx/cli (public)
  0cmplx-server/    API server (private)
  0cmplx-web/       Dashboard (private)
  0cmplx-docs/      Documentation (public)
```

## Architecture

```
CLI (thin client, public)
  |
  |  HTTPS + streaming
  v
Cloud API (api.0cmplx.com)
  |
  v
Engine (Pyodide, parsers, bridge, traps) <- proprietary
  |
  v
User's API / Ephemeral DB
```

- **Engine**: proprietary core. Pyodide sandbox, OpenAPI parser, relationship graph, host bridge, OWASP traps. Runs on our servers only.
- **Server**: Hono + TypeScript, Redis, SQLite, DDD layers, DI via container.ts. Hosts the engine.
- **Web**: Astro 6 + React 19 + Tailwind CSS 4. Dashboard at 0cmplx.com.
- **CLI**: thin client to the cloud API. Authenticates via API tokens, interactive REPL shell. Does not import the engine.
- **Docs**: @supaproxy/supadocs framework. Public.

## Deployment

- Droplet: 142.93.47.151 (Ubuntu 24.04, DigitalOcean London)
- Code: /opt/0cmplx/server, /opt/0cmplx/web
- Services: 0cmplx-server.service, 0cmplx-web.service, caddy.service
- URLs: 0cmplx.com (web), api.0cmplx.com (server), mcp.0cmplx.com (MCP)
- DNS: GoDaddy, A records for @, api, mcp

## npm packages

| Package | Repo | Purpose |
|---|---|---|
| @0cmplx/cli | cli | Command-line client (public) |

## Skills

All skills are centralised in this repo. Run them from here regardless of which repo you are working in.

| Skill | Purpose |
|---|---|
| `/deploy` | Deploy services to the DigitalOcean production droplet |
| `/dev` | Start local dev environment (server, web, CLI, Redis) |
| `/ship` | Cross-repo commit, push, PR, and merge in one go |

## Brand

- Name: 0cmplx (zero complexity)
- Domain: 0cmplx.com
- Colours: black (#0a0a0a dark), grey (#f5f5f5 light), monochrome
- Font: Geist Mono
- Logo: rotated capsule SVG

## Git workflow

- NEVER push directly to main. Always create a feature branch and open a PR.
- Branch naming: `feat/`, `fix/`, `chore/`, `docs/` prefixes.
- Squash merge to main.

## Writing standards

- British English throughout
- No em dashes or en dashes
- Sentence case for headings
- Geist Mono font in all UI
