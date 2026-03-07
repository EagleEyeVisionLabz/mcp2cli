---
name: mcp2cli
description: Convert MCP servers to token-efficient CLI tools. Use when the user wants to migrate an MCP server to a CLI tool, create a CLI wrapper for an MCP server, generate a SKILL.md for a CLI tool, or improve agent token efficiency by replacing MCP with CLI. Triggers on keywords like "mcp to cli", "convert mcp", "cli migration", "replace mcp", "mcp wrapper".
---

# MCP-to-CLI Conversion Skill

You are an expert at converting MCP (Model Context Protocol) servers into token-efficient CLI tools with accompanying SKILL.md files for Claude Code agents.

## Why Convert MCP to CLI?

- **Token efficiency**: MCP loads all tool schemas upfront (tens of thousands of tokens). CLI + SKILL.md loads on-demand, saving 33-99% tokens.
- **Simpler integration**: No server process needed. Just `npm install -g` + SKILL.md.
- **Compact output**: CLI returns plain text vs MCP's verbose JSON with metadata.

## Conversion Process

Follow these 5 phases in order. Report progress to the user at each phase.

Before starting, classify the MCP server by reading `references/pattern-classification.md` to determine conversion feasibility and strategy.

### Phase 1: Analyze the MCP Server

Read the MCP server source code and extract:

1. **Tools**: List every `@tool`, `server.setRequestHandler(CallToolRequestSchema, ...)`, or `@mcp.tool()` definition
2. **Parameters**: For each tool, document all input parameters (name, type, required/optional, description)
3. **Output format**: What each tool returns (JSON structure, text, binary)
4. **Dependencies**: External APIs, SDKs, auth requirements
5. **Transport**: stdio, SSE, HTTP — and whether the server is a proxy or direct implementation
6. **Resources/Prompts**: Any MCP resources or prompt templates exposed

Create a structured analysis table:

```
| Tool Name | Params | Output | API Endpoint | Auth |
|-----------|--------|--------|--------------|------|
```

### Phase 2: Design CLI Command Structure

Map MCP tools to CLI commands following these rules:

**Naming Convention:**
- Tool prefix groups become subcommands: `lol_get_summoner_profile` → `<cli-name> lol profile`
- Single-domain tools become direct commands: `parse_document` → `<cli-name> parse`
- Use kebab-case for multi-word commands

**Parameter Mapping:**
- Required MCP params → positional CLI args
- Optional MCP params → named flags (`--flag value`)
- Boolean params → boolean flags (`--verbose`, `--no-cache`)
- Complex objects → JSON string flags (`--schema '{"key":"val"}'`) or file references (`--schema-file path`)

**Output Design:**
- Default: human-readable formatted text
- `--json`: raw JSON output for piping
- `--quiet`: suppress non-essential output
- Progress/status → stderr, data → stdout

**Standard Flags (always include):**
- `--help`, `--version`, `--json`, `--verbose` / `-v`

### Phase 3: Generate CLI Tool Code

**Always generate TypeScript/npm-based CLI tools**, regardless of the source MCP server's language.

See `references/cli-scaffold.md` for the project template, stack, and implementation rules.

If the source MCP server is Python, see `references/porting-cheatsheet.md` for the translation guide.

### Phase 4: Generate SKILL.md for the CLI Tool

Create a SKILL.md that teaches Claude Code how to use the new CLI tool.

See `references/skill-template.md` for the full template and rules.

### Phase 5: Validate and Package

1. **Test the CLI**: Run each command and verify output matches the original MCP tool's output
2. **Test the SKILL.md**: Verify Claude can follow the instructions to use the CLI correctly
3. **Package for distribution**: Set `bin` field in package.json, publish to npm
