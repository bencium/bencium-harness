# Plan: `bencium-harness` — AI-assisted build harness as Claude Code plugin

## Context

You want a lightweight scaffolding + orchestration layer that gives an AI agent stable context, constraints, feedback loops, and verification gates while it builds a project. Distinct from:

- **BMAD-method**: heavy, multi-phase interview, generates exhaustive PRD/architecture docs.
- **Vector-DB memory systems**: retrieval-based memory with embeddings, complex infra.

The pitch is the **professional layer that separates serious AI-assisted builders**: a clean repo structure, generated planning artifacts, a living roadmap, an enforced acceptance checklist, a commit/deploy/rollback rhythm, and a growing markdown memory that survives `/clear` and gets re-injected next session. The harness is what the agent works *inside*, so it drifts less and learns from its own failures.

Outcome: ship as a private Claude Code plugin distributed via raw zip, no gating, suitable for hand-delivery to clients and self-dogfooding on Bencium projects.

## Decisions locked during grilling

| Branch | Decision |
|---|---|
| Form factor | Claude Code plugin (skills + commands + hooks). Not portable to claude.ai / Claude for Work — those users get graceful degradation by manually dropping the markdown artifacts into a Project. |
| MVP scope | Full kickoff kit: scaffold + tasks + acceptance + commit hook + decision log + loop + deploy/rollback |
| Memory model | Tiered markdown — hot `.harness/memory.md` (auto-injected via SessionStart hook), cold `.harness/archive/NNNN-*.md` (append-only). No DB, no embeddings. Agent curates promotion/demotion. |
| Distribution | Raw zip, no license, no phone-home. Unzip into `~/.claude/plugins/bencium-harness/`. Trust-based; revocation = "don't send v2." |
| Opinionation | Neutral templates; opinions live in `.harness/rules.md` which user fills or imports via optional `/bencium-import-rules`. |
| Planning artifacts | `/bencium-init` runs a 5-question interview + single LLM call (Haiku-first) and generates 1-page PRD.md, 1-page ARCHITECTURE.md, ~15-item tasks.md, ~10-item ACCEPTANCE.md. Total ~60 seconds. |
| Greenfield/brownfield | Universal `/bencium-init` auto-detects empty vs populated repo, branches internally, asks before overwriting anything. Brownfield path scans repo and produces retrospective PRD/ARCH. |
| Verification | `/bencium-verify` loads ACCEPTANCE.md and has the agent walk each unchecked item against the actual code/tests/deploy, reporting PASS/FAIL/SKIP with evidence and updating checkboxes. |
| Build loop | Living `tasks.md` with `## Now` (≤15) and `## Roadmap` sections. `/bencium-feature` appends, `/bencium-promote` moves Roadmap→Now. `/bencium-retro` triggers automatically on `/bencium-verify` failure, writes a postmortem to `archive/`, proposes memory.md + ACCEPTANCE.md updates. |
| Deploy/rollback | `.harness/config.yaml` declares `deploy.cmd` + `deploy.health` + `rollback.cmd`. `/bencium-deploy` runs `/bencium-verify` first (refuses if fail), runs deploy, runs health check, logs to memory.md. Failed deploys auto-invoke `/bencium-retro`. |
| Name | `bencium-harness` (slash prefix `/bencium-*`) |

**Flagged v1 risk:** surface area grew to 9 commands + 2 auto-skills + 1 hook. The "much simpler than BMAD" promise now relies on each command being load-bearing and fast (≤60s init, single LLM call) rather than on having fewer pieces. If users complain about volume after dogfooding, consolidate `/bencium-feature` + `/bencium-promote` into one and drop the post-commit hook.

## Recommended approach

### Plugin structure

```
bencium-harness/
├── .claude-plugin/
│   └── plugin.json                 # name, version, marketplace metadata
├── commands/
│   ├── bencium-init.md             # 5-Q interview → all artifacts
│   ├── bencium-next.md             # pop next task from Now
│   ├── bencium-decide.md           # append decision to archive/
│   ├── bencium-verify.md           # walk ACCEPTANCE.md
│   ├── bencium-promote.md          # archive ↔ hot memory promotion
│   ├── bencium-feature.md          # append to Roadmap
│   ├── bencium-retro.md            # postmortem + propose updates
│   ├── bencium-deploy.md           # verify → deploy → healthcheck → log
│   └── bencium-rollback.md         # rollback + log reason
├── skills/
│   ├── context-loader/SKILL.md     # SessionStart: inject memory.md + rules.md
│   └── decision-watcher/SKILL.md   # intent trigger on architectural talk
├── hooks/
│   └── post-commit-nag.sh          # PostToolUse on git commit → tasks.md check
├── templates/
│   ├── README.md.tmpl
│   ├── PRD.md.tmpl
│   ├── ARCHITECTURE.md.tmpl
│   ├── tasks.md.tmpl               # Now/Roadmap split
│   ├── ACCEPTANCE.md.tmpl
│   ├── rules.md.tmpl               # blank, user fills or imports
│   ├── glossary.md.tmpl
│   ├── constraints.md.tmpl
│   ├── config.yaml.tmpl            # deploy/rollback/health
│   └── memory.md.tmpl              # seeded with init Q&A
├── README.md                       # install instructions, command reference
└── INSTALL.md                      # unzip path, settings.json snippet
```

### What `/bencium-init` produces in the target project

