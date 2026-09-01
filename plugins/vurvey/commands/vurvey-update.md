---
description: Check whether the Vurvey plugin is up to date and say exactly what to run
---

Check whether the installed Vurvey plugin is behind what is published, and report it in one or two
lines.

This checks the plugin only. There is nothing else to update: the connector is hosted by Vurvey, so
its tools change on Vurvey's side with no action from the user.

## 1. Compare

Latest published version:

```bash
curl -s https://raw.githubusercontent.com/Batterii/vurvey-claude-plugin/main/.claude-plugin/marketplace.json
```

Read `plugins[0].version` from that JSON and compare it against the installed plugin version, which
is in this plugin's own `.claude-plugin/plugin.json`. If you cannot determine the installed
version, say so rather than guessing.

## 2. If it is behind

```
/plugin marketplace update Batterii/vurvey-claude-plugin
/plugin update vurvey
```

The marketplace refresh comes first. `/plugin update` compares against the cached marketplace
index, so updating without refreshing can report "already up to date" when it is not.

## 3. Offer to stop the manual checking

If it was behind, mention once that auto-update exists, and show the snippet for
`~/.claude/settings.json`:

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

## Reporting

Keep it short. If it is current, say so in one line and stop. Only expand when it is behind, and
lead with the exact command to run. If the network call fails, say so rather than claiming
everything is fine.

A stale plugin is not a likely cause of a missing tool. The tool set comes from the hosted
connector and from what the user approved, so check `/vurvey-connection` first for that.
