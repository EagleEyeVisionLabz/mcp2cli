# mcp2cli - MCP to CLI Conversion Plugin

This is a Claude Code plugin that helps convert MCP servers into token-efficient CLI tools.

## Plugin Structure

- `.claude-plugin/plugin.json` - Plugin manifest
- `skills/mcp2cli/SKILL.md` - Core conversion skill (auto-invoked)
- `skills/mcp2cli/references/` - Detailed patterns and templates
- `commands/` - Slash commands (`/convert`, `/analyze-mcp`, `/generate-skill`)
- `agents/mcp-analyzer.md` - Subagent for MCP server analysis

## Key Commands

- `/convert <path>` - Full MCP→CLI conversion workflow
- `/analyze-mcp <path>` - Analyze without converting
- `/generate-skill <cli-name>` - Generate SKILL.md for existing CLI

## Development

Test locally: `claude --plugin-dir /path/to/mcp2cli`
