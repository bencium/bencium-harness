---
description: Pick the next task from tasks.md ## Now, write a plan/spec, gate on user approval, then build
allowed-tools: Read, Edit, Write, Bash
---

You are running `/bencium-next` — the build-loop driver.

This command runs in **two enforced phases**: **PLAN** (write & display the spec, stop and wait) and **BUILD** (execute only after explicit approval). The plan IS the spec — there is no separate spec command. The whole point is to refuse to let you jump into coding without a reviewable plan.

Phase styling is defined in the `phase-banner` skill. Use it.

## Step 1: Load context

1. Read `tasks.md` from project root. If missing, tell the user to run `/bencium-init` first.
2. Read `.harness/memory.md`, `.harness/conventions.md` (if present), and `ACCEPTANCE.md`.

## Step 2: Pick the next task

From the `## Now` section, find the first unchecked task (`- [ ]`). If `## Now` is empty or all checked:

- Tell the user "Now section is clear."
- Suggest: `/bencium-verify` (if anything was just completed), `/bencium-promote` (to move Roadmap → Now), or `/bencium-feature` (to add new work).
- STOP.

## Step 2.5: PROTOTYPE detour (optional — UI tasks only)

This is the **one** place `/bencium-next` may write files before the PLAN gate — and it writes **only throwaway prototype code** to a disposable location, never real source. PLAN (Step 3) stays 100% read-only. The detour exists so you can eyeball look-and-feel *before* committing to a spec, then fold the chosen direction back into the plan. Phase styling: PROTOTYPE (orange / 🎨) in the `phase-banner` skill.

**Trigger detection.** Decide whether the picked task touches UI — same heuristic as the PLAN "Design skills to invoke" section: titles mentioning a page, screen, component, view, route, form, layout, dashboard, landing, modal, styling, theme, or any visible surface. If the task is pure backend / data / infra / refactor, **skip this step entirely** and go to Step 3 — no offer, no output.

**Offer (UI tasks only).** Print:

```
🎨 This task touches UI. Want to see pixels before planning?
   Reply 'prototype' to scaffold a throwaway preview (one disposable route, no tests).
   Reply 'plan' (or anything else) to go straight to the read-only spec.
```

**STOP and wait for the reply.**

