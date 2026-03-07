---
name: analyze-mcp
description: Analyze an MCP server without converting it
---

# /analyze-mcp - MCP Server Analysis

Analyze an MCP server's tools, resources, and prompts to assess CLI conversion feasibility.

## Usage

```
/analyze-mcp <path-to-mcp-server>
```

## Instructions

When the user runs `/analyze-mcp`:

1. Read the MCP server source code at the given path
2. Extract all tool definitions with their parameters and output formats
3. Identify the server type: proxy (remote API relay), SDK wrapper, or standalone
4. Assess conversion feasibility using the mcp2cli skill's decision guide
5. Output a structured analysis table showing:
   - Each tool's name, parameters, output format
   - Recommended CLI command mapping
   - Conversion complexity (low/medium/high)
   - Any blockers or concerns

Do NOT generate any code. This is analysis only.
