# CLI Scaffold Reference

## Stack

- `commander` for argument parsing
- Built-in `fetch` for HTTP calls (Node 18+)
- `tsup` for bundling to single ESM file
- Entry point: `#!/usr/bin/env node`

## Project Template

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

## package.json Template

```json
{
  "name": "<cli-name>",
  "version": "1.0.0",
  "type": "module",
  "bin": { "<cli-name>": "dist/cli.js" },
  "scripts": {
    "build": "tsup src/cli.ts --format esm --dts",
    "dev": "tsx src/cli.ts"
  },
  "dependencies": {
    "commander": "^12.0.0"
  },
  "devDependencies": {
    "tsup": "^8.0.0",
    "tsx": "^4.0.0",
    "typescript": "^5.0.0"
  }
}
```

## tsconfig.json Template

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "outDir": "dist",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src"]
}
```

## Implementation Rules

1. **Reuse, don't rewrite**: Copy API client logic from the MCP server. Only change the interface layer.
2. **Error codes**: Exit 0 on success, 1 on user error, 2 on API error, 3 on network error.
3. **Auth priority**: env vars → config files → `--api-key` flag.
4. **Timeouts**: Default 30s for API calls, configurable via `--timeout`.
5. **No MCP dependency**: The CLI must not import any MCP SDK packages.
6. **Output convention**: Data → stdout, status/progress → stderr.
