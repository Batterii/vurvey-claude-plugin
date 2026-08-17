---
description: Check whether the Vurvey CLI and plugin are up to date, and say exactly what to run
---

Check both halves of the Vurvey integration and report anything stale. The CLI and the plugin version independently, so check each.

## 1. The CLI

Installed version:

```bash
vurvey --version
```

Latest published version (public, no auth needed):

```bash
curl -s https://storage.googleapis.com/vurvey-cli-releases/latest
```

Compare them. If the installed version is behind, tell the user to run:

```bash
vurvey update
```

Then have them restart the MCP server (`/mcp restart vurvey` in Claude Code) so the new binary is picked up. A stale CLI is the usual cause of "that tool doesn't exist" — tools are added in CLI releases, not plugin releases.

## 2. The plugin

Latest published plugin version:

```bash
curl -s https://raw.githubusercontent.com/Batterii/vurvey-claude-plugin/main/.claude-plugin/marketplace.json
```

Read `plugins[0].version` from that JSON and compare it against the installed plugin version, which is in this plugin's own `.claude-plugin/plugin.json`. If you cannot determine the installed version, say so rather than guessing.

If the plugin is behind, tell the user to run:

```
/plugin marketplace update Batterii/vurvey-claude-plugin
/plugin update vurvey
```

The marketplace refresh comes first — `/plugin update` compares against the cached marketplace index, so updating without refreshing can report "already up to date" when it isn't.

## 3. Offer to stop the manual checking

If anything was stale, mention once that auto-update exists, and show the snippet for `~/.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "vurvey": {
      "source": { "source": "github", "repo": "Batterii/vurvey-claude-plugin" },
      "autoUpdate": true
    }
  }
}
```

Note that this keeps the *plugin* current automatically, but not the CLI binary — `vurvey update` is still a manual step, or their package manager's (`brew upgrade vurvey`).

## Reporting

Keep it short. If both are current, say so in one line and stop — don't print version tables or explain the architecture. Only expand when something is actually behind, and lead with the exact command to run.

If the network calls fail, say which one failed and report what you could determine locally rather than claiming everything is fine.
