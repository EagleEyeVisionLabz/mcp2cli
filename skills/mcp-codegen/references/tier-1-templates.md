# Tier 1: Excellent CLI Fit — Code Templates

## Local System Tool

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

## SaaS API Wrapper

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
    headers: {
      Authorization: `Bearer ${token()}`,
      'Content-Type': 'application/json',
      ...init?.headers,
    },
  });
  if (!res.ok) {
    console.error(`API error: ${res.status} ${await res.text()}`);
    process.exit(2);
  }
  return res.json();
};

const program = new Command().name('example-cli').version('1.0.0');

const issues = program.command('issues');

issues.command('list')
  .option('--status <s>', 'Filter by status')
  .option('--json', 'JSON output')
  .action(async (opts) => {
    const data = await api(`/issues?status=${opts.status ?? ''}`);
    console.log(JSON.stringify(data, null, 2));
  });

issues.command('create <title>')
  .option('--body <b>', 'Issue body')
  .action(async (title, opts) => {
    const data = await api('/issues', {
      method: 'POST',
      body: JSON.stringify({ title, body: opts.body }),
    });
    console.log(JSON.stringify(data, null, 2));
  });

program.parse();
```

## Document/Media Processing

```typescript
#!/usr/bin/env node
import { Command } from 'commander';
import { readFile } from 'fs/promises';

const program = new Command().name('doc-cli').version('1.0.0');

program
  .command('parse <file>')
  .option('-f, --format <formats...>', 'Output formats')
  .option('--json', 'JSON output')
  .action(async (file, opts) => {
    const apiKey = process.env.API_KEY;
    if (!apiKey) { console.error('Set API_KEY env var'); process.exit(1); }

    const body = new FormData();
    body.append('document', new Blob([await readFile(file)]));
    body.append('model', 'document-parse');

    const res = await fetch('https://api.example.com/v1/parse', {
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

## Infrastructure/DevOps

For DevOps tools that wrap existing CLIs (kubectl, docker, aws, terraform), prefer generating a **SKILL.md that teaches Claude the existing CLI** instead of creating a new wrapper. Only create a new CLI if the MCP server adds significant custom logic.
