# Tier 3: Partial Conversion Strategies

These patterns require selective conversion. Extract stateless operations as CLI; keep stateful operations in MCP.

## Browser Automation

**Convert to CLI:**
- `screenshot <url> --output screenshot.png`
- `snapshot <url>` (accessibility tree as text)
- `pdf <url> --output page.pdf`

**Keep as MCP:**
- Multi-step navigation chains (navigate → click → fill → submit)
- Cookie/session management
- Real-time page interaction

## Code Execution

**Convert to CLI:**
- `exec "code" --lang python` (one-shot execution)
- `exec --file script.py` (file execution)

**Keep as MCP:**
- REPL sessions with persistent variable state
- Notebook-style execution with cell dependencies
- Long-running process management

## Knowledge/Memory

**Convert to CLI:**
- `store <key> <value>` / `recall <query>` (simple key-value)
- `export --format json` (data export)

**Keep as MCP:**
- Graph traversal with context-dependent queries
- Session-aware memory that builds on previous interactions
- Entity resolution across tool calls

## Auth & Identity

**Convert to CLI:**
- Login flow using loopback server pattern:
  ```typescript
  program.command('login')
    .action(async () => {
      // Start local HTTP server on random port
      // Open browser to OAuth authorize URL with redirect to localhost
      // Receive callback with auth code
      // Exchange for token and store in ~/.config/<cli>/credentials.json
    });
  ```
- Token storage: OS keychain (`keytar`) or config file
- `auth status` / `auth logout` subcommands

**Keep as MCP:**
- Complex multi-step OAuth flows with custom scopes
- Token refresh that needs to happen transparently during other operations

## General Partial Conversion Pattern

```
┌─────────────────────────────────────┐
│         Original MCP Server         │
├──────────────────┬──────────────────┤
│   Stateless Ops  │   Stateful Ops   │
│                  │                  │
│   → CLI tool     │   → Keep MCP     │
│   get_*, list_*  │   subscribe_*    │
│   export_*       │   maintain_*     │
│   search_*       │   session_*      │
└──────────────────┴──────────────────┘
```
