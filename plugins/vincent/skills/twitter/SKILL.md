# Vincent Twitter / X

Use the Vincent Twitter MCP tools to search tweets, look up user profiles, and retrieve recent tweets from X.com (Twitter). All requests are proxied through Vincent, which handles X API authentication, rate limits, and per-call billing automatically.

Authentication is handled automatically by the MCP server via `VINCENT_API_KEY`.

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

Every response includes `_vincent.cost` and `_vincent.balance` for credit tracking.

## Important Notes

- **7-day search window**: Tweet search only returns tweets from the last 7 days (X API v2 limitation)
- Credit is deducted per call. The account must be claimed and have credit or a payment method on file.
- Rate limit: 60 requests/minute shared across all data source tools (Brave + Twitter)
- `vincent_twitter_user_tweets` requires the numeric user ID, not the username. Get the ID from `vincent_twitter_get_user` first.
- A single `DATA_SOURCES` API key works for both Twitter and Brave Search — no separate key needed.
