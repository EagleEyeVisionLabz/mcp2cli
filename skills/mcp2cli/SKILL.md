---
name: mcp2cli
description: Convert MCP servers to token-efficient CLI tools. Use when the user wants to migrate an MCP server to a CLI tool, create a CLI wrapper for an MCP server, generate a SKILL.md for a CLI tool, or improve agent token efficiency by replacing MCP with CLI. Triggers on keywords like "mcp to cli", "convert mcp", "cli migration", "replace mcp", "mcp wrapper".
---

# MCP-to-CLI Conversion Skill

You are an expert at converting MCP (Model Context Protocol) servers into token-efficient CLI tools with accompanying SKILL.md files for Claude Code agents.

## Why Convert MCP to CLI?

- **Token efficiency**: MCP loads all tool schemas upfront (tens of thousands of tokens). CLI + SKILL.md loads on-demand, saving 33-99% tokens.
- **Simpler integration**: No server process needed. Just `npm install -g` or `pip install` + SKILL.md.
- **Compact output**: CLI returns plain text vs MCP's verbose JSON with metadata.

## Conversion Process

Follow these 5 phases in order. Report progress to the user at each phase.

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
- Field selection patterns (like `desired_output_fields`) → `--fields` flag

**Output Design:**
- Default: human-readable formatted text
- `--json`: raw JSON output for piping
- `--quiet`: suppress non-essential output
- Progress/status → stderr, data → stdout

**Standard Flags (always include):**
- `--help`: usage information
- `--version`: tool version
- `--json`: JSON output mode
- `--verbose` / `-v`: detailed output

### Phase 3: Generate CLI Tool Code

Choose the implementation language based on the source MCP server:

**TypeScript/Node MCP → TypeScript CLI:**
- Use `commander` or `yargs` for argument parsing
- Reuse the MCP server's API client code directly
- Entry point: `#!/usr/bin/env node` with `bin` field in package.json
- Build with `tsup` for single-file distribution

**Python MCP → Python CLI:**
- Use `click` or `typer` for argument parsing
- Reuse the MCP server's API client, validators, utilities
- Entry point: `[project.scripts]` in pyproject.toml
- Replace `async` MCP context calls with `asyncio.run()` wrappers

**Proxy MCP (remote API relay):**
- Create a thin HTTP client CLI that calls the remote API directly
- Skip the MCP protocol layer entirely
- Add local caching where appropriate

**Code Structure Template:**
```
<cli-name>/
├── src/
│   ├── cli.ts (or cli.py)      # Entry point + command definitions
│   ├── commands/                # One file per command group
│   │   ├── <group1>.ts
│   │   └── <group2>.ts
│   ├── api/                     # API client (reused from MCP)
│   │   └── client.ts
│   └── utils/                   # Shared utilities
│       ├── output.ts            # Output formatting
│       └── config.ts            # Config file loading
├── package.json (or pyproject.toml)
├── tsconfig.json (if TS)
└── README.md
```

**Key Implementation Rules:**
1. **Reuse, don't rewrite**: Copy API client logic from the MCP server. Only change the interface layer.
2. **Error codes**: Exit 0 on success, 1 on user error, 2 on API error, 3 on network error.
3. **Auth**: Support env vars (e.g., `API_KEY`), config files, and `--api-key` flag (in that priority order).
4. **Timeouts**: Default 30s for API calls, configurable via `--timeout`.
5. **No MCP dependency**: The CLI must not import any MCP SDK packages.

### Phase 4: Generate SKILL.md for the CLI Tool

Create a SKILL.md that teaches Claude Code how to use the new CLI tool.

**Template:**
```markdown
---
name: <cli-name>
description: <What the tool does and when to use it. Include trigger keywords. Written in third person.>
---

# <CLI Tool Name>

<One-line description of what this CLI does.>

## Installation

<How to install the CLI globally>

## Commands

### `<cli-name> <command> [args] [flags]`

<Description>

**Arguments:**
- `<arg>`: <description>

**Flags:**
- `--flag`: <description>

**Example:**
\`\`\`bash
<cli-name> <command> "example-arg" --flag value
\`\`\`

<Repeat for each command>

## Output Formats

- Default: human-readable text
- `--json`: machine-readable JSON, pipeable to `jq`

## Common Patterns

<2-3 practical usage examples showing real workflows>

## Error Handling

<Common errors and how to resolve them>
```

**SKILL.md Rules:**
1. Keep under 500 lines — put extended docs in separate reference files
2. Show concrete examples for every command
3. Include the `--json | jq` piping pattern for data extraction
4. Mention env var names for authentication
5. Description field must include trigger keywords for auto-discovery

### Phase 5: Validate and Package

1. **Test the CLI**: Run each command and verify output matches the original MCP tool's output
2. **Test the SKILL.md**: Verify Claude can follow the instructions to use the CLI correctly
3. **Package for distribution**:
   - npm: Set `bin` field, publish to npm
   - pip: Set `[project.scripts]`, publish to PyPI
   - Or distribute as a Claude Code plugin with the CLI bundled

## Reference: Patterns from Real Conversions

### Pattern A: Proxy MCP → Direct API CLI (opgg-mcp style)

The MCP server is just a protocol translator. The CLI should call the API directly:

```
MCP: Client → MCP Server (stdio) → MCP Client → Remote API (HTTP/SSE)
CLI: Client → CLI → Remote API (HTTP)  [one fewer hop]
```

### Pattern B: SDK MCP → CLI Wrapper (mcp-upstage style)

The MCP server wraps an SDK with validation and file handling. Reuse all business logic:

```
MCP: @mcp.tool() decorator → validation → API call → save output
CLI: @click.command() decorator → validation → API call → save output
```

Only the interface layer changes. The core logic is identical.

### Pattern C: Complex Multi-Tool MCP → Grouped CLI

For MCP servers with many tools (10+), group by domain:

```
MCP: lol_get_profile, lol_get_matches, tft_meta_decks, val_agents
CLI: mycli lol profile, mycli lol matches, mycli tft meta, mycli val agents
```

## Decision Guide: When NOT to Convert

Keep MCP when:
- The server provides **real-time streaming** that CLI can't replicate
- Tools require **bidirectional communication** (subscriptions, live updates)
- The ecosystem has **no existing CLI** and the API is too complex for shell commands
- The server manages **persistent state** across tool calls

Convert to CLI when:
- Tools are **stateless request/response** patterns
- Output is **text or JSON** (not binary streams)
- The agent uses tools **infrequently** (MCP schema overhead > usage value)
- You want **simpler deployment** without background server processes
