---
name: ship
description: Cross-repo commit, push, PR, and merge for 0cmplx repos. Detects which repos have changes and ships them all in one go.
---

## Configuration

| Repo | Directory |
|---|---|
| server | /Users/Elvis/workspace/0cmplxHq/0cmplx-server |
| web | /Users/Elvis/workspace/0cmplxHq/0cmplx-web |
| cli | /Users/Elvis/workspace/0cmplxHq/0cmplx-cli |
| engine | /Users/Elvis/workspace/0cmplxHq/0cmplx-engine |
| docs | /Users/Elvis/workspace/0cmplxHq/0cmplx-docs |
| hub | /Users/Elvis/workspace/0cmplxHq/0cmplx-hub |

## Step 1: Detect changes

For each repo, check for uncommitted or unpushed changes:

```bash
for dir in 0cmplx-server 0cmplx-web 0cmplx-cli 0cmplx-engine 0cmplx-docs 0cmplx-hub; do
  path="/Users/Elvis/workspace/0cmplxHq/$dir"
  if [ -d "$path/.git" ]; then
    status=$(cd "$path" && git status --short)
    branch=$(cd "$path" && git branch --show-current)
    if [ -n "$status" ] || [ "$branch" != "main" ]; then
      echo "$dir ($branch): $status"
    fi
  fi
done
```

Report which repos have changes. Skip repos with no changes.

## Step 2: Confirm scope

Show the user:

```
Repos with changes:
  - server: N files modified
  - web: N files modified
  - cli: N files modified
```

Ask the user for:
1. A commit message (or use one message for all)
2. Branch name prefix (e.g. `feat/token-auth`)

If the user provides a single message, use it for all repos. If different, ask per repo.

## Step 3: Run tests

Before shipping, run tests in repos that have them:

**Server:**
```bash
cd /Users/Elvis/workspace/0cmplxHq/0cmplx-server && npx vitest run
```

**CLI:**
```bash
cd /Users/Elvis/workspace/0cmplxHq/0cmplx-cli && npm run build
```

**Web:**
```bash
cd /Users/Elvis/workspace/0cmplxHq/0cmplx-web && pnpm build
```

If any test or build fails, stop and report. Do not proceed.

## Step 4: Ship each repo

For each repo with changes, in order (server first, then web, then CLI):

### 4a. Create branch

```bash
cd <repo-dir>
git checkout main && git pull
git checkout -b <branch-name>
```

If already on a feature branch, use it.

### 4b. Stage and commit

Stage specific files (never `git add -A`):

```bash
git add <files>
git commit -m "<message>

Co-Authored-By: Claude Opus 4.6 (1M context) <noreply@anthropic.com>"
```

### 4c. Push and create PR

```bash
git push -u origin <branch-name>
gh pr create --title "<title>" --body "<body>"
```

### 4d. Merge

```bash
gh pr merge --squash --delete-branch
git checkout main && git pull
```

## Step 5: Post-ship checks

After all repos are merged:

```bash
for dir in 0cmplx-server 0cmplx-web 0cmplx-cli; do
  path="/Users/Elvis/workspace/0cmplxHq/$dir"
  branch=$(cd "$path" && git branch --show-current)
  status=$(cd "$path" && git status --short)
  echo "$dir: branch=$branch clean=$([ -z \"$status\" ] && echo yes || echo no)"
done
```

All should be on `main` with clean working trees.

## Step 6: Report

```
Shipped N repos:
  server: <commit> <PR URL> OK
  web: <commit> <PR URL> OK
  cli: <commit> <PR URL> OK

All repos on main, clean working trees.
Next: run /deploy to push to production.
```

## Safety rules

- **Tests must pass before shipping.** Never skip tests.
- **Never push to main.** Always branch, PR, merge.
- **Never stage .env files.** Check for secrets before committing.
- **Ship order matters.** Server first (API), then web (depends on API), then CLI.
- **One commit message is fine** for related cross-repo changes.
- **Ask before merging** if the user wants to review PRs first.
