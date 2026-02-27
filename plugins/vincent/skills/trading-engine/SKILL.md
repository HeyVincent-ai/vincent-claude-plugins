# Vincent Trading Engine

Use the Vincent Trading Engine MCP tools to create and manage automated trading strategies. The Trading Engine has three modes:

1. **V2 Multi-Venue Strategies** — The latest strategy framework. Define instruments, a thesis, drivers (data monitors), escalation policies, trade rules, and notifications. Supports multiple venues (starting with Polymarket). Includes a full signal pipeline, LLM decision engine, and detailed analytics.
2. **V1 Polymarket Strategies** — Create Polymarket-specific strategies with monitors (web search, Twitter, price alerts, newswire). When a monitor detects new data, an LLM evaluates it against your thesis and decides whether to trade, set rules, or alert you.
3. **Standalone Trade Rules** — Set stop-loss, take-profit, and trailing stop rules that execute automatically when price conditions are met. No LLM involved.

Authentication is handled automatically by the MCP server via `VINCENT_API_KEY`.

---

## V2 Multi-Venue Strategies

V2 strategies support multiple venues, structured signal pipelines, and richer analytics. Use V2 for new strategies.

### V2 Strategy MCP Tools

| Tool | Description |
|---|---|
| `vincent_v2_strategy_create` | Create a new V2 strategy (starts as DRAFT) |
| `vincent_v2_strategy_list` | List all V2 strategies |
| `vincent_v2_strategy_get` | Get V2 strategy details |
| `vincent_v2_strategy_update` | Update a DRAFT V2 strategy |
| `vincent_v2_strategy_activate` | Activate a DRAFT V2 strategy |
| `vincent_v2_strategy_pause` | Pause an ACTIVE V2 strategy |
| `vincent_v2_strategy_resume` | Resume a PAUSED V2 strategy |
| `vincent_v2_strategy_archive` | Archive a V2 strategy permanently |

### V2 Analytics & Monitoring Tools

| Tool | Description |
|---|---|
| `vincent_v2_portfolio` | Portfolio overview across all venues (positions, balances, totals) |
| `vincent_v2_signal_log` | Raw signals received by drivers |
| `vincent_v2_decision_log` | LLM decisions: thesis updates, trade decisions, reasoning |
| `vincent_v2_trade_log` | Trade execution log: orders, fills, results |
| `vincent_v2_performance` | Performance metrics: P&L, win rate, per-instrument breakdown |
| `vincent_v2_filter_stats` | Signal filter statistics (pass/drop at each pipeline layer) |
| `vincent_v2_escalation_stats` | Escalation policy stats (wake frequency, batch counts, threshold breaches) |

### V2 Order Management Tools

| Tool | Description |
|---|---|
| `vincent_v2_place_order` | Place a manual order on any venue (market or limit) |
| `vincent_v2_cancel_order` | Cancel an open order on a venue |
| `vincent_v2_close_position` | Close a position by placing an opposite-side market order |
| `vincent_v2_kill_switch` | Emergency: pause all strategies and cancel all orders across all venues |

### V2 Strategy Parameters

#### vincent_v2_strategy_create
- `name` (string, required): Strategy name
- `config` (object, required): V2StrategyConfig with:
  - `instruments` — Markets/tokens to trade
  - `thesis` — Your trading thesis
  - `drivers` — Data monitors (web search, Twitter, price, newswire)
  - `escalation` — When and how to wake the LLM (batch vs immediate, thresholds)
  - `tradeRules` — Automated stop-loss, take-profit, trailing stop rules
  - `notifications` — Alert configuration
- `dataSourceSecretId` (string, optional): DATA_SOURCES secret ID for driver monitoring
- `pollIntervalMinutes` (number, optional): Driver polling interval (1-1440, default: 15)

#### vincent_v2_strategy_update
- `strategyId` (string, required)
- `name`, `config`, `dataSourceSecretId`, `pollIntervalMinutes` — pass only fields to change (DRAFT only)

#### vincent_v2_strategy_activate / pause / resume / archive
- `strategyId` (string, required)

#### vincent_v2_signal_log / decision_log / trade_log
- `strategyId` (string, required)
- `limit` (number, optional): Max results (default: 50)
- `offset` (number, optional): Pagination offset

#### vincent_v2_place_order
- `instrumentId` (string, required): Token ID, ticker, etc.
- `venue` (string, required): Venue name (e.g. `"polymarket"`)
- `side` (string, required): `BUY` or `SELL`
- `size` (number, required): Order size
- `orderType` (string, required): `market` or `limit`
- `limitPrice` (number, optional): Required for limit orders

#### vincent_v2_cancel_order
- `venue` (string, required): Venue name
- `orderId` (string, required): Order ID

#### vincent_v2_close_position
- `instrumentId` (string, required): Instrument ID
- `venue` (string, required): Venue name

### V2 Common Workflows

#### Create and activate a V2 strategy
1. `vincent_v2_strategy_create` with name and config (instruments, thesis, drivers, escalation, tradeRules, notifications)
2. Review with `vincent_v2_strategy_get`
3. `vincent_v2_strategy_activate` to start driver monitoring

#### Monitor a running V2 strategy
1. `vincent_v2_signal_log` — see raw signals from drivers
2. `vincent_v2_decision_log` — see LLM reasoning and decisions
3. `vincent_v2_trade_log` — see order executions and fills
4. `vincent_v2_performance` — see P&L and win rate

#### Place a manual order through V2
1. `vincent_v2_place_order` with venue, instrumentId, side, size, and orderType
2. Orders go through policy enforcement

#### Emergency stop
1. `vincent_v2_kill_switch` — pauses all active strategies and cancels all open orders across all venues

---

## V1 Polymarket Strategies

V1 strategies are Polymarket-specific. For new strategies, prefer V2 above.

## Strategy MCP Tools

| Tool | Description |
|---|---|
| `vincent_trading_strategy_create` | Create a new strategy (starts as DRAFT) |
| `vincent_trading_strategy_list` | List all strategies |
| `vincent_trading_strategy_get` | Get strategy details |
| `vincent_trading_strategy_update` | Update a DRAFT strategy |
| `vincent_trading_strategy_activate` | Activate a DRAFT strategy (use `resume` for PAUSED) |
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
