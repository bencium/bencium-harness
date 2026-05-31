# Install bencium-harness

This plugin is distributed as a **local Claude Code marketplace**. No unzip, no `settings.json` editing, no GitHub push. One marketplace registration, one `/plugin install`.

## Test install (local, recommended for dogfooding)

### 1. Register the local marketplace (one-time)

Inside Claude Code, run:

```
/plugin marketplace add /Users/bencium/benc-harness-source
```

This adds `bencium-harness-local` to your `~/.claude/plugins/known_marketplaces.json`.

### 2. Install the plugin

```
/plugin install bencium-harness@bencium-harness-local
```

That's the one-line install. The plugin is now cached under `~/.claude/plugins/cache/bencium-harness-local/bencium-harness/0.1.0/`.

### 3. Make the post-commit hook executable

```bash
chmod +x ~/.claude/plugins/cache/bencium-harness-local/bencium-harness/0.1.0/hooks/post-commit-nag.sh
```

(Optional — needed only if you want the post-commit nag wired into your global hooks.)

### 4. Restart Claude Code

Quit and relaunch. Verify with:

```
/plugin list
```

You should see `bencium-harness@bencium-harness-local` listed as enabled. Then:

```
/bencium-init
```

should trigger the 5-question interview.

## Updating after edits

Edit files directly in `/Users/bencium/benc-harness-source/`, then in Claude Code:

```
/plugin update bencium-harness@bencium-harness-local
```

Restart and re-test.

## Public install (later, after publishing)

When ready to share publicly, add `bencium-harness` to a public marketplace (e.g. `bencium/bencium-marketplace` on GitHub). Recipients then run:

```
/plugin marketplace add bencium/bencium-marketplace
/plugin install bencium-harness@bencium-marketplace
```

Until then, keep distribution to the local marketplace path above.

## Uninstall

```
/plugin uninstall bencium-harness@bencium-harness-local
/plugin marketplace remove bencium-harness-local
```

Restart Claude Code.

## Statusline (optional)

The plugin ships a statusline script at `statusline/harness-status.sh`. Claude Code plugins cannot register a statusline declaratively, so wire it into your own `~/.claude/settings.json`:

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/plugins/cache/bencium-harness-local/bencium-harness/<VERSION>/statusline/harness-status.sh",
    "padding": 1
  }
}
```

Replace `<VERSION>` with the installed version (check with `/plugin list`). Then make the script executable:

```bash
chmod +x ~/.claude/plugins/cache/bencium-harness-local/bencium-harness/<VERSION>/statusline/harness-status.sh
```

Restart Claude Code. In any project with a `.harness/` folder you will see a two-row bar at the bottom:

```
🗺️ ▸ 📋 ▸ [🔨] ▸ ✓ ▸ 🚀 ▸ 🔥 ▸ 💭   todo-app
"Add CSV export"   5 now · 7/10 ✓   💫 haiku $0.04   ctx 38%
```

Row 1 is the phase-progress strip (current phase wrapped in `[ ]`) plus the project name from `.harness/memory.md`. Row 2 is the current task, `## Now` count, ACCEPTANCE ratio, model, session cost, and context-window usage.

Projects without `.harness/` produce no output — your previous statusline, if any, takes over. The script is pure bash + awk + grep, no dependencies, <50ms per render.

The phase comes from `.harness/state`, which each `/bencium-*` command writes on entry. If you delete it, the strip renders without a highlight.

## Use with claude.ai or Claude for Work

Plugins do not run in claude.ai or Claude for Work (web). You can still use the template files manually — see the README for the graceful-degradation path.

Command prompts have a portability note where they call `AskUserQuestion` (a Claude Code-only tool). Outside Claude Code the agent will ask the same questions inline as a numbered list — no manual edits required.
