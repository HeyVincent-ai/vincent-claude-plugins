# Vincent Trading Engine

Use the Vincent Trading Engine MCP tools to create and manage automated trading strategies on Polymarket (and future venues). The Trading Engine has two modes:

1. **Multi-Venue Strategies** — Define instruments, a thesis, drivers (data monitors), escalation policies, trade rules, and notifications. Includes a signal pipeline that filters noise before the LLM sees it, an LLM decision engine, and detailed analytics.
2. **Standalone Trade Rules** — Set stop-loss, take-profit, and trailing stop rules that execute automatically when price conditions are met. No LLM involved.

Authentication is handled automatically by the MCP server via `VINCENT_API_KEY`.

---

## Signal Pipeline

Every strategy runs a multi-stage signal pipeline. Understanding it is key to writing good strategies:

```
Drivers (web search, Twitter, price, newswire)
    │
    ▼
Raw Signals — everything the drivers pick up
    │
    ▼
Filter Layers — drop noise, duplicates, irrelevant data
    │
    ▼
Escalation Policy — batch signals, apply thresholds
    │
    ▼
LLM Invocation — only when the escalation policy decides it's worth it
    │
    ▼
Actions — trade, update thesis, create rules, alert user, or no action
```

**Key points:**
- **Signals are pre-filtered before the LLM sees them.** The filter layers drop duplicates, low-relevance noise, and data that doesn't match the strategy's instruments or drivers.
- **The LLM only wakes when the escalation policy decides it should.** Escalation can batch signals (e.g. "wake every 30 minutes with a summary") or trigger immediately on high-priority events (e.g. "price moved >5%").
- **This means the LLM is not invoked on every poll.** Unlike raw polling, the pipeline saves cost and reduces noise. Your thesis should focus on decision-making, not filtering.

Use `vincent_v2_filter_stats` and `vincent_v2_escalation_stats` to see how signals flow through the pipeline.

---

## Strategy MCP Tools

| Tool | Description |
|---|---|
| `vincent_v2_strategy_create` | Create a new strategy (starts as DRAFT) |
| `vincent_v2_strategy_list` | List all strategies |
| `vincent_v2_strategy_get` | Get strategy details |
| `vincent_v2_strategy_update` | Update a DRAFT strategy |
| `vincent_v2_strategy_activate` | Activate a DRAFT strategy |
| `vincent_v2_strategy_pause` | Pause an ACTIVE strategy |
| `vincent_v2_strategy_resume` | Resume a PAUSED strategy |
| `vincent_v2_strategy_archive` | Archive a strategy permanently |

## Analytics & Monitoring Tools

| Tool | Description |
|---|---|
| `vincent_v2_portfolio` | Portfolio overview across all venues (positions, balances, totals) |
| `vincent_v2_signal_log` | Raw signals received by drivers |
| `vincent_v2_decision_log` | LLM decisions: thesis updates, trade decisions, reasoning |
| `vincent_v2_trade_log` | Trade execution log: orders, fills, results |
| `vincent_v2_performance` | Performance metrics: P&L, win rate, per-instrument breakdown |
| `vincent_v2_filter_stats` | Signal filter statistics (pass/drop at each pipeline layer) |
| `vincent_v2_escalation_stats` | Escalation policy stats (wake frequency, batch counts, threshold breaches) |

## Order Management Tools

| Tool | Description |
|---|---|
| `vincent_v2_place_order` | Place a manual order on any venue (market or limit) |
| `vincent_v2_cancel_order` | Cancel an open order on a venue |
| `vincent_v2_close_position` | Close a position by placing an opposite-side market order |
| `vincent_v2_kill_switch` | Emergency: pause all strategies and cancel all orders across all venues |

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

---

## Strategy Parameters

### vincent_v2_strategy_create
- `name` (string, required): Strategy name
- `config` (object, required): V2StrategyConfig with:
  - `instruments` — Markets/tokens to trade
  - `thesis` — Your trading thesis (see Thesis Best Practices below)
  - `drivers` — Data monitors (web search, Twitter, price, newswire)
  - `escalation` — When and how to wake the LLM (batch vs immediate, thresholds)
  - `tradeRules` — Automated stop-loss, take-profit, trailing stop rules
  - `notifications` — Alert configuration
- `dataSourceSecretId` (string, optional): DATA_SOURCES secret ID for driver monitoring
- `pollIntervalMinutes` (number, optional): Driver polling interval (1-1440, default: 15)

### vincent_v2_strategy_update
- `strategyId` (string, required)
- `name`, `config`, `dataSourceSecretId`, `pollIntervalMinutes` — pass only fields to change (DRAFT only)

### vincent_v2_strategy_activate / pause / resume / archive
- `strategyId` (string, required)

### vincent_v2_signal_log / decision_log / trade_log
- `strategyId` (string, required)
- `limit` (number, optional): Max results (default: 50)
- `offset` (number, optional): Pagination offset

### vincent_v2_place_order
- `instrumentId` (string, required): Token ID, ticker, etc.
- `venue` (string, required): Venue name (e.g. `"polymarket"`)
- `side` (string, required): `BUY` or `SELL`
- `size` (number, required): Order size
- `orderType` (string, required): `market` or `limit`
- `limitPrice` (number, optional): Required for limit orders

### vincent_v2_cancel_order
- `venue` (string, required): Venue name
- `orderId` (string, required): Order ID

### vincent_v2_close_position
- `instrumentId` (string, required): Instrument ID
- `venue` (string, required): Venue name

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

---

## Strategy Lifecycle

```
DRAFT → ACTIVE → PAUSED → ACTIVE (resume)
                → ARCHIVED (permanent)
```

