# Vincent HyperLiquid

Use the Vincent HyperLiquid MCP tools to trade perpetuals and manage spot balances on HyperLiquid. The generated EOA **is** the HyperLiquid account — fund it directly via the HL bridge and start trading immediately with no Safe deployment or collateral approval steps.

**The agent never sees the private key.** All operations are executed server-side. The agent receives a scoped API key that can only perform actions permitted by the wallet owner's policies.

Authentication is handled automatically by the MCP server via `VINCENT_API_KEY`.

## MCP Tools

| Tool | Description |
|---|---|
| `vincent_hyperliquid_balance` | Get perps account value, withdrawable USDC, open positions, and spot balances |
| `vincent_hyperliquid_internal_transfer` | Transfer USDC between perps and spot sub-accounts |
| `vincent_hyperliquid_markets` | Get all available markets with mid prices |
| `vincent_hyperliquid_orderbook` | Get order book for a specific coin |
| `vincent_hyperliquid_trade` | Place a market or limit order (perps) |
| `vincent_hyperliquid_open_orders` | Get open orders (optionally filtered by coin) |
| `vincent_hyperliquid_trades` | Get trade history / fills (optionally filtered by coin) |
| `vincent_hyperliquid_cancel_order` | Cancel a specific order by coin and order ID |
| `vincent_hyperliquid_cancel_all` | Cancel all open orders (optionally filtered by coin) |

## Tool Parameters

### vincent_hyperliquid_balance
No parameters required.

### vincent_hyperliquid_internal_transfer
- `amount` (number, required): USDC amount to transfer
- `toPerp` (boolean, required): `true` = spot→perps, `false` = perps→spot

### vincent_hyperliquid_markets
No parameters required.

### vincent_hyperliquid_orderbook
- `coin` (string, required): Asset name (e.g. `BTC`, `ETH`, `SOL`)

### vincent_hyperliquid_trade
- `coin` (string, required): Asset name (e.g. `BTC`, `ETH`, `SOL`)
- `isBuy` (boolean, required): `true` for long, `false` for short/close
- `sz` (string, required): Size in base currency (e.g. `0.0001` BTC)
- `limitPx` (string, required): Price. For market orders, set slightly above ask (buy) or below bid (sell) to ensure fill. Recommended: `askPx * 1.005` for buys, `bidPx * 0.995` for sells.
- `orderType` (string, required): `market` (IoC) or `limit` (GTC)
- `reduceOnly` (boolean, optional): Pass `true` when closing a position to prevent accidentally opening a new one in the opposite direction

### vincent_hyperliquid_open_orders
- `coin` (string, optional): Filter by coin

### vincent_hyperliquid_trades
- `coin` (string, optional): Filter by coin

### vincent_hyperliquid_cancel_order
- `coin` (string, required): Asset name
- `oid` (number, required): Numeric order ID

### vincent_hyperliquid_cancel_all
- `coin` (string, optional): Cancel only orders for this coin. Omit to cancel all.

## Common Workflows

### Get started
1. Call `vincent_hyperliquid_balance` to check if the wallet is funded
2. If empty, tell the user to deposit USDC via the HyperLiquid bridge from Arbitrum to the wallet address

### Open a long position
1. Call `vincent_hyperliquid_markets` to get current prices
2. Call `vincent_hyperliquid_orderbook` with `coin: "BTC"` to get best ask
3. Call `vincent_hyperliquid_trade` with `isBuy: true`, `orderType: "market"`, and `limitPx` set to `bestAsk * 1.005`

### Close a position
1. Call `vincent_hyperliquid_orderbook` to get best bid
2. Call `vincent_hyperliquid_trade` with `isBuy: false`, `reduceOnly: true`, and `limitPx` set to `bestBid * 0.995`

### Transfer between sub-accounts
HyperLiquid has separate perps and spot sub-accounts. Use `vincent_hyperliquid_internal_transfer` to move USDC:
- `toPerp: true` — spot→perps (needed before perp trading)
- `toPerp: false` — perps→spot (needed before spot trading)

### Fund the wallet
Deposit USDC to the EOA address via:
- **HyperLiquid bridge** from Arbitrum: visit `https://app.hyperliquid.xyz/portfolio` and bridge USDC
- **HL→HL transfer** (`usdSend`) from another HL account — instant

Minimum for a BTC perp trade: **$2 USDC** (covers $10 notional at 20x default leverage + taker fees).

## Output Format

