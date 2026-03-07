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

### 2. FastMCP Python → Click/Typer CLI

**Signature**: Python server using `@mcp.tool()` decorators with FastMCP.

**Example**: mcp-upstage
- MCP: `@mcp.tool() async def parse_document(file_path, ctx)`
- CLI: `@click.command() def parse_document(file_path)`

**Conversion Steps**:
1. List all `@mcp.tool()` decorated functions
2. Convert each to a click/typer command
3. Replace `ctx.report_progress()` with `click.progressbar()` or `rich.progress`
4. Replace `ctx.info/warn/error()` with `click.echo()` to stderr
5. Wrap async functions with `asyncio.run()`
6. Keep all validation, API client, and utility modules unchanged

**Code Template (Python)**:
```python
#!/usr/bin/env python3
import click
import asyncio
from .api.client import call_api
from .utils.validators import validate_file

@click.group()
@click.version_option()
def cli():
    """CLI tool description."""
    pass

@cli.command()
@click.argument('file_path', type=click.Path(exists=True))
@click.option('--output-format', '-f', multiple=True, help='Output formats')
@click.option('--json', 'as_json', is_flag=True, help='JSON output')
def parse(file_path, output_format, as_json):
    """Parse a document and extract structured content."""
    validate_file(file_path)
    result = asyncio.run(call_api(file_path, list(output_format)))
    if as_json:
        click.echo(json.dumps(result, indent=2))
    else:
        click.echo(format_result(result))

if __name__ == '__main__':
    cli()
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
