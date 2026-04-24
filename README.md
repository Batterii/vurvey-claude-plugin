# Vurvey Claude Plugin

Bring your [Vurvey](https://vurvey.com) workspace into Claude. Ask Claude to list surveys, pull responses, run ad-hoc GraphQL, and more — all from a natural-language chat.

This repo is a Claude Code **plugin marketplace** plus an **MCP server config** for the [vurvey](https://github.com/Batterii/vurvey-cli) CLI. Works with Claude Code, Claude Desktop, Cursor, Zed, and any other MCP-compatible client.

## Quick install — Claude Code

1. Install the `vurvey` CLI (see [binary install](#binary-install) below).
2. Authenticate once:
   ```bash
   vurvey login
   ```
3. In Claude Code, add this marketplace and install the plugin:
   ```
   /plugin marketplace add Batterii/vurvey-claude-plugin
   /plugin install vurvey
   ```
4. Ask Claude: *"List my Vurvey surveys."*

## Quick install — Claude Desktop / Cursor / other MCP clients

1. Install the `vurvey` CLI (see [binary install](#binary-install)) and run `vurvey login`.
2. Add the following to your MCP config file:

   **Claude Desktop:** `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows).

   **Cursor:** `~/.cursor/mcp.json`.

   ```json
   {
     "mcpServers": {
       "vurvey": {
         "command": "vurvey",
         "args": ["mcp", "serve"]
       }
     }
   }
   ```
3. Restart the app. Ask the assistant to list your Vurvey surveys.

## Tools

The plugin registers six core-tier tools:

| Tool | Purpose |
|---|---|
| `vurvey_whoami` | Authenticated user info |
| `vurvey_workspace_info` | Active workspace metadata |
| `vurvey_surveys_list` | List surveys (filter by status, name, limit) |
| `vurvey_surveys_get` | Fetch a survey with its questions |
| `vurvey_graphql_query` | Run an arbitrary **read-only** GraphQL query — mutations and subscriptions are rejected |
| `vurvey_graphql_introspect` | Return the full GraphQL schema for tool discovery |

Advanced and destructive tiers (mutations, deletes) are gated behind env vars and ship in a future release. See [Tiers](#tiers) below.

## Binary install

The plugin assumes `vurvey` is on your `$PATH`. Pick one:

### Homebrew (macOS / Linux)

```bash
brew install Batterii/tap/vurvey
```

### Direct download

Grab the latest binary for your platform from [vurvey-cli releases](https://github.com/Batterii/vurvey-cli/releases) and put it on your `$PATH`.

### Build from source

Requires Go 1.25+:

```bash
git clone https://github.com/Batterii/vurvey-cli
cd vurvey-cli
make build
sudo mv vurvey /usr/local/bin/
```

## Tiers

The MCP server exposes tools in three cumulative tiers. Defaults to **core** (read + safe operations).

| Tier | How to unlock | Adds |
|---|---|---|
| core | default | 6 read tools listed above |
| advanced | `VURVEY_MCP_TIER=advanced` in `env` | mutations (create/update surveys, questions, etc.) — **ships in a future release** |
| destructive | `VURVEY_MCP_ALLOW_DESTRUCTIVE=1` **plus** `VURVEY_MCP_TIER=advanced` | deletes — **ships in a future release** |

To unlock tiers, add `env` to your MCP config. For Claude Desktop this looks like:

```json
{
  "mcpServers": {
    "vurvey": {
      "command": "vurvey",
      "args": ["mcp", "serve"],
      "env": {
        "VURVEY_MCP_TIER": "advanced"
      }
    }
  }
}
```

### Read-only kill switch

Set `VURVEY_MCP_READ_ONLY=1` to force read-only regardless of tier. Useful for demos, customer-support sessions, or "let Claude explore but never touch" setups.

## Multi-profile / multi-environment setup

If you use multiple Vurvey profiles (e.g. staging vs production) via `vurvey config profile`, wire a separate MCP server entry per profile:

```json
{
  "mcpServers": {
    "vurvey-staging": {
      "command": "vurvey",
      "args": ["mcp", "serve", "--profile", "staging"]
    },
    "vurvey-prod": {
      "command": "vurvey",
      "args": ["mcp", "serve", "--profile", "prod"]
    }
  }
}
```

Each profile authenticates independently — run `vurvey login --profile staging` (and `--profile prod`) once per profile.

## Troubleshooting

**"Not authenticated with Vurvey" on every tool call**
Run `vurvey login` in your terminal, then restart the MCP server. In Claude Code: `/mcp restart vurvey`.

**"Refusing to start MCP server against non-Vurvey host"**
Your config's `api_url` points at a host that isn't a recognized Vurvey domain. The MCP server hard-fails this case (unlike the CLI which only warns) to prevent credential leakage. Check `~/.config/vurvey/config.json`.

**"Mutations are blocked in core tier"**
You (or Claude) tried a mutation via `vurvey_graphql_query`. Either use a dedicated write tool (advanced tier, future release) or set `VURVEY_MCP_TIER=advanced` in your plugin env config once that tier ships.

**Server appears hung**
Check `~/.config/vurvey/mcp.log` for stderr output. Only JSON-RPC goes to stdout — all diagnostics go there.

**`vurvey` not found**
The plugin's MCP config uses bare `vurvey` so it must be on `$PATH`. If installing via Homebrew on Apple Silicon, you may need `/opt/homebrew/bin` on PATH — start your MCP client from a shell that has your normal `$PATH` loaded.

## Requirements

- **`vurvey` CLI v0.7.0 or newer** (first version shipping the `mcp serve` subcommand)
- A Vurvey workspace and account. Sign up at [vurvey.com](https://vurvey.com).

## Contributing

Issues and pull requests welcome. See the [vurvey-cli repo](https://github.com/Batterii/vurvey-cli) for the MCP server source; this repo only carries the plugin manifest and workflow skill.

## License

MIT — see [LICENSE](LICENSE).
