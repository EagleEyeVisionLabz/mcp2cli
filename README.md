# mcp2cli

**MCP servers are expensive. CLIs are not.**

> A Claude Code plugin that converts MCP servers into token-efficient CLI tools.

---

## Why?

MCP servers load **every tool schema** into the agent's context window on startup. For a server with 30 tools, that's ~55,000 tokens burned before the agent does anything.

A CLI tool with a SKILL.md loads **on-demand** — only when relevant — and returns compact text instead of verbose JSON.

|  | MCP | CLI + Skill |
|:---|:---|:---|
| **Startup** | Load all tool schemas into context | Nothing loaded |
| **Token cost** | ~55,000 tokens for 30 tools | 0 tokens until first use |
| **On use** | JSON + metadata per call | Load SKILL.md once (~2K tokens) |
| **Output** | Verbose structured JSON | Compact plain text |

> **Bottom line:** 94–96% token savings across all scales.

|  | MCP | CLI + Skill | Savings |
|:---|---:|---:|---:|
| 5 tools | 8,000 tok | 500 tok | **94%** |
| 30 tools | 55,000 tok | 2,000 tok | **96%** |
| 100 tools | 134,000 tok | 5,000 tok | **96%** |

> The canonical example: **Playwright** shipped a CLI + SKILL.md alongside their MCP server — specifically for coding agents where token efficiency matters.

---

## Quick Start

**1. Clone the plugin:**

```bash
git clone https://github.com/myeolinmalchi/mcp2cli.git
```

**2. Load it in Claude Code:**

```bash
claude --plugin-dir ./mcp2cli
```

**3. Convert your first MCP server:**

```
/convert ./my-mcp-server
```

That's it. The plugin analyzes the server, generates CLI code, and creates a SKILL.md — all in one step.

---

## How It Works

The plugin runs a **5-phase conversion pipeline**:

| Phase | Name | What happens | Output |
|:-----:|:-----|:-------------|:-------|
| 1 | **Analyze** | Read MCP source, extract tools, params, outputs, auth | Structured analysis table |
| 2 | **Design** | Map tool names → CLI commands, params → flags | Command structure spec |
| 3 | **Generate** | Write TypeScript CLI reusing MCP's business logic | Working npm CLI tool |
| 4 | **Skill** | Generate SKILL.md under 500 lines with examples | SKILL.md file |
| 5 | **Validate** | Run each command, compare with MCP output | Test report |

### Supported MCP Patterns

The plugin classifies MCP servers into 4 tiers by CLI conversion feasibility:

| Tier | Patterns | CLI Fit |
|:----:|:---------|:-------:|
| 1 | Local System Tool, SaaS API Wrapper, DevOps, Document/Media Processing | Excellent |
| 2 | Database, Search, Proxy, Multi-tool, Messaging, Monitoring | Good |
| 3 | Browser Automation, Code Execution, Memory, Auth/Identity | Partial |
| 4 | Aggregator/Gateway, Reasoning/Cognitive | Not recommended |

---

## Commands

| Command | Description |
|:--------|:------------|
| `/convert <path>` | Full end-to-end conversion (all 5 phases) |
| `/analyze-mcp <path>` | Analysis only — assess feasibility, no code gen |
| `/generate-skill <cli-name>` | Generate SKILL.md for an existing CLI tool |

```bash
/convert ./my-mcp-server
/convert https://github.com/user/their-mcp-server
/analyze-mcp ./my-mcp-server
/generate-skill my-existing-cli
```

---

## What's Inside

```
mcp2cli/
├── .claude-plugin/
│   └── plugin.json                        # Plugin manifest
│
├── skills/
│   └── mcp2cli/
│       ├── SKILL.md                       # Core conversion skill (auto-invoked)
│       └── references/
│           ├── pattern-classification.md  # 15 MCP patterns across 4 tiers
│           ├── cli-scaffold.md            # TypeScript CLI project template
│           ├── tier-1-templates.md        # Code templates: system, API, doc/media
│           ├── tier-2-templates.md        # Code templates: DB, proxy, multi-tool
│           ├── tier-3-strategies.md       # Partial conversion strategies
│           ├── porting-cheatsheet.md      # Python → TypeScript translation guide
│           └── skill-template.md          # SKILL.md generation template
│
├── commands/
│   ├── convert.md                         # /convert — full conversion
│   ├── analyze-mcp.md                     # /analyze-mcp — analysis only
│   └── generate-skill.md                  # /generate-skill — SKILL.md for existing CLI
│
└── agents/
    └── mcp-analyzer.md                    # Subagent for MCP source analysis
```

| Component | Count | Purpose |
|:----------|:-----:|:--------|
| Skills | 1 | Auto-invoked conversion engine |
| Reference docs | 7 | Pattern classification, code templates, porting guide |
| Commands | 3 | `/convert`, `/analyze-mcp`, `/generate-skill` |
| Agents | 1 | MCP server source code analysis |

---

## When to Convert (and When Not To)

| Convert to CLI | Keep as MCP |
|:---------------|:------------|
| Stateless request/response tools | Real-time streaming / subscriptions |
| Text or JSON output | Binary streams (audio, video) |
| Infrequently used tools (schema overhead > value) | Bidirectional communication |
| Simpler deployment desired | Complex state management across calls |

> **Rule of thumb:** If the MCP tool is basically `input → API call → output`, it should be a CLI.

---

## Contributing

1. **Add patterns** — New conversion patterns go in `skills/mcp2cli/references/`
2. **Keep it lean** — SKILL.md stays under 500 lines. Detailed docs go in `references/`
3. **Test conversions** — Point `/convert` at any MCP server and verify the output works

```bash
# Test locally
claude --plugin-dir ./mcp2cli
```

---

## References

- [Playwright CLI + SKILL.md](https://www.npmjs.com/package/@playwright/cli) — The canonical MCP-to-CLI migration
- [mcp-cli](https://www.philschmid.de/mcp-cli) — Dynamic tool discovery bridge (99% token reduction)
- [MCP vs CLI Benchmarks](https://mariozechner.at/posts/2025-08-15-mcp-vs-cli/) — 33% token efficiency advantage for CLI
- [Cloudflare Code Mode](https://blog.cloudflare.com/code-mode-mcp/) — 99.9% token reduction via single-tool MCP
- [Claude Code Skills Docs](https://code.claude.com/docs/en/skills) — Official skill authoring guide
- [Claude Code Plugins Docs](https://code.claude.com/docs/en/plugins) — Official plugin development guide
