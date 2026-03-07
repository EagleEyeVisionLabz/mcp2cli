# mcp2cli - MCP to CLI Conversion Plugin

This is a Claude Code plugin that converts MCP servers into token-efficient CLI tools.

## Skills

- `mcp-analyze` - Analyze MCP server source, classify pattern, assess feasibility
- `mcp-codegen` - Generate TypeScript CLI code from analysis (includes Python → TS porting)
- `skill-author` - Generate SKILL.md for CLI tools

## Commands

- `/convert <path>` - Full conversion (orchestrates all skills)
- `/analyze-mcp <path>` - Analysis only
- `/generate-skill <cli-name>` - Generate SKILL.md for existing CLI

## Development

Test locally: `claude --plugin-dir /path/to/mcp2cli`
