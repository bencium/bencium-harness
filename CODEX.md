if i update the source harness, how will i be able to update the codex version too?
10:01 AM


Right now, the Codex version is a copied adapter, not a live link.

That means updating:

/Users/bencium/benc-harness-source/bencium-harness

does not automatically update the Codex-installed copy at:

/Users/bencium/.codex/plugins/cache/bencium-harness-local/bencium-harness/0.4.0

or the local marketplace copy at:

/Users/bencium/.codex/local-marketplaces/bencium-harness-codex/plugins/bencium-harness

How to update it today

For now, rerun the installer I created:

node /private/tmp/install-bencium-harness-codex.mjs
That will recopy the harness source into the Codex marketplace/cache and regenerate the Codex skill wrappers.

But important caveat: that script lives in /private/tmp, so it is temporary. Also, the version path is currently hardcoded to 0.4.0, so if you bump the source plugin version to 0.4.1, the current installer should be improved before relying on it long term.

The right long-term setup

I’d use this model:

Keep editing the source harness here:
/Users/bencium/benc-harness-source/bencium-harness

Add a permanent sync script inside the harness source, for example:
/Users/bencium/benc-harness-source/bencium-harness/scripts/sync-codex-adapter.mjs

Make that script:

read .claude-plugin/plugin.json
use the real version from that manifest
auto-discover all files in commands/*.md
auto-discover all skills/*/SKILL.md
copy templates/hooks/statusline
regenerate .codex-plugin/plugin.json
update /Users/bencium/.codex/config.toml
write both:
local marketplace copy
plugin cache copy
After running the sync, restart/refresh Codex so the plugin index reloads.

Then your update flow becomes simple:

cd /Users/bencium/benc-harness-source/bencium-harness
node scripts/sync-codex-adapter.mjs
The three options

Option 1: Manual copy
Fast, but easy to forget files and break the adapter.

Option 2: Symlink Codex directly to the source
Tempting, but brittle. Codex plugin cache expects a stable installed copy, and symlinks can behave badly across refresh/cache logic.

Option 3: Permanent sync script
Best fit. Source remains authoritative, Codex gets a clean generated adapter, and version bumps become predictable.

So: for a quick update today, rerun the temp installer. For the maintainable version, I’d add the permanent sync script to the harness source and make it auto-discover commands/skills instead of relying on today’s hardcoded list.