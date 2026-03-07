# MCP Server Pattern Classification

Before converting, classify the MCP server into one of these patterns to determine feasibility and strategy.

## Tier 1: Excellent CLI Fit

| Pattern | What it wraps | Conversion strategy |
|:--------|:-------------|:-------------------|
| **Local System Tool** | Filesystem, git, docker, shell | 1:1 tool-to-subcommand mapping. No auth. Use `child_process` or `fs` directly. |
| **SaaS API Wrapper** | REST/GraphQL APIs (GitHub, Slack, etc.) | HTTP client CLI. Auth via env vars. Map CRUD operations to subcommands. |
| **Infrastructure/DevOps** | Cloud APIs, K8s, CI/CD | Often wraps existing CLIs — consider SKILL.md pointing to original CLI instead. |
| **Document/Media Processing** | OCR, PDF, image gen | File-in/file-out pattern. `<cli> process <input-file> --output <output-file>`. |

## Tier 2: Good CLI Fit

| Pattern | What it wraps | Conversion strategy |
|:--------|:-------------|:-------------------|
| **Database Query** | SQLite, Postgres, Redis | Connection string via env var. `<cli> query "SQL"`, `<cli> schema`. |
| **Search & Retrieval** | Web search, vector DBs | `<cli> search "query" --limit 10`. Index management needs subcommands. |
| **Monitoring/Observability** | Metrics, logs, APM | `<cli> query "metric"`, `<cli> logs tail`. Streaming → `--follow` flag. |
| **Communication/Messaging** | Slack, email, Discord | `<cli> send "#channel" "message"`, `<cli> read --channel`. |
| **Proxy MCP** | Remote MCP/API endpoint | Remove protocol layer, call API directly. One fewer hop. |
| **Multi-tool MCP** | 10+ tools with prefixes | Group by prefix: `lol_get_profile` → `<cli> lol profile`. |

## Tier 3: Partial/Difficult CLI Fit

| Pattern | What it wraps | Strategy |
|:--------|:-------------|:---------|
| **Browser Automation** | Playwright, Puppeteer | Individual actions work (`click`, `screenshot`), but session state is awkward. |
| **Code Execution/Sandbox** | Language runtimes | One-shot `<cli> exec "code"` works. REPL sessions need persistent process. |
| **Knowledge/Memory** | Knowledge graphs | `store`/`recall` subcommands work, but loses value without LLM loop. |
| **Auth & Identity** | OAuth, identity | OAuth redirect flow needs loopback server (like `gh auth login`). |

## Tier 4: Not Recommended for CLI

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
- Server maintains session state critical to functionality
- Real-time streaming / bidirectional communication is core
- Server aggregates or routes to other MCP servers
- Server structures LLM reasoning (no standalone CLI value)

**Partial conversion** (Tier 3):
- Extract stateless read operations as CLI
- Keep stateful/interactive operations in MCP

## How to Detect the Pattern

| Look for | Pattern |
|:---------|:--------|
| `fs`, `child_process`, `execSync`, no HTTP client | Local System Tool |
| `fetch`/`httpx` to external API + Bearer token auth | SaaS API Wrapper |
| `child_process.exec('kubectl/docker/aws ...')` | Infrastructure/DevOps |
| File read + API call + file write, MIME detection | Document/Media Processing |
| Database driver imports (`pg`, `better-sqlite3`, `ioredis`) | Database Query |
| MCP `Client` → `StreamableHTTPClientTransport` → remote | Proxy MCP |
| 10+ tools with shared prefix (`lol_*`, `tft_*`) | Multi-tool MCP |
| Browser engine imports, page/session state | Browser Automation |
| Language runtime execution, sandboxing | Code Execution |
| Knowledge graph, entity/relation storage | Knowledge/Memory |
| OAuth flows, token management | Auth & Identity |

See `tier-1-templates.md`, `tier-2-templates.md`, `tier-3-strategies.md` for code templates per pattern.
