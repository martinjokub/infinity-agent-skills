# Infinity Agent Skills

Reusable agent skills for working with StartInfinity through the Infinity MCP server.

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

## Install For Codex

Copy the skill folder into your Codex skills directory:

```powershell
Copy-Item -Recurse .\skills\infinity-mcp "$env:USERPROFILE\.codex\skills\infinity-mcp" -Force
```

Then start a new Codex session or reload skills. Use the skill when asking agents to work with Infinity data.

## Required MCP Server

The skill assumes an Infinity MCP server is available. On Martin's local Docker setup, it runs at:

```txt
http://127.0.0.1:3015/mcp
```

For other machines, update the skill's `Local Server` section to match the MCP endpoint.

## Safety Notes

The skill tells agents to prefer read-only tools first and ask for confirmation before write/destructive actions such as:

- creating/updating/archiving items
- creating/updating/archiving folders
- creating/updating/deleting attributes
- inviting/adding/removing workspace members
