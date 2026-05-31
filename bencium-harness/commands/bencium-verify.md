---
description: Walk ACCEPTANCE.md against actual repo state, report PASS/FAIL/SKIP with evidence
allowed-tools: Read, Edit, Bash, Glob, Grep
---

You are running `/bencium-verify` — the verification gate.

## Step 0: Record phase (statusline)

If `.harness/` exists, write the current phase so the optional statusline can highlight TEST:

```bash
[ -d .harness ] && printf 'phase=test\ntask=verify acceptance\nupdated=%s\n' "$(date -u +%FT%TZ)" > .harness/state
```

## Step 1: Load

1. Read `ACCEPTANCE.md` from project root. If missing, tell the user to run `/bencium-init` first.
2. Read `.harness/config.yaml` to find `verify.cmd` entries (optional — if absent, skip the hard-check phase).
3. Read `.harness/rules.md` for project-specific non-negotiables that should also be checked.

## Step 2: Hard checks (if config has them)

Run each command in `verify.cmd` sequentially via Bash. Capture exit code + last 20 lines of output. If any non-zero, record as FAIL with evidence.

Examples of typical commands users may have configured: `npm test -- --run`, `npm run typecheck`, `npm run build`, custom smoke scripts.

## Step 3: Soft checks (walk ACCEPTANCE.md — local only)

For EACH unchecked item in `ACCEPTANCE.md`:

1. **Tag check first.** If the line starts with `- [deployed]` (or has a trailing `[deployed]` tag), it is a deployed-environment check. Classify as **SKIP** with reason `awaiting post-deploy smoke — re-walked by /bencium-deploy Step 5.5 against deploy.url`. Do NOT attempt to evaluate it locally, and do NOT mark it `[x]`. Move on. (Rationale: local pass != deployed pass. The whole reason this tag exists is because tests passing locally have falsely confirmed deployed features in the past.)
2. Otherwise the line is `- [ ]` (local). Read the check carefully.
3. Determine the minimum evidence needed to call it pass/fail. Examples:
   - "All list endpoints paginated" → grep for route handlers, inspect for limit/offset/cursor
   - "Health check returns 200" → run curl against the configured URL
   - "No secrets in repo" → grep for common patterns (`SUPABASE_KEY=`, `sk_live_`, etc.) in tracked files only
   - "Bilingual EN/HU preserved" → grep for stripped diacritics or missing translation keys
   - "UI discloses backend activity" → grep for `fetch(`, `useQuery`, `useMutation`, upload handlers and confirm each has visible loading/error state (e.g. matching `isLoading`, `<Spinner`, `<Skeleton`, `toast.error`, etc.); flag any async call with no visible state binding.
4. Gather the evidence. Run commands, read files, search the codebase.
5. Classify as PASS, FAIL, or SKIP (with reason). SKIP for a local check is only acceptable when it genuinely doesn't apply yet (e.g., "deploy works" before first deploy).
6. If PASS, mark `- [x]` in ACCEPTANCE.md.
7. If FAIL or SKIP, leave the line unchanged and record the reason.

## Step 4: Report

Print the TEST banner (yellow) — see `phase-banner` skill for the codes:

```
\033[33m┌── TEST ── verify acceptance (local) ──┐\033[0m
🗺️ ▸ 📋 ▸ 🔨 ▸ [✓] ▸ 🚀 ▸ 🔥 ▸ 💭
```

Then a structured report. Show **deployed-env checks as a separate block** so the user can see at a glance that local PASS is not a deploy gate on its own.

```
/bencium-verify report (local)

Hard checks:
  ✓ tests        (npm test -- --run, 0)
  ✗ typecheck    (npm run typecheck, 2 errors in src/api.ts:42, src/db.ts:88)

Soft checks — local (ACCEPTANCE.md):
  ✓ All list endpoints paginated
    └─ Inspected 4 routes in app/api/, all use limit/offset
  ✗ Health check returns 200
    └─ curl https://app.fly.dev/health → 503
  - No secrets in repo (skipped: no scanner configured)
  ✓ UI discloses backend activity
    └─ All 7 fetch sites bound to visible loading/error state
  ✓ ...

Deployed-env checks (deferred — run /bencium-deploy SMOKE to evaluate):
  ⏭ [deployed] Upload works at https://app.example.com
    └─ awaiting post-deploy smoke
  ⏭ [deployed] No console errors on deployed page load
    └─ awaiting post-deploy smoke

Result: 7 PASS, 2 FAIL, 1 SKIP, 2 DEFERRED-TO-SMOKE
```

Note: DEFERRED-TO-SMOKE is not a failure — it just means the row cannot be evaluated locally. The deploy command will walk these. The local pass count never includes deployed rows.

Close: `\033[33m└──────────────────────────────────┘\033[0m`

