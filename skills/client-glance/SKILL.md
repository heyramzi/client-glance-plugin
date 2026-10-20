---
name: client-glance
description: Runs an agency's Client Glance workspace over its MCP tools. Use when someone asks to create a client portal, turn portal features on or off, give a client access, invite a teammate, or check how a client's portal or project looks.
---

# Client Glance

Client Glance gives an agency a password-protected portal for each client, fed by the project tools the agency already uses. Today the project source is ClickUp, and time can come from ClickUp or Everhour. More sources will follow, so check what the workspace has connected before you assume one. The `client-glance` MCP server exposes one tool per API route, named after the route with `/` and `-` turned into `_` (`portals/set-password` is `portals_set_password`).

The connection acts for one organization, the one the person picked when they signed in. Leave `organizationId` out: the tools fill it in from the connection. Call `users_me` first when you need the organization's name, plan or the person's role.

## Traps

- **A client never gets an invite.** Clients open their portal with its address and password. `invites_send` adds a *teammate* to the agency's workspace. When someone says "invite the client", they mean the portal link and password.
- **A portal sits on one project in the connected tool** (a ClickUp list or folder today). Its id is a Client Glance id like `k57...`, never the source tool's id.
- **Find a portal by name with `portals_list`** (or `portals_listActive`), then use its `_id`. Don't guess ids or slugs.

## Create a portal

`organizations_get` says which tools are connected (`clickupConnected`, `everhourConnected` and so on). The steps below are for ClickUp, the one project source today.

1. `integrations_clickup_verify`. If it says `connected: false`, stop: the person connects ClickUp in Settings (or gives you a `pk_` token for `integrations_clickup_set_token`).
2. Walk down to the client's list: `integrations_clickup_workspaces`, then `integrations_clickup_spaces` (workspaceId), then `integrations_clickup_space_hierarchy` (spaceId). Lists inside a folder come from `integrations_clickup_folder_lists` (folderId).
3. `portals_count` and `organizations_get` tell you whether the plan has room and give you the brand color and logo.
4. `portals_create` with:
   - `name` and `slug` (lowercase, dashes; the client's name usually works)
   - `clickupWorkspaceId`, `clickupListId`, `clickupListName`, `trackingScope: "list"` (or `clickupFolderId` with `trackingScope: "folder"`)
   - `branding`: `{ "primary_color": "<org brand color>", "font_family": "Geist, ui-sans-serif, system-ui", "thumbnail": "auto", "logo_url": "<org logo, if any>" }`
   - `portalIndicators`: start with `["progress", "waiting_on_client", "requests", "digest"]` unless the person asks for something else
   - `password` only if they gave one
5. `portals_openLink` (id) returns the portal's public address. If no password was set, `portals_set_password` (portalId) makes a strong one and returns it once.

Tell the person the address and the password, and that it's theirs to send to the client.

## Turn features on or off

`portals_setFeatures` takes the portal `id` plus only the switches that change:

| Ask | Arguments |
| --- | --- |
| Time tracking | `timeTrackingEnabled: true`, `timeSource: "clickup"` or `"everhour"` (Everhour needs `everhourProjectIds`) |
| Chat | `chatEnabled: true` |
| Client docs | `clientDocsEnabled: true`, `clientDocId` |
| Hours package | `packageBudgetEnabled: true`, `packageBudgetType`, `packageBudgetCapAmount`, `packageBudgetCapUnit` |
| Cards | `enableIndicators` / `disableIndicators`, e.g. `["requests"]` |

Before turning on Everhour time, check `integrations_everhour_projects` returns projects. For a ClickUp source, `integrations_clickup_capabilities` (workspaceId) says whether the plan allows time tracking.

Name, slug, branding, language and active state go through `portals_update`. If the portal belongs to a group (`portal_groups_list`, `portal_groups_listMembers`), a change to the group with `portal_groups_update` reaches every portal in it. Say that before you make it.

## Give a client access

1. Find the portal with `portals_list`.
2. `portals_openLink` for the address.
3. `portals_revealPassword` for the current password. If it returns null, offer `portals_set_password`, and warn that a new password locks out anyone using the old one.

## Invite a teammate

`invites_send` with `emails` and `role` (`member` by default, or `admin`). Only owners and admins can invite. `invites_list` shows who hasn't joined yet, and `invites_cancel` kills a link.

## Check a portal's status

- `portals_listDashboard` for a quick look across every portal.
- `portal_view_dashboard` (portalId, optional `period` like `30d`) returns what the client sees: tasks by status, what's waiting on them, time and the enabled cards.
- `portals_digest` returns the AI summary the client sees. `portals_regenerateDigest` writes a fresh one; it's throttled, so a second call right after returns the same summary.
- `portal_view_report` (period `7d` or `30d`) gives the time report.
- `analytics_trend` (sinceMs) and `analytics_visitCounts` show whether clients actually open their portals.

## Ask first

Tools marked destructive overwrite or delete something: `portals_remove`, `portals_set_password`, `portal_groups_update`, `users_removeFromOrganization`, `organizations_clearClickUpToken` and the rest. Say what will change and get a yes before you call one. Posting tools (`portal_view_comments_post`, `portal_view_chat_post`, `portal_view_requests_submit`) write to the connected project tool under the agency's account, prefixed with the portal name, so the client and the team both see it.