```
target-project/
├── README.md                       # generated from PRD + config
├── PRD.md                          # 1-page, from 5 questions
├── ARCHITECTURE.md                 # 1-page, from 5 questions
├── tasks.md                        # 15 items, Now/Roadmap
├── ACCEPTANCE.md                   # 10 checks, ties to PRD
└── .harness/
    ├── config.yaml                 # deploy + rollback commands
    ├── memory.md                   # hot context, auto-loaded each session
    ├── rules.md                    # blank
    ├── glossary.md                 # blank
    ├── constraints.md              # blank
    └── archive/                    # NNNN-decision-*.md, NNNN-retro-*.md
```

### Session start flow

1. User opens Claude Code in a project with `.harness/`.
2. `context-loader` skill (SessionStart trigger) reads `.harness/memory.md` + `.harness/rules.md` + last 3 entries in `.harness/archive/` and injects as system context.
3. Agent has fresh state but with project memory restored.

### Build loop flow (v0.2.0 — six explicit phases)

```
🗺️ roadmap → 📋 plan/spec → 🔨 build → ✓ test → 💭 reflect → 🚀 deploy
```

1. **🗺️ roadmap** — `tasks.md` is the living roadmap. `/bencium-feature` appends. `/bencium-promote` moves Roadmap → Now.
2. **📋 plan/spec** — `/bencium-next` Phase A picks the top unchecked item from `## Now`, reads `.harness/conventions.md` + `ACCEPTANCE.md`, writes a 10–25 line plan inline (cyan banner), **stops, and waits for explicit user approval**. No files are written in this phase. The plan IS the spec.
3. **🔨 build** — `/bencium-next` Phase B (green banner) is entered only after the user replies `approve`. Executes per the approved plan, marks `[x]` in `tasks.md`, prints `📝 Memory: …` confirmation.
4. **✓ test** — `/bencium-verify` (yellow banner) walks `ACCEPTANCE.md` against actual code. PASS/FAIL/SKIP with evidence. Renders the structured next-moves template at the end.
5. **💭 reflect** — `/bencium-retro` (magenta banner) auto-fires on verify failure or any deploy failure. Five-whys, writes archive entry, proposes memory.md + ACCEPTANCE.md updates with user approval.
6. **🚀 deploy** — `/bencium-deploy` (bright-blue banner) re-runs verify (refuses if any FAIL), runs deploy.cmd, health-checks 6×10s, logs to memory. `decision-watcher` nudges `/bencium-decide` whenever the agent picks between architectural alternatives. Post-commit hook nags if `tasks.md` was not updated.

### Distribution

1. Zip the `bencium-harness/` folder: `zip -r bencium-harness-0.1.0.zip bencium-harness/`.
2. Email/Dropbox/Drive to client.
3. Client unzips into `~/.claude/plugins/bencium-harness/`.
4. Client adds plugin path to `~/.claude/settings.json` per the INSTALL.md snippet.
5. Restart Claude Code. `/bencium-init` is now available.

### Graceful degradation for claude.ai / Claude for Work users

- They cannot install the plugin.
- They can still receive the zip, extract the `templates/` folder, manually copy templates into their repo, and attach the repo to a claude.ai Project.
- The agent will read the markdown files as instructions but will not have auto-injection, slash commands, hooks, or `/bencium-verify` enforcement. Document this in README.md.

## Files to create

All paths below are inside the new `bencium-harness/` plugin source repo (which itself becomes the zipped artifact):

- `.claude-plugin/plugin.json` — plugin metadata, version, command/skill/hook registration
- `commands/bencium-*.md` (9 files) — each is a slash command prompt
- `skills/context-loader/SKILL.md` — SessionStart hook + injection instructions
- `skills/decision-watcher/SKILL.md` — intent detection prompt
- `hooks/post-commit-nag.sh` — PostToolUse Bash hook script
- `templates/*.tmpl` (10 files) — Mustache-style templates (`{{product_name}}` etc.) consumed by `/bencium-init`
- `README.md`, `INSTALL.md` — recipient-facing docs

## Verification

End-to-end smoke test for the harness itself:

1. **Greenfield init**: `mkdir /tmp/harness-test && cd /tmp/harness-test && git init`. Open Claude Code. Run `/bencium-init`. Answer 5 questions. Expect: PRD.md, ARCHITECTURE.md, tasks.md, ACCEPTANCE.md, .harness/ folder created within 90 seconds. Inspect each file for sane content.
2. **Brownfield init**: `cd ~/existing-project` (any real Bencium project). Run `/bencium-init`. Expect: detection of existing code, fewer questions asked, retrospective PRD/ARCH generated without overwriting any source files.
3. **Memory persistence**: Run `/bencium-decide "use Supabase for auth"`. Run `/clear`. Confirm next session: agent recalls the decision via `context-loader` skill injection.
4. **Verify-gate**: Manually break one ACCEPTANCE.md item (e.g., add a `SELECT *` to a query if that's one of the checks). Run `/bencium-verify`. Expect: FAIL report with evidence, refusal to proceed, auto-invocation of `/bencium-retro`.
5. **Deploy gate**: Run `/bencium-deploy` with a failing verify. Expect: deploy refused, no `fly deploy` invocation, report shown.
6. **Distribution**: Zip plugin, transfer to a second machine, unzip into `~/.claude/plugins/`, edit settings.json, restart Claude Code. Run `/bencium-init` in a fresh directory. Expect: identical behavior to dev machine.

## v2 candidates (explicitly out of v1)

- License-key gating for paid distribution
- MCP server companion for claude.ai/Cursor/Codex users
- `/bencium-sprint-start` / `/bencium-sprint-end` ceremony
- Integration with existing planning skills (`/plan-ceo-review`, `/plan-eng-review`)
- Stack presets (`--preset nextjs-fly-supabase`)
- ReasoningBank/HNSW memory upgrade if pure markdown hits limits
- Public marketplace JSON
