# Vincent Trading Engine

Use the Vincent Trading Engine MCP tools to create and manage automated trading strategies for Polymarket. The Trading Engine has two modes:

1. **LLM-Powered Strategies** — Create strategies with monitors (web search, Twitter, price alerts, newswire). When a monitor detects new information, an LLM evaluates it against your thesis and decides whether to trade, set protective orders, or alert you.
2. **Standalone Trade Rules** — Set stop-loss, take-profit, and trailing stop rules that execute automatically when price conditions are met. No LLM involved.

Authentication is handled automatically by the MCP server via `VINCENT_API_KEY`.

## Strategy MCP Tools

| Tool | Description |
|---|---|
| `vincent_trading_strategy_create` | Create a new strategy (starts as DRAFT) |
| `vincent_trading_strategy_list` | List all strategies |
| `vincent_trading_strategy_get` | Get strategy details |
| `vincent_trading_strategy_update` | Update a DRAFT strategy |
| `vincent_trading_strategy_activate` | Activate a DRAFT or resume a PAUSED strategy |
| `vincent_trading_strategy_pause` | Pause an ACTIVE strategy |
| `vincent_trading_strategy_resume` | Resume a PAUSED strategy |
| `vincent_trading_strategy_archive` | Archive a strategy permanently |
| `vincent_trading_strategy_duplicate` | Duplicate a strategy as a new version (DRAFT) |
| `vincent_trading_strategy_versions` | Get version history for a strategy |
| `vincent_trading_strategy_invocations` | Get LLM invocation log (decisions, actions, costs) |
| `vincent_trading_strategy_costs` | Get aggregate LLM cost summary |

## Trade Rule MCP Tools

| Tool | Description |
|---|---|
| `vincent_trading_rule_create` | Create a stop-loss, take-profit, or trailing stop rule |
| `vincent_trading_rule_list` | List trade rules (filter by status) |
| `vincent_trading_rule_get` | Get rule details |
| `vincent_trading_rule_update` | Update trigger price of an ACTIVE rule |
| `vincent_trading_rule_cancel` | Cancel a rule |
| `vincent_trading_rule_events` | Get event log (triggers, updates, executions) |
| `vincent_trading_rule_status` | Get monitoring worker health status |

## Strategy Parameters

### vincent_trading_strategy_create
- `name` (string, required): Strategy name (1-200 chars)
- `alertPrompt` (string, required): Your thesis and instructions for the LLM (1-10000 chars)
- `pollIntervalMinutes` (number, optional): How often to check periodic monitors (1-1440, default: 15)
- `config` (object, required): Strategy configuration with:
  - `monitors.periodic.webSearch.enabled` / `.keywords[]` — Brave web search keywords
  - `monitors.periodic.twitter.enabled` / `.accounts[]` — Twitter handles to monitor
  - `monitors.live.newswire.enabled` / `.topics[]` — Finnhub market news topics
  - `monitors.live.priceAlerts.enabled` / `.triggers[]` — Price conditions (ABOVE, BELOW, CHANGE_PCT)
  - `tools.canTrade` — Allow LLM to place trades
  - `tools.canSetRules` — Allow LLM to create stop-loss/take-profit/trailing stop rules
  - `tools.maxTradeUsd` — Maximum USD per LLM-initiated trade
  - `risk.maxOpenPositions` — Max concurrent positions
  - `risk.maxPortfolioAllocationPct` — Max portfolio % per position

### vincent_trading_strategy_update
- `strategyId` (string, required)
- `name`, `alertPrompt`, `pollIntervalMinutes`, `config` — any fields to change (DRAFT only)

### vincent_trading_strategy_activate / pause / resume / archive / duplicate
- `strategyId` (string, required)

### vincent_trading_strategy_invocations
- `strategyId` (string, required)
- `limit` (number, optional): 1-500 (default: 50)
- `offset` (number, optional): Pagination offset

### vincent_trading_rule_create
- `ruleType` (string, required): `STOP_LOSS`, `TAKE_PROFIT`, or `TRAILING_STOP`
- `marketId` (string, required): Polymarket condition ID
- `tokenId` (string, required): Outcome token ID
- `triggerPrice` (number, required): Price threshold 0-1 (e.g. 0.40 = 40 cents)
- `trailingPercent` (number): Required for TRAILING_STOP (1-99). Not allowed for other types.
- `action` (object, required): `{ type: "SELL_ALL" }` or `{ type: "SELL_PARTIAL", amount: <number> }`