**If the user declines** (`plan` / silence / anything that isn't `prototype`): go to Step 3. No files written.

**If the user replies `prototype`:** record the PROTOTYPE phase for the statusline, print the banner, then run the detour.

```bash
[ -d .harness ] && printf 'phase=prototype\ntask=%s\nupdated=%s\n' "<picked task title>" "$(date -u +%FT%TZ)" > .harness/state
```

```
\033[38;5;208m┌── PROTOTYPE ── throwaway pixels, no commitment ──┐\033[0m
[🎨] ▸ 🗺️ ▸ 📋 ▸ 🔨 ▸ ✓ ▸ 💭
```

1. Invoke the `prototype` skill to scaffold a **throwaway** UI preview for this task. Its "several radically different UI variations toggleable from one route" branch is the right fit for eyeballing look-and-feel.
2. Write the prototype to a **clearly disposable location** so it can never masquerade as the implementation:
   - a `_prototype` route folder (e.g. `app/_prototype/` or `src/routes/_prototype/`), or
   - `.harness/prototype/` for standalone static HTML.
   Recommend adding the chosen path to `.gitignore`. This code is **not** counted toward `ACCEPTANCE.md`, gets **no** test, and is expected to be deleted once a direction is chosen.
3. Let the user eyeball the variations and pick a direction (or ask for tweaks). When they settle, capture the chosen direction in one or two sentences — the layout, type, color, and the one key move.
4. Close the banner:
   ```
   \033[38;5;208m└──────────────────────────────────────────────────┘\033[0m
   ```

Then proceed to Step 3 (PLAN) with the chosen direction in hand. The detour writes no `.harness/state` `phase=plan` line itself — Step 3 does that. **Do not start coding the real feature here.** The throwaway is for looking, not shipping.

## Step 3: PHASE A — PLAN (read-only, gated)

Record the PLAN phase for the statusline before printing the banner. Use the picked task title from Step 2 as the `task=` value (truncate to ~60 chars):

```bash
[ -d .harness ] && printf 'phase=plan\ntask=%s\nupdated=%s\n' "<picked task title>" "$(date -u +%FT%TZ)" > .harness/state
```

Print the PLAN banner (cyan):

```
\033[36m┌── PLAN ── review before coding ──┐\033[0m
🗺️ ▸ [📋] ▸ 🔨 ▸ ✓ ▸ 💭
```

Then write a 10–25 line plan/spec for the picked task. Structure:

```
📋 Task: <verbatim title from tasks.md>

Problem (1 sentence):
  <what user-visible thing this fixes/adds and why now>

Files to touch (predicted):
  - <path/file.tsx>
  - <path/other.ts>
  (mark new files with NEW)

Approach (3-5 bullets):
  - <step 1>
  - <step 2>
  - ...

ACCEPTANCE items this targets:
  - <verbatim ACCEPTANCE.md line>  (or "none directly — quality/refactor work")

Conventions touched (from .harness/conventions.md):
  - <which rule applies, e.g. "file size cap — current file is 412 LOC, this adds ~80, OK">
  - <or "none — trivial change">

Design skills to invoke (only if UI touched):
  - Before code:  bencium-innovative-ux-designer OR impeccable
  - For copy:     typography
  - After code:   design-review
  (skip this section if no UI involved)

Prototype direction (only if Step 2.5 ran):
  - <chosen look-and-feel in one line> — throwaway preview at <path>, delete after BUILD
  (omit this section entirely if no prototype detour ran)

Risks / open questions:
  - <one risk, or "None">

Done when:
  - <concrete observable, e.g. "form submits and the row appears in /admin within 1s">

Failing test (red):
  - File: <path/to/test.spec.ts>  (NEW or existing)
  - Assertion: <one-line description of the failing assertion that captures "Done when">
  - Run command: <e.g. `npm test -- --run path/to/test.spec.ts`>
  (For trivial tasks — typo, single-line rename, copy-only — the entire value may be:
   `none — trivial (declared)`. Anything else MUST name a real test.)

Deployed-env check (smoke):
  - <ACCEPTANCE.md [deployed] row this task adds or satisfies, or "none — not user-facing / no deployed surface area">
  (If the task touches any code path that runs on the deployed env — network, env vars,
   build pipeline, runtime — a `[deployed]` row in ACCEPTANCE.md is required. Local
   pass does not ship the feature.)
```

Close the banner:

```
\033[36m└──────────────────────────────────┘\033[0m
```

Then print the gate prompt verbatim:

```
📋 Review the plan above.
   Reply 'approve' to enter BUILD phase.
   Or tell me what to change and I will revise the plan.
   For a trivial task (typo, one-line rename) you may reply 'skip plan' to fast-track to BUILD.
```

**STOP HERE.** Do not call any tool that writes or edits files. Do not run any build commands. Do not enter Phase B.

## Step 4: PHASE B — BUILD (only after explicit approval)

Enter Phase B ONLY if the user's next reply is one of:
- `approve` / `ok` / `yes` / `go` / `lgtm` / 👍
- `skip plan` (acknowledged fast-track; print a one-line warning that the gate was bypassed)

If the user instead requests changes, revise the plan and re-print Step 3, then STOP again. Do not enter Phase B until approval is explicit.

Record the BUILD phase for the statusline before printing the banner:

```bash
[ -d .harness ] && printf 'phase=build\ntask=%s\nupdated=%s\n' "<picked task title>" "$(date -u +%FT%TZ)" > .harness/state
```

Once approved, print the BUILD banner (green):

```
\033[32m┌── BUILD ── coding in progress ──┐\033[0m
🗺️ ▸ 📋 ▸ [🔨] ▸ ✓ ▸ 🚀 ▸ 🔥 ▸ 💭
```

### Step 4.0 — RED (write the failing test first)

Before editing **any** implementation file:

1. Read the `Failing test (red):` field from the approved plan.
2. If the value is `none — trivial (declared)`:
   - Print one yellow warning line: `\033[33m⚠ TDD: trivial-task bypass (no failing test required by plan)\033[0m`
   - Skip to Step 4.1.
3. If the user replied `skip tdd` to the plan gate:
   - The plan must already declare why (added during revision). If it doesn't, refuse and ask for the justification.
   - Append to `.harness/memory.md` under `## TDD bypasses` (create the section if missing): `- YYYY-MM-DD HH:MM — TDD bypass on "<task title>" — reason: <justification from plan>`
   - Print: `\033[33m⚠ TDD bypass logged to memory.md. Implementation will proceed without a red→green test.\033[0m`
   - Skip to Step 4.1.
4. Otherwise (the normal path): write the test file declared in the plan. Run the run-command. Observe the output:
   - The test MUST fail (red).
   - The failure must be for the **expected reason** (assertion failed because the feature isn't implemented yet) — not an import error, syntax error, or "no test found". If the failure is the wrong kind, fix the test until it fails *for the right reason*, then continue.
   - Print one line summarizing the red state: `🔴 RED: <test name> failed as expected — <one-line failure summary>`
5. Only after a real red is observed may Step 4.1 edit implementation files.

### Step 4.1 — GREEN (implement)

Execute the work per the approved plan. Touch only the files declared in the plan unless you discover the plan is wrong — in which case STOP, print a one-line note ("plan needs update: X"), and ask whether to revise the plan or continue.

### Step 4.2 — confirm green

After implementation, re-run the failing test from Step 4.0. It must now pass.

- Print: `🟢 GREEN: <test name> passed`
- If it still fails, do NOT mark the task done. Either fix the implementation, or — if the test itself was wrong — revise the plan and re-enter Step 4.0.

When the task is complete:

1. Mark the task `- [x]` in `tasks.md`.
2. If the work produced a durable learning (something a future session would benefit from knowing), append a one-liner to `.harness/memory.md` under `## Learnings`:
   ```
   - YYYY-MM-DD — <one-line learning>
   ```
   Otherwise do NOT pollute memory.
3. Print the memory confirmation line (always — whether you added or not):
   ```
   📝 Memory: +1 learning appended to .harness/memory.md
   ```
   or
   ```
   📝 Memory: unchanged (no durable insight from this task)
   ```
4. Close the BUILD banner:
   ```
   \033[32m└──────────────────────────────────┘\033[0m
   ```
5. Render `${CLAUDE_PLUGIN_ROOT}/templates/next-moves.md.tmpl` to close out the BUILD phase, filling:
   - `{{phase_progress_strip}}` → `🗺️ ▸ 📋 ▸ 🔨 ▸ [✓] ▸ 🚀 ▸ 🔥 ▸ 💭` (verify highlighted next; deploy + smoke still ahead)
   - `{{verify_reason}}` → `BUILD just finished — verify walks ACCEPTANCE.md against the new state, then /bencium-deploy runs SMOKE against the deployed URL. Local green != shipped.`
   - `{{top_roadmap_items}}` → first 3 items from `tasks.md ## Roadmap`
   - `{{tail_status}}` → `🔁 Close the loop: /bencium-verify → /bencium-deploy (SMOKE) → /bencium-retro. Local + deployed must both go green before the task is shipped. Then 💡 /clear before the next slice.`

   Put `/bencium-verify` at the top of the suggestions. The template's trailing 💡 Token checkpoint block is the boundary nudge — leave it in.

### Step 4.3 — Marker block (final output — for downstream tools)

After the `next-moves` render, emit a fenced ```` ```json ```` block as the **very last** thing Phase B prints. Nothing after it — no prose, no banner, no trailing whitespace beyond the closing fence. This is the loop step's audit record: PLAN and BUILD are the heart of the loop, and until now they left no structured trace (verify/deploy/smoke/retro all do). One marker captures the executed plan *and* the build outcome, so it doubles as the per-loop-step checkpoint.

Schema:

```json
{
  "_marker": "bencium-build-result",
  "version": 1,
  "ts": "<ISO8601 UTC>",
  "phase": "build",
  "task": "<picked task title>",
  "result": "complete" | "incomplete",
  "tdd": "red-green" | "trivial-bypass" | "skip-tdd",
  "test": { "file": "<path>", "name": "<test name>", "status": "green" | "n/a" },
  "files_touched": ["<path>", "..."],
  "sha": "<short git sha of HEAD>",
  "learning": "<one-line>" | null
}
```

Rules for populating:

- `result` is `complete` only when Step 4.2 observed green (or a declared `trivial-bypass` / `skip-tdd` path that finished and the task was marked `[x]`). If BUILD stopped mid-task (plan-wrong pause in Step 4.1, an unresolved red), emit `incomplete` with whatever `files_touched` you have so far — an incomplete record still beats no record for recovery.
- `tdd` reflects which Step 4.0 path ran: `red-green` (normal), `trivial-bypass` (`none — trivial (declared)`), or `skip-tdd` (explicit bypass logged to memory).
- `test.status` is `green` on the normal path after Step 4.2 confirms it; `n/a` for `trivial-bypass` / `skip-tdd`.
- `sha` from `git rev-parse --short HEAD` (read-only — BUILD does not commit, so this is the pre-existing HEAD, used as a pointer to where the work sits).
- `learning` mirrors the one-liner written under `## Learnings`, or `null` if memory was unchanged.

## Hard rules

- **Phase A MUST NOT call Edit, Write, or any Bash command that mutates state.** Read-only tools only (Read, Glob, Grep, read-only Bash like `cat`/`ls`/`git log`). The optional PROTOTYPE detour (Step 2.5) is the **only** place `/bencium-next` may write before approval, and it writes **only throwaway prototype files** to a disposable location (`_prototype/` route or `.harness/prototype/`) — never real source, never a test, never anything counted toward ACCEPTANCE. The detour runs only for UI tasks and only when the user types `prototype`; a non-UI task or any other reply goes straight to the read-only PLAN.
- **Phase A plan MUST include `Failing test (red):` and `Deployed-env check (smoke):` fields.** Refuse to proceed if either is missing. The fields force the user to decide *what proves this works* before any code is written — both locally and on the deployed env.
- **Phase B is entered only after explicit user approval.** Silence, "looks good", or anything ambiguous does NOT count — re-prompt.
- **Phase B Step 4.0 (RED) MUST run before any implementation edit.** No writing implementation files until the declared failing test is observed in a red state for the expected reason. The only exceptions are `none — trivial (declared)` and an explicit `skip tdd` with a justification logged to `.harness/memory.md ## TDD bypasses`.
- **Do not check off a task until both the local test passes AND (if the task has a `[deployed]` ACCEPTANCE row) `/bencium-deploy` SMOKE has passed that row against the live URL.** Marking `[x]` in `tasks.md` from local-only state is the false-confirmation failure mode the loop was hardened against.
- Do not modify the order of `## Now` items unless the user asks.
- If the task is too big to finish in one go, split it during Phase A: propose 2-3 sub-tasks in the plan, get approval to edit `tasks.md`, then proceed.
- If `.harness/conventions.md` is missing, note "conventions.md not found — running without quality guardrails" in the plan instead of skipping silently.
- **The Step 4.3 marker block MUST be the last output of Phase B** — after the `next-moves` render, nothing after the closing fence (no prose, no banner, no whitespace). Downstream tools locate it as the trailing fenced JSON. Markers report state, not opinion — no recommendations inside the JSON; those stay in the prose above. The schema is versioned (`version: 1`); future changes bump the integer. Phase A (PLAN) emits no marker — it is the read-only gate, and the BUILD marker already records the plan that was executed.
