# Vurvey for Claude

Ask Claude about your Vurvey workspace in plain English.

> *"What campaigns do we have open?"*
> *"Pull the responses to our Q1 research campaign and summarize the main themes."*
> *"Run the weekly insights workflow and tell me when it finishes."*

There is no Vurvey software to install: no binary, no package manager, no login on your machine.
Vurvey hosts the connector. You copy one address out of Vurvey, add it to Claude, and approve it
once.

---

## Before you start

You need a Vurvey account, and the workspace you want to connect needs the Claude connector turned
on. If it is off, the approval screen says so and tells you to ask your Vurvey contact. Nothing on
this page will work until it is on.

---

## Install

Three steps, the same three for every client. What differs between clients is where you click.

### 1. Copy your workspace address

In Vurvey, open **Workspace settings**, then **Connected apps**. The page shows the connector
address for the workspace you are in, with a **Copy** button next to it.

The address is per workspace. Copying it from a different workspace connects a different workspace,
so copy it from the one you actually want Claude working in.

If that page says connecting Claude is not available in this Vurvey environment yet, there is no
connector for you to add and the rest of this page does not apply.

### 2. Add the address to Claude

Pick your client below.

#### Claude Desktop and claude.ai

These take the address directly, as a custom connector. They do not take this plugin, which is a
Claude Code artifact, so there is nothing to install here.

On Pro or Max:

1. Go to **Customize**, then **Connectors**.
2. Click **+**, then **Add custom connector**.
3. Paste the address into the remote MCP server URL field.
4. Give it a name that says which workspace it is, such as `Vurvey - Acme Q1`. The address ends in
   a workspace id, so the name is the only thing that will tell two Vurvey connectors apart later.
5. Leave **Advanced settings** alone. The OAuth client ID and secret there are for servers that
   cannot register a client themselves. Vurvey can, so those fields stay empty.
6. Click **Add**.
7. In a conversation, turn it on with the **+** button, then **Connectors**.

On Team or Enterprise an owner adds it once for the whole organization, under **Organization
settings**, **Connectors**, **Add**, hover **Custom**, then **Web**. After that each member goes to
their own **Customize**, **Connectors**, finds it, and clicks **Connect**.

Free accounts can hold one custom connector at a time.

#### Claude Code

Claude Code takes this plugin, which brings a skill teaching Claude which Vurvey tool to reach for
and how to chain them.

```
/plugin marketplace add Batterii/vurvey-claude-plugin
/plugin install vurvey
```

Enabling it asks you for one thing, **Workspace connector address**. Paste in the address you
copied. Nothing else to set, no file to edit, no terminal command. To change it later, or to check
what it is holding, run `/plugin manage` and open the Vurvey plugin's options.

Then start a new session. Claude Code connects plugin servers at session start.

#### Connecting more than one workspace

An address names one workspace, so a second workspace means a second connector rather than a
setting to change. In Claude Desktop and claude.ai, add another custom connector. In Claude Code,
the plugin holds one address; add each further workspace by hand, under a name of your own
choosing:

```bash
claude mcp add --transport http vurvey-acme "paste-the-other-workspace-address-here"
```

**Name each one after its workspace.** Three connectors all called `vurvey` are three entries in
`/mcp` you cannot tell apart, because their addresses differ only in a workspace id. Each connector
is approved separately in Vurvey and Claude sees each as its own set of tools. The plugin's skill
covers all of them.

### 3. Approve it in Vurvey

In Claude Code, run `/mcp`, select **vurvey**, and choose **Authenticate**. In Claude Desktop and
claude.ai, adding the connector starts this by itself, and a member on a Team or Enterprise plan
starts it with the **Connect** button on the connector their owner added.

Your browser opens on Vurvey. Sign in if you are not already, then read the approval screen. It
tells you which workspace is being connected, what the application will be able to do, and what it
can never do. Approve it, and you are done.

**Read the hosts on that screen.** Any application can register with Vurvey and name itself
whatever it likes, so the screen deliberately shows you the web address the application actually
uses rather than the name it typed in for itself. If the address is not one you expect, do not
approve it.

Then just ask a question.

---

## What Claude can do

Approving grants abilities, and each one is shown on the approval screen as a sentence rather than
as a code. There are three, and that is the entire vocabulary. Nothing outside it can be asked for,
which is why the list below of what is impossible is short and absolute.

