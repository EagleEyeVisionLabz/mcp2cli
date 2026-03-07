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

## Instructions

When the user runs `/convert`, execute these phases in order:

**Phase 1 — Analyze** (uses `mcp-analyze` skill)
1. If the path is a GitHub URL, clone the repository first
2. Read the MCP server's entry point and package manifest
3. Extract all tool definitions, parameters, and output formats
4. Classify the server pattern and assess conversion feasibility
5. Present the analysis table and ask the user to confirm before proceeding

**Phase 2 — Generate CLI** (uses `mcp-codegen` skill)
1. Design the CLI command structure based on the analysis
2. Generate TypeScript CLI code using the appropriate tier template
3. If the source is Python, refer to `mcp-codegen`'s `references/porting-cheatsheet.md` for translation patterns

**Phase 3 — Generate SKILL.md** (uses `skill-author` skill)
1. Generate a SKILL.md for the new CLI tool
2. Follow the skill template and rules

**Phase 4 — Validate**
1. Run each CLI command and verify output
2. Verify the SKILL.md is correct and complete

If no path is provided, ask the user for the path to the MCP server they want to convert.
