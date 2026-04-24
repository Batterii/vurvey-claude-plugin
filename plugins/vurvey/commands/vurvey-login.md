---
description: Check Vurvey auth state and guide the user through login if needed
---

Check the user's Vurvey auth state by calling the `vurvey_whoami` tool.

**If it returns a successful result** (email, id, name):
- Tell the user which account they're authenticated as (email + workspace name if `vurvey_workspace_info` is quick).
- Confirm tools are working. Suggest a next step like "try asking: list my surveys".

**If it returns an error containing "not authenticated" or "vurvey login"**:
- Do NOT retry the tool.
- Tell the user verbatim what to do, with the exact copy-pastable command in a code block:

  ```
  vurvey login
  ```

  If they mentioned a profile earlier in the conversation (e.g. "staging"), use `vurvey login --profile staging` instead.

- Then tell them to restart the MCP server so it picks up the refreshed token. The command depends on their client:
  - Claude Code: `/mcp restart vurvey`
  - Claude Desktop / Cursor / Codex: quit and relaunch the app (or the conversation, for Codex).

- End with: "Once that's done, ask me to list your Vurvey surveys and I'll confirm it works."

**If it returns any other error** (network, 5xx, non-Vurvey host):
- Surface the error verbatim to the user.
- Suggest checking `~/.config/vurvey/mcp.log` for server-side diagnostics.

Keep the response short — don't lecture or explain the whole MCP protocol. The user just wants to know if they're authenticated.
