# Vincent Brave Search

Use the Vincent Brave Search MCP tools for real-time web and news search. All requests are proxied through Vincent, which handles Brave API authentication, rate limits, and per-call billing automatically.

Authentication is handled automatically by the MCP server via `VINCENT_API_KEY`.

## MCP Tools

| Tool | Description | Cost |
|---|---|---|
| `vincent_brave_web_search` | Search the web | $0.005/call |
| `vincent_brave_news_search` | Search news articles | $0.005/call |

## Tool Parameters

### vincent_brave_web_search
- `q` (string, required): Search query (1-400 chars)
- `count` (number, optional): Number of results, 1-20 (default: 10)
- `offset` (number, optional): Pagination offset, 0-9
- `freshness` (string, optional): `pd` (past day), `pw` (past week), `pm` (past month), `py` (past year)
- `country` (string, optional): 2-letter country code for localized results (e.g. `us`, `gb`, `de`)

### vincent_brave_news_search
- `q` (string, required): Search query (1-400 chars)
- `count` (number, optional): Number of results, 1-20 (default: 10)
- `freshness` (string, optional): `pd` (past day), `pw` (past week), `pm` (past month), `py` (past year)

## Response Metadata

Every response includes `_vincent.cost` and `_vincent.balance` so you can track remaining credit:

```json
{
  "_vincent": {
    "costUsd": 0.005,
    "creditRemainingUsd": 4.99
  }
}
```

Warn the user when credit is running low.

## Important Notes

- Credit is deducted per call. The account must be claimed and have credit or a payment method on file.
- Rate limit: 60 requests/minute shared across all data source tools (Brave + Twitter)
- If credit runs out, calls are rejected. Tell the user to add credit at [heyvincent.ai](https://heyvincent.ai).
- A single `DATA_SOURCES` API key works for both Brave Search and Twitter — no separate key needed.