### vincent_trading_rule_update
- `ruleId` (string, required)
- `triggerPrice` (number, required): New trigger price 0-1

### vincent_trading_rule_events
- `ruleId` (string, optional): Filter by rule
- `limit` (number, optional): Default 100
- `offset` (number, optional): Default 0

## Strategy Lifecycle

```
DRAFT → ACTIVE → PAUSED → ACTIVE (resume)
                → ARCHIVED (permanent)
```

- **DRAFT**: Can be edited. Not monitoring yet.
- **ACTIVE**: Monitors running. LLM invoked when new data detected.
- **PAUSED**: Monitoring stopped. Can resume.
- **ARCHIVED**: Permanent. Cannot reactivate.

To iterate, use `vincent_trading_strategy_duplicate` to create a new DRAFT version.

## Common Workflows

### Create and activate a strategy
1. `vincent_trading_strategy_create` with name, alertPrompt, and config (monitors + tools)
2. Review the strategy details with `vincent_trading_strategy_get`
3. `vincent_trading_strategy_activate` to start monitoring

### Set stop-loss protection on a position
1. Use `vincent_polymarket_holdings` to get current positions and token IDs
2. `vincent_trading_rule_create` with `ruleType: "STOP_LOSS"`, the market/token IDs, and a trigger price
3. The rule executes automatically when the price condition is met

### Monitor strategy activity
1. `vincent_trading_strategy_invocations` — see what the LLM decided and why
2. `vincent_trading_strategy_costs` — see aggregate LLM spending
3. `vincent_trading_rule_events` — see rule triggers and executions

### Strategy + trade rules together
1. Place a bet with `vincent_polymarket_bet`
2. Create a strategy with monitors to watch the thesis
3. Set a standalone stop-loss with `vincent_trading_rule_create` for immediate downside protection
4. Activate the strategy — the LLM monitors and may create additional rules

## Alert Prompt Best Practices

The `alertPrompt` is your instructions to the LLM. Good prompts:

1. **Specific thesis**: "I believe AI tokens will rally because GPU demand is increasing. Buy any AI-related prediction market position below 40 cents."
2. **Clear action criteria**: "Only trade if the new information directly supports or contradicts the thesis. If ambiguous, alert me instead."
3. **Explicit risk**: "Never allocate more than $50 to a single position. Set a 15% trailing stop on any new position."
4. **Contextual**: "Ignore routine corporate announcements. Focus on regulatory actions, major product launches, and competitive threats."

## LLM Available Tools

When invoked, the LLM can use these tools (depending on strategy config):

| Tool | Requires |
|---|---|
| `place_trade` | `canTrade: true` |
| `set_stop_loss` / `set_take_profit` / `set_trailing_stop` | `canSetRules: true` |
| `alert_user` | Always available |
| `no_action` | Always available |

## Trade Rule Behavior

- **STOP_LOSS**: Sells when price <= triggerPrice
- **TAKE_PROFIT**: Sells when price >= triggerPrice
- **TRAILING_STOP**: Moves stop price up as price rises. Sells when price drops below the trailing stop. `trailingPercent` controls how far below the peak the stop trails (e.g. 5 = 5%).

Rule statuses: `ACTIVE` → `TRIGGERED` / `PENDING_APPROVAL` / `FAILED` / `CANCELED`

## Cost Tracking

- LLM invocations cost $0.05-$0.30 depending on context size
- Costs are deducted from the data source credit balance (`dataSourceCreditUsd`)
- Brave Search monitors: ~$0.005/call. Twitter monitors: ~$0.005-$0.01/call. Finnhub: free.
- Use `vincent_trading_strategy_costs` to check spending

## Security

- **LLM cannot bypass policies** — all trades go through Vincent's policy engine
- **Backend-side LLM key** — the OpenRouter API key never leaves the server
- **Credit gating** — no LLM invocation without sufficient credit
- **Audit trail** — every invocation recorded with full prompt, response, actions, cost, and duration
- **All trades policy-enforced** — both LLM and standalone rules respect spending limits and approval thresholds
