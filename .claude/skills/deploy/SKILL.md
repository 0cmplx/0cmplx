---
name: deploy
description: Deploy 0cmplx services to the Digital Ocean production droplet. Pulls latest main, builds, and restarts services. Always deploys from main.
---

## Configuration

- Droplet: 142.93.47.151
- SSH: root@142.93.47.151
- Code: /opt/0cmplx/server, /opt/0cmplx/web, /opt/0cmplx/docs
- Services: 0cmplx-server.service, 0cmplx-web.service
- Caddy: manages TLS for 0cmplx.com, api.0cmplx.com, mcp.0cmplx.com, docs.0cmplx.com

## Services

| Service | Systemd | Repo | Code path | URL |
|---|---|---|---|---|
| server | 0cmplx-server | 0cmplx/server | /opt/0cmplx/server | api.0cmplx.com |
| web | 0cmplx-web | 0cmplx/web | /opt/0cmplx/web | 0cmplx.com |
| docs | (static, Caddy) | 0cmplx/docs | /opt/0cmplx/docs | docs.0cmplx.com |

## Step 1: Pre-flight checks

Check which services have changes to deploy:

```bash
ssh root@142.93.47.151 "cd /opt/0cmplx/server && git fetch origin 2>/dev/null && echo '--- server ---' && git log HEAD..origin/main --oneline; cd /opt/0cmplx/web && git fetch origin 2>/dev/null && echo '--- web ---' && git log HEAD..origin/main --oneline"
```

Only deploy services with new commits. Report which services need updating.

## Step 2: Confirm deployment

Show the user what will be deployed:

```
Deploying to: 142.93.47.151
Services to update:
  - <service>: <commit summary>
  - ...
```

Ask: **"Proceed with deployment?"**

Do NOT proceed without explicit user confirmation.

## Step 3: Deploy each service

For each service that needs updating:

### 3a. Pull latest code

```bash
ssh root@142.93.47.151 "cd /opt/0cmplx/<repo> && git pull origin main"
```

### 3b. Install dependencies and build

**Server:**
```bash
ssh root@142.93.47.151 "cd /opt/0cmplx/server && npm install && npm run build"
```

**Web:**
```bash
ssh root@142.93.47.151 "cd /opt/0cmplx/web && npm install && PUBLIC_API_URL=https://api.0cmplx.com npm run build"
```

Note: web needs `PUBLIC_API_URL` set at build time (Astro bakes it in).

### 3c. Restart the service

```bash
ssh root@142.93.47.151 "systemctl restart 0cmplx-<service> && sleep 2 && systemctl is-active 0cmplx-<service>"
```

### 3d. Verify health

**Server:**
```bash
curl -s https://api.0cmplx.com/health
```

**Web:**
```bash
curl -s -o /dev/null -w "%{http_code}" https://0cmplx.com
```

**Deploy order** (if multiple services):
1. Server first (API, others depend on it)
2. Web

### Docs (separate from server/web)

Docs is a static site served by Caddy. It has no systemd service.

```bash
ssh root@142.93.47.151 "cd /opt/0cmplx/docs && git fetch origin 2>/dev/null && echo '--- docs ---' && git log HEAD..origin/main --oneline"
```

If there are changes:

```bash
ssh root@142.93.47.151 "cd /opt/0cmplx/docs && git pull origin main && npm install && npm run build"
```

No restart needed. Caddy serves `/opt/0cmplx/docs/dist-docs` as static files.

Verify:
```bash
curl -s -o /dev/null -w "%{http_code}" https://docs.0cmplx.com
```

## Step 4: Final verification

After all services are deployed:

```bash
ssh root@142.93.47.151 "systemctl is-active 0cmplx-server 0cmplx-web caddy"
```

Verify all three show `active`.

## Step 5: Report

```
Deployment complete
  server: <commit hash> <message> OK
  web: <commit hash> <message> OK
  Skipped: <services with no changes>
```

## Safety rules (NON-NEGOTIABLE)

- **PR before deploy.** Every change must be merged to main before deployment.
- **Never deploy from a feature branch.** Always main.
- **Never modify .env on the droplet** without explicit user instruction.
- **Always confirm before deploying.** This is a production environment.
- **Never run destructive commands** on the droplet (rm -rf, etc.).
- If a build fails, stop and report. Do not continue with other services.
