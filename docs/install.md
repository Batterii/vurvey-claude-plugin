# Installing the Vurvey Plugin

This plugin brings the Vurvey platform API into any MCP-compatible AI client as structured tool calls. The CLI binary (`vurvey`) runs the MCP server; the client connects over stdio.

## TL;DR: the four-command install

For users on `vurvey` **v0.19.1 or newer**, this is the entire setup for Claude Desktop, Cursor, or Codex:

```bash
brew install Batterii/vurvey/vurvey
vurvey login
vurvey workspaces list && vurvey workspaces use <id>   # login does not pick one
vurvey mcp install claude-desktop    # or: cursor | codex | all
```

That's it. `vurvey mcp install` writes the correct MCP server entry into the client's config file (JSON for Claude Desktop/Cursor, TOML for Codex), with an absolute path to the installed binary so GUI apps find it even with stripped `$PATH`. Existing MCP servers in the same file are preserved. It writes no tier, so those clients run at the CLI default, which is `advanced`: reads plus writes. See [step 2b](#2b-select-a-workspace) for why the workspace command is not optional.

**Claude Code users** still need the first three commands (the binary, the login, the workspace). Only the last is different: install the plugin instead of running `vurvey mcp install`:

```bash
brew install Batterii/vurvey/vurvey
vurvey login
vurvey workspaces list && vurvey workspaces use <id>
```

```
/plugin marketplace add Batterii/vurvey-claude-plugin
/plugin install vurvey
```

Then restart your MCP client (or the relevant server entry) and chat normally. Try:

- *"List my Vurvey surveys."*
- *"Show me the answers to question X in survey Y and summarize the sentiment."*
- *"What personas do I have in this workspace?"*

If anything fails, keep reading — the per-client sections below cover troubleshooting, multi-profile setups, read-only mode, and the equivalent hand-edit config if you'd rather skip `vurvey mcp install`.

---

## The full story

All clients share the same prerequisites:

1. **Install the CLI binary**
2. **Authenticate once** with `vurvey login`, then **select a workspace** with `vurvey workspaces use`

Then either run `vurvey mcp install <client>` to auto-configure your client, or follow the per-client section for hand-editing the config yourself.

---

## 1. Install the CLI binary

Pick whichever matches your platform.

### macOS / Linux (Homebrew, recommended)

```bash
brew install Batterii/vurvey/vurvey
```

Homebrew installs to `/opt/homebrew/bin/vurvey` on Apple Silicon, `/usr/local/bin/vurvey` on Intel Macs / Linux. The tap repo is [`Batterii/homebrew-vurvey`](https://github.com/Batterii/homebrew-vurvey); releases are signed with a GPG key shipped in the tap.

### Linux (APT — Debian / Ubuntu)

```bash
curl -fsSL https://storage.googleapis.com/vurvey-cli-releases/apt/gpg.key \
  | sudo gpg --dearmor -o /usr/share/keyrings/vurvey.gpg
echo "deb [signed-by=/usr/share/keyrings/vurvey.gpg] https://storage.googleapis.com/vurvey-cli-releases/apt stable main" \
  | sudo tee /etc/apt/sources.list.d/vurvey.list
sudo apt update && sudo apt install vurvey
```

### Linux (RPM — Fedora / RHEL)

```bash
sudo tee /etc/yum.repos.d/vurvey.repo <<'EOF'
[vurvey]
name=Vurvey CLI
baseurl=https://storage.googleapis.com/vurvey-cli-releases/rpm
enabled=1
gpgcheck=1
gpgkey=https://storage.googleapis.com/vurvey-cli-releases/apt/gpg.key
EOF
sudo dnf install vurvey
```

### Windows (Scoop)

```powershell
scoop bucket add vurvey https://github.com/Batterii/scoop-vurvey
scoop install vurvey
```

### Direct download (any platform)

Binaries + `.deb` / `.rpm` packages are at `https://storage.googleapis.com/vurvey-cli-releases/<version>/`. Each is GPG-signed; SHA-256 checksums are in `checksums.txt`.

```bash
curl -LO https://storage.googleapis.com/vurvey-cli-releases/v0.19.2/vurvey_0.19.2_darwin_arm64.tar.gz
tar -xzf vurvey_0.19.2_darwin_arm64.tar.gz
sudo mv vurvey /usr/local/bin/
```

`curl -s https://storage.googleapis.com/vurvey-cli-releases/latest` prints the current version tag if you want the newest rather than the pinned one above.

### Verify install

```bash
vurvey --version
# → vurvey version 0.19.1  (or newer)

vurvey mcp serve --help
# should list --tier, --read-only, --debug flags
```

You need **v0.19.1 or newer**. That is the release where the four delete-shaped tools (`workflow_schedules_delete`, `workflow_variables_delete`, `workflow_triggers_remove`, `capabilities_remove_workflow`) moved behind `VURVEY_MCP_ALLOW_DESTRUCTIVE=1`. On any earlier binary they are registered at the `advanced` tier, so the delete opt-in gates nothing and "deletes are off" is not true of your install. Older versions also expose fewer tools, and anything before v0.7.0 has no `vurvey mcp serve` at all.

Nothing warns you on a mismatch: the plugin does not check the binary's version, and MCP does not negotiate one, so an old CLI simply answers with a different tool set. If a tool these docs describe is missing from your client, check `vurvey --version` before assuming it is broken, then run `vurvey update` and restart the server.

---

## 2. Authenticate once

```bash
vurvey login
```

This opens a browser, runs Firebase OAuth, and caches your token at `$XDG_CONFIG_HOME/vurvey/config.json` (falls back to `~/.config/vurvey/config.json`). Tokens auto-refresh — you only need to re-run `vurvey login` when your refresh token is invalidated (usually after password changes or 60+ days of inactivity).

### 2b. Select a workspace

`vurvey login` authenticates you. It does **not** pick a workspace, and almost every tool is workspace-scoped, so this is a required second step rather than a preference:

```bash
vurvey workspaces list       # safe to run before one is selected
vurvey workspaces use <id>   # writes workspace_id into the config
```

Skip it and the first real question fails like this:

```
API response: Variable "$workspaceId" got invalid value ""; Value is not a valid UUID:
```

Three things worth knowing:

- The wrapper around that error says the tool's query may be out of sync with the schema and that a developer should look at it. That is a misread. Nothing is wrong with the tool.
- The MCP server reads `workspace_id` once, at startup. After changing it, restart the server (`/mcp restart vurvey` in Claude Code, or a new session elsewhere) or the old value stays in effect.
- `vurvey_workspaces_list` works with no workspace selected, so the assistant can show you the options. It cannot act on them for you at the `core` tier: `vurvey_workspace_switch` is an `advanced` tool and is not registered, and `vurvey_cli` refuses `workspaces use` as a non-read command.

### Verify auth

```bash
vurvey me
# → prints your user info
vurvey surveys list --limit 3
# → prints three surveys in JSON or table format
```

If these fail, the MCP integration won't work either — fix this first.

### Multi-profile setups

If you work across staging and production Vurvey environments, create named profiles:

```bash
vurvey login --profile staging
vurvey login --profile prod
```

You'll add one MCP server entry per profile in the client config (see per-client sections below).

---

## 3. Per-client setup

Pick your client:

- [Claude Code](#claude-code) — Anthropic's terminal-based agent (CLI)
- [Claude Desktop](#claude-desktop) — Anthropic's macOS / Windows desktop app
- [Cursor](#cursor) — AI-first code editor
- [Codex CLI](#codex-cli) — OpenAI's terminal coding assistant
- [Generic MCP client](#generic-mcp-client) — anything that speaks MCP stdio

---

## Claude Code

Claude Code has a first-class plugin system, so wiring is two commands, but they only wire. You still need the binary from [step 1](#1-install-the-cli-binary), a login from [step 2](#2-authenticate-once), and a selected workspace from [step 2b](#2b-select-a-workspace) first. If `which vurvey` is empty, the plugin will install fine and then have nothing to launch. If no workspace is selected, it will launch and then fail your first real question.

### Install

In any Claude Code session, run:

```
/plugin marketplace add Batterii/vurvey-claude-plugin
/plugin install vurvey
```

The first command registers this GitHub repo as a marketplace. The second installs the `vurvey` plugin from it, which wires up the MCP server config + workflow skill automatically.

### Verify it connected

```
/mcp
```

You should see `vurvey` listed with a green / "connected" indicator. If it shows as disconnected, check `~/.config/vurvey/mcp.log` — the server's stderr log.

### Use it

Just chat naturally. The bundled skill tells Claude when to reach for which tool:

> "List my Vurvey surveys."
>
> "What personas do I have in this workspace?"
>
> "Find survey answers that mention 'onboarding' and summarize the sentiment."
>
> "Show market-share data for brand 'Acme'."

Claude will use `vurvey_surveys_list`, `vurvey_personas_list`, `vurvey_answers_search`, `vurvey_brands_market_share` etc. as needed.

### Tiers: what Claude may change

The plugin ships at the **core** tier: reads only. No tool it registers can create, update, run, or delete, and `vurvey_graphql_query` refuses mutations. To allow writes, set `VURVEY_MCP_TIER` to `advanced` in the plugin's `mcp.json`; to also allow deletes, add `VURVEY_MCP_ALLOW_DESTRUCTIVE=1` on top. See the README's "What Claude can change" section, which also lists the client settings that switch off the per-call approval prompt you would otherwise be relying on at `advanced`.

Note that `advanced` is the CLI's own default. This plugin's `env` block is the only reason it runs read-only, and a client configured by `vurvey mcp install` gets no such block.

### Update the plugin

```
/plugin update vurvey
```

Or `/plugin marketplace update Batterii/vurvey-claude-plugin` to refresh the marketplace index first.

### Troubleshoot

| Symptom | Fix |
|---|---|
| `/plugin marketplace add` fails | Make sure you're on a recent Claude Code build with plugin support. Try `claude --version`. |
| Every tool returns "not authenticated" | Run `vurvey login` in a terminal, then `/mcp restart vurvey` in Claude Code. |
| Server not found | Ensure `which vurvey` in your shell returns a path. Claude Code inherits your shell's PATH on launch. |

---

## Claude Desktop

**Easiest path (v0.9.0+):**

```bash
vurvey mcp install claude-desktop
```

This writes the config for you with an absolute binary path, so you can skip the rest of this section unless you want to customize. Restart Claude Desktop afterward.

**Hand-edit path:** if you prefer to write the config manually (or you're on a pre-v0.9.0 CLI), keep reading.

### Config file location

| OS | Path |
|---|---|
| macOS | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Windows | `%APPDATA%\Claude\claude_desktop_config.json` |

Create the file if it doesn't exist. If it already has MCP servers, merge into the existing `mcpServers` object rather than overwriting.

### Minimal config

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

### Reveal the file on macOS

```bash
open "$HOME/Library/Application Support/Claude"
```

Then drag `claude_desktop_config.json` into your editor.

### Multi-profile config

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

### Read-only mode

Add an `env` block if you want Claude Desktop to have read-only access regardless of which tier ships later:

```json
{
  "mcpServers": {
    "vurvey": {
      "command": "vurvey",
      "args": ["mcp", "serve"],
      "env": { "VURVEY_MCP_READ_ONLY": "1" }
    }
  }
}
```

### Apply the config

Quit and relaunch Claude Desktop. You should see a hammer icon in the input box indicating available tools. Click it to see the Vurvey tools listed.

### Use it

Chat as usual:

> "List the surveys in my Vurvey workspace and show the three most recent."
>
> "For survey X, pull the answers to question Y and summarize the sentiment."
>
> "What brands do we track? Give me market-share data for the top two."

### Troubleshoot

| Symptom | Fix |
|---|---|
| No hammer icon after launch | The config is malformed. Validate with `cat "<path>" \| python3 -m json.tool`. |
| Hammer icon but Vurvey tools not listed | Check the server actually starts: run `vurvey mcp serve` in a terminal with `< /dev/null` and see if it responds to JSON-RPC. |
| "Command not found: vurvey" in Claude Desktop logs | Claude Desktop on macOS launches from the Dock, which uses a minimal PATH. Fix by using the absolute path in the config: `"command": "/opt/homebrew/bin/vurvey"` (Apple Silicon) or `"command": "/usr/local/bin/vurvey"`. |
| Auth errors on every call | Run `vurvey login` in a terminal. Claude Desktop picks up the refreshed token on the next tool call (the server auto-refreshes, and the flock-protected config rewrite is safe). |

---

## Cursor

**Easiest path (v0.9.0+):**

```bash
vurvey mcp install cursor
```

Writes `~/.cursor/mcp.json` with an absolute binary path. Skip the rest of this section unless you want per-project config or need to customize.

**Hand-edit path:** Cursor supports MCP servers natively via its own `mcp.json` file — per-project or per-user (global).

### Config file locations

| Scope | Path |
|---|---|
| Per-project (recommended when working on Vurvey code) | `<project-root>/.cursor/mcp.json` |
| Per-user (global, applies everywhere) | `~/.cursor/mcp.json` |

The format matches Claude Desktop exactly.

### Minimal config

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

### Apply

Restart Cursor. Open the chat/composer, expand the "MCP Servers" panel (look for a server / plug icon), confirm `vurvey` shows as connected.

### Use it

Same natural-language prompts work. Cursor's composer has access to the tools as part of any chat:

> "Pull the response distribution from my most recent Vurvey survey and generate a matplotlib script to visualize it."

Cursor is nice for mixed workflows — it can call `vurvey_surveys_get` AND edit your local analysis script in the same session.

### Troubleshoot

Same issues as Claude Desktop generally apply. If Cursor's MCP panel shows a connection error, hover it for the stderr output. Most problems are:

- PATH not finding `vurvey` when Cursor spawns the server → use the absolute path.
- Stale auth → `vurvey login` + restart Cursor.

---

## Codex CLI

**Easiest path (v0.9.0+):**

```bash
vurvey mcp install codex
```

Writes `~/.codex/config.toml` for you with an absolute binary path. Skip the rest of this section unless you want to customize.

**Hand-edit path:** Codex CLI supports MCP servers via a TOML config — slightly different shape from Claude Desktop / Cursor (which use JSON), but the same fields underneath.

Requires **Codex CLI v0.124.0 or newer**. Check with `codex --version`.

### Config file location

```
~/.codex/config.toml
```

Create it if it doesn't exist.

### Minimal config

```toml
[mcp_servers.vurvey]
command = "vurvey"
args = ["mcp", "serve"]
```

That's it — no enclosing object, no `mcpServers` wrapper. Each MCP server is its own TOML table named `[mcp_servers.<name>]`.

### Config fields

| Field | Required | Purpose |
|---|---|---|
| `command` | yes | Executable name (resolved against `$PATH`) or absolute path |
| `args` | no | Array of CLI arguments |
| `env` | no | Map of env vars; supports `${VAR_NAME}` substitution from the parent shell |
| `enabled` | no | Bool (default `true`). Set `false` to disable without deleting the section |
| `cwd` | no | Working directory for the process |

### Multi-profile config

```toml
[mcp_servers.vurvey-staging]
command = "vurvey"
args = ["mcp", "serve", "--profile", "staging"]

[mcp_servers.vurvey-prod]
command = "vurvey"
args = ["mcp", "serve", "--profile", "prod"]
```

### Read-only mode

```toml
[mcp_servers.vurvey]
command = "vurvey"
args = ["mcp", "serve"]
env = { VURVEY_MCP_READ_ONLY = "1" }
```

### Verify it connected

```bash
codex mcp list
```

You should see `vurvey` listed with status "enabled". The `Auth` column may show "Unsupported" — that's normal and doesn't mean anything is wrong (Codex doesn't validate MCP server auth at config time; you only find out auth works when a tool call succeeds or returns the "run vurvey login" error).

### Use it

Just start a Codex conversation and ask naturally:

```
codex
> use the vurvey tools to show my surveys and pull answer summaries for the most recent one
```

Codex auto-discovers the tools at conversation start — no `/mcp` command or equivalent is needed. If the server isn't running, you'll see an error at session init.

### Troubleshoot

| Symptom | Fix |
|---|---|
| `codex mcp list` shows `vurvey` as disabled | Check `enabled = true` (or omit the field) in `config.toml`. Malformed TOML silently disables the entry — validate with `codex mcp list` output. |
| "server not found" / "command failed to start" | Use an absolute path in `command`: `command = "/opt/homebrew/bin/vurvey"`. Codex's `$PATH` at launch may differ from your shell's. |
| Every call returns auth error | Run `vurvey login` in a terminal. Start a new Codex conversation — Codex binds MCP servers at session start, so you need a new session to pick up the refreshed token (it auto-refreshes on subsequent tool calls, but if your token is completely missing Codex fails at init). |
| MCP server crashed mid-session | Codex does NOT auto-restart crashed MCP servers. Exit the conversation with `ctrl+d` and start a new one. |
| TOML syntax error breaks Codex | Back up and rewrite from the minimal-config snippet. Codex's TOML parser is strict; a missing `=` or stray quote fails the whole file. |

### Management commands

```bash
codex mcp list                # show all MCP servers + status
codex mcp add vurvey          # interactive setup (alternative to editing config.toml by hand)
codex mcp remove vurvey       # unregister
codex mcp --help              # full reference
```

---

## Generic MCP client

Any client that speaks the [MCP stdio protocol](https://modelcontextprotocol.io/) can launch the Vurvey server. The command is:

```
vurvey mcp serve
```

The server accepts the standard `initialize` → `tools/list` → `tools/call` lifecycle. Tool schemas are self-describing via `tools/list`. The server won't emit anything to stdout until the client sends an `initialize` request (we have a regression test guarding this).

Optional env vars:

| Env var | Effect |
|---|---|
| `VURVEY_MCP_READ_ONLY=1` | Force read-only regardless of tier |
| `VURVEY_MCP_TIER=advanced` | 82 tools: reads plus create, update, run, schedule, switch, and arbitrary non-delete GraphQL mutations. **This is the CLI default**, so it is what you get when no `env` block is present. |
| `VURVEY_MCP_TIER=core` | 53 tools, read-only, mutations rejected. This is what the Claude Code plugin pins. |
| `VURVEY_MCP_ALLOW_DESTRUCTIVE=1` (+ advanced) | 86 tools: adds the four delete tools and allows `deleteX` mutations |

---

## What tools you get

53 read-only tools at the `core` tier the plugin ships, 82 at `advanced`, 86 with deletes enabled. Measured against `vurvey` v0.19.2 by sending `tools/list` to `vurvey mcp serve`; add one to each against an environment that permits GraphQL introspection. Full breakdown and provenance in the [README](../README.md#what-claude-can-change). Highlights:

| Group | Tools |
|---|---|
| Identity | `whoami`, `workspace_info` |
| Surveys | `surveys_list`, `surveys_get` |
| Questions | `questions_list`, `questions_get` |
| Answers | `answers_list`, `answers_get`, `answers_search` |
| Workflows | `workflows_list`, `workflows_get`, `workflows_history` |
| Personas | `personas_list`, `personas_get`, `personas_members` |
| Brands | `brands_list`, `brands_get`, `brands_insights`, `brands_market_share` |
| Media | `clips_list`, `clips_get`, `files_get`, `file_tags_list` |
| GraphQL | `graphql_query` (queries at any tier; mutations at `advanced`; mutations whose field name starts with delete/remove/destroy only at `destructive`), `graphql_introspect` |

**`vurvey_graphql_query` is not read-only.** It sends whatever GraphQL you hand it to the Vurvey API under your own credentials. At `core` it refuses mutations outright. At `advanced` it runs arbitrary mutations, and the only thing refused is a field name matching `delete|remove|destroy` followed by a capital letter. That is a regular expression over the query text, not a check against what the mutation actually does, so an `archiveX`, `disableX`, `purgeX`, or `resetX` mutation executes. It is one of the reasons the plugin ships at `core`.

All tool names are prefixed `vurvey_` (e.g. `vurvey_surveys_list`). The full list with arguments is discoverable via each client's tool browser, or programmatically by sending a `tools/list` request.

---

## Common issues across all clients

**"not authenticated with Vurvey"** — run `vurvey login` in a terminal, then restart your MCP client (or just the `vurvey` server entry, where supported).

**"refusing to start MCP server against non-Vurvey host"** — your config's `api_url` isn't a recognized Vurvey domain. The MCP server hard-fails this (the CLI only warns) to prevent credential leakage. Check `~/.config/vurvey/config.json`.

**"mutations require advanced tier"**: the server is running at `core`, which is what the Claude Code plugin ships, so this is expected rather than broken. Read [Tiers: what Claude may change](#tiers-what-claude-may-change) first, then set `VURVEY_MCP_TIER=advanced` in your MCP config's `env` block and restart the server if you do want writes.

**`Variable "$workspaceId" got invalid value ""`**: no workspace is selected. Run `vurvey workspaces list`, then `vurvey workspaces use <id>`, then restart the server. See [2b. Select a workspace](#2b-select-a-workspace). The error's own wording suggests a schema problem in the tool; that is a misread.

**"destructive operations require VURVEY_MCP_ALLOW_DESTRUCTIVE"** — deletes are gated on purpose. Add that env var only if you intend to let the assistant delete things.

**Server appears hung** — check `~/.config/vurvey/mcp.log`. Stdout is reserved for JSON-RPC; all diagnostics go to the log file or stderr.

**Need to file an issue?** [github.com/Batterii/vurvey-claude-plugin/issues](https://github.com/Batterii/vurvey-claude-plugin/issues).
