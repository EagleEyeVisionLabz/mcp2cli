# SKILL.md Generation Template

Use this template when generating SKILL.md files for converted CLI tools.

## Template

```markdown
---
name: {cli-name}
description: {What the tool does. When to use it. Trigger keywords. Third person voice. Max 200 chars.}
---

# {CLI Tool Name}

{One sentence: what this tool does.}

## Prerequisites

- Install: `{install-command}`
- Auth: Set `{ENV_VAR}` environment variable

## Commands

### `{cli-name} {command} {<required-arg>} [optional-arg] [flags]`

{Brief description of what this command does.}

| Argument/Flag | Type | Required | Description |
|--------------|------|----------|-------------|
| `<arg>` | string | yes | {description} |
| `--flag` | string | no | {description} (default: {default}) |

**Example:**
\`\`\`bash
{cli-name} {command} "example" --flag value
\`\`\`

**Output:**
\`\`\`
{example output}
\`\`\`

{Repeat ### block for each command}

## Output Modes

| Flag | Format | Use Case |
|------|--------|----------|
| (default) | Formatted text | Human reading |
| `--json` | Raw JSON | Piping to `jq`, programmatic use |
| `--quiet` | Minimal | Scripts, CI/CD |

## Common Workflows

### {Workflow Name}
\`\`\`bash
# Step 1: {description}
{cli-name} {command1} "arg"

# Step 2: {description}
{cli-name} {command2} --json | jq '.field'
\`\`\`

## Troubleshooting

| Error | Cause | Fix |
|-------|-------|-----|
| `{error message}` | {cause} | {fix} |
```

## Guidelines

1. **Length**: Target 200-400 lines. Never exceed 500.
2. **Examples**: Every command MUST have at least one concrete example with realistic arguments.
3. **Description field**: Include 3-5 trigger keywords that match user intent. Examples:
   - "gaming statistics" for game data APIs
   - "document parsing OCR" for document tools
   - "browser testing automation" for web tools
4. **Output examples**: Show what the user will actually see. Truncate long outputs with `...`
5. **Workflow section**: Show 2-3 real multi-step scenarios. This is what distinguishes a good SKILL.md.
6. **No MCP references**: The SKILL.md should not mention MCP at all. It's a CLI tool now.
