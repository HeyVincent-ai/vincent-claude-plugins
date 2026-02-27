# Vincent Plugins for Claude Code

The official [Vincent](https://heyvincent.ai) plugin marketplace for Claude Code. Gives your agent access to secure crypto wallets, Polymarket trading, and real-time data.

## Quick Start

```
/plugin marketplace add HeyVincent-ai/vincent-claude-plugins
/plugin install vincent@vincent-plugins
export VINCENT_API_KEY="ssk_your_key_here"
```

Restart Claude Code and you're ready to go.

## What's included

### vincent

Connects Claude Code to Vincent's MCP server, giving your agent access to:

- **Smart Wallets** -- Transfer tokens, swap on DEXs, interact with DeFi protocols
- **Polymarket** -- Browse prediction markets, place bets, manage positions
- **Trading Engine** -- Multi-venue automated trading strategies (V2) with instruments, drivers, escalation policies, and full analytics. Also includes V1 Polymarket strategies and standalone stop-loss, take-profit, and trailing stop rules
- **Web Search** -- Real-time web and news search via Brave
- **Twitter/X** -- Search tweets, look up profiles, get recent activity
- **Credentials** -- Securely store API keys, passwords, and tokens; write to `.env` files without exposing values in the agent's context

All operations are secured by Vincent's policy engine with spending limits, allowlists, and optional human approval via Telegram.

### Skills included

| Skill | Description |
|---|---|
| `wallet` | Transfer tokens, swap on DEXs, check balances on any EVM chain |
| `polymarket` | Browse markets, place bets, view positions, cancel orders |
| `trading-engine` | Multi-venue automated strategies (V2) with signal pipelines, LLM decisions, and analytics. V1 Polymarket strategies with monitors. Standalone stop-loss/take-profit/trailing stop rules. |
| `brave-search` | Real-time web and news search via Brave Search |
| `twitter` | Search tweets, look up profiles, get recent activity |
| `credentials` | Securely store and manage credentials, write to `.env` without exposing values |
| `vincent-setup` | Guided setup wizard -- run `/vincent-setup` to configure your API key |

## Get your API key

1. Sign up at [heyvincent.ai](https://heyvincent.ai)
2. Create an account (Smart Wallet, Polymarket Wallet, or Data Source)
3. Go to **API Keys** and create a new key
4. Export it: `export VINCENT_API_KEY="ssk_..."`

## Links

- [Vincent Dashboard](https://heyvincent.ai)
- [Docs](https://github.com/HeyVincent-ai/Vincent/tree/main/docs)
- [Plugin Details](./plugins/vincent/README.md)
