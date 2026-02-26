# Vincent Polymarket

Use the Vincent Polymarket MCP tools to trade on prediction markets. Browse markets, place bets, track holdings, manage orders, and cancel positions — all without exposing private keys. Wallets use Gnosis Safe on Polygon with gasless trading through Polymarket's relayer.

Authentication is handled automatically by the MCP server via `VINCENT_API_KEY`.

## MCP Tools

| Tool | Description |
|---|---|
| `vincent_polymarket_markets` | Search or list prediction markets |
| `vincent_polymarket_market` | Get details for a specific market by condition ID |
| `vincent_polymarket_orderbook` | Get the order book for an outcome token |
| `vincent_polymarket_bet` | Place a BUY or SELL order |
| `vincent_polymarket_positions` | Get open positions and orders |
| `vincent_polymarket_holdings` | Get portfolio holdings with P&L |
| `vincent_polymarket_trades` | Get trade history |
| `vincent_polymarket_balance` | Get wallet collateral (USDC.e) balance |
| `vincent_polymarket_cancel_order` | Cancel a specific open order |
| `vincent_polymarket_cancel_all` | Cancel all open orders |

## Tool Parameters

### vincent_polymarket_markets
- `query` (string, optional): Keyword search (e.g. "bitcoin", "election")
- `active` (boolean, optional): Only show markets accepting orders
- `limit` (number, optional): 1-100 results
- `nextCursor` (string, optional): Pagination cursor from previous response

### vincent_polymarket_market
- `conditionId` (string, required): Market condition ID

### vincent_polymarket_orderbook
- `tokenId` (string, required): Outcome token ID (from market's `tokenIds` array)

### vincent_polymarket_bet
- `tokenId` (string, required): Outcome token ID
- `side` (string, required): `BUY` or `SELL`
- `amount` (number, required): For BUY — USD to spend. For SELL — number of shares to sell. Minimum $1.
- `price` (number, optional): Limit price 0.01-0.99. Omit for market order. **Always use market orders unless the user specifies a limit price.**

### vincent_polymarket_positions
- `market` (string, optional): Filter by condition ID

### vincent_polymarket_trades
- `market` (string, optional): Filter by condition ID

### vincent_polymarket_cancel_order
- `orderId` (string, required): Order ID to cancel

## Common Workflows

### Discover and bet on a market
1. Call `vincent_polymarket_markets` with `query` and `active: true`
2. Each market has `outcomes` (e.g. ["Yes", "No"]) and `tokenIds` at matching indices
3. Call `vincent_polymarket_orderbook` with the token ID you want to check pricing
4. Call `vincent_polymarket_bet` with `tokenId`, `side: "BUY"`, and `amount`
5. Share the user's Polymarket profile link: `https://polymarket.com/profile/<walletAddress>`

### Check positions and P&L
1. Call `vincent_polymarket_holdings` — returns shares, entry price, current price, unrealized P&L
2. Use this to decide whether to sell or set trade rules

### Sell a position
1. Call `vincent_polymarket_holdings` to get current share count
2. Call `vincent_polymarket_bet` with `side: "SELL"` and `amount` = number of shares
3. **Wait a few seconds after buying before selling** — shares need time to settle on-chain

### Fund the wallet
The wallet needs USDC.e (bridged USDC) on Polygon. The user must send USDC.e to the wallet address (from `vincent_polymarket_balance`).
- USDC.e contract: `0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174`
- **Do not send native USDC** (`0x3c499c542cEF5E3811e1192ce70d8cC03d5c3359`)
- Minimum bet: $1

## Security

- **No gas needed.** All Polymarket transactions are gasless via Polymarket's relayer.
- **Private keys never leave the server.** The agent gets a scoped Bearer token.
- **Server-side policy enforcement.** The wallet owner sets spending limits and approval thresholds at [heyvincent.ai](https://heyvincent.ai).
- **Human-in-the-loop.** Trades above the approval threshold trigger Telegram notifications.
- If a trade violates a policy, the response explains which policy was triggered.
- If a trade returns `status: "pending_approval"`, the wallet owner must approve via Telegram.

## Important Notes

- Always use `tokenIds` from the market response — not the `conditionId`. Each outcome has a token ID at the same index as the `outcomes` array.
- `"No orderbook exists for the requested token id"` means the market is closed, resolved, or you used the wrong ID.
- After any bet, share the user's Polymarket profile: `https://polymarket.com/profile/<walletAddress>`
- The first `vincent_polymarket_balance` call triggers Safe deployment and collateral approval (30-60 seconds).
- Use short keyword phrases in search. Stop-words like "or" can cause empty results.
