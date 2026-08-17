---
name: vurvey
description: Use when the user asks about Vurvey surveys, questions, answers, responses, workflows, capabilities, personas, brands, chat threads, clips, or files in their Vurvey workspace. Triggers on phrases like "check my surveys", "list surveys", "show workflows", "find brand insights", "search answers", "vurvey personas", "analyze survey responses", "what's in my Vurvey workspace", "what capabilities do we have", or "run a Vurvey GraphQL query". Only activates when the vurvey MCP server is available.
---

# Vurvey Workflow Guide

The `vurvey` MCP server exposes the Vurvey platform API as structured tools. Use this skill to understand **when to reach for which tool** and **how to chain them** for common Vurvey research workflows.

## Start here

`vurvey_workspace_overview` is the cheapest way to orient. One call returns the user, the active workspace, the 5 most recent surveys, and the 5 most recent workflows. Prefer it over chaining `whoami` + `workspace_info` + `surveys_list` + `workflows_list` on the first turn.

## Available tools (advanced tier, 84 tools)

The plugin pins `VURVEY_MCP_TIER=advanced`: all reads, plus create / update / run / schedule. Deletes are gated behind `VURVEY_MCP_ALLOW_DESTRUCTIVE=1` and are **not** available by default.

The read tools are listed first, then the write tools under [Changing things](#changing-things).

### Identity & workspace

| Tool | Purpose |
|---|---|
| `vurvey_workspace_overview` | Composite first-turn anchor: user + workspace + recent surveys + recent workflows in one call. |
| `vurvey_whoami` | Confirm the authenticated user. Use when auth state is unclear. |
| `vurvey_workspace_info` | Active workspace metadata (name, plan, enabled features). |
| `vurvey_workspaces_list` | List every workspace the user belongs to. |
| `vurvey_environment_get` / `vurvey_environments_list` | Which Vurvey environment the CLI is pointed at, and what else is available. |
| `vurvey_cli` | Describe the underlying CLI (version, config path). Diagnostic. |

### Surveys, questions, answers

| Tool | Purpose |
|---|---|
| `vurvey_surveys_list` | List surveys. Filters: `status` (`DRAFT`, `OPEN`, `CLOSED`, `ARCHIVED`), `name`, `limit`, `cursor`. |
| `vurvey_surveys_get` | Fetch one survey by id, including questions and response count. |
| `vurvey_surveys_find_by_name` | Substring resolver — one call instead of list-then-filter. Prefer when the user names a survey. |
| `vurvey_questions_list` / `vurvey_questions_get` | Questions in a survey (`survey_id`); a single question with its choices (`id`). |
| `vurvey_answers_list` / `vurvey_answers_get` | Answers for a question (`question_id`); a single answer (`id`). |
| `vurvey_answers_search` | Free-text search across all answers in the workspace. |

### Responses

| Tool | Purpose |
|---|---|
| `vurvey_responses_list` / `vurvey_responses_get` | Survey responses — the respondent-level view above individual answers. |
| `vurvey_responses_export` | Bulk-fetch up to 1000 responses in one call. Use for analysis instead of looping `responses_get`. |

### Workflows (orchestrations)

| Tool | Purpose |
|---|---|
| `vurvey_workflows_list` / `vurvey_workflows_get` | Inventory; a single workflow definition. |
| `vurvey_workflows_status` | Current run state. Use before assuming a workflow is idle. |
| `vurvey_workflows_history` / `vurvey_workflows_history_entry` | Past runs; one run in detail. |
| `vurvey_workflow_templates_list` / `vurvey_workflow_templates_get` | Reusable workflow templates. |
| `vurvey_workflow_schedules_list` / `vurvey_workflow_schedules_get` | Scheduled recurrence for a workflow. |
| `vurvey_workflow_triggers_list` / `vurvey_workflow_triggers_get` | What causes a workflow to fire. |
| `vurvey_workflow_variables_list` | Variables bound into a workflow run. |

### Capabilities

| Tool | Purpose |
|---|---|
| `vurvey_capabilities_list` / `vurvey_capabilities_get` | Capabilities configured in the workspace. |
| `vurvey_capabilities_pipeline_progress` | How far a capability's pipeline has gotten. Use when the user asks why output is missing. |
| `vurvey_capability_blueprints_list` / `vurvey_capability_blueprints_get` | Prebuilt capability blueprints available to deploy. |

### Personas & brands

| Tool | Purpose |
|---|---|
| `vurvey_personas_list` / `vurvey_personas_get` | AI personas in the workspace. |
| `vurvey_personas_members` | Member accounts attached to a persona (`id`). |
| `vurvey_brands_list` / `vurvey_brands_get` | Brands in the workspace. |
| `vurvey_brands_insights` | Insights report for a brand (`id`). |
| `vurvey_brands_market_share` | Market share **by brand name, not id**. |

### Chat

| Tool | Purpose |
|---|---|
| `vurvey_chat_list` / `vurvey_chat_get` | Existing Vurvey chat threads and their messages. |
| `vurvey_chat_message_grounding` | The sources a chat answer was grounded in. Use when the user asks "where did that come from?". |
| `vurvey_chat_export_markdown` | Export a thread as markdown. |

### Media

| Tool | Purpose |
|---|---|
| `vurvey_clips_list` / `vurvey_clips_get` | Video clips for a survey (`survey_id`); a single clip (`id`). |
| `vurvey_files_get` | File metadata (`id`). |
| `vurvey_file_tags_list` | File-tag keys in the workspace. |

### GraphQL escape hatch

| Tool | Purpose |
|---|---|
| `vurvey_graphql_query` | Arbitrary GraphQL operation. Queries always allowed; mutations allowed at this tier; delete-pattern mutations (`deleteX`/`removeX`/`destroyX`) rejected. Subscriptions unsupported. |

`vurvey_graphql_introspect` registers only when the target API allows introspection. Production has it disabled, so against production the tool is absent by design — don't tell the user it's broken. Don't send `__schema` / `__type` queries through `vurvey_graphql_query` either; they're rejected.

### Anything else: the CLI escape hatch

`vurvey_cli` runs any `vurvey` CLI subcommand and returns its output. Reach for it when no dedicated tool covers the request — billing, contacts, segments, collections, training sets, people models, respondents, rewards, system prompts, templates, transcripts, attributes, discovery, reels.

- **Discover before guessing:** call with `args: ["<resource>", "--help"]` (e.g. `["billing", "--help"]`) to see real subcommands and flags. Don't invent flags.
- **Don't prepend `vurvey`** — pass `["workspaces", "list"]`, not `["vurvey", "workspaces", "list"]`.
- **Blocked regardless of tier:** `login`, `logout`, `mcp`, `graphql`, `config set`, `config profile`. Use `vurvey_graphql_query` for GraphQL and `vurvey_environment_switch` / `vurvey_workspace_switch` for context changes. If the user needs to log in, tell them to run `vurvey login` in their own terminal.

### When a tool seems to be missing

If the user asks for something this guide describes but you can't see the tool, the most likely cause is a stale CLI binary — tools ship in CLI releases, not plugin releases, so an old binary exposes an old tool set.

Don't silently substitute a workaround. Check the version and say what you found:

```bash
vurvey --version
curl -s https://storage.googleapis.com/vurvey-cli-releases/latest
```

If the installed version is behind, tell the user to run `vurvey update` and then `/mcp restart vurvey`. The `/vurvey-update` command does this check for both the CLI and the plugin.

Two other causes worth ruling out before blaming the version: a delete-shaped tool is absent by design (see below), and `vurvey_graphql_introspect` is absent against production on purpose.

## Changing things

Create, update, run, schedule, and switch are available. Deleting is not.

Two different gates apply, so be precise about which surface you're on:

- **Dedicated tools:** the delete-shaped ones (`vurvey_workflow_schedules_delete`, `vurvey_workflow_variables_delete`, `vurvey_workflow_triggers_remove`, `vurvey_capabilities_remove_workflow`) are not registered unless `VURVEY_MCP_ALLOW_DESTRUCTIVE=1`. If you can't see a tool, it's off.
- **`vurvey_cli` escape hatch:** `delete`, `delete-many`, `revert`, `deactivate`, and `cancel` subcommands are refused at this tier even though the equivalent dedicated tool may exist.
- **`vurvey_graphql_query`:** `deleteX` / `removeX` / `destroyX` mutations are refused.

| Intent | Tools |
|---|---|
| Build a workflow | `vurvey_workflows_create`, `vurvey_workflows_create_auto`, `vurvey_workflows_create_from_template`, `vurvey_workflows_duplicate`, `vurvey_workflows_clone_from_history` |
| Change a workflow | `vurvey_workflows_update` |
| Control a run | `vurvey_workflows_run`, `vurvey_workflows_pause`, `vurvey_workflows_resume`, `vurvey_workflows_cancel` |
| Reports | `vurvey_workflows_report_regenerate`, `vurvey_workflows_report_update`, `vurvey_workflows_report_share` |
| Automate | `vurvey_workflow_schedules_create`, `vurvey_workflow_triggers_add`, `vurvey_workflow_triggers_update`, `vurvey_workflow_variables_create`, `vurvey_workflow_variables_activate` |
| Capabilities | `vurvey_capabilities_create`, `vurvey_capabilities_update`, `vurvey_capabilities_activate`, `vurvey_capabilities_quick_start`, `vurvey_capabilities_deploy_from_blueprint`, `vurvey_capabilities_add_workflow`, `vurvey_capabilities_run_workflow`, `vurvey_capabilities_set_schedule` |
| Chat | `vurvey_chat_send` |
| Context | `vurvey_workspace_switch`, `vurvey_environment_switch` (session-only; neither persists to the config file) |

**How to behave when writing:**

- **Confirm the target before acting.** Resolve the id first (`vurvey_surveys_find_by_name`, `vurvey_workflows_list`) and say which workspace you're in. Acting on the wrong workspace is the most likely real mistake.
- **Say what you're about to do** in one line before a create/update/run, especially when the user's phrasing was ambiguous.
- **Don't chain writes speculatively.** Do the one thing asked, report the result, then continue.
- **Check `vurvey_workflows_status` before starting a run** that may already be in flight.
- **A delete request is not a failure to route around.** Tell the user deletes are disabled and that they can enable them with `VURVEY_MCP_ALLOW_DESTRUCTIVE=1`, or do it in the web app. Do not attempt the same delete through `vurvey_cli` or `vurvey_graphql_query` — those are gated too, and working around a safety gate is not something to do on the user's behalf.

## Picking the right tool

- **Default to the specific tool** when one exists (`vurvey_surveys_list` over a hand-written `surveys { ... }` query). Specific tools return stable shapes and are schema-validated in the CLI's CI.
- **Use the composite tools to save round-trips.** `vurvey_workspace_overview` for orientation, `vurvey_surveys_find_by_name` instead of list-then-filter, `vurvey_responses_export` instead of looping.
- **Fall back to `vurvey_graphql_query`** for a field no dedicated tool covers, or to join across resources. *"Which surveys have more than 100 responses?"* is one query rather than many calls.
- **Brand market share is by NAME.** Confirm the exact name with `vurvey_brands_list` first if unsure.

## Common workflows

**"What's in my workspace?"** → `vurvey_workspace_overview`. One call; only drill down if the user asks.

**"Tell me about survey X"** → `vurvey_surveys_find_by_name` with the name, then `vurvey_surveys_get` with the returned id.

**"Analyze responses to survey X"** → `vurvey_surveys_get` for the questions, then `vurvey_responses_export` for the bulk response set. Summarize; never fabricate quotes.

**"Find what users said about <topic>"** → `vurvey_answers_search` with `query`, then `vurvey_answers_get` for full context on the interesting hits.

**"Why is this workflow not producing output?"** → `vurvey_workflows_status`, then `vurvey_workflows_history` for recent runs, then `vurvey_workflow_triggers_list` to check what should fire it. For capability pipelines, `vurvey_capabilities_pipeline_progress`.

**"Where did this chat answer come from?"** → `vurvey_chat_get` for the thread, then `vurvey_chat_message_grounding` for the cited sources.

**"What brands do we cover, and how do they stack up?"** → `vurvey_brands_list`, then `vurvey_brands_insights` per brand, then `vurvey_brands_market_share` passing the brand name.

## Troubleshooting

**Assume the user is not technical.** They are willing and capable of running whatever you give them, but they will not know what a `$PATH`, a config directory, or a Firebase project is, and they should not need to. So:

- Give **one copy-pasteable block at a time**, then ask what it printed. Do not hand over a six-step script and hope.
- Say what the command is for in plain words first ("this checks whether the Vurvey app is installed on your machine").
- **Never ask them to edit JSON by hand** unless there is no alternative, and if there is none, give them the complete file contents rather than a description of the edit.
- Do not explain the architecture unless they ask. They want it working.
- Tell them plainly when a step needs their terminal rather than this chat. They cannot always tell the difference.

### Diagnose in this order

Work top down and stop at the first thing that is wrong. Later symptoms are usually caused by earlier causes.

**1. Are the Vurvey tools loaded at all?**

If you cannot see any `vurvey_*` tools in this session, the problem is the plugin or the CLI, not the user's account. Ask them to run `/mcp` and tell you whether `vurvey` is listed and connected.

- Not listed at all → the plugin's components did not load. Go to [Reinstalling the plugin](#reinstalling-the-plugin).
- Listed but failed or disconnected → the CLI is missing or cannot start. Go to step 2.

**2. Is the CLI installed?**

The plugin is only configuration. It does not contain the Vurvey program, and it cannot install it. Have them run:

```bash
which vurvey
```

Empty output means it is not installed:

```bash
brew install Batterii/vurvey/vurvey
vurvey login
```

If `which vurvey` prints a path but the server still will not start, the client cannot see it on its `$PATH`. Have them use that exact path in the MCP config `command` field (commonly `/opt/homebrew/bin/vurvey`).

**3. Is the CLI new enough?**

Tools ship in CLI releases, not plugin releases, so a tool described in this guide can be missing simply because the binary is old. This is the single most common cause of "that tool doesn't exist".

```bash
vurvey --version
curl -s https://storage.googleapis.com/vurvey-cli-releases/latest
```

Behind → `vurvey update`, then `/mcp restart vurvey`. `/vurvey-update` runs this check for both the CLI and the plugin.

**4. Are they logged in?**

A tool returning *"not authenticated with Vurvey"* means there is no valid token.

- **Do not retry the tool in a loop.** It cannot recover on its own.
- Give them `vurvey login` and tell them it must be run in their **terminal**, not in this chat. You cannot log them in.
- Then `/mcp restart vurvey` so the server picks up the new token.
- `/vurvey-login` performs this check and reports back, so it is a good first thing to hand them.

Tokens refresh automatically, so a user who logged in weeks ago normally stays logged in. Re-authentication is usually needed only after a password change or a long gap.

### Environment problems (staging vs production)

Vurvey has three environments: **production** (`api.vurvey.app`), **staging** (`api-staging.vurvey.dev`), and **experimental** (`api-experimental.vurvey.dev`). They are genuinely separate systems with separate accounts and separate data. An account in one does not exist in the others.

Use `vurvey_environment_get` to see which one this session is pointed at, `vurvey_environments_list` for what is configured, and `vurvey_environment_switch` to change it. Switching works by **profile**: the target environment must already have a profile with a completed login, or the switch will fail with nothing to switch to.

To set up a new environment, the user runs this in their terminal, once per environment:

```bash
vurvey login --profile staging
```

**If they report `Access denied: Invalid or expired auth token`,** and especially if it happened right after a login that appeared to succeed, this is a known bug in CLI versions before the fix landed. Signing in with a `--api-url` override authenticated against the wrong environment's identity system, producing a token the target environment correctly refused. It looks like an expired token but nothing is expired.

The fix:

```bash
vurvey update
```

If they cannot update yet, this works on any version because it sets the environment before logging in rather than during:

```bash
vurvey login --profile staging
vurvey --profile staging config set api-url https://api-staging.vurvey.dev
```

Do not tell them to retry the same failing login, and do not tell them their account is expired or broken. It is not.

### Reinstalling the plugin

When components are missing or the plugin is behaving oddly, a clean reinstall is the reliable fix. A plain `/plugin install` is **not** enough: the marketplace listing is cached locally, so reinstalling can pull the same stale copy that caused the problem. The marketplace has to be removed and re-added.

Give these to the user one line at a time, in a Claude Code session:

```
/plugin marketplace remove vurvey
/plugin marketplace add Batterii/vurvey-claude-plugin
/plugin install vurvey
```

Then tell them to **start a new Claude Code session**. MCP servers connect at session start, so a newly installed server will not appear in the session they are currently in. `/reload-plugins` alone does not reconnect MCP servers.

To confirm it worked, have them run `/mcp` and look for `vurvey` as connected.

### Things that look broken but are not

Do not send the user chasing any of these:

- **`vurvey_graphql_introspect` is missing.** Production disables schema introspection, so the tool is not registered there. Expected. Do not send `__schema` or `__type` queries through `vurvey_graphql_query` either; they are rejected.
- **A delete tool is missing.** Deletes are gated off by design. See [Changing things](#changing-things).
- **"refusing to start against non-Vurvey host".** The configured API URL is not a Vurvey domain. This guard exists to stop credentials being sent somewhere they should not go. Have them run `vurvey config get api-url` and check it against the three environments above.
- **A tool returns an empty list.** Usually a real empty result or the wrong workspace, not a failure. Check `vurvey_workspace_info` and offer `vurvey_workspaces_list` before treating it as a bug.

### When you are genuinely stuck

Say so, and give them the diagnostic to send their engineering contact rather than guessing further:

```bash
vurvey --version && vurvey config get api-url && vurvey status -o json
```

The MCP server's own log is at `~/.config/vurvey/mcp.log`. Only protocol traffic goes to stdout, so real errors land there. Never invent a cause you have not confirmed, and never tell a user their data is gone or their account is broken on the basis of a tool error.

## What this skill doesn't do

- **No deletes.** See [Changing things](#changing-things). Point the user at the env var or the web app; don't route around the gate.
- **No login.** Auth happens in the user's terminal. You cannot log them in.
- **No subscriptions.** Real-time event streams are not exposed over MCP.
- **No file uploads.** Multipart uploads aren't wired through the server — direct users to the web UI for CSV/media.
- **No Vurvey engineering docs.** This server exposes workspace *data*. Questions about how the Vurvey platform is built are not answerable from these tools.

## Version + installation

- Requires the `vurvey` CLI on `$PATH`. v0.17.x exposes the 84-tool advanced surface described here; older binaries expose fewer.
- Install: `brew install Batterii/vurvey/vurvey`, then `vurvey login`. The plugin does not ship the binary.
- The server auto-refreshes Firebase tokens; users run `vurvey login` once per profile.
- The plugin is versioned in `plugins/vurvey/.claude-plugin/plugin.json`.
