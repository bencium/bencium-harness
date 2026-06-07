# bencium-harness

> The professional layer between vibe-coding and BMAD-method. A Claude Code plugin that gives AI agents stable context, an enforced acceptance gate, and a memory that survives `/clear`.

## The problem

AI agents drift. They forget decisions across `/clear`. They scaffold code that nobody checks against the original requirements. They ship without verification, then can't explain why. A blank repo and a clever prompt aren't enough — what's missing is a feedback loop and a memory that the agent works *inside*.

## What this is

`bencium-harness` is a small Claude Code plugin (9 slash commands, 2 auto-skills, 1 hook) that scaffolds a project with a **living roadmap**, an **acceptance checklist that gates deploy**, a **decision log that auto-loads next session**, and a **retro loop** that runs on failure. The agent drifts less because it has structure to work against and memory to come back to.

One command — `/bencium-init` — runs a 5-question interview and produces a complete planning kit in ~60 seconds: PRD, ARCHITECTURE, tasks.md, ACCEPTANCE.md, and a `.harness/` folder with config, memory, rules, and an archive for decisions.

## Nothing vanishes

A workflow without checkpoints forgets itself — and invisible work becomes invisible failure. So every boundary in the loop leaves a written trace: each real decision is recorded with the options weighed and why this one won; every loop-boundary command emits a timestamped, machine-readable result; and the `## Session handoff` block keeps a fixed-shape note of where you are so a `/clear` or crash always has something structured to resume from. The newest pieces are the **build record** (`/bencium-next` now emits its own marker, so the busiest phase of the loop is no longer the only one without an audit trail) and the **structured handoff** (five named fields instead of freeform prose). The decision log, the per-boundary markers, the recovery path — those the harness always did. Together they're one advantage: you can see exactly what happened, and you can always pick up where you left off.

## How it compares

| | bencium-harness | BMAD-method | Just a CLAUDE.md |
|---|---|---|---|
| Setup time | ~60 sec | Multi-phase interview | Instant |
| Memory model | Tiered markdown (`memory.md` + archive) | Heavy generated docs | Static file |
| Verification gate | `/bencium-verify` against `ACCEPTANCE.md` | — | — |
| Deploy/rollback | Built-in, refuses on verify fail | — | — |
| Infra needed | None | None | None |
| Survives `/clear` | Auto-injected | Re-read manually | Yes |

## Who it's for

Solo builders and small teams using Claude Code on real projects — not throwaway demos. Especially valuable for **regulated or healthcare work** where unverified deploys are not OK, and for any project where you want a paper trail of decisions and a refusal-to-deploy when acceptance criteria slip.

## Install

Two slash commands inside Claude Code — no unzip, no `settings.json` editing.

```
/plugin marketplace add bencium/bencium-marketplace
/plugin install bencium-harness@bencium-marketplace
```

Restart Claude Code, then verify:

```
/plugin list           # bencium-harness@bencium-marketplace should be enabled
/bencium-init          # triggers the 5-question interview
```

Developing the plugin itself? Add this repo as a local marketplace instead: `/plugin marketplace add /path/to/benc-harness-source` then `/plugin install bencium-harness@bencium-harness-local`. Full details (updating, local dev, uninstall) live in [`INSTALL.md`](INSTALL.md).

**Uninstall:** `/plugin uninstall bencium-harness@bencium-marketplace` then `/plugin marketplace remove bencium-marketplace`, restart.

### Why not project-scoped?

Project-scope install (`<repo>/.claude/plugins/`) sounds tempting but breaks for the main use case:

- **Chicken-and-egg.** `/bencium-init` needs to exist *before* the project does. A project-scoped plugin can't scaffold an empty repo.
- **N copies, N updates.** Every repo needs its own install, and every harness update has to be re-applied everywhere.
- **Git noise.** Either you commit the plugin files (defeats the point of versioning your own code) or you `.gitignore` them (defeats the "share with team" reason for project scope in the first place).