## Step 5: On failure

If any FAIL exists:

- Do NOT proceed with deploy or any destructive action.
- Render `${CLAUDE_PLUGIN_ROOT}/templates/next-moves.md.tmpl`, filling:
  - `{{phase_progress_strip}}` → `🗺️ ▸ 📋 ▸ 🔨 ▸ ✓ ▸ 🚀 ▸ 🔥 ▸ [💭]` (reflect highlighted next)
  - `{{verify_reason}}` → `N checks still failing — fix and re-run` (or omit the verify block entirely)
  - `{{top_roadmap_items}}` → read from `tasks.md ## Roadmap` (first 3 items)
  - `{{tail_status}}` → `📝 Memory: unchanged (verify failed — no memory line until clean)`
- Put `/bencium-retro` at the top of the rendered output.
- Offer to fix the first failing check now (yes/no/list-all-first).

## Step 6: On full pass

Append to `.harness/memory.md` under `## Verifications`:

```
- YYYY-MM-DD HH:MM — /bencium-verify clean (N checks)
```

Render `${CLAUDE_PLUGIN_ROOT}/templates/next-moves.md.tmpl`, filling:
- `{{phase_progress_strip}}` → `🗺️ ▸ 📋 ▸ 🔨 ▸ ✓ ▸ [🚀] ▸ 🔥 ▸ 💭` (deploy highlighted next; smoke still pending)
- `{{verify_reason}}` → `local clean — `/bencium-deploy` is gated open. Deployed-env rows will be evaluated by SMOKE after deploy; the loop is not complete until they pass.`
- `{{top_roadmap_items}}` → read from `tasks.md ## Roadmap` (first 3 items)
- `{{tail_status}}` → `📝 Memory: +1 verification line appended to .harness/memory.md`

Put `/bencium-deploy` at the top of the rendered output and tell the user the gate is open.

## Step 7: Memory confirmation

Whether the result was PASS or FAIL, print one explicit memory line so the user knows what landed in `.harness/memory.md`:

```
📝 Memory: +1 verification line appended      (on full PASS)
```
or
```
📝 Memory: unchanged                          (on any FAIL)
```

## Step 8: Marker block (final output — for downstream tools)

Emit a fenced ```` ```json ```` block as the **very last** thing the command prints. Nothing after it — no closing banner, no extra prose, no trailing whitespace beyond the closing fence. This is how CI scripts, dashboards, and follow-up commands locate the result.

Schema:

```json
{
  "_marker": "bencium-verify-result",
  "version": 1,
  "ts": "<ISO8601 UTC>",
  "phase": "test",
  "result": "pass" | "fail" | "partial",
  "hard_checks": [
    { "name": "<verify.cmd entry>", "status": "pass" | "fail", "evidence": "<one-line>", "exit_code": <int> }
  ],
  "soft_checks_local": {
    "pass": <int>,
    "fail": <int>,
    "skip": <int>,
    "items": [
      { "text": "<ACCEPTANCE row text>", "status": "pass" | "fail" | "skip", "evidence": "<one-line>" }
    ]
  },
  "deferred_to_smoke": [
    { "text": "<row text>", "tag": "deployed" }
  ]
}
```

Rules for populating:

- `result` is `pass` only if every hard check passes AND every local soft check is `pass`. Any `fail` → `fail`. All `pass` with any `skip` → `partial`. `deferred_to_smoke` entries do NOT count toward the result (they are evaluated by `/bencium-deploy` SMOKE).
- `evidence` strings come from the same prose report in Step 4 — one line each, no markdown.
- `hard_checks` is an empty array if `.harness/config.yaml` has no `verify.cmd` entries.
- `deferred_to_smoke` is an empty array if no `[deployed]`-tagged rows exist.

## Hard rules

- Never mark a check `[x]` without actual evidence (file path, grep result, command output).
- Never invent results. If you can't verify a check, mark it SKIP with a reason — don't guess.
- **Never evaluate `[deployed]`-tagged rows locally.** They are SKIP with reason `awaiting post-deploy smoke` until `/bencium-deploy` walks them against `deploy.url`. Marking one `[x]` from local state is a false confirmation — the exact failure mode this gate was built to prevent.
- Do not modify source code during verify. Only update ACCEPTANCE.md checkboxes and .harness/memory.md.
- Never skip the TEST banner or the memory confirmation line — they're how the user knows where they are in the loop.
- **The Step 8 marker block MUST be the last output.** Nothing after the closing fence — no prose, no banner close, no whitespace. Downstream tools locate it as the trailing fenced JSON.
- **Markers report state, not opinion.** No "recommendations" inside the JSON — those stay in prose above. The marker schema is versioned (`version: 1`); future schema changes bump the integer.
