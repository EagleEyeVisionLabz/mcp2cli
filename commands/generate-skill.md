---
name: generate-skill
description: Generate a SKILL.md file for an existing CLI tool
---

# /generate-skill - SKILL.md Generator

Generate a SKILL.md file for an existing CLI tool so Claude Code can use it effectively.

## Usage

```
/generate-skill <cli-tool-name>
```

## Instructions

When the user runs `/generate-skill`, use the `skill-author` skill to:

1. Run `<cli-tool-name> --help` to discover available commands
2. For each subcommand, run `<cli-tool-name> <subcommand> --help` to get detailed usage
3. If the tool has a man page or README, read it for additional context
4. Generate a SKILL.md following the template
5. Place the generated SKILL.md at `skills/<cli-tool-name>/SKILL.md` in the current project
