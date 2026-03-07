# MCP-to-CLI Conversion Patterns Reference

## Tier 1: Excellent CLI Fit

### 1-A. Local System Tool → Direct CLI

**Signature**: MCP server wrapping filesystem, git, docker, or shell operations. No external API calls.

**Detection**: Imports like `fs`, `child_process`, `execSync`, `os`, `path`. No HTTP client.

**Conversion Steps**:
1. Map each tool to a subcommand (`read_file` → `<cli> read`, `list_dir` → `<cli> ls`)
2. Use Node.js built-ins (`fs/promises`, `child_process`) directly
3. No auth needed
4. File paths as positional args, options as flags

**Code Template**:
```typescript
#!/usr/bin/env node
import { Command } from 'commander';
import { readFile, readdir, stat } from 'fs/promises';
import { resolve } from 'path';

const program = new Command().name('fs-cli').version('1.0.0');

program
  .command('read <path>')
  .option('--encoding <enc>', 'File encoding', 'utf-8')
  .action(async (path, opts) => {
    console.log(await readFile(resolve(path), opts.encoding));
  });

program
  .command('ls <dir>')
  .option('-l, --long', 'Detailed listing')
  .action(async (dir, opts) => {
    const entries = await readdir(resolve(dir), { withFileTypes: true });
    for (const e of entries) {
      if (opts.long) {
        const s = await stat(resolve(dir, e.name));
        console.log(`${e.isDirectory() ? 'd' : '-'} ${s.size.toString().padStart(8)} ${e.name}`);
      } else {
        console.log(e.name);
      }
    }
  });

program.parse();
```

### 1-B. SaaS API Wrapper → HTTP Client CLI

**Signature**: MCP server calling external REST/GraphQL APIs with auth tokens.

**Detection**: `fetch`, `httpx`, `axios` calls to external URLs. Bearer token auth headers.

**Conversion Steps**:
1. Extract API base URL and endpoint paths
2. Auth via env var (e.g., `GITHUB_TOKEN`), falling back to `--token` flag
3. Map CRUD operations: `list_issues` → `<cli> issues list`, `create_issue` → `<cli> issues create`
4. Complex payloads → JSON flag (`--body '{"title":"..."}'`) or individual flags

**Code Template**:
```typescript
#!/usr/bin/env node
import { Command } from 'commander';

const API = 'https://api.example.com/v1';
const token = () => {
  const t = process.env.API_TOKEN;
  if (!t) { console.error('Set API_TOKEN env var'); process.exit(1); }
  return t;
};
const api = async (path: string, init?: RequestInit) => {
  const res = await fetch(`${API}${path}`, {
    ...init,
    headers: { Authorization: `Bearer ${token()}`, 'Content-Type': 'application/json', ...init?.headers },
  });
  if (!res.ok) { console.error(`API error: ${res.status} ${await res.text()}`); process.exit(2); }
  return res.json();
};

const program = new Command().name('example-cli').version('1.0.0');

const issues = program.command('issues');
issues.command('list').option('--status <s>', 'Filter by status')
  .action(async (opts) => console.log(JSON.stringify(await api(`/issues?status=${opts.status ?? ''}`), null, 2)));
issues.command('create <title>').option('--body <b>', 'Issue body')
  .action(async (title, opts) => console.log(JSON.stringify(await api('/issues', {
    method: 'POST', body: JSON.stringify({ title, body: opts.body }),
  }), null, 2)));

program.parse();
```

### 1-C. Infrastructure/DevOps → Wrapper or SKILL.md

**Signature**: MCP server wrapping `kubectl`, `docker`, `terraform`, or cloud CLIs.

**Detection**: `child_process.exec('kubectl ...')`, AWS SDK imports, etc.

**Strategy**: If a good CLI already exists (kubectl, docker, aws, gh), don't rewrite — generate a SKILL.md that teaches Claude how to use the existing CLI directly. Only create a new CLI if the MCP server adds significant logic on top.

### 1-D. Document/Media Processing → File I/O CLI

**Signature**: MCP server doing OCR, PDF parsing, image generation via local models or remote APIs.

**Detection**: File read + API call + file write pattern. MIME type detection. Base64 encoding.

**Conversion Steps**:
1. Input: file path as positional arg
2. Output: `--output <path>` flag, default to stdout
3. Processing options as flags (`--format`, `--model`, `--quality`)
4. Progress reporting via `ora` spinner or stderr dots

## Tier 2: Good CLI Fit

### 2-A. Database Query → SQL CLI

**Signature**: MCP server connecting to a database engine and running queries.

**Detection**: Database driver imports (`pg`, `better-sqlite3`, `mysql2`, `ioredis`).

**Code Template**:
```typescript
#!/usr/bin/env node
import { Command } from 'commander';
import Database from 'better-sqlite3';

const program = new Command().name('db-cli').version('1.0.0');

program
  .command('query <sql>')
  .option('-d, --database <path>', 'Database file', './data.db')
  .option('--json', 'JSON output')
  .action((sql, opts) => {
    const db = new Database(opts.database, { readonly: true });
    const rows = db.prepare(sql).all();
    if (opts.json) {
      console.log(JSON.stringify(rows, null, 2));
    } else {
      if (rows.length === 0) { console.log('No results'); return; }
      const cols = Object.keys(rows[0]);
      console.log(cols.join('\t'));
      for (const row of rows) console.log(cols.map(c => row[c]).join('\t'));
    }
  });

program
  .command('tables')
  .option('-d, --database <path>', 'Database file', './data.db')
  .action((opts) => {
    const db = new Database(opts.database, { readonly: true });
    const tables = db.prepare("SELECT name FROM sqlite_master WHERE type='table'").all();
    tables.forEach(t => console.log(t.name));
  });

program.parse();
```

