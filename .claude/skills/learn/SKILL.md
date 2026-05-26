---
name: learn
description: Update skills and memory after encountering and fixing an issue. Run automatically after any fix before continuing with the next task.
---

## When to run

Run this skill immediately after fixing any issue that involved:
- A workaround or non-obvious fix (cache clearing, port conflicts, env vars, etc.)
- A deployment problem (build steps, service restarts, missing files, etc.)
- A dev environment gotcha (stale processes, wrong server, hot reload failures, etc.)
- A cross-repo coordination issue (wrong order, missing dependency, etc.)
- A security or auth issue (token handling, cookie behaviour, etc.)
- Any fix that took more than one attempt to get right

Do NOT continue to the next task until this skill completes.

## Step 1: Identify the lesson

Summarise:
1. What went wrong
2. What the fix was
3. Which skill(s) should have prevented it

## Step 2: Check existing skills

Read all skills in this repo:

```bash
for skill in /Users/Elvis/workspace/0cmplxHq/0cmplx-hub/.claude/skills/*/SKILL.md; do
  echo "=== $(basename $(dirname $skill)) ==="
  head -3 "$skill"
  echo ""
done
```

Determine if the lesson belongs in:
- An existing skill (add a step, gotcha, or safety rule)
- A new skill (if no existing skill covers this area)
- Memory only (if it is project knowledge, not a repeatable workflow)

## Step 3: Update the skill

If updating an existing skill, add the lesson in the most relevant section:
- **Gotchas** section: for dev environment surprises
- **Safety rules** section: for things that must never happen
- **Step N** subsection: for missing procedural steps

If creating a new skill, follow the standard format:
```markdown
---
name: skill-name
description: One-line description
---

## Step 1: ...
```

## Step 4: Update memory if needed

If the lesson is project knowledge (not a workflow), update the memory file:

```
/Users/Elvis/.claude/projects/-Users-Elvis-workspace-supaproxyhq-supaproxy/memory/0cmplx.md
```

Add to the relevant section (deployment, dev environment, gotchas, etc.).

## Step 5: Update CLAUDE.md if needed

If a new skill was created, add it to the skills table in CLAUDE.md:

```
/Users/Elvis/workspace/0cmplxHq/0cmplx-hub/CLAUDE.md
```

## Step 6: Confirm

Report:
```
Lesson captured:
  Issue: <what went wrong>
  Fix: <what fixed it>
  Updated: <skill name> (added <section>)

Continuing with next task.
```

## Examples of lessons from recent sessions

These are real issues that should have been captured in skills:

- **Vite cache serves stale code**: `/dev` skill should always clear `node_modules/.vite`
- **tsx watch does not pick up new files**: `/dev` gotcha, restart server after adding files
- **Port 4322 occupied by old ropuppy app**: `/dev` kills stale processes before starting
- **tsc does not copy YAML files to dist**: `/deploy` and `/ship` should verify non-TS assets
- **Browser DNS cache prevents access to new domains**: deployment gotcha, flush with `dscacheutil`
- **Logout fetch aborted by page navigation**: `keepalive: true` on logout requests
- **CLI token visible in terminal history**: raw mode stdin for hidden input
- **CLI logout only clears local credentials**: must also notify server for activity logging
