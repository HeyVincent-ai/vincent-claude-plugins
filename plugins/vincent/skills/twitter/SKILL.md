# Vincent Twitter / X

Use the Vincent Twitter MCP tools to search tweets, look up user profiles, and retrieve recent tweets from X.com (Twitter). All requests are proxied through Vincent, which handles X API authentication, rate limits, and per-call billing automatically.

Authentication is handled automatically by the MCP server via `VINCENT_API_KEY`.

**No Twitter API keys to manage.** The MCP server authenticates with Twitter on your behalf. The agent cannot access the upstream API directly or bypass the proxy's credit and rate-limit enforcement.

## MCP Tools

| Tool | Description | Cost |
|---|---|---|
| `vincent_twitter_search` | Search recent tweets (last 7 days) | $0.01/call |
| `vincent_twitter_get_tweet` | Get a specific tweet by ID | $0.005/call |
| `vincent_twitter_get_user` | Look up a user profile by username | $0.005/call |
| `vincent_twitter_user_tweets` | Get a user's recent tweets by user ID | $0.01/call |

## Tool Parameters

### vincent_twitter_search
- `q` (string, required): Search query (1-512 chars)
- `max_results` (number, optional): 10-100 (default: 10)
- `start_time` (string, optional): ISO 8601 datetime, earliest tweets
- `end_time` (string, optional): ISO 8601 datetime, latest tweets

### vincent_twitter_get_tweet
- `tweetId` (string, required): Tweet ID

### vincent_twitter_get_user
- `username` (string, required): Twitter handle (without @)

### vincent_twitter_user_tweets
- `userId` (string, required): Numeric user ID (from `vincent_twitter_get_user` response)
- `max_results` (number, optional): 10-100 (default: 10)

## Common Workflows

### Research a topic
1. Call `vincent_twitter_search` with keywords
2. Returns tweet text, author ID, creation time, and metrics (likes, retweets, replies)

### Look up a specific account
1. Call `vincent_twitter_get_user` with the username to get the profile and numeric user ID
2. Call `vincent_twitter_user_tweets` with the user ID to get their recent tweets

## Response Metadata

Every response includes a `_vincent` object with cost and credit tracking:

```json
{
  "_vincent": {
    "costUsd": 0.01,
    "creditRemainingUsd": 4.99
  }
}
```

Use `creditRemainingUsd` to warn the user when credit is running low.

## Output Format

Tweet search results:

```json
{
  "data": [
    {
      "id": "123456789",
      "text": "Tweet content here",
      "created_at": "2026-02-26T12:00:00.000Z",
      "author_id": "987654321",
      "public_metrics": {
        "like_count": 42,
        "retweet_count": 10,
        "reply_count": 5
      }
    }
  ],
  "_vincent": {
    "costUsd": 0.01,
    "creditRemainingUsd": 4.99
  }
}
```

User profile responses include `description`, `public_metrics` (followers/following counts), `profile_image_url`, and `verified`.

## Error Handling

| Error | Cause | Resolution |
|-------|-------|------------|
| `401 Unauthorized` | Invalid or missing API key | Check that `VINCENT_API_KEY` is set correctly |
| `402 Insufficient Credit` | Credit balance is zero and no payment method on file | User must add credit at heyvincent.ai |
| `429 Rate Limited` | Exceeded 60 requests/minute | Wait and retry with backoff |
| `User not found` | Username doesn't exist on Twitter | Verify the username spelling |

## Rate Limits

- 60 requests per minute per API key across all data source endpoints (Twitter + Brave Search combined)
- If rate limited, you'll receive a `429` response. Wait and retry.

## Credit Management

| Tool | Description |
| --- | --- |
| `vincent_credit_balance` | Check current credit balance and top-up options |
| `vincent_add_credits` | Get x402 payment instructions for purchasing credits |

## Important Notes

- **7-day search window**: Tweet search only returns tweets from the last 7 days (X API v2 limitation).
- Credit is deducted per call. The account must have credit or a payment method on file.
- A single `DATA_SOURCES` API key works for both Twitter and Brave Search — no separate key needed.
- `vincent_twitter_user_tweets` requires the numeric user ID, not the username. Get the ID from `vincent_twitter_get_user` first.
- If credit runs out, calls are rejected. Tell the user to add credit at [heyvincent.ai](https://heyvincent.ai).