The only legitimate project-scope reasons: pinning an older harness version to one specific repo, or shipping the harness *with* a client repo so their team auto-gets it on clone. Neither applies to solo dogfooding.

## The 60-second demo

```
$ cd ~/new-project
$ git init
$ # open Claude Code
/bencium-init
```

You get 5 questions (product name, problem, primary user, success metric, stack). One LLM call later you have:

```
new-project/
├── README.md
├── CLAUDE.md              # project-root agent context, auto-loaded
├── PRD.md                 # 1-page product requirements
├── ARCHITECTURE.md        # 1-page system architecture
├── tasks.md               # Now (≤15) / Roadmap split
├── ACCEPTANCE.md          # 10 testable acceptance checks
└── .harness/
    ├── config.yaml        # deploy + rollback + verify commands
    ├── memory.md          # hot context, auto-loaded each session
    ├── rules.md           # project non-negotiables
    ├── glossary.md
    ├── constraints.md
    └── archive/           # NNNN-decision-*.md, NNNN-retro-*.md
```

Brownfield repos are auto-detected — `/bencium-init` scans existing code and produces retrospective PRD/ARCH without overwriting anything.

## How to start a new project with bencium-harness with a vague idea?

Short answer: describe your idea in the chat, then run `/bencium-init` in the empty project dir. That one command runs a 5-question interview and generates the entire scaffold in a single ~60s pass.

**The greenfield-from-a-prompt flow:**

```bash
mkdir my-project && cd my-project && git init
```

Then in the chat:

