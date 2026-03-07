# MCP-to-CLI Conversion Patterns Reference

## Pattern Catalog

### 1. Proxy MCP → Direct HTTP CLI

**Signature**: MCP server that creates a `Client` connecting to a remote endpoint and a `Server` on stdio, forwarding requests.

**Example**: opgg-mcp
- MCP: `Client → StdioServer → StreamableHTTPClientTransport → Remote API`
- CLI: `CLI → httpx/fetch → Remote API`

**Conversion Steps**:
1. Identify the remote API URL from the transport config
2. Extract the tool list from `ListToolsRequestSchema` handler
3. Create CLI commands that call the remote API directly via HTTP
4. Map `desired_output_fields` patterns to `--fields` flags
5. No MCP SDK dependency needed in the CLI

**Code Template (TypeScript)**:
```typescript
#!/usr/bin/env node
import { Command } from 'commander';

const API_BASE = 'https://api.example.com';

const program = new Command()
  .name('example-cli')
  .version('1.0.0');

program
  .command('tool-name <required-arg>')
  .option('--fields <fields...>', 'Output fields to include')
  .option('--json', 'JSON output')
  .action(async (arg, opts) => {
    const res = await fetch(`${API_BASE}/endpoint`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ arg, fields: opts.fields }),
    });
    const data = await res.json();
    if (opts.json) {
      console.log(JSON.stringify(data, null, 2));
    } else {
      // formatted output
      console.log(formatOutput(data));
    }
  });

program.parse();
```

### 2. FastMCP Python → TypeScript CLI

**Signature**: Python server using `@mcp.tool()` decorators with FastMCP.

**Conversion Steps**:
1. List all `@mcp.tool()` decorated functions
2. Create a `commander` command for each tool
3. Port `httpx` calls → `fetch`
4. Port `aiofiles` → `fs/promises`
5. Port `tenacity` retry → simple retry loop or `p-retry`
6. Port validators/constants as-is (logic is language-agnostic)
7. Replace `ctx.report_progress()` with stderr logging or `ora` spinner

**Code Template (TypeScript)**:
```typescript
#!/usr/bin/env node
import { Command } from 'commander';
import { readFile } from 'fs/promises';

const program = new Command()
  .name('upstage-cli')
  .version('1.0.0');

program
  .command('parse <file>')
  .option('-f, --output-format <formats...>', 'Output formats')
  .option('--json', 'JSON output')
  .action(async (file, opts) => {
    const apiKey = process.env.UPSTAGE_API_KEY;
    if (!apiKey) { console.error('Set UPSTAGE_API_KEY'); process.exit(1); }

    const body = new FormData();
    body.append('document', new Blob([await readFile(file)]));
    body.append('model', 'document-parse');

    const res = await fetch('https://api.upstage.ai/v1/document-digitization', {
      method: 'POST',
      headers: { Authorization: `Bearer ${apiKey}` },
      body,
    });
    const data = await res.json();

    if (opts.json) {
      console.log(JSON.stringify(data, null, 2));
    } else {
      console.log(data.content?.text ?? JSON.stringify(data, null, 2));
    }
  });

program.parse();
```

### 3. Multi-Tool MCP → Grouped Subcommand CLI

**Signature**: MCP server with 10+ tools, often with naming prefixes.

**Conversion Steps**:
1. Group tools by prefix: `lol_*`, `tft_*`, `val_*`
2. Create command groups for each prefix
3. Strip prefix for subcommand name: `lol_get_profile` → `lol profile`
4. Share auth/config across all command groups

**Code Template (TypeScript)**:
```typescript
const lol = program.command('lol').description('League of Legends');
const tft = program.command('tft').description('Teamfight Tactics');

lol.command('profile <summoner>')
  .option('--region <region>', 'Server region', 'na')
  .action(async (summoner, opts) => { /* ... */ });

lol.command('matches <summoner>')
  .option('--limit <n>', 'Number of matches', '20')
  .action(async (summoner, opts) => { /* ... */ });

tft.command('meta')
  .option('--count <n>', 'Number of decks', '10')
  .action(async (opts) => { /* ... */ });
```

### 4. Stateful/Streaming MCP → Hybrid Approach

**Signature**: MCP server that maintains state or uses subscriptions.

**When to use**: Browser automation, real-time data feeds, collaborative tools.

**Approach**: Keep MCP for stateful operations, but extract stateless read-only operations as CLI commands.

**Example**:
- Keep MCP: `subscribe_to_updates`, `maintain_session`
- Convert to CLI: `get_status`, `list_items`, `export_data`

## Token Efficiency Reference

| Scenario | MCP Tokens | CLI Tokens | Savings |
|----------|-----------|-----------|---------|
| 5 tools, simple API | ~8,000 | ~500 (SKILL.md) | 94% |
| 30 tools, complex API | ~55,000 | ~2,000 (SKILL.md) | 96% |
| 100 tools, multi-server | ~134,000 | ~5,000 (SKILL.md) | 96% |
| Per invocation overhead | ~200-500 | ~50-100 | 75% |

## Field Selection Pattern Conversion

MCP servers often use structured field selection (like opgg's `desired_output_fields`):

```
MCP: desired_output_fields: ["data.summoner.{name,level}", "data.stats[].{win,lose}"]
CLI: --fields "name,level" --fields "stats.win,stats.lose"
  or: --fields "data.summoner.{name,level}"
  or: mycli profile "Faker" --json | jq '.data.summoner | {name, level}'
```

Prefer the `jq` piping approach as it's universally understood and more flexible.
