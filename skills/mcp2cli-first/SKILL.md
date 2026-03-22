---
name: mcp2cli-first
description: Use when calling any MCP server tool — enforces mcp2cli with --toon as the primary access method for 96-99% token savings over raw MCP tool calls. Triggers on any MCP server interaction, Wolfram queries, Moneypenny portfolio operations, or external API access via MCP.
---

# mcp2cli-First — Token-Efficient MCP Access

**Every MCP tool call MUST go through `mcp2cli --toon` first.** Raw MCP tool calls are the fallback, not the default.

## Iron Law

**NO RAW MCP TOOL CALLS WHEN mcp2cli IS AVAILABLE.**

Raw MCP tool calls waste 2,000-5,000 tokens per invocation on JSON schema overhead. `mcp2cli --toon` delivers the same result in 50-200 tokens. This is a 96-99% savings per call.

## How to Use

```bash
# Pattern: mcp2cli --mcp <server-url-or-name> --toon <tool-name> --input '{"param": "value"}'

# Wolfram Alpha
mcp2cli --mcp wolfram --toon WolframAlpha --input '{"input": "solve 3x^2 + 2x - 5 = 0"}'

# Moneypenny (Portfolio Intelligence)
mcp2cli --mcp http://10.0.0.112:3101/mcp --toon portfolio_summary
mcp2cli --mcp http://10.0.0.112:3101/mcp --toon github_issues --input '{"repo": "verifieddit-www"}'
mcp2cli --mcp http://10.0.0.112:3101/mcp --toon matrix_notify --input '{"message": "Deploy complete"}'

# Any OpenAPI spec
mcp2cli --spec https://api.example.com/openapi.json --toon --list
mcp2cli --spec ./api.json --toon get-users --input '{"limit": 10}'

# Baked configurations (saved servers)
mcp2cli @moneypenny --toon portfolio_brief
```

## Key Flags

| Flag | Purpose |
|------|---------|
| `--toon` | **Token-efficient output** — always use this |
| `--jq <filter>` | Filter JSON response (e.g., `--jq '.items[].name'`) |
| `--head N` | Truncate to first N results |
| `--list` | Discover available tools on a server |
| `--search PATTERN` | Filter tool list by name pattern |

## When to Fall Back to Raw MCP

Only use raw MCP tool calls (`mcp__server__tool`) when:
1. `mcp2cli` is not installed (check with `which mcp2cli`)
2. `mcp2cli` fails with a connection error AND the raw MCP tool is available
3. The tool requires streaming or interactive output that mcp2cli doesn't support

**Log the fallback:** If you fall back to raw MCP, note it as a friction signal for the improvement tracker.

## Token Savings Example

| Scenario | Raw MCP | mcp2cli --toon | Savings |
|----------|---------|----------------|---------|
| Single Wolfram query | ~3,500 tokens | ~150 tokens | 96% |
| Portfolio summary | ~4,200 tokens | ~200 tokens | 95% |
| 10 queries per session | ~35,000 tokens | ~1,500 tokens | 96% |
| Heavy session (30 queries) | ~105,000 tokens | ~4,500 tokens | 96% |

At 30 queries, that's **100K tokens saved** — 10% of a 1M context window.
