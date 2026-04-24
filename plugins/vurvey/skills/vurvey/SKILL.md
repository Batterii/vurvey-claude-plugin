---
name: vurvey
description: Use when the user asks about Vurvey surveys, workflows, personas, brands, or any data in their Vurvey workspace. Triggers on phrases like "check my surveys", "list surveys", "vurvey workspace", "analyze survey responses", "what's in my Vurvey workspace", or "run a Vurvey GraphQL query". Only activates when the vurvey MCP server is available.
---

# Vurvey Workflow Guide

The `vurvey` MCP server exposes the Vurvey platform API as structured tools. Use this skill to understand **when to reach for which tool** and **how to chain them** for common Vurvey research workflows.

## Available tools (core tier)

| Tool | Purpose |
|---|---|
| `vurvey_whoami` | Confirm the authenticated user. Use first if auth state is unclear. |
| `vurvey_workspace_info` | Active workspace metadata (name, plan, enabled features). |
| `vurvey_surveys_list` | List surveys. Optional filters: `status` (`DRAFT`, `OPEN`, `CLOSED`, `ARCHIVED`), `name` (substring), `limit`, `cursor`. |
| `vurvey_surveys_get` | Fetch a single survey by id, including questions and response count. |
| `vurvey_graphql_query` | Escape hatch — run an arbitrary read-only GraphQL query. Mutations and subscriptions are rejected. |
| `vurvey_graphql_introspect` | Full schema introspection. Use when you need to discover a query or type that isn't exposed as a dedicated tool. |

## Picking the right tool

- **Default to the specific tool** when one exists (`vurvey_surveys_list` over a custom `surveys { ... }` graphql query). Specific tools return more stable shapes and are schema-validated in the CLI's CI.
- **Fall back to `vurvey_graphql_query`** when the user wants a field that isn't covered by a specific tool, or when they want to join across resources. Example: *"Which surveys have more than 100 responses?"* is easier as one GraphQL query than two round-trips.
- **Use `vurvey_graphql_introspect` first** if you don't know what's in the schema. It's cheap and self-documenting.

## Common workflows

**"List my surveys" / "Show open surveys":**
1. `vurvey_surveys_list` with no args (default: all surveys, up to limit).
2. If the user asked for open ones specifically, pass `status: "OPEN"`.

**"Tell me about survey X":**
1. If the user gave an id, `vurvey_surveys_get` with `id: <guid>`.
2. If they gave a name, `vurvey_surveys_list` with `name: <substring>` to find the id, then `vurvey_surveys_get`.

**"Analyze responses to survey X":**
1. `vurvey_surveys_get` to pull the survey + its questions.
2. For each question, use `vurvey_graphql_query` to pull answers — example query: `query($qid: GUID!) { answers(questionId: $qid, limit: 100) { text responderId sentiment } }` (use `vurvey_graphql_introspect` first to confirm the real field names).
3. Summarize or compare the answer set; never fabricate quotes.

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

- **No mutations.** The core tier is read-only. If the user asks to create, update, or delete something in Vurvey, tell them to use the `vurvey` CLI directly (`vurvey surveys create ...`) or the web UI.
- **No subscriptions.** Real-time event streams (chat, events) are not exposed through the core MCP surface.
- **No file uploads.** Multipart uploads aren't wired through the MCP server — direct them to the web UI for CSV/media uploads.

## Version + installation

- Requires the `vurvey` CLI (v0.7.0+) on `$PATH`. See the plugin [README](../../../README.md) for install paths.
- The MCP server auto-refreshes Firebase tokens; users only need to run `vurvey login` once per profile.
- The plugin itself is versioned in `plugins/vurvey/.claude-plugin/plugin.json`.
