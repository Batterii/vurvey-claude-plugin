---
description: Report which Vurvey workspace this connector is bound to and what it can do
---

Report the state of the Vurvey connection in one short answer. Do not lecture, and do not explain
MCP or OAuth unless asked.

## 1. Is the connector here?

If you cannot see any `vurvey_*` tools in this session, stop and say so. The connector is not
connected. Tell the user which one to do, matching their client:

- Claude Code: run `/mcp`, select `vurvey`, and choose **Authenticate**.
- Claude Desktop or claude.ai: open **Customize**, then **Connectors**, and click **Connect** on
  the Vurvey connector.

If `vurvey` is listed but will not connect, and Claude Code reports its URL as unset or invalid,
the plugin's **Workspace connector address** option is empty. That is the cause, and saying
"go and copy the address again" instead sends them in a circle. Tell them to run `/plugin manage`,
open the Vurvey plugin's options, paste the address in, and start a new session.

If `vurvey` is not listed at all, they have not added the address yet. Point them at
**Workspace settings**, then **Connected apps** in Vurvey, where the address and its **Copy**
button are.

## 2. Who and where

Call `vurvey_whoami` and `vurvey_workspace_info`. Report the account and the workspace name in one
line, so the user can tell at a glance whether it is the workspace they meant.

If a call is refused, do not retry it in a loop. A refusal here usually means the connection was
ended in Vurvey, or the user's membership or role in that workspace changed. Both take effect on
the very next request. Send them to **Connected apps** to check, and tell them approving again is
what restores it.

## 3. What it can do

Say what this connection can do, based on the tools you can actually see in this session, not on an
assumption:

- Only read tools visible: it can read this workspace and nothing else.
- Write tools such as `vurvey_workflows_create` or `vurvey_chat_send` visible: it can also create
  and edit content here.
- Run tools such as `vurvey_workflows_run` visible: it can also run workflows that already exist
  here.

Add one sentence either way: it can never delete anything, change billing, change who is in the
workspace, or change security settings, because no ability grants those.

If the user expected an ability that is missing, the cause is one of two things and it is worth
naming both: they did not approve it, or their role in that workspace cannot back it. A workspace
administrator can tell them which.

## 4. Close

If everything is working, say so in one line and suggest a next step, such as asking what is in the
workspace. If something is wrong, give them the single thing to do next and stop there.

Never tell the user to install anything, run a command in a terminal, log in, or restart a server.
The connector is hosted by Vurvey and none of those apply. `/mcp` and `/plugin manage` are the
exception, because they are Claude Code's own menus rather than anything on their machine.
