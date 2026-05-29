# 0cmplx

Query composition engine for AI agents. Upload a SQL schema, ask questions in natural language, get answers from a sandboxed database.

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
CLI / MCP Client (e.g. Supaproxy)
  |
  |  HTTPS + MCP protocol
  v
Cloud API (api.0cmplx.com)
  |
  v
Engine (schema parser, relationship graph, query composer)
  |
  v
Pyodide sandbox + ephemeral DB
```

- **Engine**: proprietary core. SQL schema parser, relationship graph, query composition (natural language to SQL), Pyodide sandbox, OWASP security traps. Runs on our servers only.
- **Server**: Hono + TypeScript, Redis, SQLite, DDD layers, DI via container.ts. Hosts the engine. Exposes MCP tools (query, mutate, describe).
- **Web**: Astro 6 + React 19 + Tailwind CSS 4. Dashboard at 0cmplx.com.
- **CLI**: thin client to the cloud API. Authenticates via API tokens. Does not import the engine.
- **Docs**: @supaproxy/supadocs framework. Public.

## How 0cmplx works

1. User uploads a SQL schema
2. Engine parses tables, columns, FKs, constraints into a relationship graph
3. MCP server exposes tools: `query`, `mutate`, `describe`
4. Caller sends natural language (e.g. "Get status for user with phone +2781...")
5. Engine composes SQL using schema knowledge, executes in sandboxed DB
6. High confidence: executes immediately, returns data
7. Low confidence: returns what it could not resolve, caller clarifies

## 0cmplx and Supaproxy

0cmplx is the infrastructure layer. Supaproxy is the governance layer. They connect via MCP.

| | 0cmplx | Supaproxy |
|---|---|---|
| Knows | Database schema, relationships | Business rules, user context |
| Decides | How to query the data | What to ask for and why |
| Owns | Query composition, sandbox execution | Policy, compliance, routing |
| Sells | "Give me your schema, ask me anything" | "Governed AI for your team" |

Supaproxy tells 0cmplx exactly what to fetch, never why. 0cmplx returns data, never decisions. Neither product imports or depends on the other. MCP is the boundary.

## Deployment

- Droplet: 142.93.47.151 (Ubuntu 24.04, DigitalOcean London)
- Code: /opt/0cmplx/server, /opt/0cmplx/web, /opt/0cmplx/docs
- Services: 0cmplx-server.service, 0cmplx-web.service, caddy.service
- URLs: 0cmplx.com (web), api.0cmplx.com (server), mcp.0cmplx.com (MCP), docs.0cmplx.com (docs)
- DNS: GoDaddy, A records for @, api, mcp, docs

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
| `/learn` | Capture lessons from fixes into skills and memory. Run after every non-trivial fix before continuing. |
| `/audit` | Audit codebase for DDD, SOLID, clean architecture, and 0cmplx conventions. |

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
