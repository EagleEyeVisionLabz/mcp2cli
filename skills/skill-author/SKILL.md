---
name: skill-author
description: Generate a SKILL.md file for a CLI tool so Claude Code can use it effectively. Use when creating SKILL.md, writing skill files, documenting CLI tools for agents, or making a CLI agent-friendly. Triggers on "generate skill", "create skill.md", "write skill", "skill for cli", "agent documentation".
---

# SKILL.md Authoring

Generate a SKILL.md file that teaches Claude Code how to use a CLI tool.

## When to Use

- After `mcp-codegen` produces a CLI tool (as part of `/convert`)
- Standalone via `/generate-skill` for any existing CLI tool

## Process

1. Discover the CLI's commands via `<cli> --help` and `<cli> <subcommand> --help`
2. Generate a SKILL.md following the template in `references/skill-template.md`

## Rules

1. Keep under 500 lines — put extended docs in separate reference files
2. Show concrete examples for every command
3. Include the `--json | jq` piping pattern for data extraction
4. Mention env var names for authentication
5. Description field must include trigger keywords for auto-discovery
6. Do NOT mention MCP in the generated SKILL.md — it's a CLI tool now
