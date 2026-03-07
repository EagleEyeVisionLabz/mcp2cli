---
name: mcp-analyze
description: Analyze MCP server source code to classify its pattern and assess CLI conversion feasibility. Use when the user wants to understand an MCP server's architecture, list its tools, or determine if it can be converted to a CLI. Triggers on "analyze mcp", "mcp server pattern", "conversion feasibility", "what type of mcp".
---

# MCP Server Analysis

Analyze an MCP server's source code to extract tools, classify its architecture pattern, and assess CLI conversion feasibility.

## How to Analyze

Read the MCP server source code and extract:

1. **Tools**: List every `@tool`, `server.setRequestHandler(CallToolRequestSchema, ...)`, or `@mcp.tool()` definition
2. **Parameters**: For each tool, document all input parameters (name, type, required/optional, description)
3. **Output format**: What each tool returns (JSON structure, text, binary)
4. **Dependencies**: External APIs, SDKs, auth requirements
5. **Transport**: stdio, SSE, HTTP — and whether the server is a proxy or direct implementation
6. **Resources/Prompts**: Any MCP resources or prompt templates exposed

## Output Format

Create a structured analysis table:

```
| Tool Name | Params | Output | API Endpoint | Auth |
|-----------|--------|--------|--------------|------|
```

Then classify the server using `references/pattern-classification.md` and report:
- Pattern type and tier (1-4)
- CLI conversion feasibility (Excellent / Good / Partial / Not recommended)
- Recommended conversion strategy
- Any blockers or concerns
