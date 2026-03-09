# Vincent Brave Search

Use the Vincent Brave Search MCP tools for real-time web and news search. All requests are proxied through Vincent, which handles Brave API authentication, rate limits, and per-call billing automatically.

Authentication is handled automatically by the MCP server via `VINCENT_API_KEY`.

**No Brave API keys to manage.** The MCP server authenticates with Brave Search on your behalf. The agent cannot access the upstream API directly or bypass the proxy's credit and rate-limit enforcement.

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

Every response includes a `_vincent` object with cost and credit tracking:

```json
{
  "_vincent": {
    "costUsd": 0.005,
    "creditRemainingUsd": 4.99
  }
}
```

Use `creditRemainingUsd` to warn the user when credit is running low.

## Output Format

Web search results:

```json
{
  "web": {
    "results": [
      {
        "title": "Article Title",
        "url": "https://example.com/article",
        "description": "A brief description of the article content."
      }
    ]
  },
  "_vincent": {
    "costUsd": 0.005,
    "creditRemainingUsd": 4.99
  }
}
```

News search results follow the same structure with additional `age` and `source` fields per result.

## Error Handling

| Error | Cause | Resolution |
|-------|-------|------------|
| `401 Unauthorized` | Invalid or missing API key | Check that `VINCENT_API_KEY` is set correctly |
| `402 Insufficient Credit` | Credit balance is zero and no payment method on file | User must add credit at heyvincent.ai |
| `429 Rate Limited` | Exceeded 60 requests/minute | Wait and retry with backoff |

## Rate Limits

- 60 requests per minute per API key across all data source endpoints (Twitter + Brave Search combined)
- If rate limited, you'll receive a `429` response. Wait and retry.

## Credit Management

| Tool | Description |
| --- | --- |
| `vincent_credit_balance` | Check current credit balance and top-up options |
| `vincent_add_credits` | Get x402 payment instructions for purchasing credits |

## Important Notes

- Credit is deducted per call. The account must have credit or a payment method on file.
- A single `DATA_SOURCES` API key works for both Brave Search and Twitter — no separate key needed.
- If credit runs out, calls are rejected. Tell the user to add credit at [heyvincent.ai](https://heyvincent.ai).
