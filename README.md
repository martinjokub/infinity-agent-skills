# Infinity Agent Skills

Reusable agent skills for working with StartInfinity through the Infinity MCP server.

Current version: `0.1.4`

Changelog:

```txt
CHANGELOG.md
```

Companion MCP server repository:

```txt
https://github.com/martinjokub/infinity-mcp-server
```

## Included Skills

```txt
skills/infinity-mcp/SKILL.md
```

This skill routes agents directly to the right Infinity MCP tools for:

- workspace discovery and members
- board listing/getting/creation
- folder trees and item counts
- folder create/update/archive
- attribute create/update/delete
- item and subitem create/update/archive

It also records known API limitations, such as the fact that the Morava API docs expose board list/get/create but not board update/delete.

## Install For An Agent Runtime

Copy the skill folder into the skills directory used by your agent runtime.

```powershell
Copy-Item -Recurse .\skills\infinity-mcp "path\to\your\agent\skills\infinity-mcp" -Force
```

Then start a new agent session or reload skills. Use the skill when asking agents to work with Infinity data.

## Required MCP Server

The skill assumes an Infinity MCP server is available. Configure the MCP endpoint for your own machine or deployment.

For cloud Docker setup, the skill tells agents to:

1. use this skills repo first,
2. clone `https://github.com/martinjokub/infinity-mcp-server.git`,
3. install into the folder chosen by the user,
4. store the Infinity token in the encrypted credential store,
5. generate an MCP API key,
6. test the server,
7. configure Codex or the MCP client with the MCP URL and API key.

If you use the companion MCP server's example Docker setup, the default local endpoint is:

```txt
http://127.0.0.1:3015/mcp
```

HTTP MCP clients must send:

```txt
Authorization: Bearer <mcp-api-key>
```

The MCP API key is separate from the Infinity token. The server maps it to an encrypted Infinity credential profile.

For other machines, update your agent config or the skill's `Local Server` section to match your MCP endpoint.

See `config/infinity-mcp.config.example.json` for a neutral configuration template.

## Versioning

This project uses semver-compatible versions in the form `x.x.y`. Treat the final number as the change counter requested for the project: increment it by `+1` for each update or edit unless a higher-level version bump is explicitly requested.

The current version is stored in `VERSION` and summarized in `CHANGELOG.md`.

## Safety Notes

The skill tells agents to prefer read-only tools first and ask for confirmation before write/destructive actions such as:

- creating/updating/archiving items
- creating/updating/archiving folders
- creating/updating/deleting attributes
- inviting/adding/removing workspace members
