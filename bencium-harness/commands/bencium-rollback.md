---
description: Run the configured rollback command and log the reason
allowed-tools: Read, Bash, Edit, AskUserQuestion
argument-hint: "<reason for rollback, optional>"
---

You are running `/bencium-rollback` with argument: `$ARGUMENTS`

## Step 0: Record phase (statusline)

```bash
[ -d .harness ] && printf 'phase=deploy\ntask=rollback\nupdated=%s\n' "$(date -u +%FT%TZ)" > .harness/state
```

## Step 1: Load config

Read `.harness/config.yaml`. If `rollback.cmd` is missing, tell the user to add it and stop.

## Step 2: Capture the reason

If `$ARGUMENTS` is empty, ask: "Why are you rolling back? (one line — required)".

The reason must be specific enough to act on later (e.g., "auth flow broken on prod, users can't sign in" — not just "broken").

## Step 3: Confirm

Show:

```
Rollback target: <rollback.cmd>
Reason:          <reason>
```

**Portability note.** If you are running inside Claude Code, use the `AskUserQuestion` tool for the confirmation prompt below. If `AskUserQuestion` is unavailable (claude.ai, Cursor, Codex, or any non-Claude-Code host), ask inline (`Proceed with rollback? (yes/no)`) and wait for the reply. Either way, do not run the rollback command without explicit confirmation.

Ask: "Proceed?" (yes/no). On no, STOP.

## Step 4: Execute

Run `rollback.cmd` via Bash. Stream output. Capture exit code.

If non-zero:

- Print failure output
- Tell the user the rollback itself failed — manual intervention needed
- Do NOT log a "successful rollback" to memory
- STOP

## Step 5: Health check (if configured)

If `deploy.health` exists, run it (with retries as in `/bencium-deploy`). The rolled-back version should pass the same health check.

If health still fails post-rollback, escalate: tell the user the rolled-back version is also unhealthy — this is a deeper issue.

## Step 6: Log

Append to `.harness/memory.md` under `## Deploys`:

```
- YYYY-MM-DD HH:MM — ROLLED BACK from <prev-sha> — <reason>
```

## Step 7: Suggest retro

Print with the DEPLOY banner (bright blue) — see `phase-banner` skill:

```
\033[94m┌── DEPLOY ── rollback complete ──┐\033[0m
🚀 Rolled back from <prev-sha>. Health: ✓
📝 Memory: +1 rollback line appended to .harness/memory.md under ## Deploys
\033[94m└───────────────────────────────────┘\033[0m

Next: /bencium-retro — capture the root cause and propose an ACCEPTANCE.md check so this doesn't recur.
```

## Hard rules

- Never roll back without a captured reason.
- Never claim success if the rollback command exit code was non-zero.
- The retro suggestion at the end is not optional — every rollback is a learning opportunity.