1. **Type your idea first** — e.g. "A SaaS invoice generator for freelancers, Next.js + Supabase, deploy to Vercel." There's no `--seed` flag, but `/bencium-init` reads the conversation, so your prompt pre-fills the answers.
2. **Run `/bencium-init`.** Empty dir → it auto-detects greenfield, skips repo discovery, and goes straight to 5 questions (it'll pre-suggest answers from what you typed):
   - **Product** — what is this? (one sentence)
   - **User** — who's it for?
   - **Killer feature** — the one thing it must do well
   - **Stack** — e.g. Next.js + Supabase + Vercel
   - **Deploy target** — e.g. `vercel --prod`, plus a health-check URL/command
3. **It writes everything in one pass:** `README.md`, `PRD.md`, `ARCHITECTURE.md`, `tasks.md`, `ACCEPTANCE.md`, `CLAUDE.md` at root, plus a `.harness/` dir (`config.yaml`, `memory.md`, `conventions.md`, `rules.md`, `glossary.md`, `constraints.md`, `archive/`). Add `.harness/state` to `.gitignore`.

## End-to-end walkthrough: empty directory → shipped app

The full loop, in order, for a brand-new toy app:

```bash
mkdir todo-app && cd todo-app
git init
# open Claude Code in this directory
```

**1. Scaffold the harness** — answers 5 questions, writes all planning artifacts.
```
/bencium-init
```
Result: `CLAUDE.md`, `PRD.md`, `ARCHITECTURE.md`, `tasks.md` (~15 tasks split Now/Roadmap), `ACCEPTANCE.md` (~10 checks), `.harness/` with `config.yaml`, `memory.md`, `rules.md`, `glossary.md`, `constraints.md`, `archive/`.

**2. Edit `.harness/config.yaml`** — fill in your deploy, health, and rollback commands (e.g. `fly deploy`, `curl /health`, `fly releases rollback`).

**3. Start the build loop.**
```
/bencium-next
```
This runs in two enforced phases:

- **📋 PLAN (cyan banner)** — picks the top unchecked task, reads `.harness/conventions.md` and `ACCEPTANCE.md`, then writes a 10–25 line plan/spec inline (files to touch, approach, acceptance items targeted, conventions touched, risks, "done when"). **Stops and waits for your explicit approval.** No files are written.
- **🔨 BUILD (green banner)** — entered only after you reply `approve`. Executes the work, marks the task `[x]` in `tasks.md`, and prints an explicit `📝 Memory: …` line so you know what (if anything) landed in `.harness/memory.md`.

The plan IS the spec — there is no separate spec command. The harness will refuse to let the agent jump into coding without showing you the plan first.

*If [Context7](https://github.com/upstash/context7) MCP is installed, the agent will pull live library docs while planning.*

As the agent encounters architectural choices, the `decision-watcher` skill nudges:
```
/bencium-decide "use SQLite for local persistence"
```
This appends an ADR to `.harness/archive/NNNN-decision-*.md` and updates `memory.md` so the choice survives `/clear`.

**4. Commit.** The `post-commit-nag` hook checks if `tasks.md` was updated; if not, it prints a one-line reminder. Never blocks.

**5. New ideas arrive mid-build** — don't derail the Now list.
```
/bencium-feature "export tasks as CSV"
```
Appends to `tasks.md ## Roadmap`. When current slice is done:
```
/bencium-promote
```
Moves Roadmap items into Now (and demotes stale `memory.md` entries to archive).

**6. Repeat steps 3–5** until the current slice of `ACCEPTANCE.md` items should pass.

**7. Verify.**
```
/bencium-verify
```
Walks every unchecked item in `ACCEPTANCE.md` against the actual code, tests, and deployed state. Reports `PASS`/`FAIL`/`SKIP` with evidence and updates checkboxes. If anything fails, `/bencium-retro` auto-fires, writes a postmortem to `.harness/archive/`, and proposes updates to `memory.md` and `ACCEPTANCE.md`.

*If [claude-in-chrome](https://github.com/anthropics/claude-in-chrome) MCP is installed, the verify step can include live UI / screenshot checks for `[deployed]` acceptance rows.*

**8. Deploy.**
```
/bencium-deploy
```
Runs `/bencium-verify` first — **refuses to deploy if it fails**. On pass: runs your `deploy.cmd`, hits the health check, logs the release to `memory.md`. A failed deploy auto-invokes `/bencium-retro`.

**9. Rollback when needed.**
```
/bencium-rollback "health check timeout on /api/tasks"
```
Runs the configured rollback command and logs the reason to the archive.

**10. Next session.** Run `/clear` or close and reopen Claude Code. The `context-loader` skill auto-injects `memory.md` + `rules.md` + the last 3 archive entries at SessionStart, so the agent resumes with full project memory. If you stopped mid-task, the `## Session handoff` block in `memory.md` — a fixed five-field shape (active task · phase · up next · blockers · uncommitted), refreshed at any loop boundary per the next-moves "Token checkpoint" nudge — tells the agent exactly where it left off, with structured fields rather than freeform prose. Back to step 3.

That's the whole loop. Nine commands, two auto-skills, one hook — designed to be the rhythm of how you actually build.

## Commands

| Command | What it does |
|---|---|
| `/bencium-init` | Scaffold the harness in the current repo (greenfield or brownfield, auto-detected). Now also writes `.harness/conventions.md`. |
| `/bencium-next` | **Two-phase:** plan/spec (📋 cyan, read-only, gated on your approval) → build (🔨 green, executes). The plan IS the spec. |
| `/bencium-feature "desc"` | Append a feature to `tasks.md ## Roadmap`. |
| `/bencium-promote` | Move Roadmap items into Now, or demote stale memory entries to archive. |
| `/bencium-decide "title"` | Log an architectural decision to `.harness/archive/`. |
| `/bencium-verify` | Walk `ACCEPTANCE.md` against the actual code. Report pass/fail with evidence. Inspects for gamed tests (weakened/skipped assertions) and can delegate a skeptical second opinion to an independent reviewer subagent. |
| `/bencium-deploy` | Verify → deploy → health check → log. Refuses on verify fail. |
| `/bencium-rollback "reason"` | Run the configured rollback command and log the reason. |
| `/bencium-retro` | Postmortem after a failure. Proposes memory + acceptance updates. |

## Skills and hooks

- **`context-loader`** — at session start, injects `.harness/memory.md` + `conventions.md` + `rules.md` + recent archive entries so the agent picks up where it left off.
- **`decision-watcher`** — nudges you to run `/bencium-decide` when it detects you're picking between technical alternatives.
- **`phase-banner`** — single source of truth for the ANSI color scheme, banners, and emoji that distinguish each phase of the loop (PLAN cyan/📋, BUILD green/🔨, TEST yellow/✓, REFLECT magenta/💭, DEPLOY bright-blue/🚀). Every command reads from this skill so styling stays consistent.
- **`post-commit-nag`** — after a `git commit`, if `tasks.md` wasn't updated, prints a one-line reminder. Never blocks. No-op outside harness projects.
- **`statusline/harness-status.sh`** (optional) — two-row Claude Code status bar showing the phase-progress strip, project name, current task, ACCEPTANCE ratio, model, cost, and context-window %. Plugins can't ship statuslines declaratively, so wire it into `~/.claude/settings.json` per [INSTALL.md](INSTALL.md#statusline-optional). Silent outside harness projects.

## Machine-readable markers

Every loop-boundary command emits a fenced ```` ```json ```` block as its last output, so CI scripts, dashboards, and follow-up commands can consume results without scraping prose. Schema version `1`. The five markers:

| Marker | Producer | Contains |
|---|---|---|
| `bencium-build-result` | `/bencium-next` Step 4.3 | task, complete/incomplete, TDD path (red-green / trivial-bypass / skip-tdd), declared test + status, files touched, HEAD sha, optional learning |
| `bencium-verify-result` | `/bencium-verify` Step 8 | hard checks, local soft checks (pass/fail/skip + items + evidence), deferred-to-smoke entries |
| `bencium-smoke-result` | `/bencium-deploy` Step 5.5 | `deploy_url`, deployed sha, per-`[deployed]`-row result against the live URL |
| `bencium-deploy-result` | `/bencium-deploy` Step 6 | sha, target, deploy exit code, health check, smoke summary |
| `bencium-retro-result` | `/bencium-retro` Step 5 | retro id, archive path, trigger, root cause, the three proposals (memory line, ACCEPTANCE row, hook path) |

Use case — CI gate: `grep -A20 '"_marker": "bencium-verify-result"' agent-output.log | jq -r '.result' | grep -qx pass || exit 1`. Use case — "fix the findings" follow-up: prompt the agent to read the prior turn's marker JSON and address each FAIL row. The harness doesn't ship a tool for either pattern; it ships the contract.

## Recommended MCPs (optional)

The harness does not depend on any MCP server. Two trust-vetted MCPs pair naturally with the loop if you want to extend agent capability beyond the codebase:

- **[Context7](https://github.com/upstash/context7)** (Upstash) — live library documentation fetched at plan time. Useful inside `/bencium-next` Phase A when the agent is choosing an API surface or arguing about a version. Install via the upstream repo's instructions.
- **[claude-in-chrome](https://github.com/anthropics/claude-in-chrome)** (Anthropic) — browser automation for `/bencium-verify` rows that require live UI inspection (screenshots, console errors, network requests). Useful for any `ACCEPTANCE.md` row tagged `[deployed]`. Install via the upstream repo's instructions.

Both are optional. **Do not install MCPs from anonymous or single-maintainer sources** — MCPs run with your local privileges and the harness's trust posture requires multi-maintainer governance plus verifiable identity.

## Memory isolation across projects

Plugin code lives globally (`~/.claude/plugins/bencium-harness/`), but **all data lives per-repo** inside that project's `.harness/` folder. The `context-loader` skill only reads from the current working directory's `.harness/memory.md`, so project A's decisions never leak into project B. Opening a different project gives the agent a clean slate scoped to that repo.

**Edge cases to know:**
- **Worktrees / Conductor** on the same repo share one `.harness/memory.md` — parallel `/bencium-decide` writes from two windows can race.
- **Monorepos** with a single root `.harness/` share memory across all packages. If you want per-package isolation, run one `.harness/` per package directory (not currently a documented pattern).
- **Forks** inherit committed memory. Useful for handoffs, surprising if you didn't expect it.

## Use with claude.ai or Claude for Work

The plugin runs only in Claude Code (CLI). On claude.ai or Claude for Work you can still get partial value:

1. Unzip and grab the `templates/` folder.
2. Manually copy the templates into your repo (rename `.tmpl` to real names, fill placeholders).
3. Attach the repo to a Project.

You lose slash commands, auto-injection, and `/bencium-verify` enforcement. You keep the structural value of having `tasks.md`, `ACCEPTANCE.md`, and `.harness/memory.md` in the repo for the agent to read.

## The six-phase loop

```
🗺️ roadmap    →    📋 plan/spec    →    🔨 build    →    ✓ test    →    💭 reflect    →    🚀 deploy
tasks.md        /bencium-next A      /bencium-next B   /bencium-verify   /bencium-retro   /bencium-deploy
                ↑ MANDATORY GATE ↑
                (refuses to code without approval)
```

Every phase has a distinct ANSI-colored banner in the CLI so you always know whether the agent is thinking (PLAN/SPEC, cyan) or executing (BUILD, green; DEPLOY, bright blue). The phase-progress strip at the top of every report shows where you are in the loop.

## Versioning

Each release has both a SemVer number (used by Claude Code's `/plugin install` / `/plugin update`) and a human codename in the format `<Month> <Theme>` (used in CHANGELOG and conversation). The codename is what you remember; the SemVer is what the plugin manager uses. Latest: **Jun Lantern (v0.5.0)**. Theme vocabulary and full history live in [CHANGELOG.md](CHANGELOG.md).

Git tag format: `v<SemVer>-<kebab-codename>` (e.g. `v0.2.0-may-gate`). The leading `v<SemVer>` keeps tags sorted correctly; the trailing codename makes the tag list scannable.

## Status

**v0.5.0 — Jun Lantern.** Makes harness reports legible to downstream automation. `/bencium-verify`, `/bencium-deploy` SMOKE, `/bencium-deploy` success, and `/bencium-retro` each emit a machine-readable JSON marker block as their final output (schema `version: 1`) — CI scripts, deploy gates, dashboards, and "fix the findings" follow-ups can parse the prior turn's marker without scraping prose. `rules.md.tmpl` re-scaffolded into DO / DO NOT / Ordering-invariants format. New "Ordering invariants" section in `conventions.md.tmpl` names the harness's four encoded invariants (PLAN→BUILD, RED→GREEN, VERIFY→DEPLOY, DEPLOY→SMOKE) and shows users how to declare their own.

**v0.4.0 — Jun Compass.** Closed the three Level-1 harness gaps: `/bencium-retro` proposes **hooks** for destructive-op / repeated-mistake / env-quirk failure classes (writes `.harness/hooks/NNNN-*.sh` + settings.json snippet). Conventions added **LSP** as a pre-edit check. `/bencium-init` brownfield uses **parallel Explore subagents** (3-agent cap: structure / data / tests+CI) for large repos.

**v0.3.0 — Jun Gate.** Closed the local-pass-doesn't-mean-shipped gap with a TDD red→green gate in `/bencium-next` BUILD and a `🔥 SMOKE` phase in `/bencium-deploy`. Added the two-row Claude Code statusline and AskUserQuestion portability notes.

Future versions may add an MCP server companion for non-CLI surfaces, sprint ceremony commands, stack presets, and integration with other planning skills.
