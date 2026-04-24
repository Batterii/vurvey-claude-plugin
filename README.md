# Vurvey Claude Plugin

Bring your [Vurvey](https://vurvey.com) workspace into your AI assistant. Ask Claude, Cursor, or Codex to list surveys, search answers, pull brand insights, run ad-hoc GraphQL, and more — all from a natural-language chat.

Works with **Claude Code**, **Claude Desktop**, **Cursor**, **OpenAI Codex CLI**, and any MCP-compatible client.

## Quick install (3 commands)

```bash
brew install Batterii/vurvey/vurvey
vurvey login
vurvey mcp install claude-desktop    # or: cursor | codex | all
```

That's the full install. `vurvey mcp install` detects the client's config file, writes the correct MCP server entry with an absolute binary path (so GUI apps find it), and preserves any MCP servers you already had configured.

**Claude Code users** — even simpler, install the plugin itself:

```
/plugin marketplace add Batterii/vurvey-claude-plugin
/plugin install vurvey
```

More detail per client (config paths, multi-profile setup, troubleshooting): [`docs/install.md`](docs/install.md).

### Try these prompts

Once installed, ask your assistant things like:

- *"List my Vurvey surveys."*
- *"Show me the questions in survey X."*
- *"Search Vurvey answers for mentions of 'onboarding' and summarize the sentiment."*
- *"What personas do I have in this workspace, and who's a member of each?"*
- *"Show me brand insights for Acme and pull the market-share data."*
- *"Compare the response counts of my three most recent surveys."*
- *"What workflows have run in the last week? Show me the latest history for each."*

## What you get

**25 tools across 9 resource groups** in the default (read-only) tier:

| Group | Tools |
|---|---|
| Identity | `vurvey_whoami`, `vurvey_workspace_info` |
| Surveys | `vurvey_surveys_list`, `vurvey_surveys_get` |
| Questions | `vurvey_questions_list`, `vurvey_questions_get` |
| Answers | `vurvey_answers_list`, `vurvey_answers_get`, `vurvey_answers_search` |
| Workflows | `vurvey_workflows_list`, `vurvey_workflows_get`, `vurvey_workflows_history` |
| Personas | `vurvey_personas_list`, `vurvey_personas_get`, `vurvey_personas_members` |
| Brands | `vurvey_brands_list`, `vurvey_brands_get`, `vurvey_brands_insights`, `vurvey_brands_market_share` |
| Media | `vurvey_clips_list`, `vurvey_clips_get`, `vurvey_files_get`, `vurvey_file_tags_list` |
| GraphQL escape hatch | `vurvey_graphql_query` (read-only), `vurvey_graphql_introspect` |

Advanced tools (mutations, deletes) are gated behind env vars and ship in a future release.

## Tiers

The MCP server exposes tools in three cumulative tiers. Defaults to **core** (read + safe operations).

| Tier | How to unlock | Adds |
|---|---|---|
| core | default | 25 read tools listed above |
| advanced | `VURVEY_MCP_TIER=advanced` | mutations (create/update) — **future release** |
| destructive | `VURVEY_MCP_ALLOW_DESTRUCTIVE=1` **plus** `VURVEY_MCP_TIER=advanced` | deletes — **future release** |

Set `VURVEY_MCP_READ_ONLY=1` to force read-only regardless of tier.

## Multi-profile setups

If you work across staging and production Vurvey environments, run `vurvey login --profile <name>` once per profile and wire one MCP server entry per profile. Config snippets for each client are in [`docs/install.md`](docs/install.md).

## Requirements

- **`vurvey` CLI v0.8.0 or newer** on `$PATH` for the full 25-tool surface
- A Vurvey workspace and account — sign up at [vurvey.com](https://vurvey.com)

## Troubleshooting

Common issues live in the per-client sections of [`docs/install.md`](docs/install.md#common-issues-across-all-clients). The most frequent ones:

- **"Not authenticated"** → run `vurvey login`, then restart the MCP server in your client.
- **"Command not found: vurvey"** → use an absolute path in your MCP config (`/opt/homebrew/bin/vurvey` on Apple Silicon). Your MCP client's `$PATH` at spawn time may differ from your shell's.
- **Server hung** → check `~/.config/vurvey/mcp.log`. Only JSON-RPC goes to stdout; all diagnostics go to the log file.

## Contributing

Issues and pull requests welcome on this plugin repo. The MCP server source lives in `Batterii/vurvey-cli` (private — access for Vurvey staff / collaborators).

## License

MIT — see [LICENSE](LICENSE).
