---
name: convert
description: Start an MCP-to-CLI conversion workflow
---

# /convert - MCP to CLI Conversion

Convert an MCP server to a token-efficient CLI tool.

## Usage

```
/convert <path-to-mcp-server>
```

## What This Does

1. Analyzes the MCP server at the given path
2. Extracts all tool definitions, parameters, and output formats
3. Designs a CLI command structure
4. Generates the CLI tool code
5. Creates a SKILL.md for Claude Code integration
6. Validates the conversion

## Instructions

When the user runs `/convert`, follow the mcp2cli skill's 5-phase conversion process.

If no path is provided, ask the user for the path to the MCP server they want to convert.

If the path is a GitHub URL, clone the repository first using `git clone`.

Start by reading the MCP server's entry point file and package manifest to understand the scope of the conversion.
