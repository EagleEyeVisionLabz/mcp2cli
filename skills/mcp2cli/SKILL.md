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

**Always generate TypeScript/npm-based CLI tools**, regardless of the source MCP server's language. The source language doesn't matter — MCP servers are API wrappers, and the CLI just needs to call the same APIs.

Benefits of npm-based CLI:
- `npx <cli-name>` — run without global install
- `bin` field in package.json — clean global install via `npm i -g`
- `tsup` — bundle to a single file, no dependency issues
- Cross-platform without Python version conflicts

**Stack:**
- `commander` for argument parsing
- `node-fetch` or built-in `fetch` for HTTP calls
- `tsup` for bundling to single ESM file
- Entry point: `#!/usr/bin/env node`

**Porting API logic from Python MCP servers:**
- Translate `httpx` calls → `fetch`
- Translate `aiofiles` → `fs/promises`
- Translate `tenacity` retry → simple retry loop or `p-retry`
- Translate `click.progressbar` → `ora` or stderr logging
- Translate validators/constants as-is (logic is language-agnostic)

**Code Structure Template:**
```
<cli-name>/
├── src/
│   ├── cli.ts                   # Entry point + command definitions
│   ├── commands/                # One file per command group
│   │   ├── <group1>.ts
│   │   └── <group2>.ts
│   ├── api/
│   │   └── client.ts            # HTTP client (ported from MCP)
│   └── utils/
│       ├── output.ts            # Output formatting (JSON/text)
│       └── config.ts            # Config & env var loading
├── package.json                 # bin field + dependencies
├── tsconfig.json
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

## MCP Server Pattern Classification

Before converting, classify the MCP server into one of these patterns. Each has different conversion strategies.

### Tier 1: Excellent CLI Fit

| Pattern | What it wraps | Conversion strategy |
|:--------|:-------------|:-------------------|
| **Local System Tool** | Filesystem, git, docker, shell | 1:1 tool-to-subcommand mapping. No auth. Use `child_process` or `fs` directly. |
| **SaaS API Wrapper** | REST/GraphQL APIs (GitHub, Slack, etc.) | HTTP client CLI. Auth via env vars. Map CRUD operations to subcommands. |
| **Infrastructure/DevOps** | Cloud APIs, K8s, CI/CD | Often wraps existing CLIs — consider thin wrapper or SKILL.md pointing to original CLI. |
| **Document/Media Processing** | OCR, PDF, image gen | File-in/file-out pattern. `<cli> process <input-file> --output <output-file>`. |

### Tier 2: Good CLI Fit

| Pattern | What it wraps | Conversion strategy |
|:--------|:-------------|:-------------------|
| **Database Query** | SQLite, Postgres, Redis | Connection string via env var. `<cli> query "SQL"`, `<cli> schema`. Format output as table or JSON. |
| **Search & Retrieval** | Web search, vector DBs | `<cli> search "query" --limit 10`. Simple for search; index management needs subcommands. |
| **Monitoring/Observability** | Metrics, logs, APM | `<cli> query "metric > threshold"`, `<cli> logs tail`. Streaming → `--follow` flag. |
| **Communication/Messaging** | Slack, email, Discord | `<cli> send "#channel" "message"`, `<cli> read --channel`. |
| **Proxy MCP** | Remote MCP/API endpoint | Remove protocol layer, call API directly. One fewer hop. |
| **Multi-tool MCP** | 10+ tools with prefixes | Group by prefix: `lol_get_profile` → `<cli> lol profile`. |

### Tier 3: Partial/Difficult CLI Fit

| Pattern | What it wraps | Strategy |
|:--------|:-------------|:---------|
| **Browser Automation** | Playwright, Puppeteer | Individual actions work (`click`, `screenshot`), but session state is awkward. Consider script-based approach. |
| **Code Execution/Sandbox** | Language runtimes | One-shot `<cli> exec "code"` works. REPL sessions need persistent process. |
| **Knowledge/Memory** | Knowledge graphs, memory stores | `store`/`recall` subcommands work, but loses value without LLM loop. |
| **Auth & Identity** | OAuth, identity platforms | OAuth browser redirect flow needs loopback server (like `gh auth login`). API key auth is trivial. |

### Tier 4: Not Recommended for CLI

| Pattern | Why |
|:--------|:----|
| **Aggregator/Gateway** | Infrastructure for MCP ecosystems, not end-user tools. |
| **Reasoning/Cognitive** | Structures LLM's own reasoning. No CLI value without AI loop. |

## Decision Guide

**Convert to CLI when:**
- Tools are stateless request/response (Tier 1 & 2)
- Output is text or JSON
- The MCP server wraps an API, filesystem, or database
- Schema overhead > actual usage value

**Keep as MCP when:**
- Server maintains session state critical to functionality (browser sessions, REPL)
- Real-time streaming / bidirectional communication is core
- Server aggregates or routes to other MCP servers
- Server structures LLM reasoning (no standalone CLI value)

**Partial conversion** (Tier 3):
- Extract stateless read operations as CLI
- Keep stateful/interactive operations in MCP
- Example: Browser automation → `screenshot` and `snapshot` as CLI, keep `navigate`+`click` chains in MCP

See `references/patterns.md` for code templates for each pattern.