| Ability | What it means |
|---|---|
| **Read this workspace** | Claude can read what your role can already see here, which may include campaigns, survey responses, datasets, agents and workflows. |
| **Create and edit content here** | Claude can draft and change workflows and capabilities here, along with their schedules, triggers and variables, and send messages in Vurvey chat. Campaigns, agents and datasets are read-only through the connector, however editable they are in the web app. |
| **Run workflows that already exist here** | Claude can run a workflow this workspace already has and read what it produces. Running is separate from creating: this ability alone cannot make a new workflow. |

**Your own role is the ceiling.** Claude acts as you and never beyond what your role in this
workspace already allows. A Guest can only ever grant the read ability, whatever the application
asked for. If your role changes, or you are removed from the workspace, the connector is cut back
to the new ceiling on its very next request, with no waiting for a token to expire.

**Approving in one workspace grants nothing in any other.** The address names one workspace, the
access token is bound to that exact address, and the token is refused anywhere else.

### What is not possible at all

These are not defaults you can change. There is no ability that expresses them, so approving
everything on the screen still cannot reach them:

- **Delete anything.** Not a campaign, a response, a dataset, an agent or a workflow.
- **Change billing**, your plan, or any payment details.
- **Add or remove people**, invite anyone, or change what someone's role can do.
- **Change single sign-on** or any other workspace security setting.

---

## Where your login lives now

There is no Vurvey login on your machine, and Claude never sees your Vurvey password. You type it
into Vurvey's own sign-in page in your own browser, the same as when you use the web app.

What approving creates is a record inside Vurvey, on your account, for that one workspace. Your
Claude client holds an access token, and that token is worth nothing on its own: Vurvey reads the
record behind it on **every single request**, with nothing cached in front of it. That is what
makes ending the connection immediate rather than eventual, and it is why the token sitting in your
client is not the thing you have to go and clean up.

Vurvey keeps a record of every call the connector makes, including the ones it refuses, and rate
limits a connector that calls too fast.

---

## Writes are previewed before they run

Anything that changes something, or runs a workflow, takes two calls rather than one.

The first call **executes nothing**. Vurvey answers with a preview naming the tool, the workspace
and the call's short arguments, plus a token bound to that exact call: that grant, that tool, those
exact arguments. The second call carries the token back, and Vurvey rebuilds the binding from the
arguments it is handed the second time. If anything changed in between, the binding does not match
and the call is refused. The token is single use and cannot be spent in another workspace.

**The preview is a description of the call, not a copy of it.** Only short scalars come back
verbatim. A string over 120 characters is shown as its shape, `<string:512 chars>`, and anything
structured is shown as `<structured value, not shown>`. That is deliberate: the preview is built
by the same reduction as Vurvey's audit record, so respondent free text and a report password
cannot be copied into either by a write that carries them. The binding is over the **full**
arguments, so the call that runs is byte for byte the call that was proposed, but if you want to
know what is inside a value the preview did not print, ask Claude to say what it is about to send.

**Here is the honest limit of that.** Both calls come from the same caller, so a model can call the
preview and send the confirmation back in the same turn without you seeing either one. What the two
phases actually buy you is that a write cannot happen by accident on the way past, that a preview
of one thing can never authorize a different thing, and that both halves land in Vurvey's audit
record. It is a cost and a receipt. It is not a promise that a person looked at it.

**Do not read your client's approval prompt as the missing guarantee either.** Claude clients do
prompt before a tool call by default, but ordinary settings switch that off:

- a permission rule that allowlists the Vurvey tools. Installed as this plugin they are named
  `mcp__plugin_vurvey_vurvey__*`; added by hand as a server called `vurvey` they are
  `mcp__vurvey__*`.
- `"defaultMode": "bypassPermissions"` in Claude Code settings
- launching Claude Code with `--dangerously-skip-permissions`
- the equivalent auto-run mode in another client

That prompt lives in the client, not in Vurvey, so Vurvey cannot promise you anything about it. If
you want a hard limit on writes, the one Vurvey can actually keep is your role and the abilities you
approved.

---

## Your workspace data goes to Anthropic

Anything Claude reads from this workspace becomes part of your Claude conversation and is sent to
Anthropic, exactly like text you paste in yourself. That is how Claude answers from it.

