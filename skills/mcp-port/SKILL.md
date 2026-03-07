---
name: mcp-port
description: Port Python MCP server code to TypeScript for CLI generation. Use when converting Python API clients, validators, or utilities to TypeScript. Triggers on "port python", "python to typescript", "translate python", "convert python code", "httpx to fetch".
---

# Python → TypeScript Porting

Translate Python MCP server code (API clients, validators, utilities) to TypeScript for use in CLI tools.

## When to Use

Use this skill when the source MCP server is written in Python and `mcp-codegen` needs to port the business logic to TypeScript.

## Quick Reference

| Python | TypeScript |
|:-------|:-----------|
| `httpx.AsyncClient` | `fetch` (built-in Node 18+) |
| `aiofiles` | `fs/promises` |
| `tenacity` retry | `p-retry` or manual retry loop |
| `click` / `typer` | `commander` |
| `pydantic` models | TypeScript interfaces + runtime checks |
| `os.getenv()` | `process.env` |
| `pathlib.Path` | `path.resolve()` / `path.join()` |
| `json.dumps(indent=2)` | `JSON.stringify(data, null, 2)` |
| `asyncio.run()` | Top-level await (ESM) |

For detailed pattern translations (HTTP requests, file uploads, retry logic, progress reporting), see `references/porting-cheatsheet.md`.
