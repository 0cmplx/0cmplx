# 0cmplx

Research and experimental platform for testing AI agents, MCP servers, and tool integrations.

## Repositories

| Repo | Visibility | URL | Purpose |
|---|---|---|---|
| 0cmplx | Public | github.com/0cmplx/0cmplx | Central governance hub (this repo) |
| engine | Public | github.com/0cmplx/engine | @0cmplx/engine: shared core (Pyodide, parsers, bridge, traps) |
| cli | Public | github.com/0cmplx/cli | @0cmplx/cli: command-line interface |
| server | Public | github.com/0cmplx/server | Hono API server + MCP protocol |
| web | Public | github.com/0cmplx/web | Astro 6 + React 19 dashboard |
| docs | Public | github.com/0cmplx/docs | Documentation site (@supaproxy/supadocs) |

## Local directories

```
/Users/Elvis/workspace/0cmplxHq/
  0cmplx-hub/       this repo
  0cmplx-engine/    @0cmplx/engine
  0cmplx-cli/       @0cmplx/cli
  0cmplx-server/    API server
  0cmplx-web/       Dashboard
  0cmplx-docs/      Documentation
```

## Brand

- Name: 0cmplx (zero complexity)
- Domain: 0cmplx.com
- Colours: black (#0a0a0a dark), grey (#f5f5f5 light), monochrome
- Font: Geist Mono
- Logo: rotated capsule SVG

## Architecture

- Engine: shared npm package, Pyodide sandbox, OpenAPI parser, relationship graph, host bridge, OWASP traps
- Server: Hono + TypeScript, Redis, SQLite, DDD layers, DI via container.ts
- Web: Astro 6 + React 19 + Tailwind CSS 4
- CLI: thin wrapper over engine, terminal output
- Docs: @supaproxy/supadocs framework

## npm packages

| Package | Repo | Purpose |
|---|---|---|
| @0cmplx/engine | engine | Shared core |
| @0cmplx/cli | cli | Command-line interface |

## Git workflow

- NEVER push directly to main. Always create a feature branch and open a PR.
- Branch naming: `feat/`, `fix/`, `chore/`, `docs/` prefixes.
- Squash merge to main.

## Writing standards

- British English throughout
- No em dashes or en dashes
- Sentence case for headings
- Geist Mono font in all UI