That includes respondent free text in survey answers and responses, and workspace member names
attached to an agent. Treat a question that pulls data the way you would treat pasting that data
into the chat window.

Only connect a workspace whose contents you are willing to share with Anthropic.

Two things that do **not** happen: Claude cannot see anything your own Vurvey account cannot see,
and your Vurvey password never reaches it.

---

## Ending the connection

**Workspace settings**, then **Connected apps**, in Vurvey. Every connection anyone in the
workspace has approved is listed there, with who approved it, what it can do, and when it was last
used. A workspace administrator can end any of them; if you are not one, ask yours.

Claude stops on its very next request to the workspace. There is no cache to wait out.

What that does not do is reach backwards. Anything Claude already read is still sitting in the
Claude conversations it was copied into, and disconnecting does not touch those.

Reconnecting means approving again from scratch.

---

## Troubleshooting

| What you see | What is wrong | Fix |
|---|---|---|
| `URL is unset or invalid, open /plugin manage and configure vurvey options` | The plugin's **Workspace connector address** option is empty | `/plugin manage`, open the Vurvey plugin's options, paste the address, start a new session |
| `MCP server "vurvey" has a "url" but no "type"` | A hand-written config entry is missing `"type": "http"` | Use the entry from this plugin, or `claude mcp add --transport http` |
| The server is listed but never connects | You have not approved it yet | `/mcp`, select `vurvey`, **Authenticate** |
| The approval screen says the workspace does not have the connector turned on | It is an entitlement on the workspace, not something you can switch on yourself | Ask your Vurvey contact |
| The approval screen says your role does not allow anything that was asked for | Your role in that workspace has no ability the application requested | Ask a workspace administrator about your role |
| Approving worked, then the browser failed to hand the code back | The local callback did not complete | Paste the full callback URL from your address bar into the prompt Claude Code shows |
| Calls suddenly refused, nothing changed on your side | The connection was ended in Vurvey, or your membership or role changed | Check **Connected apps**, then approve again if you should still have it |
| Answers are about the wrong workspace | The address you added belongs to another workspace | Add the workspace you want as a second connector under its own name, see [Connecting more than one workspace](#connecting-more-than-one-workspace) |
| No Vurvey tools at all in Claude Code | The plugin's marketplace listing is cached | `/plugin marketplace update Batterii/vurvey-claude-plugin`, `/plugin update vurvey`, then a new session |

To sign out of the connector on the client side, use **Clear authentication** in Claude Code's
`/mcp` menu, or `claude mcp logout vurvey`. That drops the token your client is holding. It does
not end the grant in Vurvey, so use **Connected apps** if that is what you meant.

---

## Staff and local development

**Customers do not need any of this.** This section is for Vurvey staff and for anyone developing
against the platform locally.

The `vurvey` CLI still runs an MCP server over stdio against whichever environment your profile
points at, which is how you exercise the tool surface without the hosted connector in front of it:

```bash
vurvey login
vurvey workspaces use <id>
vurvey mcp serve
```

`vurvey mcp install <client>` writes that entry into a client config for you. The server defaults
to the `core` tier, which is queries only; `VURVEY_MCP_TIER=advanced` adds writes and
`VURVEY_MCP_ALLOW_DESTRUCTIVE=1` on top of that adds the delete tools. `VURVEY_MCP_READ_ONLY=1`
pins it read-only whatever the tier says. The local escape hatches, the `vurvey_cli` subprocess
tool and GraphQL mutations, are off unless `VURVEY_MCP_UNSAFE_LOCAL=1` is set on the server
process.

None of that is reachable through the hosted connector, and the difference is deliberate rather
than incidental. The hosted build has no `vurvey_cli`, no `vurvey_graphql_query` and no
`vurvey_graphql_introspect`, because a subprocess runs in the host's environment and an
agent-written GraphQL document is the one thing Vurvey cannot resolve to a known operation and
re-authorize. It has no environment or workspace-switching tools either, because a hosted request
is bound to one workspace by its token and has no business changing the host's session. Everything
it can run is a fixed, named operation that Vurvey re-authorizes against your live grant.

The CLI source, including the hosted server, is in `Batterii/vurvey-cli` (private, Vurvey staff and
collaborators).

---

## Contributing

Issues and PRs welcome here.

## License

MIT, see [LICENSE](LICENSE).
