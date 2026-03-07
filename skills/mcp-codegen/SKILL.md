---
name: mcp-codegen
description: Generate TypeScript CLI tool code from an MCP server analysis. Use when the user wants to create a CLI tool, generate CLI code, scaffold a CLI project, or design CLI command structure from MCP tools. Triggers on "generate cli", "create cli", "cli code", "scaffold cli", "mcp to typescript".
---

# CLI Code Generation

Generate a TypeScript/npm-based CLI tool from an analyzed MCP server.

## Prerequisites

Run `mcp-analyze` first to get the tool list, parameters, and pattern classification.

## Step 1: Design CLI Command Structure

Map MCP tools to CLI commands:

**Naming Convention:**
- Tool prefix groups become subcommands: `lol_get_summoner_profile` → `<cli> lol profile`
- Single-domain tools become direct commands: `parse_document` → `<cli> parse`
- Use kebab-case for multi-word commands

**Parameter Mapping:**
- Required MCP params → positional CLI args
- Optional MCP params → named flags (`--flag value`)
- Boolean params → boolean flags (`--verbose`, `--no-cache`)
- Complex objects → JSON string flags or `--file` references

**Output Design:**
- Default: human-readable formatted text
- `--json`: raw JSON output for piping
- Progress/status → stderr, data → stdout

**Standard Flags (always include):**
- `--help`, `--version`, `--json`, `--verbose` / `-v`

## Step 2: Generate Code

**Always generate TypeScript/npm-based CLI tools**, regardless of the source MCP server's language.

See `references/cli-scaffold.md` for the project template, stack, and implementation rules.

Select the appropriate code template from `references/tier-1-templates.md` or `references/tier-2-templates.md` based on the pattern classification from `mcp-analyze`.

If the pattern is Tier 3, see `references/tier-3-strategies.md` for partial conversion approach.
