---
name: infinity-mcp
description: Use this skill whenever the user asks to read, search, list, create, update, comment on, or organize StartInfinity / Infinity workspaces, boards, folders, items, subitems, attributes, or comments through the Infinity MCP server. Use it especially for requests involving Infinity item URLs, "add a comment", "change status", board/folder trees, item fields, ChatGPT custom app OAuth, or deciding which Infinity MCP tool is safe to call.
---

# Infinity MCP Workflow

Use the Infinity MCP server when the user asks to inspect, create, update, or archive StartInfinity workspaces, boards, folders, items, or subitems.

## Production Endpoint

Known production MCP endpoint:

```txt
https://infinity.aimastersdata.com/mcp
```

Transport:

```txt
streamable HTTP
```

Use the static Codex/admin MCP API key for full tool access. ChatGPT custom app OAuth may intentionally expose a smaller safe tool set.

## Client Surfaces

There are two intentional tool surfaces:

- Static Codex/admin API key: full read, write, admin, and setup tool set.
- ChatGPT custom app OAuth: restricted read/navigation tools plus `infinity_add_item_comment`.

If an OAuth client cannot see a write/update tool, that may be deliberate. Do not work around the allowlist by guessing another write path.

## Agent-Led Cloud Docker Install

Use this workflow when the user asks an agent to install or configure Infinity MCP on a cloud Docker server.

First get or use this skill repository:

```txt
https://github.com/martinjokub/infinity-agent-skills.git
```

Then install the MCP server from:

```txt
https://github.com/martinjokub/infinity-mcp-server.git
```

Ask the user for these values if they are not already provided:

- SSH target for the cloud server, for example `root@example.com`.
- Install folder on the server. The user controls this path; do not invent one silently.
- Infinity developer token from `https://app.startinfinity.com/profile/developer/tokens`.
- MCP user name, default `codex`.
- Access level: `read-only`, `read-write`, or `admin`.
- MCP URL the client should use, preferably an HTTPS reverse-proxy URL.

Run setup commands on the cloud server, outside Docker, in the chosen install folder. Do not ask the user to enter the Docker container for setup.

Create or update:

```txt
docker-compose.yml
.env
config/mcp-users.json
data/credentials.enc.json
config/<mcp-user>-mcp-key.txt
```

Store the Infinity token only in the encrypted credential store. Do not expose it in Docker environment variables. The MCP API key is what Codex or another MCP client uses to call `/mcp`.

After setup, test:

1. `/health` works.
2. `/mcp` fails without `Authorization`.
3. `/mcp` works with `Authorization: Bearer <generated MCP API key>`.
4. `infinity_get_profile` works.

Then configure the MCP client:

```txt
name: infinity
transport: streamable HTTP
url: <MCP URL>
authorization header: Bearer <generated MCP API key>
```

Report back the install folder, repo commit used, Docker Compose service, created files, MCP URL, key file path, test proof, and MCP client config location.

## Endpoint Coverage Truth

Do not spend time looking for unsupported board/workspace CRUD unless the user explicitly asks for a fresh API audit.

Documented Morava coverage:

- Workspaces: list only.
- Workspace members: list, invite, add, remove.
- Boards: list, get, create only. There is no documented board update/delete endpoint.
- Folders: list, get, create, update, delete/archive.
- Attributes: list, get, create, update, delete.
- Items: list, get, create, update, delete/archive.
- Subitems: handled as items with `parent_id`.

Other documented areas exist but are outside the core board/folder/item workflow: comments, attachments, views, references, hooks, and time tracking.

Updated practical coverage:

- Comments: list, get, add/create, update, delete.
- Attachments, views, references, hooks, and time tracking remain outside the current core MCP workflow unless the user asks to add those tools.

## Local Server

Use the Infinity MCP endpoint configured for the current machine. If using the companion MCP server's example Docker setup, the default local endpoint is:

```txt
http://127.0.0.1:3015/mcp
```

Health endpoint:

```txt
http://127.0.0.1:3015/health
```

HTTP MCP requests require:

```txt
Authorization: Bearer <mcp-api-key>
```

The MCP API key is not an Infinity token. It authorizes the MCP client to use one server-side encrypted Infinity credential profile. On this local machine, the generated key may be stored in the local ignored file `D:\AI\infinity\mcp\config\local-client-key.txt`; otherwise use the agent/client's configured secret source.