**balance:**
```json
{
  "walletAddress": "0x...",
  "accountValue": "105.23",
  "withdrawable": "95.00",
  "positions": [
    {
      "position": {
        "coin": "BTC",
        "szi": "0.0001",
        "entryPx": "105200.0",
        "positionValue": "10.52",
        "unrealizedPnl": "0.05",
        "liquidationPx": null,
        "leverage": { "type": "cross", "value": 20 }
      },
      "type": "oneWay"
    }
  ],
  "spotBalances": [
    {
      "coin": "USDC",
      "token": 0,
      "hold": "0.0",
      "total": "50.0"
    }
  ]
}
```

**markets:**
```json
{
  "BTC": "105234.5",
  "ETH": "3412.0",
  "SOL": "185.3"
}
```

**orderbook:**
```json
{
  "coin": "BTC",
  "levels": [
    [["105200.0", "0.5", 3], ["105100.0", "1.2", 5]],
    [["105300.0", "0.3", 2], ["105400.0", "0.8", 4]]
  ]
}
```
`levels[0]` = bids (descending), `levels[1]` = asks (ascending). Each entry is `[price, size, numOrders]`. Best bid: `levels[0][0][0]`, best ask: `levels[1][0][0]`.

**trade (executed):**
```json
{
  "orderId": 12345678,
  "status": "executed",
  "transactionLogId": "clx...",
  "walletAddress": "0x...",
  "fillDetails": {
    "totalSz": "0.0001",
    "avgPx": "105250.0"
  }
}
```

**trade (pending approval):**
```json
{
  "status": "pending_approval",
  "transactionLogId": "clx...",
  "reason": "Exceeds approval threshold"
}
```

**trade (denied):**
```json
{
  "status": "denied",
  "transactionLogId": "clx...",
  "reason": "Exceeds daily spending limit"
}
```

**open-orders:**
```json
{
  "walletAddress": "0x...",
  "openOrders": [
    {
      "coin": "BTC",
      "side": "B",
      "limitPx": "100000.0",
      "sz": "0.0001",
      "oid": 12345678,
      "timestamp": 1700000000000,
      "origSz": "0.0001"
    }
  ]
}
```
`side`: `"B"` = buy/long, `"A"` = ask/sell.

**trades (fills):**
```json
{
  "walletAddress": "0x...",
  "fills": [
    {
      "coin": "BTC",
      "px": "105200.0",
      "sz": "0.0001",
      "side": "B",
      "time": 1700000000000,
      "dir": "Open Long",
      "closedPnl": "0",
      "fee": "0.0105",
      "oid": 12345678
    }
  ]
}
```

## Error Handling

| Error | Cause | Resolution |
|-------|-------|------------|
| `401 Unauthorized` | Invalid or missing API key | Check that `VINCENT_API_KEY` is set correctly |
| `status: "denied"` | Trade blocked by server-side policy | User must adjust policies at heyvincent.ai |
| `status: "pending_approval"` | Trade exceeds approval threshold | Do not retry — wallet owner receives Telegram notification to approve/deny |
| `400 Bad Request` | Invalid parameters (e.g. non-numeric oid, bad coin) | Fix the parameter values |
| `429 Rate Limited` | Too many requests | Wait and retry with backoff |
| `500 TRADE_FAILED` | HyperLiquid rejected the order (e.g. insufficient margin, bad price) | Check account balance and order parameters |

## Policies (Server-Side Enforcement)

The wallet owner controls what the agent can do by setting policies at [heyvincent.ai](https://heyvincent.ai). All policies are enforced server-side before any trade executes.

| Policy                      | What it does                                                     |
| --------------------------- | ---------------------------------------------------------------- |
| **Spending limit (per tx)** | Max USD notional per trade                                       |
| **Spending limit (daily)**  | Max USD notional per rolling 24 hours                            |
| **Spending limit (weekly)** | Max USD notional per rolling 7 days                              |
| **Require approval**        | Every trade needs human approval via Telegram                    |
| **Approval threshold**      | Trades above a USD amount need human approval via Telegram       |

If a trade is blocked, the API returns `status: "denied"` with the reason. If approval is needed, `status: "pending_approval"` is returned and the wallet owner receives a Telegram notification.

## Important Notes

- **No gas required.** HyperLiquid L1 is gasless — all perp trades settle natively.
- **Perps and spot sub-accounts.** The generated EOA has both a perps sub-account (cross-margin) and a spot sub-account. Use `vincent_hyperliquid_internal_transfer` to move USDC between them. Deposits via the HL bridge land in the perps account by default.
- **Minimum notional:** $10 (e.g. 0.0001 BTC at $100k/BTC). Default leverage is 20x cross-margin.
- **Never try to access raw secret values.** The private key stays server-side.
- For market orders, always set `limitPx` slightly outside the best price (`* 1.005` for buys, `* 0.995` for sells) to guarantee IoC fill at the current market price.
- If a trade returns `status: "pending_approval"`, do not retry — wait for the wallet owner to respond via Telegram.