### 2-B. Search & Retrieval → Search CLI

**Signature**: MCP server calling search APIs or vector databases.

**Conversion Steps**:
1. `<cli> search "query" --limit 10 --type web`
2. For vector DBs with index management: `<cli> index add <file>`, `<cli> index search "query"`
3. Output: ranked results with title, snippet, URL/score

### 2-C. Proxy MCP → Direct HTTP CLI

**Signature**: MCP server that creates a `Client` connecting to a remote endpoint and a `Server` on stdio, forwarding requests transparently.

**Conversion Steps**:
1. Identify the remote API URL from the transport config
2. Extract the tool list from `ListToolsRequestSchema` handler
3. Create CLI commands that call the remote API directly via HTTP
4. Map `desired_output_fields` patterns to `--json | jq` piping
5. No MCP SDK dependency needed

### 2-D. Multi-Tool MCP → Grouped Subcommand CLI

**Signature**: MCP server with 10+ tools, often with naming prefixes.

**Code Template**:
```typescript
const lol = program.command('lol').description('League of Legends');
const tft = program.command('tft').description('Teamfight Tactics');

lol.command('profile <summoner>')
  .option('--region <region>', 'Server region', 'na')
  .action(async (summoner, opts) => { /* ... */ });

tft.command('meta')
  .option('--count <n>', 'Number of decks', '10')
  .action(async (opts) => { /* ... */ });
```

### 2-E. Communication/Messaging → Message CLI

**Signature**: MCP server wrapping Slack, email, Discord APIs.

**Conversion Steps**:
1. `<cli> send "#channel" "message"` or `<cli> send --to user@email.com --subject "..." --body "..."`
2. `<cli> read --channel "#general" --limit 20`
3. Auth via env var (`SLACK_TOKEN`, etc.)

### 2-F. Monitoring/Observability → Metrics CLI

**Conversion Steps**:
1. `<cli> query "cpu_usage > 90%" --from 1h`
2. `<cli> logs --service api --level error --follow`
3. `--follow` flag for streaming (outputs to stdout continuously)

## Tier 3: Partial Conversion

### 3-A. Browser Automation → Script-Based CLI

**Strategy**: Convert individual stateless actions to CLI commands. Session management via a persistent browser process.

**Convertible**: `screenshot <url>`, `snapshot <url>` (accessibility tree), `pdf <url>`
**Keep MCP**: Multi-step navigation chains, form filling sequences, cookie/session state

### 3-B. Code Execution → One-Shot CLI

**Convertible**: `<cli> exec "print(1+1)" --lang python`
**Keep MCP**: REPL sessions, persistent variable state, notebook-style execution

### 3-C. Knowledge/Memory → Store/Recall CLI

**Convertible**: `<cli> store "key" "value"`, `<cli> recall "query"`
**Limitation**: Loses value without LLM loop. Only useful as a key-value utility.

### 3-D. Auth & Identity → Login Flow CLI

**Strategy**: Use loopback server pattern for OAuth (like `gh auth login`). Store tokens in OS keychain or `~/.config/<cli>/credentials.json`.

## Tier 4: Not Recommended

| Pattern | Reason |
|:--------|:-------|
| **Aggregator/Gateway** | Infrastructure for MCP routing, not end-user tools |
| **Reasoning/Cognitive** | Structures LLM reasoning — no standalone value |

## Cross-Cutting Concerns

### Python → TypeScript Porting Cheatsheet

| Python | TypeScript |
|:-------|:-----------|
| `httpx.AsyncClient` | `fetch` (built-in) |
| `aiofiles` | `fs/promises` |
| `tenacity` retry | `p-retry` or manual retry loop |
| `click` / `typer` | `commander` |
| `pydantic` models | TypeScript interfaces + runtime validation |
| `os.getenv()` | `process.env` |
| `pathlib.Path` | `path.resolve()` / `path.join()` |
| `mimetypes.guess_type()` | `mime-types` package |
| `json.dumps(indent=2)` | `JSON.stringify(data, null, 2)` |
| `asyncio.run()` | Top-level await (ESM) |

### Token Efficiency Reference

| Scenario | MCP Tokens | CLI Tokens | Savings |
|:---------|----------:|----------:|--------:|
| 5 tools, simple API | ~8,000 | ~500 | 94% |
| 30 tools, complex API | ~55,000 | ~2,000 | 96% |
| 100 tools, multi-server | ~134,000 | ~5,000 | 96% |
| Per invocation overhead | ~200-500 | ~50-100 | 75% |

### Output Formatting Conventions

- Data → stdout, status/progress → stderr
- `--json` flag: raw JSON for piping to `jq`
- Default: human-readable formatted text
- Tables: tab-separated for easy parsing, or aligned columns for readability
- Exit codes: 0 success, 1 user error, 2 API error, 3 network error
