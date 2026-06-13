# bencium-harness

> A plugin for agentic coding tools like codex and opencode. It gives AI agents steady context, an acceptance checklist that must pass before anything ships, and a memory that survives a context reset (when the chat history is wiped between sessions).

## The problem

AI agents drift. They forget decisions when the chat is reset. They write code that nobody checks against the original requirements. They ship without verifying, then can't explain why. A blank repo and a clever prompt aren't enough — what's missing is a feedback loop and a memory the agent works *inside*.

## What this is

One command — `/bencium-init` — runs a 5-question interview and produces a complete planning kit in about 60 seconds: product requirements, architecture, a task list, an acceptance checklist, and a `.harness/` folder with config, memory, rules, and an archive for decisions.

## Nothing vanishes

A workflow without checkpoints forgets itself — and invisible work becomes invisible failure. So every boundary in the loop leaves a written trace: each real decision is recorded. Together they're one advantage: you can see exactly what happened, and you can always pick up where you left off.

## How it compares

| | bencium-harness | BMAD-method | Just a context file |
|---|---|---|---|
| Setup time | ~60 sec | Multi-phase interview | Instant |
| Memory model | Tiered markdown (`memory.md` + archive) | Heavy generated docs | Static file |
| Verification gate | `/bencium-verify` against `ACCEPTANCE.md` | — | — |
| Deploy/rollback | Built-in, refuses on verify fail | — | — |
| Infrastructure needed | None | None | None |
| Survives a context reset | Auto-injected | Re-read manually | Yes |

## Who it's for

Solo builders and small teams using agentic coding tools like codex or opencode on real projects — not throwaway demos. Especially valuable for **regulated or healthcare work** where unverified deploys are not OK, and for any project where you want a paper trail of decisions and a refusal to deploy when acceptance criteria slip.

## The 60-second demo

```
$ cd ~/new-project
$ git init
$ # open your agentic coding tool
/bencium-init
```

You get 5 questions (product name, problem, primary user, success metric, stack). One model call later you have:

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

Existing projects (ones that already have code) are auto-detected — `/bencium-init` scans the code and produces a retrospective product spec and architecture without overwriting anything.

## How to start a new project with bencium-harness from a vague idea

Short answer: describe your idea in the chat, then run `/bencium-init` in the empty project folder. That one command runs a 5-question interview and generates the entire scaffold in a single ~60s pass.

> **Secret tip:** One faster option if you don't want to wait for the full cycle: ask for a throwaway design prototype (a quick one-route scaffold) to eyeball the look-and-feel, then fold the chosen direction back into the proper creative phase. That trades process rigor for seeing pixels sooner.

## Commands

| Command | What it does |
|---|---|
| `/bencium-init` | Scaffold the harness in the current repo (new or existing project, auto-detected). Also writes `.harness/conventions.md`. |
| `/bencium-next` | **Two-phase:** plan/spec (read-only, waits for your approval) → build (executes). The plan IS the spec. For UI tasks it first offers an optional **prototype detour** — a throwaway one-route preview to eyeball look-and-feel before the spec is written. |
| `/bencium-feature "desc"` | Append a feature to `tasks.md ## Roadmap`. |
| `/bencium-promote` | Move Roadmap items into Now, or demote stale memory entries to archive. |
| `/bencium-decide "title"` | Log a technical decision (and why) to `.harness/archive/`. |
| `/bencium-verify` | Walk `ACCEPTANCE.md` against the actual code. Report pass/fail with evidence. Inspects for gamed tests (weakened or skipped checks) and can delegate a skeptical second opinion to an independent reviewer. |
| `/bencium-deploy` | Verify → deploy → health check → log. Refuses on verify fail. |
| `/bencium-rollback "reason"` | Run the configured rollback command and log the reason. |
| `/bencium-retro` | Postmortem after a failure. Proposes memory and acceptance updates. |

## Use in tools without slash-command plugins

Some tools don't support installable slash-command plugins. You can still get partial value:

1. Grab the `templates/` folder.
2. Copy the templates into your repo (rename `.tmpl` to real names, fill placeholders).
3. Point your tool at the repo.

You lose slash commands, auto-injection, and `/bencium-verify` enforcement. You keep the structural value of having `tasks.md`, `ACCEPTANCE.md`, and `.harness/memory.md` in the repo for the agent to read.