If the user asks to use Infinity data, prefer this HTTP MCP endpoint instead of calling the Infinity API directly. It already handles MCP auth, Infinity credentials, API version headers, pagination shape, and Infinity value payload conversion.

If this endpoint is different on the current machine, use the local agent/client config instead of assuming a hard-coded path or personal directory.

## Auth And Scopes

Do not ask the user for an Infinity token during normal tool use. The MCP server resolves the caller's MCP API key to an encrypted Infinity credential profile.

Tool access is scope-gated:

- `infinity:read`: profile, list, and get tools.
- `infinity:write`: create and update tools.
- `infinity:admin`: archive/delete and workspace member management tools.

If a tool returns a missing-scope error, explain which MCP scope is needed. Do not retry the same write/destructive tool with guessed credentials.

## Discovery Order

For most tasks, use this order:

1. `infinity_list_workspaces`
2. `infinity_list_boards`
3. `infinity_list_folders`
4. `infinity_list_attributes`
5. Item or folder operation

Do not guess board, folder, item, or attribute IDs if they can be discovered with read-only tools.

For an Infinity item URL like:

```txt
https://app.startinfinity.com/b/<board-slug>/<folder-id>/<item-id>?...
```

resolve the workspace ID separately. The first path segment after `/b/` is usually a board ID/slug, not a workspace ID.

## Tool Router

Route user intent directly:

- "Show my workspaces" -> `infinity_list_workspaces`
- "Who has access?" / "members" -> `infinity_list_workspace_members`
- "Invite/add/remove workspace member" -> `infinity_invite_workspace_member`, `infinity_add_workspace_member`, or `infinity_remove_workspace_member`
- "Show boards" -> `infinity_list_boards`
- "Create board" -> `infinity_create_board`
- "Update/rename/delete board" -> explain that Morava docs do not expose board update/delete; suggest creating a new board or using Infinity UI
- "Show folders/tree" -> `infinity_list_folders`, then nest by `parent_id`
- "Create/update/delete folder" -> `infinity_create_folder`, `infinity_update_folder`, `infinity_archive_folder`
- "Show fields/columns" -> `infinity_list_attributes`
- "Create/update/delete field/column" -> `infinity_create_attribute`, `infinity_update_attribute`, `infinity_delete_attribute`
- "Show/list items" -> `infinity_list_items`
- "Get item detail" -> `infinity_get_item`, use `expand` when values or related objects are needed
- "Create/update/delete item" -> `infinity_create_item`, `infinity_update_item`, `infinity_archive_item`
- "Subitems" -> `infinity_list_subitems` or `infinity_create_subitem`; subitems are items whose `parent_id` is set
- "Add comment/note/progress update/message to item" -> `infinity_add_item_comment`
- "Show comments" -> `infinity_list_comments`
- "Get/update/delete comment" -> `infinity_get_comment`, `infinity_update_comment`, or `infinity_delete_comment`

Use read-only tools first when IDs are not known. For write actions, resolve IDs first, then perform one targeted write.

## Comment Workflows

When the user asks to add a comment, note, progress update, activity note, or message to an item, use comment tools, not item update tools.

Preferred tool:

```txt
infinity_add_item_comment
```

Use it with:

```json
{
  "workspace_id": "<workspace id>",
  "board_id": "<board id>",
  "item_id": "<item id>",
  "comment_text": "Plain text comment",
  "response_format": "json"
}
```

Do not use `infinity_update_item` to add a comment. Updating an item changes fields such as status or description and can falsely appear successful while no comment is created.

After adding a comment, verify with:

```txt
infinity_list_comments
```

Report success only after the created comment appears or the add-comment tool returns a concrete created comment object.

Ask for explicit confirmation before using `infinity_update_comment` or `infinity_delete_comment`.

## Efficient Tool Use

When using the MCP server from code, use the MCP result's `structuredContent` instead of parsing `content[0].text`. The text output can be truncated for large item lists, but `structuredContent` keeps the full page object.

Use `response_format: "json"` for programmatic work.

Always paginate list tools:

```js
async function collectPages(client, name, args = {}) {
  const out = [];
  let after;
  for (let i = 0; i < 100; i++) {
    const result = await client.callTool({
      name,
      arguments: { ...args, limit: 100, ...(after ? { after } : {}), response_format: "json" },
    });
    const page = result.structuredContent;
    out.push(...(page.data ?? []));
    if (!page.has_more || !page.after || page.after === after) break;
    after = page.after;
  }
  return out;
}
```

For folder item counts, call `infinity_list_items` with `folder_id`. Do not list all board items and count client-side unless the user explicitly needs a board-wide item analysis.

For folder trees, use `parent_id` from `infinity_list_folders` to nest folders. Example order:

1. `infinity_list_workspaces`
2. `infinity_list_boards`
3. `infinity_list_folders`
4. For each folder, `infinity_list_items` with `folder_id`

## Board Workflows

Available board tools:

- `infinity_list_boards`
- `infinity_get_board`
- `infinity_create_board`

There is no documented board update/delete endpoint in `2026-04-20.morava`. If a user asks to rename, recolor, archive, or delete a board, do not search repeatedly or try guessed `PUT`/`DELETE` board endpoints. State the limitation and offer the closest practical path.

When creating a board, use:

```json
{
  "workspace_id": "52593",
  "name": "Board Name",
  "description": "Optional description",
  "color": "#1EB9C8"
}
```

The API adds the current user automatically.

## Workspace Workflows

Available workspace tools:

- `infinity_list_workspaces`
- `infinity_list_workspace_members`
- `infinity_invite_workspace_member`
- `infinity_add_workspace_member`
- `infinity_remove_workspace_member`

There is no documented workspace create/get/update/delete endpoint in Morava. Direct workspace management beyond members should be done in the Infinity UI unless newer docs are explicitly requested.

## Item Values

Item create/update tools accept `values` as an object keyed by attribute ID:

```json
{
  "attribute-id-1": "Text value",
  "attribute-id-2": true,
  "attribute-id-3": ["label-id"]
}
```

The MCP server converts this to Infinity's API format:

```json
[
  { "attribute_id": "attribute-id-1", "data": "Text value" }
]
```

Always inspect attributes first when the field type or attribute ID is uncertain.

## Attribute Workflows

Attributes are board-level fields. Creating an attribute does not automatically mean every folder shows it. After creating an attribute, check or update the folder's `attribute_ids` if the field needs to appear in a specific folder.

Fast path for creating a new field:

1. `infinity_list_boards`
2. `infinity_list_folders`
3. `infinity_create_attribute`
4. `infinity_get_folder`
5. `infinity_update_folder` with the folder's existing `attribute_ids` plus the new attribute ID

Use `infinity_list_attributes` before item writes to map human field names to attribute IDs.

## Common Requests

### Show Workspace > Board > Folder Tree

Use the Docker MCP endpoint with a Streamable HTTP client and the configured MCP API key. Collect pages with `structuredContent`, then:

1. Print each workspace name.
2. Under it, print each board name.
3. Under it, print folders, nested by `parent_id`.
4. Add item counts by calling `infinity_list_items` per folder with `folder_id`.

Keep output readable. If the tree is very deep, show the main tree and summarize or compress repeated deep branches unless the user asks for every folder.

### Create Or Update Items

Before writing:

1. Use read-only tools to resolve workspace, board, folder, and attribute IDs.
2. Confirm destructive or externally visible changes when needed.
3. Pass item values keyed by attribute ID.
4. Use `infinity_get_item` after update/create if confirmation is needed.

### Create Subitems

Use `infinity_create_subitem` when the user says subitem. It sends an item create request with `parent_id` set to the parent item ID. If the parent item ID is unknown, list or search items first and ask only if ambiguity remains.

## Safety

Prefer read-only tools first. Ask for confirmation before using:

- `infinity_create_item`
- `infinity_update_item`
- `infinity_archive_item`
- `infinity_create_board`
- `infinity_invite_workspace_member`
- `infinity_add_workspace_member`
- `infinity_remove_workspace_member`
- `infinity_create_attribute`
- `infinity_update_attribute`
- `infinity_delete_attribute`
- `infinity_create_folder`
- `infinity_update_folder`
- `infinity_archive_folder`
- `infinity_create_subitem`

Archive tools change remote Infinity data.
