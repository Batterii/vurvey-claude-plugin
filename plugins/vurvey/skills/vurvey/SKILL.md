---
name: vurvey
description: Use when the user asks about Vurvey surveys, questions, answers, workflows, personas, brands, clips, files, or any data in their Vurvey workspace. Triggers on phrases like "check my surveys", "list surveys", "show workflows", "find brand insights", "search answers", "vurvey personas", "analyze survey responses", "what's in my Vurvey workspace", or "run a Vurvey GraphQL query". Only activates when the vurvey MCP server is available.
---

# Vurvey Workflow Guide

The `vurvey` MCP server exposes the Vurvey platform API as structured tools. Use this skill to understand **when to reach for which tool** and **how to chain them** for common Vurvey research workflows.

## Available tools (core tier, 25 tools)

### Identity & workspace

| Tool | Purpose |
|---|---|
| `vurvey_whoami` | Confirm the authenticated user. Use first if auth state is unclear. |
| `vurvey_workspace_info` | Active workspace metadata (name, plan, enabled features). |

### Surveys

| Tool | Purpose |
|---|---|
| `vurvey_surveys_list` | List surveys. Optional filters: `status` (`DRAFT`, `OPEN`, `CLOSED`, `ARCHIVED`), `name` (substring), `limit`, `cursor`. |
| `vurvey_surveys_get` | Fetch a single survey by id, including questions and response count. |

### Questions & answers

| Tool | Purpose |
|---|---|
| `vurvey_questions_list` | List questions in a survey. Requires `survey_id`. |
| `vurvey_questions_get` | Fetch a single question with its choices. Requires `id`. |
| `vurvey_answers_list` | List answers for a question. Requires `question_id`. |
| `vurvey_answers_get` | Fetch a single answer. Requires `id`. |
| `vurvey_answers_search` | Free-text search across answers in the workspace. Optional `query` and `limit`. |

### Workflows (orchestrations)

| Tool | Purpose |
|---|---|
| `vurvey_workflows_list` | List workflows in the workspace. |
| `vurvey_workflows_get` | Fetch a single workflow definition. Requires `id`. |
| `vurvey_workflows_history` | Run history for a workflow. Requires `id`. |

### Personas

| Tool | Purpose |
|---|---|
| `vurvey_personas_list` | List AI personas in the workspace. |
| `vurvey_personas_get` | Fetch a single persona. Requires `id`. |
| `vurvey_personas_members` | List the member accounts attached to a persona. Requires `id`. |

### Brands

| Tool | Purpose |
|---|---|
| `vurvey_brands_list` | List brands in the workspace. |
| `vurvey_brands_get` | Fetch a single brand. Requires `id`. |
| `vurvey_brands_insights` | Pull the insights report for a brand. Requires `id`. |
| `vurvey_brands_market_share` | Market-share data for a brand by **name** (not id). Requires `name`. |

### Media

| Tool | Purpose |
|---|---|
| `vurvey_clips_list` | List video clips for a survey. Requires `survey_id`. |
| `vurvey_clips_get` | Fetch a single clip. Requires `id`. |
| `vurvey_files_get` | Fetch file metadata. Requires `id`. |
| `vurvey_file_tags_list` | List file-tag keys in the workspace. |

### GraphQL escape hatch

| Tool | Purpose |
|---|---|
| `vurvey_graphql_query` | Arbitrary **read-only** GraphQL query. Mutations and subscriptions are rejected. |
| `vurvey_graphql_introspect` | Full schema introspection. Use when you need to discover a query or type that isn't exposed as a dedicated tool. |

## Picking the right tool

- **Default to the specific tool** when one exists (`vurvey_surveys_list` over a custom `surveys { ... }` graphql query). Specific tools return more stable shapes and are schema-validated in the CLI's CI.
- **Fall back to `vurvey_graphql_query`** when the user wants a field that isn't covered by a specific tool, or when they want to join across resources. Example: *"Which surveys have more than 100 responses?"* is easier as one GraphQL query than many round-trips.
- **Use `vurvey_graphql_introspect` first** if you don't know what's in the schema. It's cheap and self-documenting.
- **Brand market share is by NAME, not id.** If the user asks for market share, use `vurvey_brands_list` first if needed to confirm the exact brand name, then pass that to `vurvey_brands_market_share`.

## Common workflows

**"List my surveys" / "Show open surveys":**
1. `vurvey_surveys_list` with no args (default: all surveys, up to limit).
2. If the user asked for open ones specifically, pass `status: "OPEN"`.

**"Tell me about survey X":**
1. If the user gave an id, `vurvey_surveys_get` with `id: <guid>`.
2. If they gave a name, `vurvey_surveys_list` with `name: <substring>` to find the id, then `vurvey_surveys_get`.

**"Analyze responses to survey X":**
1. `vurvey_surveys_get` to pull the survey + its questions.
2. For each question, `vurvey_answers_list` with `question_id: <qid>` to pull answers.
3. Summarize or compare the answer set; never fabricate quotes.

**"Find what users said about <topic>":**
1. `vurvey_answers_search` with `query: "<topic>"` for a workspace-wide search. Pass `limit` if the workspace is large.
2. For each interesting answer, optionally pull the full context via `vurvey_answers_get`.

**"What workflows are running in this workspace?":**
1. `vurvey_workflows_list` for the inventory.
2. For a specific one, `vurvey_workflows_get` with `id` for its definition, or `vurvey_workflows_history` to see how it's been performing.

**"Show me the personas in this workspace":**
1. `vurvey_personas_list`.
2. If the user asks "who has access to persona X?", chain to `vurvey_personas_members` with the id.

**"What brands do we cover, and how do they stack up?":**
1. `vurvey_brands_list` for the inventory.
2. `vurvey_brands_insights` per brand for detailed insights.
3. `vurvey_brands_market_share` passing the brand **name** for market-share data.

**"What's in my workspace?" / diagnostic:**
1. `vurvey_whoami` → confirms identity.
2. `vurvey_workspace_info` → workspace features + plan.
3. `vurvey_surveys_list` with `limit: 10` → quick inventory.

## Authentication handling

If a tool returns an error like *"not authenticated with Vurvey. In your terminal, run: vurvey login"*:

- **Don't try to retry in a loop** — the tool can't recover on its own.
- **Surface the instruction to the user verbatim.** They need to run the command in a shell outside Claude.
- If they're using multi-profile setups, suggest `vurvey login --profile <name>` matching the profile their MCP server was started with.
- After login, Claude Code users should run `/mcp restart vurvey` so the server picks up the refreshed config.

## What this skill doesn't do

- **No mutations.** The core tier is read-only. If the user asks to create, update, or delete something in Vurvey, tell them to use the `vurvey` CLI directly (e.g. `vurvey surveys create ...`, `vurvey personas update ...`) or the web UI. Advanced / destructive tiers ship in a future release.
- **No subscriptions.** Real-time event streams (chat, events) are not exposed through the core MCP surface.
- **No file uploads.** Multipart uploads aren't wired through the MCP server — direct users to the web UI for CSV/media uploads.

## Version + installation

- Requires the `vurvey` CLI (v0.8.0+) on `$PATH` for the full 25-tool surface. v0.7.0 shipped the walking-skeleton 6 tools; v0.8.0 added the remaining 19.
- Install: `brew install Batterii/vurvey/vurvey`
- The MCP server auto-refreshes Firebase tokens; users only need to run `vurvey login` once per profile.
- The plugin itself is versioned in `plugins/vurvey/.claude-plugin/plugin.json`.
