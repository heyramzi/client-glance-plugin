# Client Glance for Claude Code

Run your agency's Client Glance workspace (clientglance.com) from Claude Code: create a portal for a client, turn features like time tracking or chat on, hand a client their link and password, invite a teammate, and check what a client sees before a call.

The plugin adds the Client Glance MCP server (`https://clientglance.com/mcp`) and one skill that knows the common agency workflows.

## Install

```
/plugin marketplace add heyramzi/client-glance-plugin
/plugin install client-glance@client-glance
```

Then run `/mcp`, pick `client-glance` and choose Authenticate. Your browser opens clientglance.com: sign in with the email you use for Client Glance, pick the workspace and press Allow.

You need a Client Glance account with your project tool connected. Today that's ClickUp, with Everhour as an option for time. Start one at [clientglance.com](https://clientglance.com).

## Try

- "Which client portals are active?"
- "Create a client portal for Acme."
- "Turn on time tracking for the Acme portal."
- "Send me the Acme portal's link and password."
- "Invite sam@acme.com to our workspace as a member."

## Access

Only the workspace you picked when you signed in. Tools that change or delete something are marked destructive, so Claude asks first. Disconnect any time under Settings, Connected apps.

## Help

[clientglance.com/support](https://clientglance.com/support) or hello@clientglance.com. Privacy: [clientglance.com/privacy](https://clientglance.com/privacy).