- **DRAFT**: Can be edited. Not monitoring yet.
- **ACTIVE**: Drivers running, signal pipeline processing, LLM invoked per escalation policy.
- **PAUSED**: Pipeline stopped. Can resume.
- **ARCHIVED**: Permanent. Cannot reactivate.

---

## Thesis Best Practices

The `thesis` field in your strategy config is the instructions for the backend LLM. Because the signal pipeline filters noise before the LLM sees it, write your thesis differently than you would for a raw polling system:

### Focus on decisions, not filtering
The pipeline handles noise filtering. Signals that reach the LLM have already survived the filter layers — treat them as pre-qualified and worth evaluating. Don't waste thesis tokens telling the LLM to "ignore routine news" or "only pay attention to X" — the drivers and filters already do that.

### Be specific about actions
Tell the LLM exactly what to do when it sees a signal, not what to ignore:

**Good:**
> "Buy YES on any AI regulation market below $0.30 when new restrictive legislation is introduced. Size positions at $25 max. Set a 10% trailing stop on new positions. If the market moves above $0.60, take profit on half."

**Bad:**
> "Watch AI regulation news. Ignore routine stuff. Only trade if something important happens."

### Include risk parameters in the thesis
The LLM can create trade rules. Tell it your risk tolerance:

**Good:**
> "Never hold more than $100 in a single market. Always set a stop-loss at 20% below entry. If a position doubles, sell half and move the stop to breakeven."

### Give context on your conviction
Help the LLM understand why you believe what you believe:

**Good:**
> "I believe the Fed will cut rates in Q1 2026 based on softening labor data. Buy YES on rate-cut markets when employment reports come in weak. Sell or hedge if inflation data surprises to the upside."

### Example thesis prompts

**Event-driven:**
> "Trade the FIFA World Cup 2026 winner market. Buy YES on teams that win knockout-round matches at prices below $0.40. Sell if the team is eliminated. Size: $20 per position, trailing stop at 15%."

**Macro thesis:**
> "I'm bearish on US tech regulation passing before midterms. Buy NO on any US tech regulation market below $0.50. Max $50 per position. Take profit at $0.75. Stop loss at $0.25. Alert me if any bill advances past committee."

**Arbitrage / rebalancing:**
> "Monitor all Bitcoin price prediction markets. If any YES token trades more than 5 cents below the consensus across markets, buy it. Sell when it converges. Max $30 per trade."

---

## LLM Available Tools

When the backend LLM is invoked by the escalation policy, it can take these actions:

| Action | Description |
|---|---|
| **Place trade** | Buy or sell on any configured venue. Subject to policy limits (max trade size, max positions, portfolio allocation). |
| **Update thesis** | Refine the strategy's thesis based on new information. The updated thesis persists for future invocations. |
| **Create trade rules** | Set stop-loss, take-profit, or trailing stop rules on positions. These execute automatically without further LLM involvement. |
| **Alert user** | Send a notification to the user (e.g. "Thesis is at risk — regulatory bill advancing"). |
| **No action** | Decide the signal doesn't warrant action. This is logged in the decision log. |

All trades placed by the LLM go through Vincent's policy engine — the LLM cannot bypass spending limits, position limits, or approval requirements.

---

## Trade Rule Behavior

- **STOP_LOSS**: Sells when price <= triggerPrice
- **TAKE_PROFIT**: Sells when price >= triggerPrice
- **TRAILING_STOP**: Moves stop price up as price rises. Sells when price drops below the trailing stop. `trailingPercent` controls how far below the peak the stop trails (e.g. 5 = 5%).

Rule statuses: `ACTIVE` → `TRIGGERED` / `PENDING_APPROVAL` / `FAILED` / `CANCELED`

---

## Common Workflows

### Create and activate a strategy
1. `vincent_v2_strategy_create` with name and config (instruments, thesis, drivers, escalation, tradeRules, notifications)
2. Review with `vincent_v2_strategy_get`
3. `vincent_v2_strategy_activate` to start the signal pipeline

### Monitor a running strategy
1. `vincent_v2_signal_log` — see raw signals from drivers
2. `vincent_v2_filter_stats` — see what the filters passed or dropped
3. `vincent_v2_decision_log` — see LLM reasoning and decisions
4. `vincent_v2_trade_log` — see order executions and fills
5. `vincent_v2_performance` — see P&L and win rate

### Set stop-loss protection on a position
1. Use `vincent_polymarket_holdings` to get current positions and token IDs
2. `vincent_trading_rule_create` with `ruleType: "STOP_LOSS"`, the market/token IDs, and a trigger price
3. The rule executes automatically when the price condition is met

### Place a manual order
1. `vincent_v2_place_order` with venue, instrumentId, side, size, and orderType
2. Orders go through policy enforcement

### Emergency stop
1. `vincent_v2_kill_switch` — pauses all active strategies and cancels all open orders across all venues

---

## Cost Tracking

- LLM invocations cost $0.05-$0.30 depending on context size
- Costs are deducted from the data source credit balance (`dataSourceCreditUsd`)
- Brave Search drivers: ~$0.005/call. Twitter drivers: ~$0.005-$0.01/call. Newswire: free.
- Use `vincent_trading_strategy_costs` to check spending

## Security

- **LLM cannot bypass policies** — all trades go through Vincent's policy engine
- **Backend-side LLM key** — the OpenRouter API key never leaves the server
- **Credit gating** — no LLM invocation without sufficient credit
- **Audit trail** — every invocation recorded with full prompt, response, actions, cost, and duration
- **All trades policy-enforced** — both LLM and standalone rules respect spending limits and approval thresholds
