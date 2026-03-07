# Tier 2: Good CLI Fit — Code Templates

## Database Query

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
      for (const row of rows) console.log(cols.map(c => (row as any)[c]).join('\t'));
    }
  });

program
  .command('tables')
  .option('-d, --database <path>', 'Database file', './data.db')
  .action((opts) => {
    const db = new Database(opts.database, { readonly: true });
    const tables = db.prepare("SELECT name FROM sqlite_master WHERE type='table'").all();
    tables.forEach((t: any) => console.log(t.name));
  });

program.parse();
```

## Proxy MCP → Direct HTTP

```typescript
#!/usr/bin/env node
import { Command } from 'commander';

const API_BASE = 'https://api.example.com';

const program = new Command().name('proxy-cli').version('1.0.0');

program
  .command('call <tool-name>')
  .option('--params <json>', 'Tool parameters as JSON', '{}')
  .option('--json', 'JSON output')
  .action(async (toolName, opts) => {
    const res = await fetch(`${API_BASE}/tools/${toolName}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: opts.params,
    });
    const data = await res.json();
    console.log(JSON.stringify(data, null, opts.json ? 2 : undefined));
  });

program.parse();
```

## Multi-Tool → Grouped Subcommands

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

## Search & Retrieval

```typescript
program
  .command('search <query>')
  .option('--limit <n>', 'Max results', '10')
  .option('--type <type>', 'Search type (web, docs, code)')
  .option('--json', 'JSON output')
  .action(async (query, opts) => {
    const data = await api(`/search?q=${encodeURIComponent(query)}&limit=${opts.limit}&type=${opts.type ?? 'web'}`);
    if (opts.json) {
      console.log(JSON.stringify(data, null, 2));
    } else {
      for (const r of data.results) {
        console.log(`${r.title}\n  ${r.url}\n  ${r.snippet}\n`);
      }
    }
  });
```

## Communication/Messaging

```typescript
program.command('send <channel> <message>')
  .action(async (channel, message) => {
    await api('/messages', {
      method: 'POST',
      body: JSON.stringify({ channel, text: message }),
    });
    console.log(`Sent to ${channel}`);
  });

program.command('read')
  .option('--channel <ch>', 'Channel name')
  .option('--limit <n>', 'Number of messages', '20')
  .action(async (opts) => {
    const data = await api(`/messages?channel=${opts.channel}&limit=${opts.limit}`);
    for (const m of data.messages) {
      console.log(`[${m.user}] ${m.text}`);
    }
  });
```
