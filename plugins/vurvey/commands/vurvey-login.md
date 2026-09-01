---
description: Check Vurvey auth state and guide the user through login if needed
---

Check the user's Vurvey auth state by calling the `vurvey_whoami` tool.

**If it returns a successful result** (email, id, name), check the workspace next. Logging in does not select one, and almost every other tool is workspace-scoped, so this is the step that decides whether the next question works. Call `vurvey_environment_get` and read `workspace_id`.

- **`workspace_id` is set:** tell the user which account they're authenticated as and which workspace they're in (`vurvey_workspace_info` for the name). Confirm tools are working and suggest a next step like "try asking: list my surveys".
- **`workspace_id` is empty:** do not suggest listing surveys, because that call is what fails. Say the login worked and one thing is left. Call `vurvey_workspaces_list`, which works with no workspace selected, and show them the options. Then give them this to run in their own terminal, with the id filled in:

  ```
  vurvey workspaces use <id>
  ```

  Followed by `/mcp restart vurvey`, because the server reads the workspace once at startup. You cannot do this for them: `vurvey_workspace_switch` is an `advanced` tool and is not registered at the tier this plugin ships, and `vurvey_cli` refuses `workspaces use` as a non-read command.

If a call fails with `Variable "$workspaceId" got invalid value ""`, that is the same missing selection. Nothing is broken, and the wrapper text suggesting a schema problem is wrong for this case. Hand them the two commands above rather than repeating the error.

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

- End by telling them to run `/vurvey-login` again once they're back, so the workspace check above runs before they ask a real question.

**If it returns any other error** (network, 5xx, non-Vurvey host):
- Surface the error verbatim to the user.
- Suggest checking `~/.config/vurvey/mcp.log` for server-side diagnostics.

Keep the response short — don't lecture or explain the whole MCP protocol. The user just wants to know if they're authenticated.
